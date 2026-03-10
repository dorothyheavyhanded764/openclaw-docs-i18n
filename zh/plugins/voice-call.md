

  扩展

  
# 语音通话插件

通过插件（plugin）为 OpenClaw 提供语音通话功能。支持外呼通知和基于呼入策略的多轮对话。当前支持的提供商：

- `twilio`（Programmable Voice + Media Streams）
- `telnyx`（Call Control v2）
- `plivo`（Voice API + XML transfer + GetInput speech）
- `mock`（开发模式/无网络）

快速上手流程：

- 安装插件
- 重启网关
- 在 `plugins.entries.voice-call.config` 下配置
- 使用 `openclaw voicecall ...` 或 `voice_call` 工具

## 运行位置（本地 vs 远程）

语音通话插件运行在**网关进程内部**。如果你使用远程网关，需要在**运行网关的机器上**安装并配置插件，然后重启网关以加载它。

## 安装

### 方式 A：从 npm 安装（推荐）

```bash
openclaw plugins install @openclaw/voice-call
```

安装后重启网关。

### 方式 B：从本地文件夹安装（开发模式，无需复制）

```bash
openclaw plugins install ./extensions/voice-call
cd ./extensions/voice-call && pnpm install
```

安装后重启网关。

## 配置

在 `plugins.entries.voice-call.config` 下设置配置：

```json
{
  plugins: {
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio", // 或 "telnyx" | "plivo" | "mock"
          fromNumber: "+15550001234",
          toNumber: "+15550005678",

          twilio: {
            accountSid: "ACxxxxxxxx",
            authToken: "...",
          },

          telnyx: {
            apiKey: "...",
            connectionId: "...",
            // Telnyx webhook 公钥，来自 Telnyx Mission Control Portal
            //（Base64 字符串；也可通过 TELNYX_PUBLIC_KEY 环境变量设置）
            publicKey: "...",
          },

          plivo: {
            authId: "MAxxxxxxxxxxxxxxxxxxxx",
            authToken: "...",
          },

          // Webhook 服务器
          serve: {
            port: 3334,
            path: "/voice/webhook",
          },

          // Webhook 安全性（推荐用于隧道/代理场景）
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
            trustedProxyIPs: ["100.64.0.1"],
          },

          // 公开暴露方式（选一种）
          // publicUrl: "https://example.ngrok.app/voice/webhook",
          // tunnel: { provider: "ngrok" },
          // tailscale: { mode: "funnel", path: "/voice/webhook" }

          outbound: {
            defaultMode: "notify", // notify | conversation
          },

          streaming: {
            enabled: true,
            streamPath: "/voice/stream",
            preStartTimeoutMs: 5000,
            maxPendingConnections: 32,
            maxPendingConnectionsPerIp: 4,
            maxConnections: 128,
          },
        },
      },
    },
  },
}
```

注意事项：

- Twilio 和 Telnyx 都需要**公网可访问的** webhook URL
- Plivo 也需要**公网可访问的** webhook URL
- `mock` 是本地开发提供商（无网络调用）
- Telnyx 需要 `telnyx.publicKey`（或 `TELNYX_PUBLIC_KEY`），除非 `skipSignatureVerification` 为 true
- `skipSignatureVerification` 仅用于本地测试
- 如果你使用 ngrok 免费版，请将 `publicUrl` 设置为准确的 ngrok URL；签名验证始终会执行
- `tunnel.allowNgrokFreeTierLoopbackBypass: true` 仅在 `tunnel.provider="ngrok"` 且 `serve.bind` 为环回地址（ngrok 本地代理）时，才允许带有无效签名的 Twilio webhook。仅用于本地开发
- Ngrok 免费版 URL 可能会变化或出现中间页；如果 `publicUrl` 发生变化，Twilio 签名验证会失败。生产环境建议使用稳定域名或 Tailscale funnel
- 流式传输安全默认值：
  - `streaming.preStartTimeoutMs` 关闭从未发送有效 `start` 帧的套接字
  - `streaming.maxPendingConnections` 限制未经身份验证的预启动套接字总数
  - `streaming.maxPendingConnectionsPerIp` 限制每个源 IP 的未经身份验证的预启动套接字数
  - `streaming.maxConnections` 限制打开的媒体流套接字总数（预启动 + 活动）

## 陈旧通话清理器

使用 `staleCallReaperSeconds` 结束那些从未收到终止 webhook 的通话（例如未完成的通知模式通话）。默认值为 `0`（禁用）。推荐设置：

