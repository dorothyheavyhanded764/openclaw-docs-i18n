

  CLI 命令

  
# node

运行一个**无头节点（node）主机**，它会连接到 Gateway WebSocket，并在本机暴露 `system.run` 和 `system.which` 能力。

## 什么时候需要节点主机？

当你想让智能体（agent）在网络中的**其他机器上执行命令**，又不想在那台机器上安装完整的 macOS 伴侣应用时，节点主机就是最佳选择。典型场景包括：

- 在远程 Linux/Windows 服务器上执行命令（构建服务器、实验室机器、NAS 等）
- 把命令执行**沙盒化**在 gateway 上，但把已批准的任务委托给其他主机执行
- 为自动化流程或 CI 节点提供一个轻量级、无界面的执行目标

放心，所有命令执行仍然受**执行审批**机制和节点主机上的智能体白名单约束，你可以精确控制谁能执行什么命令。

## 浏览器代理（零配置）

如果节点上的 `browser.enabled` 没有被禁用，节点主机会自动启用浏览器代理功能。这样一来，智能体（agent）就可以在该节点上进行浏览器自动化操作，无需任何额外配置。

如果不需要这个功能，可以在节点配置中禁用：

```json
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```

## 前台运行

```bash
openclaw node run --host <gateway-host> --port 18789
```

**参数说明：**

- `--host `：Gateway WebSocket 主机地址（默认：`127.0.0.1`）
- `--port `：Gateway WebSocket 端口（默认：`18789`）
- `--tls`：使用 TLS 加密与 gateway 的连接
- `--tls-fingerprint `：预期的 TLS 证书指纹（sha256 格式）
- `--node-id `：自定义节点 ID（会清除配对令牌）
- `--display-name `：自定义节点显示名称

## 后台服务

把无头节点主机安装为用户服务，让它常驻后台运行：

```bash
openclaw node install --host <gateway-host> --port 18789
```

**参数说明：**

- `--host `：Gateway WebSocket 主机地址（默认：`127.0.0.1`）
- `--port `：Gateway WebSocket 端口（默认：`18789`）
- `--tls`：使用 TLS 加密与 gateway 的连接
- `--tls-fingerprint `：预期的 TLS 证书指纹（sha256 格式）
- `--node-id `：自定义节点 ID（会清除配对令牌）
- `--display-name `：自定义节点显示名称
- `--runtime `：服务运行时环境（`node` 或 `bun`）
- `--force`：强制重新安装/覆盖已有安装

**服务管理命令：**

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

如果你只是临时运行节点主机，不想安装服务，直接用 `openclaw node run` 即可。服务管理命令支持 `--json` 参数，输出机器可读的 JSON 格式。

## 配对流程

节点主机首次连接时，会在 Gateway 上创建一个待审批的设备配对请求（角色为 `node`）。你需要手动批准：

```bash
openclaw devices list
openclaw devices approve <requestId>
```

配对成功后，节点主机会将节点 ID、令牌、显示名称和 gateway 连接信息保存在 `~/.openclaw/node.json` 文件中。

## 执行审批

`system.run` 命令的执行受本地审批规则约束，相关配置和命令如下：

- 配置文件：`~/.openclaw/exec-approvals.json`
- 文档：[执行审批](../tools/exec-approvals.md)
- 从 Gateway 编辑：`openclaw approvals --node <id|name|ip>`

[models](./models.md)[nodes](./nodes.md)