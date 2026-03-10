

  内置工具

  
# apply_patch 工具

当你需要对多个文件进行批量修改时，使用单个 `edit` 调用往往会显得脆弱且容易出错。`apply_patch` 工具正是为解决这类场景而设计的——它使用结构化的补丁格式，让你可以在一次调用中安全地完成多个文件的新增、修改和删除操作。

工具接受一个 `input` 字符串，里面包裹着一个或多个文件操作：

```markdown
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```

## 参数

- `input`（必需）：完整的补丁内容，必须包含 `*** Begin Patch` 和 `*** End Patch` 标记。

## 使用注意

- 补丁中的文件路径支持相对路径（相对于工作区目录）和绝对路径两种形式。
- `tools.exec.applyPatch.workspaceOnly` 默认为 `true`，即仅允许在工作区内操作。除非你有意让 `apply_patch` 在工作区目录外写入或删除文件，否则不建议将其设为 `false`。
- 使用 `*** Move to:` 可以在 `*** Update File:` 块内实现文件重命名。
- `*** End of File` 用于标记仅向文件末尾追加内容的场景。
- 这是一个实验性功能，默认禁用。如需启用，请设置 `tools.exec.applyPatch.enabled`。
- 目前仅支持 OpenAI 模型（包括 OpenAI Codex）。你可以通过 `tools.exec.applyPatch.allowModels` 按模型进行限制。
- 所有相关配置都位于 `tools.exec` 配置项下。

## 示例

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```

[工具](../tools.md)[Brave 搜索](../brave-search.md)