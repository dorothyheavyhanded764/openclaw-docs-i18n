

  CLI 命令

  
# completion

生成 shell 补全脚本，并可选择将其安装到你的 shell 配置文件中。

## 用法

```bash
openclaw completion
openclaw completion --shell zsh
openclaw completion --install
openclaw completion --shell fish --install
openclaw completion --write-state
openclaw completion --shell bash --write-state
```

## 选项

-   `-s, --shell `：目标 shell（`zsh`、`bash`、`powershell`、`fish`；默认：`zsh`）
-   `-i, --install`：通过在 shell 配置文件中添加 source 行来安装补全
-   `--write-state`：将补全脚本写入 `$OPENCLAW_STATE_DIR/completions`，而不输出到 stdout
-   `-y, --yes`：跳过安装确认提示

## 注意事项

-   `--install` 会在你的 shell 配置文件中写入一个小的"OpenClaw Completion"块，并指向缓存的脚本。
-   如果不使用 `--install` 或 `--write-state`，命令会将脚本打印到 stdout。
-   补全生成时会预先加载命令树，因此嵌套的子命令也会被包含在内。

[clawbot](./clawbot.md)[config](./config.md)