

  网络与发现

  
# 网络模型

绝大多数操作都经由网关（`openclaw gateway`）处理——这是一个常驻后台的进程，负责管理所有频道连接和 WebSocket 控制平面。

## 核心规则

-   **一机一网关**是推荐配置。网关是唯一被允许持有 WhatsApp Web 会话的进程。如果你需要运行救援机器人或实现严格的资源隔离，可以启动多个网关实例，但必须使用独立的配置文件和端口。详见[多网关](./multiple-gateways.md)。
-   **本地回环优先**：网关的 WebSocket 默认绑定 `ws://127.0.0.1:18789`。配置向导会自动生成网关令牌，即使是本地回环连接也不例外。若要通过 tailnet 访问，需要运行 `openclaw gateway --bind tailnet --token ...`，因为非本地回环绑定必须使用令牌认证。
-   节点根据实际需求，通过局域网、tailnet 或 SSH 隧道连接到网关 WebSocket。旧版的 TCP 桥接方式已弃用。
-   画布主机由网关的 HTTP 服务器提供，使用**与网关相同的端口**（默认 `18789`）：
    -   `/__openclaw__/canvas/`
    -   `/__openclaw__/a2ui/` 当配置了 `gateway.auth` 且网关绑定到非本地回环地址时，这些路由会受到网关认证保护。节点客户端会使用与当前活跃 WebSocket 会话绑定的节点级能力 URL。详见[网关配置](./configuration.md)（`canvasHost`、`gateway` 字段）。
-   远程使用通常通过 SSH 隧道或 tailnet VPN 实现。详见[远程访问](./remote.md)和[服务发现](./discovery.md)。

[本地模型](./local-models.md)[网关配对](./pairing.md)