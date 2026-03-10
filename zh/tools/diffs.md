

  内置工具

  
# 差异/补丁（diffs）

`diffs` 是一个可选的插件工具，内置了简洁的系统指导，并配有配套技能，可以将变更内容转换为只读的差异工件供智能体（agent）使用。

**它能帮你做什么？** 当智能体需要向用户展示文件修改、代码变更或文本差异时，这个工具可以将变更可视化，生成可供查看器展示或以文件形式发送的输出。

它接受两种输入方式：

- `before` 和 `after` 文本——分别提供修改前后的完整内容
- 统一的 `patch`——直接提供差异补丁格式

它可以返回：

- 网关查看器 URL，用于在画布中展示
- 渲染后的文件路径（PNG 或 PDF），用于消息附件发送
- 同时输出以上两种

启用后，插件会自动在系统提示空间中加入简洁的使用指导，同时提供一个详细技能供智能体在需要完整指令时调用。

## 快速开始

1. 启用插件。
2. 需要在画布中展示差异？使用 `mode: "view"` 调用 `diffs`。
3. 需要通过聊天渠道发送文件附件？使用 `mode: "file"` 调用 `diffs`。
4. 两种输出都需要？使用 `mode: "both"` 调用 `diffs`。

## 启用插件

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
      },
    },
  },
}
```

## 禁用内置系统指导

如果你想保留 `diffs` 工具但不需要它自带的系统提示指导，可以将 `plugins.entries.diffs.hooks.allowPromptInjection` 设为 `false`：

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        hooks: {
          allowPromptInjection: false,
        },
      },
    },
  },
}
```

这会阻止 diffs 插件的 `before_prompt_build` 钩子，但插件本身、工具和配套技能仍然可用。如果你想同时禁用指导和工具，直接禁用整个插件即可。

## 典型的智能体工作流

1. 智能体调用 `diffs` 工具。
2. 智能体读取返回结果中的 `details` 字段。
3. 智能体根据需要选择：
   - 用 `canvas present` 打开 `details.viewerUrl` 在画布中展示
   - 通过 `message` 工具发送 `details.filePath`（使用 `path` 或 `filePath` 参数）
   - 两种方式都用

## 输入示例

使用 before/after 模式，适合直接提供修改前后的完整内容：

```json
{
  "before": "# Hello\n\nOne",
  "after": "# Hello\n\nTwo",
  "path": "docs/example.md",
  "mode": "view"
}
```

使用 patch 模式，适合已有统一差异格式的情况：

```json
{
  "patch": "diff --git a/src/example.ts b/src/example.ts\n--- a/src/example.ts\n+++ b/src/example.ts\n@@ -1 +1 @@\n-const x = 1;\n+const x = 2;\n",
  "mode": "both"
}
```

## 工具输入参考

以下字段均为可选，除非特别说明：

- `before` (`string`)：原始文本。当没有提供 `patch` 时，必须与 `after` 一起提供。
- `after` (`string`)：修改后的文本。当没有提供 `patch` 时，必须与 `before` 一起提供。
- `patch` (`string`)：统一差异格式的文本。与 `before`/`after` 互斥，不能同时使用。
- `path` (`string`)：在 before/after 模式下显示的文件名。
- `lang` (`string`)：在 before/after 模式下的语法高亮语言提示。
- `title` (`string`)：查看器标题，覆盖默认值。
- `mode` (`"view" | "file" | "both"`)：输出模式。默认使用插件配置中的 `defaults.mode`。
- `theme` (`"light" | "dark"`)：查看器主题。默认使用插件配置中的 `defaults.theme`。
- `layout` (`"unified" | "split"`)：差异布局方式。默认使用插件配置中的 `defaults.layout`。
- `expandUnchanged` (`boolean`)：当有完整上下文时，展开未更改的部分。此参数仅支持单次调用设置，不是插件默认配置项。
- `fileFormat` (`"png" | "pdf"`)：渲染文件的格式。默认使用插件配置中的 `defaults.fileFormat`。
- `fileQuality` (`"standard" | "hq" | "print"`)：PNG 或 PDF 渲染的质量预设。
- `fileScale` (`number`)：设备缩放比例覆盖，范围 `1`-`4`。
- `fileMaxWidth` (`number`)：最大渲染宽度（CSS 像素），范围 `640`-`2400`。
- `ttlSeconds` (`number`)：查看器工件的存活时间（秒）。默认 1800 秒，最大 21600 秒。
- `baseUrl` (`string`)：查看器 URL 的来源地址覆盖。必须是 `http` 或 `https` 协议，不能包含查询参数或哈希。

