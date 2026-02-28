# OpenClaw 项目介绍

## Why（为什么）

### 项目解决的问题

OpenClaw 旨在解决当前 AI 助手使用中的几个核心痛点：

**1. 隐私与数据安全问题**

传统的 AI 助手服务（如 ChatGPT 的网页版）需要将用户的对话内容发送到第三方服务器，这引发了严重的隐私担忧。用户的消息、文件和个人信息都可能被第三方存储和使用。OpenClaw 通过让用户在自有设备上运行 AI 助手，从根本上解决了这一问题。用户的所有数据都保存在本地，不经过任何第三方服务器。

**2. 多平台消息管理碎片化**

现代用户通常使用多个即时通讯平台（WhatsApp、Telegram、Slack、Discord 等）进行沟通。管理多个平台的 AI 助手需要在不同应用之间切换，体验割裂。OpenClaw 提供了统一的消息网关，支持同时连接 14+ 个即时通讯平台，用户可以在任何平台上与同一个 AI 助手交互。

**3. AI 助手的可定制性与扩展性受限**

大多数商业 AI 助手提供的是"黑盒"服务，用户无法深度定制 AI 的行为、工具能力或工作流程。OpenClaw 基于开源架构，支持 Skills 平台（ClawHub），允许用户或开发者创建、安装和管理自定义技能，实现真正的个性化 AI 助手。

**4. 云端 AI 的延迟与成本问题**

使用云端 AI 服务存在网络延迟，特别是在需要实时响应的场景下（如语音交互、自动化任务）。OpenClaw 支持本地运行（通过 Docker 沙箱或直接部署），可以显著降低延迟并减少对云端 API 的依赖。

**5. 设备生态的割裂**

用户通常在电脑、手机、平板等多个设备上工作，但大多数 AI 助手缺乏良好的跨设备体验。OpenClaw 提供了 macOS 菜单栏应用、iOS 节点和 Android 节点，通过统一的 Gateway 协议实现设备间的无缝协作。

### 项目背景与动机

OpenClaw 的创建动机源于对"真正属于用户的 AI 助手"的追求。传统的 AI 助手服务虽然在技术上先进，但在数据所有权、隐私保护、定制自由度和跨设备体验方面存在根本性的局限。创始团队认为，AI 助手应该像个人电脑一样——完全由用户拥有和控制，而不是作为一种租赁服务存在。

该项目特别强调本地优先（Local-First）的设计理念，这意味着：

- 所有用户数据默认存储在本地设备上
- 即使在离线状态下，核心功能也能正常工作
- 用户完全掌控自己的数据和 AI 行为
- 不存在供应商锁定问题

### 为什么现有方案不够好

现有的 AI 助手解决方案各有局限：

| 方案类型 | 局限性 |
|---------|-------|
| ChatGPT 网页版 | 数据发送到 OpenAI 服务器，无本地部署选项，定制能力有限 |
| Claude 网页版 | 类似 ChatGPT，无法深度集成到用户工作流 |
| 开源 LLM（如 Ollama） | 缺乏成熟的对话管理和多平台集成能力 |
| 商业 API 方案 | 需要自己构建所有集成，部署和维护成本高 |
| 自托管解决方案 | 通常只支持单一平台，缺乏统一管理界面 |

OpenClaw 的差异化在于：它是一个完整的、开箱即用的个人 AI 助手解决方案，同时提供了企业级开源项目的专业品质和社区支持。

### 目标用户

OpenClaw 主要面向以下用户群体：

- **技术爱好者与开发者**：希望完全控制自己的 AI 助手，定制工作流程
- **隐私敏感用户**：不愿意将个人对话数据发送给第三方服务
- **多平台用户**：在多个即时通讯平台上需要统一的 AI 助手体验
- **效率追求者**：希望通过自动化和 AI 助手提升日常工作效率
- **小型团队**：需要一个可自托管的团队 AI 助手解决方案
- **创作者与专业人士**：需要 AI 助手与创意工具和专业工作流深度集成

## What（是什么）

### 项目概述

OpenClaw 是一个开源的个人 AI 助手平台，采用本地优先架构设计，让用户能够在自己的设备上运行功能完整的 AI 助手。它通过统一的 Gateway 控制平面管理多个即时通讯平台的连接，提供语音唤醒、实时对话和可视化画布等高级功能。

### 核心功能

**1. 多通道消息集成**

OpenClaw 原生支持 14+ 个即时通讯平台的深度集成：

- **主流平台**：WhatsApp、Telegram、Slack、Discord、Google Chat、Signal、iMessage
- **企业平台**：Microsoft Teams
- **扩展平台**：Matrix、Zalo、Zalo Personal、WebChat
- **iMessage 替代方案**：BlueBubbles（推荐）、传统 iMessage

