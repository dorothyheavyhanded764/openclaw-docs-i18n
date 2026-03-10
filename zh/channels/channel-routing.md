

  配置

  
# 频道路由

在 OpenClaw 中，回复消息会**被路由回这条消息最初所在的频道（channel）**。
模型本身并不会"选择往哪个频道回复"；真正的路由决策完全由宿主侧（host）的配置来控制，是**确定性的（deterministic）**。

本页会帮你搞清楚三件事：

- **OpenClaw 如何为每条入站消息选择目标智能体（agent）**
- **会话键 / 会话密钥（session key / sessionKey）是如何构造的**
- **在多频道、多账号场景下，如何配置路由与广播群组（broadcast groups）**

---

## 关键术语

在开始具体配置之前，先约定几个在 OpenClaw 里经常出现的关键词，方便你后面理解路由行为。

- **频道（channel）**：指具体的消息平台类型，例如 `whatsapp`、`telegram`、`discord`、`slack`、`signal`、`imessage`、`webchat` 等。
- **账户 ID（accountId）**：同一个频道（channel）下的具体账户实例，例如多个 Telegram 机器人、多个 Slack 工作区等。
- **频道默认账户（`channels..defaultAccount`）**：
  当一条出站消息没有显式指定 `accountId` 时，`channels..defaultAccount` 会决定默认使用哪个账户。
  - **推荐做法**：在多账户场景下，只要同一频道里配置了两个或更多账户，就显式设置一个默认账户（`defaultAccount` 或 `accounts.default`）。
    否则回退逻辑会选用第一个规范化后的账户 ID，结果可能和你预期的不一致。
- **智能体 ID（agentId）**：一个独立的智能体工作空间和会话存储（workspace + session store），你可以把它理解为一个"独立的大脑"。
- **会话键 / 会话密钥（session key / sessionKey）**：用于给会话"分桶"的键值。它同时决定：
  - **上下文如何聚合**（哪些消息算作同一条"对话线"），以及
  - **并发如何控制**（同一会话下如何排队或串行执行）。

---

## 会话键结构示例（Session key shapes）

OpenClaw 使用会话键（session key）来区分"这一条对话属于哪一个会话桶"。不同类型的会话（私聊、群组、频道、线程）会有不同的键结构。

### 直接消息（direct messages）

对于直接消息（direct messages / 私聊），默认策略是把它们都汇总到该智能体的**主会话（main session）**，方便你在一条时间线上查看历史记录：

- `agent::`（默认值为 `agent:main:main`）

换句话说，只要你没有额外拆分，某个智能体下的所有私聊消息，都会共享同一个主会话。

### 群组与频道（groups / channels）

群组消息和频道 / 房间消息则会按频道类型和目标 ID 拆分到不同的会话桶中，**彼此互不干扰**：

- 群组：`agent:::group:`
- 频道 / 房间：`agent:::channel:`

这样可以保证"同一个智能体在不同群里说的话"，不会被混到一条会话线里。

### 线程与话题（threads / topics）

有些平台支持线程或话题，OpenClaw 会在基础键上再追加一段标识，让同一条回复链拥有独立会话：

- Slack / Discord 线程：在基础键后追加 `:thread:`
- Telegram 论坛话题：在群组键中嵌入 `:topic:`

示例：

- `agent:main:telegram:group:-1001234567890:topic:42`
- `agent:main:discord:channel:123456:thread:987654`

---

## 主私信路由固定（session.dmScope）

当 `session.dmScope` 设为 `main` 时，所有直接消息（direct messages）都会汇入同一个主会话（main session），方便你在一条时间线上查看私聊历史。

但这也带来一个问题：如果不加限制，并非真正"拥有"这个会话的人，也可能通过私信改写这条主会话的 `lastRoute`。

为了避免这种情况，OpenClaw 在满足以下条件时，会根据 `allowFrom` 允许来源列表（allowFrom allowlist）推断出一个**固定的会话所有者（session owner）**：

- `allowFrom` 中**只有一个**非通配符项；
- 该项可以规范化为该频道上的**具体发送者 ID**；
- 当前入站私信的发送者 **不是** 这个固定所有者。

当以上条件都满足，且发送者与"固定所有者"不匹配时：

- OpenClaw 依然会记录这条入站消息的会话元数据（session metadata）；
- 但会**跳过更新主会话的 `lastRoute`**，避免错误地把主会话路由"转让"给其他人。

---

## 路由规则：如何选择智能体（agent）

对每一条入站消息，OpenClaw 最终只会选择**一个智能体（agentId）** 来处理。你可以把下面这套规则理解为"从最精确到最宽泛"的优先级列表：

