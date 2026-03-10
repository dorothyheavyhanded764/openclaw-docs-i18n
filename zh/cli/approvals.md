

  CLI 命令

  
# approvals

这条命令帮你管理**本地主机**、**网关主机**或**节点主机**上的执行审批（approval）。默认操作的是本地磁盘上的审批文件；加 `--gateway` 可以操作网关，加 `--node` 则操作指定节点。

相关主题：

-   执行审批详解：[执行审批](../tools/exec-approvals.md)
-   节点说明：[节点](../nodes.md)

## 常用命令

```bash
openclaw approvals get
openclaw approvals get --node <id|name|ip>
openclaw approvals get --gateway
```

## 从文件导入审批规则

```bash
openclaw approvals set --file ./exec-approvals.json
openclaw approvals set --node <id|name|ip> --file ./exec-approvals.json
openclaw approvals set --gateway --file ./exec-approvals.json
```

## 允许列表快捷操作

```bash
openclaw approvals allowlist add "~/Projects/**/bin/rg"
openclaw approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
openclaw approvals allowlist add --agent "*" "/usr/bin/uname"

openclaw approvals allowlist remove "~/Projects/**/bin/rg"
```

## 注意事项

-   `--node` 的节点标识支持 id、名称、ip 或 id 前缀，与 `openclaw nodes` 命令使用相同的解析逻辑。
-   `--agent` 默认为 `"*"`，表示规则对所有智能体（agent）生效。
-   目标节点主机必须支持 `system.execApprovals.get/set` 能力（如 macOS 应用或无头节点主机）。
-   审批文件按主机分别存储，位置为 `~/.openclaw/exec-approvals.json`。

[agents](./agents.md)[browser](./browser.md)