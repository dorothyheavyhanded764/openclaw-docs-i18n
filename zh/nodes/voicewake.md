

  媒体与设备

  
# 语音唤醒

OpenClaw 将**唤醒词作为单一全局列表**进行管理，由 **Gateway** 统一控制：

- **没有按节点单独设置的唤醒词**——所有设备共享同一份列表
- **任意节点或应用界面都可以编辑**该列表；修改由 Gateway 持久化并广播给所有客户端
- macOS 和 iOS 保留本地的**语音唤醒开关**（本地交互和权限机制有所不同）
- Android 目前禁用语音唤醒，使用语音标签页中的手动麦克风流程

## 存储（Gateway 主机）

唤醒词存储在 Gateway 主机的以下位置：

```
~/.openclaw/settings/voicewake.json
```

文件结构：

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```

## 协议

### 方法

- `voicewake.get` → 返回 `{ triggers: string[] }`
- `voicewake.set` 参数 `{ triggers: string[] }` → 返回 `{ triggers: string[] }`

注意事项：

- 触发词会被规范化（去除首尾空格、过滤空值）
- 空列表会回退到默认值
- 出于安全考虑，有数量和长度上限

### 事件

- `voicewake.changed` 负载 `{ triggers: string[] }`

接收者：

- 所有 WebSocket 客户端（macOS 应用、WebChat 等）
- 所有已连接的节点（iOS/Android），节点连接时也会收到初始"当前状态"推送

## 客户端行为

### macOS 应用

- 使用全局列表控制 `VoiceWakeRuntime` 触发检测
- 在语音唤醒设置中编辑"触发词"时调用 `voicewake.set`，依赖广播事件保持其他客户端同步

### iOS 节点

- 使用全局列表进行 `VoiceWakeManager` 触发检测
- 在设置中编辑唤醒词时调用 `voicewake.set`（通过 Gateway WebSocket），同时保持本地唤醒词检测的响应性

### Android 节点

- 目前 Android 运行时和设置中禁用了语音唤醒
- Android 语音功能使用语音标签页中的手动麦克风捕获，而非唤醒词触发

[对话模式](./talk.md)[位置命令](./location-command.md)