### 验证和限制

- `before` 和 `after` 各自最大 512 KiB。
- `patch` 最大 2 MiB。
- `path` 最大 2048 字节。
- `lang` 最大 128 字节。
- `title` 最大 1024 字节。
- 补丁复杂度上限：最多 128 个文件，总计不超过 120000 行。
- `patch` 与 `before`/`after` 不能同时提供，否则会被拒绝。
- 渲染文件的安全限制（适用于 PNG 和 PDF）：
  - `fileQuality: "standard"`：最大 8 MP（800 万渲染像素）。
  - `fileQuality: "hq"`：最大 14 MP（1400 万渲染像素）。
  - `fileQuality: "print"`：最大 24 MP（2400 万渲染像素）。
  - PDF 另有最多 50 页的限制。

## 输出详情契约

工具在 `details` 字段下返回结构化的元数据。

**创建查看器时返回的公共字段：**

- `artifactId`：工件唯一标识
- `viewerUrl`：查看器访问地址
- `viewerPath`：查看器路径
- `title`：标题
- `expiresAt`：过期时间
- `inputKind`：输入类型
- `fileCount`：文件数量
- `mode`：输出模式

**渲染 PNG 或 PDF 时返回的文件字段：**

- `filePath`：渲染后的文件路径
- `path`：与 `filePath` 值相同，用于与消息工具兼容
- `fileBytes`：文件字节大小
- `fileFormat`：文件格式
- `fileQuality`：渲染质量
- `fileScale`：缩放比例
- `fileMaxWidth`：最大宽度

### 模式行为一览

- `mode: "view"`：仅返回查看器相关字段。
- `mode: "file"`：仅返回文件字段，不创建查看器工件。
- `mode: "both"`：同时返回查看器和文件字段。如果文件渲染失败，查看器仍会正常返回，并附带 `fileError` 错误信息。

## 折叠的未更改部分

查看器可能会显示类似「N 行未修改」的折叠行。关于这些行的展开控件：

- 展开控件是否出现取决于输入类型，并非所有情况都能保证。
- 当渲染的差异包含可展开的上下文数据时，会出现展开控件——这对于 before/after 输入是典型情况。
- 对于许多统一补丁格式的输入，由于解析后的补丁块中不包含被省略的上下文内容，这些折叠行可能没有展开控件。这是预期行为，并非查看器故障。
- `expandUnchanged` 参数仅在实际存在可展开上下文时才会生效。

## 插件默认配置

你可以在 `~/.openclaw/openclaw.json` 中设置全局默认值：

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          defaults: {
            fontFamily: "Fira Code",
            fontSize: 15,
            lineSpacing: 1.6,
            layout: "unified",
            showLineNumbers: true,
            diffIndicators: "bars",
            wordWrap: true,
            background: true,
            theme: "dark",
            fileFormat: "png",
            fileQuality: "standard",
            fileScale: 2,
            fileMaxWidth: 960,
            mode: "both",
          },
        },
      },
    },
  },
}
```

**支持的默认配置项：**

- `fontFamily`：字体
- `fontSize`：字号
- `lineSpacing`：行间距
- `layout`：布局方式
- `showLineNumbers`：是否显示行号
- `diffIndicators`：差异指示器样式
- `wordWrap`：是否自动换行
- `background`：是否显示背景
- `theme`：主题
- `fileFormat`：文件格式
- `fileQuality`：渲染质量
- `fileScale`：缩放比例
- `fileMaxWidth`：最大宽度
- `mode`：输出模式

调用工具时显式指定的参数会覆盖这些默认值。

## 安全配置

- `security.allowRemoteViewer` (`boolean`，默认 `false`)
  - `false`：拒绝来自非本机的查看器路由请求。
  - `true`：如果令牌化路径验证通过，则允许远程访问查看器。

配置示例：

```json
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          security: {
            allowRemoteViewer: false,
          },
        },
      },
    },
  },
}
```

## 工件生命周期和存储

- 工件存储在临时目录的子文件夹中：`$TMPDIR/openclaw-diffs`。
- 查看器工件的元数据包含：
  - 随机生成的工件 ID（20 位十六进制字符）
  - 随机生成的令牌（48 位十六进制字符）
  - `createdAt` 创建时间和 `expiresAt` 过期时间
  - 存储的 `viewer.html` 文件路径
- 未指定时，默认查看器 TTL 为 30 分钟。
- 最大可接受的查看器 TTL 为 6 小时。
- 清理程序会在创建工件后择机运行。
- 过期的工件会被自动删除。
- 如果元数据丢失，后备清理程序会删除超过 24 小时的陈旧文件夹。

## 查看器 URL 和网络行为

**查看器路由：**

- `/plugins/diffs/view/{artifactId}/{token}`

**查看器资源：**

- `/plugins/diffs/assets/viewer.js`
- `/plugins/diffs/assets/viewer-runtime.js`

**URL 构建规则：**

- 如果提供了 `baseUrl`，会在严格验证后使用它。
- 如果没有提供 `baseUrl`，查看器 URL 默认使用本机环回地址 `127.0.0.1`。
- 如果网关绑定模式为 `custom` 且设置了 `gateway.customBindHost`，则使用该主机地址。

**`baseUrl` 的规则：**

- 必须使用 `http://` 或 `https://` 协议。
- 不允许包含查询参数或哈希片段。
- 可以是纯来源地址，也可以带有基础路径。