- **生产环境：** 对于通知类流程，设置为 `120`–`300` 秒
- 保持此值**高于 `maxDurationSeconds`**，让正常通话有时间完成。建议设为 `maxDurationSeconds + 30–60` 秒

示例：

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          maxDurationSeconds: 300,
          staleCallReaperSeconds: 360,
        },
      },
    },
  },
}
```

## Webhook 安全性

当代理或隧道位于网关前面时，插件会重建用于签名验证的公网 URL。以下选项控制信任哪些转发头部：

- `webhookSecurity.allowedHosts`：对转发头部中的主机名进行白名单验证
- `webhookSecurity.trustForwardingHeaders`：无需白名单即可信任转发头部
- `webhookSecurity.trustedProxyIPs`：仅当请求的远程 IP 匹配列表时才信任转发头部

Twilio 和 Plivo 启用了 webhook 重放保护。重放的有效 webhook 请求会被确认但跳过副作用处理。Twilio 对话轮次在 `` 回调中包含每轮令牌，因此陈旧/重放的语音回调无法满足较新的待处理转录轮次。

使用稳定公网主机的示例：

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          publicUrl: "https://voice.example.com/voice/webhook",
          webhookSecurity: {
            allowedHosts: ["voice.example.com"],
          },
        },
      },
    },
  },
}
```

## 通话的 TTS

语音通话使用核心的 `messages.tts` 配置（OpenAI 或 ElevenLabs）进行流式语音合成。你可以在插件配置下使用**相同结构**覆盖它——它会与 `messages.tts` 深度合并。

```json
{
  tts: {
    provider: "elevenlabs",
    elevenlabs: {
      voiceId: "pMsXgVXv3BLzUgSXRplE",
      modelId: "eleven_multilingual_v2",
    },
  },
}
```

注意事项：

- **Edge TTS 在语音通话中被忽略**（电话音频需要 PCM；Edge 输出不可靠）
- 当启用 Twilio 媒体流时使用核心 TTS；否则通话回退到提供商的原生语音

### 更多示例

仅使用核心 TTS（不覆盖）：

```json
{
  messages: {
    tts: {
      provider: "openai",
      openai: { voice: "alloy" },
    },
  },
}
```

仅为通话覆盖为 ElevenLabs（其他地方保持核心默认值）：

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            provider: "elevenlabs",
            elevenlabs: {
              apiKey: "elevenlabs_key",
              voiceId: "pMsXgVXv3BLzUgSXRplE",
              modelId: "eleven_multilingual_v2",
            },
          },
        },
      },
    },
  },
}
```

仅为通话覆盖 OpenAI 模型（深度合并示例）：

```json
{
  plugins: {
    entries: {
      "voice-call": {
        config: {
          tts: {
            openai: {
              model: "gpt-4o-mini-tts",
              voice: "marin",
            },
          },
        },
      },
    },
  },
}
```

## 呼入通话

呼入策略默认为 `disabled`。要启用呼入通话，请设置：

```json
{
  inboundPolicy: "allowlist",
  allowFrom: ["+15550001234"],
  inboundGreeting: "你好！有什么可以帮您？",
}
```

自动响应使用智能体（agent）系统。可通过以下参数调整：

- `responseModel`
- `responseSystemPrompt`
- `responseTimeoutMs`

## CLI

```bash
openclaw voicecall call --to "+15555550123" --message "来自 OpenClaw 的问候"
openclaw voicecall continue --call-id <id> --message "有什么问题吗？"
openclaw voicecall speak --call-id <id> --message "请稍等"
openclaw voicecall end --call-id <id>
openclaw voicecall status --call-id <id>
openclaw voicecall tail
openclaw voicecall expose --mode funnel
```

## 智能体工具

工具名称：`voice_call`

操作：

- `initiate_call`（message, to?, mode?）
- `continue_call`（callId, message）
- `speak_to_user`（callId, message）
- `end_call`（callId）
- `get_status`（callId）

本仓库在 `skills/voice-call/SKILL.md` 提供了配套的技能文档。

## 网关 RPC

- `voicecall.initiate`（`to?`, `message`, `mode?`）
- `voicecall.continue`（`callId`, `message`）
- `voicecall.speak`（`callId`, `message`）
- `voicecall.end`（`callId`）
- `voicecall.status`（`callId`）

[社区插件](./community.md)[Zalo 个人插件](./zalouser.md)