

  技能

  
# 技能配置

所有技能相关的配置都集中在 `~/.openclaw/openclaw.json` 文件的 `skills` 字段下。你可以在这里控制哪些技能可用、从哪里加载自定义技能、以及为每个技能单独设置环境变量和密钥。

```json
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun（Gateway 运行时仍为 Node，不建议用 bun）
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // 或直接写明文字符串
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## 字段说明

### 全局配置字段

- `allowBundled`：可选，用于限制**内置技能**的白名单。设置后，只有列表中的内置技能会被加载（不影响托管技能和工作区技能）。
- `load.extraDirs`：额外的技能目录路径，OpenClaw 会扫描这些目录（优先级最低）。
- `load.watch`：是否监听技能文件夹变化并自动刷新（默认：`true`）。
- `load.watchDebounceMs`：监听事件的防抖间隔，单位毫秒（默认：`250`）。
- `install.preferBrew`：安装依赖时优先使用 Homebrew（默认：`true`）。
- `install.nodeManager`：Node 包管理器选择（`npm` | `pnpm` | `yarn` | `bun`，默认：`npm`）。注意这只影响**技能安装**，Gateway 运行时仍需 Node（Bun 对 WhatsApp/Telegram 支持不佳，不推荐）。
- `entries.`：针对单个技能的覆盖配置。

### 单技能配置字段

在 `entries` 下，你可以为每个技能单独设置：

- `enabled`：设为 `false` 可禁用某个技能，即使它已内置或安装。
- `env`：为智能体运行时注入环境变量（仅当该变量尚未设置时生效）。
- `apiKey`：便捷字段，用于需要 API 密钥的技能。支持明文字符串或 SecretRef 对象（格式：`{ source, provider, id }`）。

## 注意事项

- `entries` 下的键名默认对应技能名称。如果技能在 `metadata.openclaw.skillKey` 中定义了自定义键名，则使用该键名。
- 开启监听后，技能文件的修改会在智能体的下一轮对话中自动生效。

### 沙盒模式下的环境变量

当会话运行在**沙盒模式**时，技能进程会在 Docker 容器内执行。沙盒环境**不会**继承主机的 `process.env`，你需要通过以下方式传递环境变量：

- 在 `agents.defaults.sandbox.docker.env` 中设置（或针对单个智能体设置 `agents.list[].sandbox.docker.env`）
- 将环境变量直接打包到自定义沙盒镜像中

注意：全局 `env` 和 `skills.entries..env/apiKey` 仅对**主机模式**运行有效。

[技能](./skills.md)[ClawHub](./clawhub.md)