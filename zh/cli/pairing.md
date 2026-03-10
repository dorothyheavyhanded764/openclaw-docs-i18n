

  CLI 命令

  
# pairing

审批或检查直接消息配对请求（用于支持配对的频道）。相关内容：

-   配对流程：[Pairing](../channels/pairing.md)

## 命令

```bash
openclaw pairing list telegram
openclaw pairing list --channel telegram --account work
openclaw pairing list telegram --json

openclaw pairing approve telegram <code>
openclaw pairing approve --channel telegram --account work <code> --notify
```

## 注意事项

-   频道输入：可以位置传递（`pairing list telegram`）或使用 `--channel `。
-   `pairing list` 支持 `--account ` 用于多账户频道。
-   `pairing approve` 支持 `--account ` 和 `--notify`。
-   如果只配置了一个支持配对的频道，允许使用 `pairing approve `。

[onboard](./onboard.md)[plugins](./plugins.md)