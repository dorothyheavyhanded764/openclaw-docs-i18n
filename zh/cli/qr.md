

  CLI 命令

  
# qr

根据当前网关配置生成 iOS 配对二维码（QR code）和设置码，方便您在手机上快速完成设备绑定。

## 用法

```bash
openclaw qr
openclaw qr --setup-code-only
openclaw qr --json
openclaw qr --remote
openclaw qr --url wss://gateway.example/ws --token '<token>'
```

## 选项

-   `--remote`: 使用配置中的 `gateway.remote.url` 及远程令牌或密码
-   `--url `: 指定 payload 中使用的网关 URL
-   `--public-url `: 指定 payload 中使用的公共 URL
-   `--token `: 指定 payload 中使用的网关令牌
-   `--password `: 指定 payload 中使用的网关密码
-   `--setup-code-only`: 仅输出设置码
-   `--no-ascii`: 跳过 ASCII 二维码渲染
-   `--json`: 以 JSON 格式输出（包含 `setupCode`、`gatewayUrl`、`auth`、`urlSource`）

## 注意事项

-   `--token` 与 `--password` 不能同时使用。
-   使用 `--remote` 时，如果远程凭证以 SecretRef 形式配置且未通过 `--token` 或 `--password` 显式指定，命令会从当前网关快照中解析凭证。若网关不可用，命令将立即报错退出。
-   不使用 `--remote` 时，本地网关的认证 SecretRef 会按以下规则解析：
    -   当令牌认证优先时（显式设置 `gateway.auth.mode="token"`，或推断模式下无密码来源胜出），解析 `gateway.auth.token`。
    -   当密码认证优先时（显式设置 `gateway.auth.mode="password"`，或推断模式下无令牌胜出），解析 `gateway.auth.password`。
-   若同时配置了 `gateway.auth.token` 和 `gateway.auth.password`（包括 SecretRef）但未设置 `gateway.auth.mode`，设置码解析将失败，需要显式指定认证模式。
-   网关版本兼容性：此命令要求网关支持 `secrets.resolve` 方法，旧版网关会返回未知方法错误。
-   扫描二维码后，需执行以下命令完成设备配对审批：
    -   `openclaw devices list`
    -   `openclaw devices approve `

[插件](./plugins.md)[重置](./reset.md)