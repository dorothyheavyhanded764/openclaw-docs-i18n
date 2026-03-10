

  引导

  
# 智能体引导

引导/初始化是智能体**首次运行**时的核心流程，用于准备工作区并收集身份信息。它发生在入门引导之后，也就是智能体第一次启动的时候。

## 引导流程做了什么

智能体首次启动时，OpenClaw 会对工作区进行初始化（默认路径为 `~/.openclaw/workspace`）：

-   创建初始文件：`AGENTS.md`、`BOOTSTRAP.md`、`IDENTITY.md`、`USER.md`
-   运行简短的问答流程，逐个问题收集信息
-   将身份信息和偏好设置写入 `IDENTITY.md`、`USER.md`、`SOUL.md`
-   完成后自动删除 `BOOTSTRAP.md`，确保流程只执行一次

## 在哪里运行

引导流程始终在**网关主机**上执行。如果你使用 macOS 应用连接到远程网关，工作区和引导文件都会存放在那台远程机器上。

> **ℹ️** 当网关运行在另一台机器上时，请直接在网关主机上编辑工作区文件（例如 `user@gateway-host:~/.openclaw/workspace`）。

## 相关文档

-   macOS 应用入门：[入门引导](./onboarding.md)
-   工作区结构：[智能体工作区](../concepts/agent-workspace.md)

[OAuth](../concepts/oauth.md)[会话管理](../concepts/session.md)