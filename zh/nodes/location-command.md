

  媒体与设备

  
# 位置命令

本节介绍如何通过节点（node）获取设备位置信息，包括权限配置、命令调用方式以及后台定位的实现思路。

## 快速概览

- `location.get` 是一个节点命令（通过 `node.invoke` 调用）
- 默认关闭
- 设置提供三档选择：关闭 / 使用时 / 始终
- 独立开关：精确位置

## 为什么用选择器而非简单开关？

操作系统权限是多层次的。我们可以在应用内提供选择器，但实际授权仍由操作系统决定：

- **iOS/macOS**：用户可以在系统提示或设置中选择"使用时"或"始终"。应用可以请求升级权限，但系统可能要求用户手动前往设置页面
- **Android**：后台位置是单独的权限；在 Android 10+ 上，通常需要引导用户进入设置流程
- 精确位置是单独的授权项（iOS 14+ 称为"精确位置"，Android 区分"精确"和"粗略"）

简单来说：UI 中的选择器控制我们请求的权限级别，实际授权状态则由操作系统设置决定。

## 设置模型

每个节点设备的配置项：

- `location.enabledMode`: `off | whileUsing | always`
- `location.preciseEnabled`: 布尔值

UI 行为：

- 选择 `whileUsing` → 请求前台权限
- 选择 `always` → 先确保 `whileUsing` 权限，再请求后台权限（必要时引导用户前往设置）
- 如果操作系统拒绝了请求级别，则回退到已授予的最高级别并显示状态

## 权限映射 (node.permissions)

此项可选。macOS 节点通过权限映射报告 `location` 状态；iOS/Android 可能不报告此项。

## 命令：location.get

通过 `node.invoke` 调用。建议参数：

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

响应负载：

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

错误码（稳定值）：

- `LOCATION_DISABLED`: 选择器已关闭
- `LOCATION_PERMISSION_REQUIRED`: 缺少请求模式所需的权限
- `LOCATION_BACKGROUND_UNAVAILABLE`: 应用在后台但仅允许"使用时"模式
- `LOCATION_TIMEOUT`: 超时未能获取定位
- `LOCATION_UNAVAILABLE`: 系统故障 / 无可用定位源

## 后台行为（规划中）

目标是让智能体（agent）即使在节点处于后台时也能请求位置，但需满足以下条件：

- 用户选择了"始终"
- 操作系统授予后台位置权限
- 应用被允许在后台运行以获取位置（iOS 后台模式 / Android 前台服务或特殊许可）

推送触发流程（规划中）：

1. Gateway 向节点发送推送（iOS 静默推送或 Android FCM 数据消息）
2. 节点短暂唤醒并向设备请求位置
3. 节点将负载转发给 Gateway

注意事项：

- **iOS**: 需要"始终"权限 + 后台位置模式。静默推送可能被系统限流，预计会出现间歇性失败
- **Android**: 后台定位可能需要前台服务支持，否则可能被拒绝

## 模型/工具集成

- 工具接口：`nodes` 工具提供 `location_get` 操作（需指定节点）
- CLI: `openclaw nodes location get --node `
- 智能体（agent）指南：仅在用户已启用位置功能并理解其范围时调用

## UX 文案建议

- 关闭: "位置共享已禁用。"
- 使用时: "仅在 OpenClaw 打开时可用。"
- 始终: "允许后台位置。需要系统权限。"
- 精确位置: "使用精确 GPS 定位。关闭后共享大致位置。"

[语音唤醒](./voicewake.md)[文本转语音](../tts.md)