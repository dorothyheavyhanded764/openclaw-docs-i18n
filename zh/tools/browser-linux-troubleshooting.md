

  浏览器（browser）

  
# 浏览器故障排除（troubleshooting）

## 问题：启动 Chrome CDP 时端口 18800 失败

当 OpenClaw 的浏览器控制服务器尝试启动 Chrome、Brave、Edge 或 Chromium 时，可能会遇到这个错误：

```json
{"error":"Error: Failed to start Chrome CDP on port 18800 for profile \"openclaw\"."}
```

这节内容将帮你诊断并解决这个问题。

### 根本原因

在 Ubuntu（以及许多 Linux 发行版）上，系统默认的 Chromium 其实是个 **snap 包**。问题在于，snap 的 AppArmor 沙盒限制会干扰 OpenClaw 启动和管理浏览器进程的方式。

当你运行 `apt install chromium` 时，安装的只是一个指向 snap 的"占位包"：

```
Note, selecting 'chromium-browser' instead of 'chromium'
chromium-browser is already the newest version (2:1snap1-0ubuntu2).
```

注意，这**不是**真正的浏览器，只是一个包装脚本而已。

### 解决方案 1：安装 Google Chrome（推荐）

最简单可靠的方法是安装官方 Google Chrome 的 `.deb` 包——它不受 snap 沙盒限制：

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt --fix-broken install -y  # 如果提示依赖问题，运行这行
```

安装完成后，更新 OpenClaw 配置文件（`~/.openclaw/openclaw.json`），指定 Chrome 的路径：

```json
{
  "browser": {
    "enabled": true,
    "executablePath": "/usr/bin/google-chrome-stable",
    "headless": true,
    "noSandbox": true
  }
}
```

### 解决方案 2：配合"仅附加模式"使用 Snap Chromium

如果你必须使用 snap 版的 Chromium，可以让 OpenClaw 以"附加模式"运行——即连接到一个你手动启动的浏览器实例。

1. 首先修改配置，启用附加模式：

```json
{
  "browser": {
    "enabled": true,
    "attachOnly": true,
    "headless": true,
    "noSandbox": true
  }
}
```

2. 然后手动启动 Chromium，开启远程调试端口：

```
chromium-browser --headless --no-sandbox --disable-gpu \
  --remote-debugging-port=18800 \
  --user-data-dir=$HOME/.openclaw/browser/openclaw/user-data \
  about:blank &
```

3. 可选：创建一个 systemd 用户服务，让浏览器开机自启：

```bash
# ~/.config/systemd/user/openclaw-browser.service
[Unit]
Description=OpenClaw Browser (Chrome CDP)
After=network.target

[Service]
ExecStart=/snap/bin/chromium --headless --no-sandbox --disable-gpu --remote-debugging-port=18800 --user-data-dir=%h/.openclaw/browser/openclaw/user-data about:blank
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

启用服务：

```bash
systemctl --user enable --now openclaw-browser.service
```

### 验证浏览器是否正常工作

检查浏览器服务状态：

```bash
curl -s http://127.0.0.1:18791/ | jq '{running, pid, chosenBrowser}'
```

测试浏览器功能：

```bash
curl -s -X POST http://127.0.0.1:18791/start
curl -s http://127.0.0.1:18791/tabs
```

### 配置字段说明

| 选项 | 说明 | 默认值 |
| --- | --- | --- |
| `browser.enabled` | 是否启用浏览器控制功能 | `true` |
| `browser.executablePath` | 浏览器可执行文件路径，支持 Chrome、Brave、Edge、Chromium 等基于 Chromium 的浏览器 | 自动检测，优先使用系统默认的 Chromium 系浏览器 |
| `browser.headless` | 是否以无头模式运行（不显示图形界面） | `false` |
| `browser.noSandbox` | 是否添加 `--no-sandbox` 标志，某些 Linux 环境需要此选项 | `false` |
| `browser.attachOnly` | 仅附加模式：不自动启动浏览器，只连接到已存在的实例 | `false` |
| `browser.cdpPort` | Chrome DevTools 协议监听端口 | `18800` |

## 问题：Chrome 扩展中继已运行，但没有标签页连接

如果你使用的是 `chrome` 配置文件（即扩展中继模式），这个错误表示 OpenClaw 期望浏览器扩展已附加到一个活动的标签页。解决方法有两种：

1. **使用托管浏览器模式**：运行 `openclaw browser start --browser-profile openclaw`，或在配置中设置 `browser.defaultProfile: "openclaw"`。

2. **正确使用扩展中继**：安装 OpenClaw 浏览器扩展，打开一个标签页，然后点击扩展图标将其附加到当前页面。

注意事项：

- `chrome` 配置文件会优先使用你**系统的默认 Chromium 系浏览器**。
- 本地的 `openclaw` 配置文件会自动分配 `cdpPort` 和 `cdpUrl`，这两个字段只在连接远程 CDP 时才需要手动设置。

[Chrome 扩展](./chrome-extension.md)[智能体发送](./agent-send.md)