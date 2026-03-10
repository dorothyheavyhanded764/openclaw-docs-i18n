

  提供商

  
# Qwen

Qwen 为 Qwen Coder 和 Qwen Vision 模型提供免费层级的 OAuth 认证流程，每日可发送 2,000 次请求（具体限制以 Qwen 官方为准）。

## 启用插件

```bash
openclaw plugins enable qwen-portal-auth
```

启用后请重启 Gateway。

## 认证

```bash
openclaw models auth login --provider qwen-portal --set-default
```

这条命令会启动 Qwen 的设备码 OAuth 认证流程，将提供商配置写入 `models.json`，同时创建一个 `qwen` 别名方便后续快速切换。

## 模型 ID

- `qwen-portal/coder-model`
- `qwen-portal/vision-model`

切换模型：

```bash
openclaw models set qwen-portal/coder-model
```

## 复用 Qwen Code CLI 的登录状态

如果你已经通过 Qwen Code CLI 登录过，OpenClaw 在加载认证存储时会自动从 `~/.qwen/oauth_creds.json` 同步凭据。不过你仍需要在 `models.json` 中有 `models.providers.qwen-portal` 配置项——运行上面的登录命令即可自动创建。

## 注意事项

- 令牌会自动刷新。如果刷新失败或访问权限被撤销，重新运行登录命令即可。
- 默认的基础 URL 是 `https://portal.qwen.ai/v1`。如果 Qwen 提供了不同的端点，可以通过 `models.providers.qwen-portal.baseUrl` 配置项覆盖。
- 更多提供商相关的通用规则，请参阅[模型提供商](../concepts/model-providers.md)。

[千帆](./qianfan.md)[合成数据](./synthetic.md)