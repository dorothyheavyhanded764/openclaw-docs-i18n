

  第一步

  
# 快速入门

本节帮你从零开始，用最少的配置完成第一次对话。

> **ℹ️** 想最快开始聊天？直接打开控制界面（无需配置频道）。运行 `openclaw dashboard` 在浏览器中聊天，或在网关主机上访问 `http://127.0.0.1:18789/`。详见 [仪表板](../web/dashboard.md) 和 [控制界面](../web/control-ui.md)。

## 前提条件

- Node 22 或更高版本

> **💡** 不确定当前版本？用 `node --version` 查一下。

## 快速安装（CLI）

### 步骤 1：安装 OpenClaw（推荐）

> **ℹ️** 其他安装方式和系统要求，见 [安装](../install.md)。

### 步骤 2：运行配置向导

```bash
openclaw onboard --install-daemon
```

向导会帮你完成认证、网关设置，以及可选的频道配置。详见 [配置向导](./wizard.md)。

### 步骤 3：确认网关状态

如果已安装服务，现在应该已经在运行了：

```bash
openclaw gateway status
```

### 步骤 4：打开控制界面

```bash
openclaw dashboard
```

> **✅** 控制界面正常加载，说明你的网关已经可以使用了。

## 可选操作

适合快速测试或排查问题。

```bash
openclaw gateway --port 18789
```

需要先配置好频道。

```bash
openclaw message send --target +15555550123 --message "Hello from OpenClaw"
```

## 常用环境变量

如果你用服务账户运行 OpenClaw，或想自定义配置和数据的存放位置：

- `OPENCLAW_HOME`：设置主目录，用于内部路径解析。
- `OPENCLAW_STATE_DIR`：覆盖状态数据目录。
- `OPENCLAW_CONFIG_PATH`：覆盖配置文件路径。

完整环境变量说明见 [环境变量](../help/environment.md)。

## 进阶阅读

## 完成后你将拥有

- 一个运行中的网关（Gateway）
- 已配置好的认证
- 可用的控制界面，或一个已连接的频道（channel）

## 接下来

- 私信安全与配对审批：[配对](../channels/pairing.md)
- 连接更多频道：[频道](../channels.md)
- 高级工作流与源码安装：[完整设置](./setup.md)

[功能特性](../concepts/features.md)[配置概览](./onboarding-overview.md)