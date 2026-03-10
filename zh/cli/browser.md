

  CLI 命令

  
# browser

这一节帮助你通过命令行控制浏览器——无论是管理标签页、获取页面快照，还是执行点击、输入等自动化操作。

相关链接：

-   浏览器工具 + API：[浏览器工具](../tools/browser.md)
-   Chrome 扩展中继：[Chrome 扩展](../tools/chrome-extension.md)

## 常用标志

以下标志适用于大多数 `browser` 子命令：

-   `--url `：网关 WebSocket URL（默认使用配置值）
-   `--token `：网关令牌（如需认证）
-   `--timeout `：请求超时时间（毫秒）
-   `--browser-profile `：指定浏览器（browser）配置文件（默认使用配置值）
-   `--json`：输出 JSON 格式（部分命令支持）

## 快速开始（本地）

先感受一下基本操作：

```bash
openclaw browser --browser-profile chrome tabs
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

## 配置文件（Profiles）

配置文件定义了浏览器（browser）的连接方式。OpenClaw 提供两种开箱即用的模式：

-   `openclaw`：启动或附加到一个由 OpenClaw 托管的独立 Chrome 实例，使用隔离的用户数据目录，适合自动化任务
-   `chrome`：通过 Chrome 扩展中继控制你日常使用的 Chrome 标签页，适合需要复用已有浏览环境的场景

管理配置文件：

```bash
openclaw browser profiles
openclaw browser create-profile --name work --color "#FF5A36"
openclaw browser delete-profile --name work
```

使用指定配置文件：

```bash
openclaw browser --browser-profile work tabs
```

## 标签页操作

查看当前标签页、打开新页面、切换焦点、关闭标签：

```bash
openclaw browser tabs
openclaw browser open https://docs.openclaw.ai
openclaw browser focus <targetId>
openclaw browser close <targetId>
```

## 快照、截图与交互操作

获取页面快照（用于智能体理解页面结构）：

```bash
openclaw browser snapshot
```

截取页面图片：

```bash
openclaw browser screenshot
```

执行导航、点击、输入（基于元素引用的 UI 自动化）：

```bash
openclaw browser navigate https://example.com
openclaw browser click <ref>
openclaw browser type <ref> "hello"
```

## Chrome 扩展中继（手动附加模式）

如果你想用智能体（agent）控制自己正在用的 Chrome 标签页，可以通过 Chrome 扩展实现。注意：扩展不会自动附加到标签页，你需要手动点击工具栏按钮来建立连接。

首先安装扩展到本地：

```bash
openclaw browser extension install
openclaw browser extension path
```

然后在 Chrome 中操作：打开 `chrome://extensions`，启用"开发者模式"，点击"加载已解压的扩展程序"，选择上面命令输出的文件夹即可。

完整配置指南请参考：[Chrome 扩展](../tools/chrome-extension.md)

## 远程浏览器控制（节点主机代理）

当网关运行在与浏览器不同的机器上时，你可以在装有 Chrome/Brave/Edge/Chromium 的机器上运行一个**节点主机**。网关会自动将浏览器操作代理到该节点，无需单独部署浏览器控制服务。

配置说明：

-   使用 `gateway.nodes.browser.mode` 控制自动路由行为
-   如果有多个节点连接，使用 `gateway.nodes.browser.node` 指定固定节点

更多远程访问与安全配置请参考：[浏览器工具](../tools/browser.md)、[远程访问](../gateway/remote.md)、[Tailscale](../gateway/tailscale.md)、[安全](../gateway/security.md)

[approvals](./approvals.md)[channels](./channels.md)