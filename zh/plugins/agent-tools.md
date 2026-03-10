

  扩展

  
# 插件代理工具

OpenClaw 插件（plugin）可以注册**代理工具**（JSON-schema 函数），这些工具在智能体（agent）运行时会暴露给 LLM。工具分为两类：**必需工具**始终可用，**可选工具**需要手动启用。代理工具在主配置的 `tools` 下配置，也可以在每个智能体的 `agents.list[].tools` 下单独配置。允许列表/拒绝列表策略控制智能体可以调用哪些工具。

## 基础工具

```typescript
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```

## 可选工具（手动启用）

可选工具**永远不会**自动启用。用户必须将它们添加到智能体的允许列表中。

```bash
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a local workflow",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

在 `agents.list[].tools.allow`（或全局 `tools.allow`）中启用可选工具：

```json
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool", // 具体工具名称
            "workflow", // 插件 ID（启用该插件的所有工具）
            "group:plugins", // 所有插件工具
          ],
        },
      },
    ],
  },
}
```

其他影响工具可用性的配置选项：

- 如果允许列表仅包含插件工具名称，则视为插件选择启用；核心工具仍保持启用，除非你在允许列表中也包含核心工具或工具组
- `tools.profile` / `agents.list[].tools.profile`（基础允许列表）
- `tools.byProvider` / `agents.list[].tools.byProvider`（按提供商的允许/拒绝）
- `tools.sandbox.tools.*`（沙盒环境下的工具策略）

## 规则与提示

- 工具名称**不得**与核心工具名称冲突，冲突的工具会被跳过
- 在允许列表中使用的插件 ID 不得与核心工具名称冲突
- 对于会触发副作用或需要额外二进制文件/凭据的工具，建议使用 `optional: true`

[插件清单](./manifest.md)[OpenProse](../prose.md)