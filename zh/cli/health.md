

  CLI 命令

  
# health

从运行中的网关获取健康状态。

```bash
openclaw health
openclaw health --json
openclaw health --verbose
```

注意事项：

-   `--verbose` 会运行实时探测，当配置了多个账户时会打印各账户的耗时。
-   当配置了多个智能体时，输出包含各智能体的会话存储信息。

[gateway](./gateway.md)[hooks](./hooks.md)