所有平台通过统一的会话模型管理，用户可以在任何平台上与同一个 AI 助手自然对话。

**2. Pi Agent 运行时**

OpenClaw 集成了 pi-agent 运行时，提供：

- RPC 模式的工具流（Tool Streaming）和块流（Block Streaming）
- 支持 Anthropic 和 OpenAI 模型
- 可配置的思考深度（Thinking Level）和详细程度（Verbose Level）
- 内置的会话管理（Session）支持直接对话、群组隔离和回复策略

**3. Gateway 控制平面**

Gateway 是 OpenClaw 的核心控制组件，提供：

- WebSocket 控制平面（ws://127.0.0.1:18789）
- CLI 表面：gateway、agent、send、wizard、doctor 等命令
- 会话管理和配置持久化
- 定时任务（Cron)和 Webhook 支持
- Control UI 和 WebChat 服务

**4. 语音与对话功能**

- **Voice Wake**：macOS/iOS/Android 的语音唤醒功能
- **Talk Mode**：持续对话模式，支持 ElevenLabs 语音合成
- 媒体管道：图像、音频、视频处理，支持转录钩子

**5. Live Canvas 可视化画布**

基于 A2UI 协议的 agent 驱动可视化工作空间，支持：

- 推送和重置画布内容
- 评估（Eval）和快照功能
- 丰富的 UI 组件集成

**6. 工具与自动化**

- **Browser Control**：专用 Chrome/Chromium 浏览器控制
- **Canvas 操作**：画布内容的程序化控制
- **Nodes 工具**：相机拍摄、屏幕录制、位置获取、通知
- **Cron 定时任务**：自动化工作流程
- **Webhook**：外部事件触发

**7. Skills 平台**

ClawHub 技能注册表支持：

- 打包技能（Bundle Skills）：随 OpenClaw 预装
- 管理技能（Managed Skills）：由 ClawHub 集中管理
- 工作区技能（Workspace Skills）：用户自定义的本地技能
- 安装门控和 UI 集成

### 主要特性

**跨平台一致性**：无论用户在哪个平台上与 OpenClaw 交互，体验都是一致的。

**隐私优先设计**：所有数据默认本地存储，用户完全掌控。

**模块化架构**：Gateway、Channels、Agents、Tools 都是独立的模块，便于扩展和维护。

**开发者友好**：提供完整的 SDK、详细的文档和活跃的社区支持。

**企业级品质**：严格的代码审查、自动化测试（Vitest，70% 覆盖率要求）和 CI/CD 流程。

### 与竞品的差异化

| 特性 | OpenClaw | ChatGPT | 自托管方案 |
|-----|---------|---------|----------|
| 数据存储 | 本地优先 | 云端 | 可配置 |
| 平台集成 | 14+ 平台统一 | 仅官方渠道 | 通常单一 |
| 部署复杂度 | 一键安装 | 无需部署 | 高度复杂 |
| 定制能力 | Skills 平台 | 有限 | 需自行开发 |
| 开源程度 | 完全开源 | 闭源 | 视项目而定 |
| 语音支持 | 原生 Voice Wake | 有限 | 需额外配置 |
| 移动端 | iOS/Android 节点 | 官方 App | 需自行构建 |

### 技术栈概述

**核心语言与运行时**

- **TypeScript**：主开发语言，严格类型检查
- **Node.js 22+**：运行环境，支持 Bun 和 npm/pnpm
- **pnpm**：包管理器，提供严格的依赖管理

**主要依赖**

- **通信框架**：grammY（Telegram）、@slack/bolt（Slack）、discord.js（Discord）、@whiskeysockets/baileys（WhatsApp）
- **AI 运行时**：@mariozechner/pi-agent-core、@mariozechner/pi-ai
- **Web 框架**：Hono、Express
- **测试框架**：Vitest（V8 覆盖率，70% 阈值）

**平台支持**

- **macOS**：菜单栏应用，使用 SwiftUI 和 Observation 框架
- **iOS**：原生应用，使用 SwiftUI
- **Android**：原生应用，使用 Kotlin/Jetpack Compose
- **Linux**：服务器部署优化
- **Windows**：WSL2 支持（推荐）

## How（怎么做）

### 快速开始

**环境要求**

- Node.js >= 22.12.0
- pnpm（推荐）或 npm、bun
- macOS、Linux 或 Windows WSL2

**安装 OpenClaw**

```bash
# 使用 npm 安装
npm install -g openclaw@latest

# 或使用 pnpm 安装
pnpm add -g openclaw@latest

# 运行安装向导（推荐）
openclaw onboard --install-daemon
```