1. **精确对等体匹配（exact peer match）**
   - 在 `bindings` 中同时指定 `peer.kind` + `peer.id` 的绑定。
2. **父对等体匹配（parent peer match）**
   - 线程沿用所属消息的绑定（thread 继承）。
3. **服务器 + 角色匹配（Discord guild + roles）**
   - 通过 `guildId` + `roles` 进行匹配。
4. **仅服务器匹配（Discord guild match）**
   - 只使用 `guildId`。
5. **团队匹配（Slack team match）**
   - 通过 `teamId` 进行匹配。
6. **账户匹配（account match）**
   - 在指定频道上的某个具体账户（`accountId`）。
7. **仅频道匹配（channel match）**
   - 该频道上的任意账户，`accountId: "*"`。
8. **默认智能体（default agent）**
   - 首选 `agents.list[].default`；
   - 若未指定，回退为 `agents.list` 中的第一个条目，再回退到 `main`。

当一个绑定中同时包含多个匹配字段（例如 `peer`、`guildId`、`teamId`、`roles`）时：

- **所有提供的字段都必须匹配**，该绑定才会生效；
- 一旦匹配成功，对应的智能体（`agentId`）就决定了具体使用哪个**工作区（workspace）**和**会话存储（session store）**。

---

## 广播群组（Broadcast Groups）：为同一对等体运行多个智能体

有时你会希望在同一个对等体（peer）上，**同时运行多个智能体（multi-agent）**，例如：

- 在一个 WhatsApp 群组里，同时跑"助手智能体 A"和"日志记录智能体 B"；
- 对同一个手机号既做用户支持（support），又做对话日志归档（logger）。

**广播群组（broadcast groups）** 允许你在 **OpenClaw 本来就会回复的场景** 下，为同一个对等体配置**多个智能体一起运行**。典型场景包括：

- WhatsApp 群组里，在提及（mention）或激活模式（activation mode）判断通过之后。

配置示例：

```json
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"],
  },
}
```

这里的 `strategy: "parallel"` 表示这些智能体会**并行运行**。更多详细用法可以参考英文页的 [Broadcast Groups](./broadcast-groups.md)。

---

## 配置总览（Config overview）

在实际配置里，你主要会接触到两个块：

- **`agents.list`**：智能体列表，用来声明每个智能体的 ID、名称、工作区路径（workspace）、模型等信息；
- **`bindings`**：路由绑定规则，用来把入站的频道 / 账户 / 对等体（peers）映射到具体的智能体（agentId）。

示例：

```json
{
  agents: {
    list: [{ id: "support", name: "Support", workspace: "~/.openclaw/workspace-support" }],
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" },
  ],
}
```

你可以根据自己的团队结构，把不同频道 / 群组映射到不同的智能体上，实现类似"客服专用智能体""内部工具智能体"等划分。

---

## 会话存储（Session storage）

默认情况下，会话存储（session store）会保存在 OpenClaw 的状态目录下（默认是 `~/.openclaw`）：

- `~/.openclaw/agents//sessions/sessions.json`
- 同一目录下还会存放该会话的 JSONL 格式会话记录（transcript，JSONL）

如果你希望把会话数据存到别的地方（例如单独的磁盘挂载或云盘路径），可以通过 `session.store` 配置项来重写存储路径，并结合 `{agentId}` 模板占位符来区分不同智能体。

---

## WebChat 行为（WebChat behavior）

WebChat 前端会直接挂载到当前选中的智能体（selected agent）上，并且默认使用该智能体的主会话（main session）。
这意味着：

- 你可以在 WebChat 里，看到这个智能体在不同频道（channel）上的上下文汇总；
- WebChat 更适合作为"这个智能体整体状态"的窗口，而不是只看单一平台的对话。

如果你在配置中改变了会话键策略，也要记得考虑 WebChat 想要呈现的是"跨频道的统一视图"还是"按渠道拆分的视图"。

---

## 回复上下文（Reply context）

当一条消息是"回复"（reply）时，OpenClaw 会尽量把有用的被回复信息一并放到入站上下文中，方便智能体做出更合适的回答。

入站回复通常会包含：

- `ReplyToId`：被回复消息的 ID；
- `ReplyToBody`：被回复消息的内容；
- `ReplyToSender`：被回复消息的发送者信息。

同时，被引用的内容还会以一个形如 `[Replying to ...]` 的小块追加到 `Body` 字段中。
这样做的好处是：无论具体来源于哪个频道（channel），你在智能体侧看到的回复上下文结构都是一致的。

[广播群组（Broadcast Groups）](./broadcast-groups.md) [频道位置解析（Channel Location Parsing）](./location.md)