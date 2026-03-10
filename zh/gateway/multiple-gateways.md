

  配置与运维

  
# 多网关

大多数情况下，一台主机只需要运行一个网关就足够了——单个网关完全可以同时处理多个消息连接和智能体。但如果你需要更强的隔离性或冗余能力（比如搭建救援机器人），可以运行多个独立的网关实例，各自使用隔离的配置文件和端口。

## 隔离检查清单（必读）

-   `OPENCLAW_CONFIG_PATH` — 每个实例独立的配置文件
-   `OPENCLAW_STATE_DIR` — 每个实例独立的会话数据、凭据和缓存目录
-   `agents.defaults.workspace` — 每个实例独立的工作空间根目录
-   `gateway.port`（或 `--port`）— 每个实例使用不同的端口
-   派生端口（浏览器/画布）之间不能有任何重叠

如果以上任意一项被多个实例共享，就会遇到配置竞争和端口冲突问题。

## 推荐：使用 profile（--profile）

使用 profile 可以自动管理 `OPENCLAW_STATE_DIR` 和 `OPENCLAW_CONFIG_PATH` 的作用域，还会为系统服务名称自动添加后缀。

```bash
# 主实例
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# 救援实例
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

为每个 profile 安装系统服务：

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## 救援机器人指南

在同一台主机上运行第二个网关时，它需要拥有独立的：

-   配置文件/配置
-   状态目录
-   工作空间
-   基础端口（以及派生端口）

这样救援机器人就能与主机器人完全隔离——当主机器人出现故障时，救援机器人可以独立进行调试或应用配置变更。端口间隔建议：基础端口之间至少预留 20 个端口的间隔，这样派生的浏览器/画布/CDP 端口就永远不会发生冲突。

### 安装步骤（救援机器人）

```bash
# 主机器人（现有实例或全新安装，不使用 --profile 参数）
# 运行在端口 18789，以及 Chrome CDP/Canvas 等派生端口
openclaw onboard
openclaw gateway install

# 救援机器人（隔离的 profile + 端口）
openclaw --profile rescue onboard
# 注意事项：
# - 工作空间名称默认会自动添加 -rescue 后缀
# - 端口建议至少距离 18789 有 20 个端口的间隔，
#   更好的做法是选择完全不同的基础端口，比如 19789
# - 其余引导流程与正常安装相同

# 安装系统服务（如果引导过程中没有自动安装）
openclaw --profile rescue gateway install
```

## 端口映射（派生关系）

基础端口 = `gateway.port`（或 `OPENCLAW_GATEWAY_PORT` / `--port`）。

-   浏览器控制服务端口 = 基础端口 + 2（仅限本地回环）
-   画布主机由网关 HTTP 服务器提供（使用与 `gateway.port` 相同的端口）
-   浏览器配置文件的 CDP 端口从 `browser.controlPort + 9 .. + 108` 自动分配

如果你在配置文件或环境变量中手动覆盖了这些端口，务必确保每个实例的端口都是唯一的。

## 浏览器/CDP 注意事项（常见陷阱）

-   **绝对不要**在多个实例中将 `browser.cdpUrl` 设置为相同的值。
-   每个实例都需要自己独立的浏览器控制端口和 CDP 端口范围（从各自的网关端口派生）。
-   如果需要显式指定 CDP 端口，请为每个实例单独设置 `browser.profiles..cdpPort`。
-   使用远程 Chrome 时：使用 `browser.profiles..cdpUrl`（每个配置文件、每个实例独立设置）。

## 手动环境变量示例

```
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```

## 快速检查命令

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```

[后台执行与进程工具](./background-process.md)[故障排除](./troubleshooting.md)