**启动 Gateway**

```bash
# 启动 Gateway 服务
openclaw gateway --port 18789 --verbose

# 或作为后台服务运行
openclaw gateway run --bind loopback --port 18789
```

**发送测试消息**

```bash
# 直接发送消息
openclaw message send --to +1234567890 --message "Hello from OpenClaw"

# 与 AI 助手对话
openclaw agent --message "Ship checklist" --thinking high
```

### 从源码构建（开发模式）

```bash
# 克隆仓库
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# 安装依赖
pnpm install

# 构建用户界面
pnpm ui:build

# 构建项目
pnpm build

# 运行安装向导
pnpm openclaw onboard --install-daemon

# 开发模式运行（自动重载）
pnpm gateway:watch
```

### 核心配置说明

**基础配置（~/.openclaw/openclaw.json）**

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5",  // 推荐模型
  },
  gateway: {
    bind: "loopback",                      // 绑定地址
    port: 18789,                            // 端口
  },
  channels: {
    telegram: {
      botToken: "YOUR_BOT_TOKEN",          // Telegram 机器人令牌
    },
    discord: {
      token: "YOUR_DISCORD_BOT_TOKEN",     // Discord 机器人令牌
    },
  },
}
```

**模型配置**

OpenClaw 支持多种模型提供商：

- **Anthropic**（推荐）：Claude Pro/Max，配合 Opus 4.5
- **OpenAI**：ChatGPT、Codex

```json5
{
  models: {
    primary: {
      provider: "anthropic",
      model: "claude-opus-4-5",
      apiKey: "${ANTHROPIC_API_KEY}",
    },
    fallback: {
      provider: "openai",
      model: "gpt-4o",
    },
  },
}
```

**安全配置**

```json5
{
  dmPolicy: "pairing",  // 配对模式：新用户需要配对码
  allowFrom: ["*"],     // 允许的白名单（配对后可设为 "*"）
  sandbox: {
    mode: "non-main",   // 非主会话使用沙箱
  },
}
```

### 最佳实践建议

**1. 部署架构选择**

- **个人使用**：macOS 菜单栏应用 + 本地 Gateway
- **服务器部署**：Linux 服务器 + 远程访问（Tailscale/SSH）
- **团队使用**：Docker 部署 + 集中管理

**2. 安全配置**

- 始终使用配对模式（dmPolicy: "pairing"）防止未授权访问
- 定期运行 `openclaw doctor` 检查配置问题
- 使用 Tailscale Serve/Funnel 时强制设置密码认证

**3. 性能优化**

- 考虑使用 Docker 沙箱运行非信任会话
- 合理配置会话清理策略（Session Pruning）
- 使用模型故障转移（Model Failover）确保可用性

**4. 技能管理**

- 从 ClawHub 探索有用的技能
- 为特定工作流创建自定义 Skills
- 使用版本控制管理 Workspace Skills

**5. 监控与调试**

- 使用 `openclaw channels status --probe` 检查通道状态
- 查看 Gateway 日志（/tmp/openclaw-gateway.log）
- 运行 `openclaw doctor` 解决常见问题

### 常用命令速查

| 命令 | 用途 |
|-----|------|
| `openclaw onboard` | 安装向导，引导初始配置 |
| `openclaw gateway` | 启动/管理 Gateway 服务 |
| `openclaw agent` | 与 AI 助手对话 |
| `openclaw message send` | 发送消息到指定通道 |
| `openclaw channels status` | 检查所有通道状态 |
| `openclaw channels login` | 登录新通道 |
| `openclaw doctor` | 诊断和修复问题 |
| `openclaw update` | 更新 OpenClaw |
| `openclaw nodes` | 管理 iOS/Android 节点 |
| `openclaw pairing approve` | 批准配对请求 |

### 故障排除

**Gateway 无法启动**

```bash
# 检查端口占用
ss -ltnp | rg 18789

# 查看日志
tail -n 120 /tmp/openclaw-gateway.log

# 重启服务
pkill -9 -f openclaw-gateway
openclaw gateway run --bind loopback --port 18789
```

**通道连接问题**

```bash
# 全面诊断
openclaw doctor

# 检查特定通道状态
openclaw channels status --probe
```

**权限问题（macOS）**

- 确保在系统设置中授予必要的权限
- 使用签名构建以保持权限持久化
- 参考 `docs/mac/permissions.md`

---

**项目地址**：https://github.com/openclaw/openclaw
**官方网站**：https://openclaw.ai
**文档**：https://docs.openclaw.ai
**Discord 社区**：https://discord.gg/clawd
