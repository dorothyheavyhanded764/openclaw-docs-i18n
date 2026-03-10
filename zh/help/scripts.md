

  环境与调试

  
# 脚本

`scripts/` 目录包含用于本地工作流和运维任务的辅助脚本。当某个任务明确需要使用脚本时才使用它们；否则请优先使用 CLI。

## 规范

- 脚本是**可选的**，除非在文档或发布检查清单中被明确引用。
- 如果存在对应的 CLI 接口，请优先使用 CLI（例如：认证监控使用 `openclaw models status --check`）。
- 请假设脚本是主机特定的；在新机器上运行前请先阅读脚本内容。

## 认证监控脚本

认证监控脚本的文档位于：[/automation/auth-monitoring](../automation/auth-monitoring.md)

## 添加新脚本时

- 保持脚本功能单一且有清晰的文档说明。
- 在相关文档中添加简短条目（如果没有相关文档，请创建一份）。

[测试](./testing.md)[Node + tsx 崩溃](../debug/node-issue.md)