## 安全模型

**查看器安全加固：**

- 默认仅限本机访问。
- 使用令牌化的查看器路径，配合严格的 ID 和令牌验证。
- 查看器响应使用 CSP 策略：
  - `default-src 'none'`
  - 脚本和资源仅允许来自自身
  - 禁止出站连接 `connect-src`
- 启用远程访问时的失败限流：
  - 60 秒内最多允许 40 次失败
  - 超限后锁定 60 秒（返回 `429 Too Many Requests`）

**文件渲染安全加固：**

- 截图浏览器的请求路由默认采用拒绝策略。
- 仅允许访问 `http://127.0.0.1/plugins/diffs/assets/*` 的本地查看器资源。
- 阻止所有外部网络请求。

## 文件模式的浏览器要求

使用 `mode: "file"` 或 `mode: "both"` 需要安装 Chromium 兼容的浏览器。查找顺序：

1. OpenClaw 配置中的 `browser.executablePath`。
2. 环境变量：
   - `OPENCLAW_BROWSER_EXECUTABLE_PATH`
   - `BROWSER_EXECUTABLE_PATH`
   - `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH`
3. 平台自动发现已安装的浏览器路径。

**常见的错误提示：**

- `Diff PNG/PDF 渲染需要兼容 Chromium 的浏览器...`

**解决方法：** 安装 Chrome、Chromium、Edge 或 Brave 浏览器，或设置上述可执行路径环境变量之一。

## 故障排除

### 输入验证错误

- `Provide patch or both before and after text.`
  - 需要同时提供 `before` 和 `after`，或者提供 `patch`。
- `Provide either patch or before/after input, not both.`
  - 不要混用两种输入模式。
- `Invalid baseUrl: ...`
  - 使用 `http(s)` 协议的来源地址，可带路径，但不能有查询参数或哈希。
- `{field} exceeds maximum size (...)`
  - 减小输入数据的大小。
- 大型补丁被拒绝
  - 减少补丁中的文件数量或总行数。

### 查看器无法访问

- 查看器 URL 默认解析到 `127.0.0.1`（本机）。
- 如需远程访问，可以：
  - 每次调用时传递 `baseUrl` 参数，或
  - 使用 `gateway.bind=custom` 并设置 `gateway.customBindHost`
- 只有确实需要外部访问查看器时，才启用 `security.allowRemoteViewer`。

### 未修改行没有展开按钮

- 当 patch 输入本身不包含可展开的上下文时，这种情况是正常的。
- 这不代表查看器有问题。

### 找不到工件

- 工件可能已超过 TTL 过期。
- 令牌或路径可能已被修改。
- 清理程序可能已删除陈旧数据。

## 操作建议

- 在画布中进行本地交互式审查时，优先使用 `mode: "view"`。
- 需要通过聊天渠道发送文件附件时，优先使用 `mode: "file"`。
- 除非你的部署确实需要远程访问查看器，否则保持 `allowRemoteViewer` 禁用。
- 对于敏感内容的差异，设置较短的 `ttlSeconds`。
- 避免在差异输入中包含敏感信息（如密钥、凭证等）。
- 如果聊天渠道对图片压缩较激进（如 Telegram 或 WhatsApp），建议使用 PDF 输出（`fileFormat: "pdf"`）。

## 相关文档

- [工具概述](../tools.md)
- [插件](./plugin.md)
- [浏览器](./browser.md)