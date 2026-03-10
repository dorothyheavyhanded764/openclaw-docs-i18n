

  CLI 命令

  
# acp

运行 [Agent Client Protocol (ACP)](https://agentclientprotocol.com/) 桥接器，与 OpenClaw 网关通信。这个命令通过 stdio 与 IDE 进行 ACP 通信，然后通过 WebSocket 把提示转发给网关，同时维护 ACP 会话与网关会话密钥之间的映射。

## 用法

```bash
openclaw acp

# 连接远程网关
openclaw acp --url wss://gateway-host:18789 --token <token>

# 连接远程网关（从文件读取令牌）
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# 附加到已有的会话密钥
openclaw acp --session agent:main:main

# 通过标签附加（会话必须已存在）
openclaw acp --session-label "support inbox"

# 在首个提示之前重置会话密钥
openclaw acp --session agent:main:main --reset-session
```

## ACP 客户端（调试用）

内置的 ACP 客户端可以帮你在没有 IDE 的情况下验证桥接器是否正常工作。它会启动 ACP 桥接器，让你可以交互式地输入提示。

```bash
openclaw acp client

# 让启动的桥接器连接远程网关
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# 覆盖服务器命令（默认是 openclaw）
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

客户端调试模式的权限模型：

- 自动批准采用白名单机制，仅适用于受信任的核心工具 ID
- `read` 自动批准仅限于当前工作目录（设置了 `--cwd` 时生效）
- 未知/非核心工具名、越界读取、危险工具始终需要显式确认
- 服务端提供的 `toolCall.kind` 视为不可信的元数据，不作为授权依据

## 使用场景

当你的 IDE（或其他客户端）使用 Agent Client Protocol，而你想让它驱动 OpenClaw 网关会话时，就需要用到 ACP。

基本步骤：

1. 确保网关已运行（本地或远程均可）
2. 配置网关目标地址（通过配置文件或命令行参数）
3. 让你的 IDE 通过 stdio 运行 `openclaw acp`

持久化配置示例：

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

直接运行示例（不写入配置）：

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
# 本地进程安全起见，推荐这种方式
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```

## 选择智能体（agent）

ACP 本身不直接选择智能体，而是通过网关会话密钥来路由。如果你想指定某个智能体，就用智能体作用域的会话密钥：

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

每个 ACP 会话映射到一个网关会话密钥。一个智能体可以有多个会话；如果不指定密钥或标签，ACP 默认使用隔离的 `acp:` 会话。

## Zed 编辑器配置

在 `~/.config/zed/settings.json` 中添加自定义 ACP 智能体（也可以用 Zed 的设置 UI）：

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

如果需要指定网关或智能体：

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

配置好后，在 Zed 里打开 Agent 面板，选择"OpenClaw ACP"就能开始对话了。

## 会话映射

默认情况下，ACP 会话会获得一个带 `acp:` 前缀的独立网关会话密钥。如果你想复用已有的会话，可以通过密钥或标签指定：

- `--session `：使用指定的网关会话密钥
- `--session-label `：通过标签查找已有会话
- `--reset-session`：为该密钥生成新的会话 ID（密钥不变，对话记录重置）

如果你的 ACP 客户端支持元数据，可以在每次会话时覆盖：

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

更多关于会话密钥的内容，参见 [/concepts/session](../concepts/session.md)。

## 参数说明

- `--url `：网关 WebSocket URL（配置了 gateway.remote.url 时默认使用该值）
- `--token `：网关认证令牌
- `--token-file `：从文件读取网关认证令牌
- `--password `：网关认证密码
- `--password-file `：从文件读取网关认证密码
- `--session `：默认会话密钥
- `--session-label `：要解析的默认会话标签
- `--require-existing`：会话密钥/标签不存在时报错
- `--reset-session`：首次使用前重置会话密钥
- `--no-prefix-cwd`：不在提示前添加工作目录前缀
- `--verbose, -v`：向 stderr 输出详细日志

安全提示：

- `--token` 和 `--password` 在某些系统的进程列表中可能可见
- 推荐使用 `--token-file`/`--password-file` 或环境变量（`OPENCLAW_GATEWAY_TOKEN`、`OPENCLAW_GATEWAY_PASSWORD`）
- ACP 运行时后端子进程会收到 `OPENCLAW_SHELL=acp`，可用于编写针对性的 shell/profile 规则
- `openclaw acp client` 会在启动的桥接器进程上设置 `OPENCLAW_SHELL=acp-client`

### acp client 参数

- `--cwd `：ACP 会话的工作目录
- `--server `：ACP 服务器命令（默认：`openclaw`）
- `--server-args <args...>`：传给 ACP 服务器的额外参数
- `--server-verbose`：在 ACP 服务器上启用详细日志
- `--verbose, -v`：客户端详细日志

[CLI 参考](../cli.md)[agent](./agent.md)