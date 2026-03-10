

  媒体与设备

  
# 节点故障排除

当节点在状态中可见但节点工具执行失败时，请参考本页面进行排查。

## 诊断命令阶梯

按顺序执行以下命令：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

然后运行节点相关检查：

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

健康状态指标：

- 节点已连接并完成 `node` 角色的配对
- `nodes describe` 输出包含你正在调用的能力
- 执行审批显示预期的模式和允许列表

## 前台要求

`canvas.*`、`camera.*` 和 `screen.*` 在 iOS/Android 节点上仅支持前台操作。快速排查步骤：

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

如果看到 `NODE_BACKGROUND_UNAVAILABLE`，请将节点应用切换到前台后重试。

## 权限矩阵

| 能力 | iOS | Android | macOS 节点应用 | 典型错误码 |
| --- | --- | --- | --- | --- |
| `camera.snap`, `camera.clip` | 摄像头（视频需麦克风） | 摄像头（视频需麦克风） | 摄像头（视频需麦克风） | `*_PERMISSION_REQUIRED` |
| `screen.record` | 屏幕录制（麦克风可选） | 屏幕捕获提示（麦克风可选） | 屏幕录制 | `*_PERMISSION_REQUIRED` |
| `location.get` | 使用时或始终（取决于模式） | 根据模式的前台/后台位置 | 位置权限 | `LOCATION_PERMISSION_REQUIRED` |
| `system.run` | 不适用（节点主机路径） | 不适用（节点主机路径） | 需要执行审批 | `SYSTEM_RUN_DENIED` |

## 配对与审批的区别

这是两个独立的检查点：

1. **设备配对**：该节点能否连接到 Gateway？
2. **执行审批**：该节点能否运行特定的 shell 命令？

快速检查命令：

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

如果配对缺失，请先批准节点设备。如果配对正常但 `system.run` 失败，请检查执行审批/允许列表配置。

## 常见节点错误码

| 错误码 | 含义与解决方法 |
| --- | --- |
| `NODE_BACKGROUND_UNAVAILABLE` | 应用在后台运行；切换到前台 |
| `CAMERA_DISABLED` | 节点设置中摄像头开关已禁用 |
| `*_PERMISSION_REQUIRED` | 操作系统权限缺失或被拒绝 |
| `LOCATION_DISABLED` | 位置模式已关闭 |
| `LOCATION_PERMISSION_REQUIRED` | 请求的位置模式未被授予 |
| `LOCATION_BACKGROUND_UNAVAILABLE` | 应用在后台但仅有"使用时"权限 |
| `SYSTEM_RUN_DENIED: approval required` | 执行请求需要明确批准 |
| `SYSTEM_RUN_DENIED: allowlist miss` | 命令被允许列表模式阻止。Windows 节点主机上，`cmd.exe /c ...` 等形式在允许列表模式下会被视为未命中，除非通过询问流程批准 |

## 快速恢复流程

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

如果仍无法解决：

- 重新批准设备配对
- 重新打开节点应用（前台模式）
- 重新授予操作系统权限
- 重建或调整执行审批策略

## 相关文档

- [节点概览](./index.md)
- [摄像头捕获](./camera.md)
- [位置命令](./location-command.md)
- [执行审批](../tools/exec-approvals.md)
- [设备配对](../gateway/pairing.md)

[节点](../nodes.md)[媒体理解](./media-understanding.md)