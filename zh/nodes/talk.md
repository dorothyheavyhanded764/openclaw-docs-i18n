

  媒体与设备

  
# 对话模式

对话模式让你可以与智能体（agent）进行连续的语音对话，形成一个完整的交互循环：

1. 监听你的语音
2. 将转录文本发送给模型（主会话，`chat.send`）
3. 等待模型响应
4. 通过 ElevenLabs 朗读回复（流式播放）

## 行为特性 (macOS)

- 启用对话模式时，**始终显示悬浮覆盖层**
- 清晰展示 **监听 → 思考 → 说话** 三个阶段
- 检测到**短暂停顿**（静音窗口）后，发送当前转录文本
- 回复会**写入 WebChat**（与打字输入效果相同）
- **语音打断**（默认开启）：助手正在说话时，如果你开始讲话，系统会立即停止播放并记录打断时间戳，用于下一次提示

## 回复中的语音指令

助手可以在回复开头添加**单行 JSON** 来动态控制语音：

```json
{ "voice": "<voice-id>", "once": true }
```

规则：

- 仅第一个非空行有效
- 未知键会被忽略
- `once: true` 仅对当前回复生效
- 不带 `once` 时，该语音会成为对话模式的新默认语音
- JSON 行会在 TTS 播放前被移除

支持的键：

- `voice` / `voice_id` / `voiceId`
- `model` / `model_id` / `modelId`
- `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
- `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
- `once`

## 配置 (~/.openclaw/openclaw.json)

```json
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

默认值：

- `interruptOnSpeech`: `true`
- `voiceId`: 回退到 `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID`（或 API 密钥可用时使用第一个 ElevenLabs 语音）
- `modelId`: 未设置时默认为 `eleven_v3`
- `apiKey`: 回退到 `ELEVENLABS_API_KEY`（或网关 shell 配置，如果可用）
- `outputFormat`: macOS/iOS 默认为 `pcm_44100`，Android 默认为 `pcm_24000`（设置为 `mp3_*` 可强制使用 MP3 流）

## macOS 用户界面

- 菜单栏切换：**Talk**
- 配置标签页：**Talk Mode** 组（语音 ID + 打断开关）
- 悬浮覆盖层：
  - **监听中**：云朵随麦克风电平脉动
  - **思考中**：下沉动画
  - **说话中**：辐射状圆环动画
  - 点击云朵：停止说话
  - 点击 X：退出对话模式

## 注意事项

- 需要语音识别和麦克风权限
- 使用 `chat.send` 发送到会话键 `main`
- TTS 使用 ElevenLabs 流式 API 和 `ELEVENLABS_API_KEY`，在 macOS/iOS/Android 上采用增量播放实现低延迟
- `eleven_v3` 的 `stability` 只能是 `0.0`、`0.5` 或 `1.0`；其他模型接受 `0..1` 范围
- `latency_tier` 设置时验证为 `0..4`
- Android 支持 `pcm_16000`、`pcm_22050`、`pcm_24000` 和 `pcm_44100` 输出格式，用于低延迟 AudioTrack 流式播放

[摄像头捕获](./camera.md)[语音唤醒](./voicewake.md)