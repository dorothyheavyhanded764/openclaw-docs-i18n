

  消息平台

  
# Google Chat

状态：已就绪，支持通过 Google Chat API webhook（仅 HTTP）进行私聊和群组空间通信。

## 快速设置（新手入门）

本节帮助你完成 Google Chat 频道的基本配置，让智能体能够接收和回复消息。

1.  创建 Google Cloud 项目并启用 **Google Chat API**。
    -   访问：[Google Chat API 凭据页面](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
    -   如果 API 尚未启用，请先启用它。
2.  创建 **服务账号（Service Account）**：
    -   点击 **创建凭据** > **服务账号**。
    -   随意命名（例如 `openclaw-chat`）。
    -   权限留空，直接点击 **继续**。
    -   可访问的主体留空，点击 **完成**。
3.  创建并下载 **JSON 密钥**：
    -   在服务账号列表中，点击刚才创建的账号。
    -   进入 **密钥** 标签页。
    -   点击 **添加密钥** > **创建新密钥**。
    -   选择 **JSON** 格式，点击 **创建**。
4.  将下载的 JSON 文件保存到网关主机（例如 `~/.openclaw/googlechat-service-account.json`）。
5.  在 [Google Cloud Console Chat 配置页面](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) 创建 Google Chat 应用：
    -   填写 **应用信息**：
        -   **应用名称**：例如 `OpenClaw`
        -   **头像 URL**：例如 `https://openclaw.ai/logo.png`
        -   **描述**：例如 `个人 AI 助手`
    -   启用 **交互功能**。
    -   在 **功能** 下勾选 **加入空间和群组对话**。
    -   在 **连接设置** 下选择 **HTTP 端点 URL**。
    -   在 **触发器** 下选择 **对所有触发器使用通用 HTTP 端点 URL**，并填写你的网关公网 URL 加上 `/googlechat` 路径。
        -   *提示：运行 `openclaw status` 可查看网关的公网 URL。*
    -   在 **可见性** 下勾选 **使此 Chat 应用对 `<你的域名>` 中的特定人员和群组可用**。
    -   在文本框中输入你的邮箱地址（例如 `user@example.com`）。
    -   点击底部的 **保存**。
6.  **启用应用状态**：
    -   保存后，**刷新页面**。
    -   找到 **应用状态** 部分（通常在保存后显示在页面顶部或底部）。
    -   将状态改为 **上线 - 对用户可用**。
    -   再次点击 **保存**。
7.  配置 OpenClaw，指定服务账号路径和 webhook 受众：
    -   环境变量方式：`GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
    -   或配置文件方式：`channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`
8.  设置 webhook 受众类型和值（需与 Chat 应用配置匹配）。
9.  启动网关。Google Chat 会向你的 webhook 路径发送 POST 请求。

## 添加到 Google Chat

网关运行起来后，且你的邮箱已添加到可见性列表，就可以开始使用了：

1.  打开 [Google Chat](https://chat.google.com/)。
2.  点击 **私信** 旁边的 **+** 号。
3.  在搜索栏中输入你在 Google Cloud Console 配置的 **应用名称**。
    -   **注意**：这是私有应用，不会出现在应用市场的列表中，必须通过名称搜索才能找到。
4.  从搜索结果中选择你的机器人。
5.  点击 **添加** 或 **聊天** 开始一对一对话。
6.  发送"Hello"测试智能体！

## 公网 URL（仅 Webhook）

Google Chat webhook 需要公网可访问的 HTTPS 端点。出于安全考虑，**只把 `/googlechat` 路径暴露到公网**，OpenClaw 仪表板和其他敏感端点应保持在私有网络中。

### 方案 A：Tailscale Funnel（推荐）

使用 Tailscale Serve 暴露私有仪表板，用 Funnel 暴露公网 webhook 路径。这样 `/` 路径保持私有，只有 `/googlechat` 对外开放。

1.  **检查网关绑定的地址：**

    复制

    ```bash
    ss -tlnp | grep 18789
    ```

    记下 IP 地址（可能是 `127.0.0.1`、`0.0.0.0` 或你的 Tailscale IP 如 `100.x.x.x`）。
2.  **将仪表板仅暴露给 tailnet（端口 8443）：**

    复制

    ```bash
    # 如果绑定到 localhost (127.0.0.1 或 0.0.0.0):
    tailscale serve --bg --https 8443 http://127.0.0.1:18789

    # 如果仅绑定到 Tailscale IP (例如 100.106.161.80):
    tailscale serve --bg --https 8443 http://100.106.161.80:18789
    ```

3.  **仅将 webhook 路径暴露到公网：**

    复制

    ```bash
    # 如果绑定到 localhost (127.0.0.1 或 0.0.0.0):
    tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

    # 如果仅绑定到 Tailscale IP (例如 100.106.161.80):
    tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
    ```

4.  **授权节点使用 Funnel：** 如果提示授权，访问输出中显示的授权 URL，在 tailnet 策略中为此节点启用 Funnel。
5.  **验证配置：**

    复制

    ```
    tailscale serve status
    tailscale funnel status
    ```

你的公网 webhook URL 为：`https://<节点名称>..ts.net/googlechat`，私有仪表板地址为：`https://<节点名称>..ts.net:8443/`。在 Google Chat 应用配置中使用公网 URL（不带 `:8443`）。

> 注意：此配置在重启后仍然有效。如需移除，运行 `tailscale funnel reset` 和 `tailscale serve reset`。

### 方案 B：反向代理（Caddy）

如果你使用 Caddy 等反向代理，只代理特定路径：

```
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

这样配置后，所有访问 `your-domain.com/` 的请求都会被忽略或返回 404，而 `your-domain.com/googlechat` 会安全地路由到 OpenClaw。

### 方案 C：Cloudflare Tunnel

配置隧道的入口规则，只路由 webhook 路径：

-   **路径**：`/googlechat` -> `http://localhost:18789/googlechat`
-   **默认规则**：HTTP 404（未找到）

## 工作原理

了解 Google Chat 与 OpenClaw 的交互流程：

1.  Google Chat 向网关发送 webhook POST 请求，每个请求都带有 `Authorization: Bearer ` 标头。
    -   当标头存在时，OpenClaw 会在读取和解析完整请求体之前先验证身份。
    -   对于请求体中携带 `authorizationEventObject.systemIdToken` 的 Google Workspace 插件请求，采用更严格的预认证体量限制。
2.  OpenClaw 根据配置的 `audienceType` 和 `audience` 验证令牌：
    -   `audienceType: "app-url"` → 受众为你的 HTTPS webhook URL。
    -   `audienceType: "project-number"` → 受众为 Cloud 项目编号。
3.  消息按空间类型路由：
    -   私聊会话键：`agent::googlechat:dm:`
    -   群组会话键：`agent::googlechat:group:`
4.  私聊默认采用配对机制。未识别的发送者会收到配对码，使用以下命令批准：
    -   `openclaw pairing approve googlechat `
5.  群组空间默认需要 @提及才能触发。如果提及检测需要应用的用户名，请配置 `botUser`。

## 目标标识符

用于消息投递和允许列表的标识符格式：

-   私信：`users/`（推荐）
-   原始邮箱地址 `name@example.com` 是可变的，仅在配置了 `channels.googlechat.dangerouslyAllowNameMatching: true` 时用于直接匹配允许列表。
-   已弃用：`users/` 会被视为用户 ID，而非邮箱允许列表。
-   群组空间：`spaces/`

## 配置要点

```json
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      // 或 serviceAccountRef: { source: "file", provider: "filemain", id: "/channels/googlechat/serviceAccount" }
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // 可选；有助于提及检测
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "简短回答即可。",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

注意事项：

-   服务账号凭据也可以通过 `serviceAccount`（JSON 字符串）内联传递。
-   也支持 `serviceAccountRef`（环境变量/文件 SecretRef），包括 `channels.googlechat.accounts..serviceAccountRef` 下的每个账号引用。
-   如果未设置 `webhookPath`，默认路径为 `/googlechat`。
-   `dangerouslyAllowNameMatching` 可重新启用可变邮箱主体匹配用于允许列表（应急兼容模式）。
-   启用 `actions.reactions` 后，可通过 `reactions` 工具和 `channels action` 使用表情回应功能。
-   `typingIndicator` 支持 `none`、`message`（默认）和 `reaction`（需要用户 OAuth）。
-   附件通过 Chat API 下载并存储在媒体管道中（大小受 `mediaMaxMb` 限制）。

密钥管理详情参考：[密钥管理](../gateway/secrets.md)。

## 故障排除

### 405 方法不允许

如果 Google Cloud Logs Explorer 显示类似错误：

```bash
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

说明 webhook 处理器未注册。常见原因：

1.  **频道未配置**：配置中缺少 `channels.googlechat` 部分。运行以下命令验证：

    复制

    ```bash
    openclaw config get channels.googlechat
    ```

    如果返回"Config path not found"，请添加配置（参见[配置要点](#config-highlights)）。
2.  **插件未启用**：检查插件状态：

    复制

    ```bash
    openclaw plugins list | grep googlechat
    ```

    如果显示"disabled"，请在配置中添加 `plugins.entries.googlechat.enabled: true`。
3.  **网关未重启**：添加配置后需要重启网关：

    复制

    ```bash
    openclaw gateway restart
    ```

验证频道是否正在运行：

```bash
openclaw channels status
# 应显示：Google Chat default: enabled, configured, ...
```

### 其他问题

-   使用 `openclaw channels status --probe` 检查身份验证错误或缺少受众配置。
-   如果没有消息到达，确认 Chat 应用的 webhook URL 和事件订阅配置正确。
-   如果提及门控阻止了回复，将 `botUser` 设置为应用的用户资源名称并检查 `requireMention` 配置。
-   发送测试消息时使用 `openclaw logs --follow` 查看请求是否到达网关。

相关文档：

-   [网关配置](../gateway/configuration.md)
-   [安全性](../gateway/security.md)
-   [表情回应](../tools/reactions.md)

[飞书](./feishu.md)[iMessage](./imessage.md)