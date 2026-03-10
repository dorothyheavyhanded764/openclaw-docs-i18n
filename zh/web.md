

  Web 界面

  
# Web

网关（Gateway）在启动时，会在同一个端口上同时提供 WebSocket 服务和一个轻量的**浏览器控制界面**（Control UI，基于 Vite + Lit 构建）。

你可以通过以下地址访问：

- 默认地址：`http://:18789/`
- 如果你配置了路径前缀：`http://:18789/`（通过 `gateway.controlUi.basePath` 设置）

控制界面能做什么？详见 [控制界面](./web/control-ui.md)。这一页我们重点讲：**如何绑定端口、如何配置安全策略、以及怎么让网关对外暴露**。

---

## Webhooks

只要在配置里启用 `hooks.enabled=true`，网关就会在同一个 HTTP 服务器上再开一个 Webhook 端点。

关于 Webhook 的认证方式和请求体格式，推荐你直接看 [网关配置](./gateway/configuration.md) 里的 `hooks` 小节，里面有完整说明。

---

## 配置说明（默认启用）

只要静态资源存在（`dist/control-ui` 目录），控制界面就会**默认启用**。

如果你想手动开关，或者自定义访问路径，可以这样配：

```json
{
  gateway: {
    controlUi: { enabled: true, basePath: "/openclaw" }, // basePath 可选
  },
}
```

---

## 通过 Tailscale 访问

如果你希望从外部网络安全地访问网关，Tailscale 是推荐的做法。有三种模式可选，下面我们逐个说清楚：每种模式适合什么场景、怎么配置、有什么要注意的。

### 模式一：Tailscale Serve（推荐）

**推荐做法**：让网关只监听本地回环地址（loopback），然后让 Tailscale Serve 做反向代理。

为什么推荐这样？因为 Tailscale 会自动帮你处理 HTTPS 和身份认证，你不需要自己管令牌或密码。

配置很简单：

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" },
  },
}
```

然后启动网关：

```bash
openclaw gateway
```

访问地址：

- `https:///`（如果你设置了 `basePath`，就是 `https:///`）

**适用场景**：你希望安全地从 Tailnet 内任何设备访问网关，又不想自己管理认证凭据。

### 模式二：Tailnet 绑定 + 令牌认证

如果你希望网关直接绑定到 Tailscale 网络（Tailnet），而不是通过 Serve 代理，可以用这个模式。

配置如下：

```json
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" },
  },
}
```

启动网关：

```bash
openclaw gateway
```

访问地址：

- `http://<tailscale-ip>:18789/`（或你配置的 `basePath` 路径）

**注意**：任何非 loopback 的绑定方式，都**必须**配置认证（令牌或密码），否则网关会拒绝启动。

**适用场景**：你需要在 Tailnet 内多设备直接访问网关，且愿意自己管理令牌。

### 模式三：公网访问（Funnel）

如果你需要临时把网关暴露到公网——比如做演示、远程调试——可以用 Funnel 模式。

配置如下：

```json
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" }, // 也可以用环境变量 OPENCLAW_GATEWAY_PASSWORD
  },
}
```

**重要**：Funnel 会把你的网关暴露到公网，所以**必须**使用密码认证，而且密码要足够强。

**适用场景**：临时演示、远程调试。不建议长期使用。

---

## 安全注意事项

在部署控制界面和 Webhook 之前，建议你先搞清楚这几条安全规则：

### 1. 默认需要认证

网关默认要求认证——令牌/密码 或 Tailscale 身份头，不允许匿名访问。

### 2. 非 loopback 绑定必须有认证

任何非本地回环的绑定（比如 `tailnet`、`0.0.0.0`），都**必须**设置共享令牌或密码，通过 `gateway.auth` 或环境变量配置。

### 3. 向导默认会生成令牌

即使绑定到 loopback，`openclaw setup` 向导也会默认生成一个网关令牌。

### 4. 控制界面如何传递凭据

控制界面在建立 WebSocket 连接时，会通过 `connect.params.auth.token` 或 `connect.params.auth.password` 发送凭据。

### 5. 非 loopback 部署必须设置 allowedOrigins

如果你要把控制界面部署到非本地环境，必须显式设置 `gateway.controlUi.allowedOrigins`（填写完整的来源 origin）。

为什么？为了防止 CSRF 攻击。如果不设置，网关会拒绝启动。

### 6. 谨慎使用 Host 头回退模式

`gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` 可以启用基于 Host 头的 origin 回退。

但这是一种**危险的安全降级**，不建议在生产环境使用。

### 7. Tailscale 身份头可以替代令牌

在 Serve 模式下，如果 `gateway.auth.allowTailscale` 设为 `true`，Tailscale 的身份头就可以满足控制界面和 WebSocket 的认证要求——你不需要额外的令牌或密码。

但要注意：

- HTTP API 端点仍然需要令牌或密码
- 如果你想强制要求显式凭据，可以设 `gateway.auth.allowTailscale: false`

详见 [Tailscale](./gateway/tailscale.md) 和 [安全](./gateway/security.md)。

**前提**：这种"无令牌模式"假设你信任运行网关的那台主机。

### 8. Funnel 模式强制密码认证

`gateway.tailscale.mode: "funnel"` 要求 `gateway.auth.mode: "password"`（共享密码）。

---

## 构建 UI

网关从 `dist/control-ui` 目录读取静态文件来提供控制界面。

如果你是从源码构建，需要先编译前端资源：

```bash
pnpm ui:build # 首次运行时会自动安装 UI 依赖
```

构建完成后，控制界面就会随网关一起启动。

[贡献者威胁模型](./security/CONTRIBUTING-THREAT-MODEL.md)[控制界面](./web/control-ui.md)