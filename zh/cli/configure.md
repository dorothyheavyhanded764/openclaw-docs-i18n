

  CLI 命令

  
# configure

通过交互式向导快速配置凭证、设备和智能体（agent）默认值。

`configure` 命令提供了一个友好的交互式界面，帮你完成以下设置：

- 配置 API 凭证和认证信息
- 管理设备绑定
- 设置智能体（agent）的默认参数
- 指定模型白名单（决定 `/model` 命令和模型选择器中显示哪些模型）

:::tip 小技巧
不带子命令直接运行 `openclaw config` 也会打开同样的配置向导。如需非交互式编辑，可使用 `openclaw config get|set|unset` 命令。
:::

**相关链接：**

- 网关配置参考：[Configuration](../gateway/configuration.md)
- 配置 CLI：[Config](./config.md)

## 使用须知

- 选择网关运行位置时会自动更新 `gateway.mode`。如果你只想修改这一项，可以在其他配置步骤选择"跳过"。
- 针对频道类服务（Slack/Discord/Matrix/Microsoft Teams），配置向导会提示你设置频道/房间白名单。你可以输入名称或 ID，向导会自动将名称解析为对应的 ID。
- 运行守护进程安装时，如果启用了令牌认证，需要提供有效令牌。`gateway.auth.token` 由 SecretRef 管理，`configure` 会验证 SecretRef 的有效性，但不会将解析后的明文令牌写入 supervisord 的服务环境配置中。
- 如果令牌认证所需的 SecretRef 无法解析，`configure` 会阻止守护进程安装，并给出具体的修复建议。
- 如果同时配置了 `gateway.auth.token` 和 `gateway.auth.password`，但未设置 `gateway.auth.mode`，`configure` 会暂停守护进程安装，直到你明确指定认证模式。

## 示例

```bash
# 启动交互式配置向导
openclaw configure

# 只配置模型和频道部分
openclaw configure --section model --section channels
```

[config](./config.md)[cron](./cron.md)