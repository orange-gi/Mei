# nanobot：超轻量级个人 AI 助手

## 一、为什么需要 nanobot

### 1.1 项目背景与动机

在当前人工智能助手领域，主流的 AI Agent 框架往往存在一个共同的问题：功能过于复杂、体积臃肿。以 Clawdbot 为例，虽然它提供了丰富的功能，但其代码库规模超过 43 万行，这使得普通用户难以理解、修改和扩展。对于研究者而言，大型代码库不仅增加了学习成本，也限制了快速迭代和实验的能力。

nanobot 的诞生正是为了解决这一困境。该项目由香港大学数据科学实验室（HKUDS）发起，专注于打造一个**极简主义**的个人 AI 助手框架。项目的核心理念是：**用最少的代码实现核心功能**，让每一个人都能轻松拥有自己的 AI 助手。

### 1.2 现有方案的不足

现有的 AI 助手解决方案普遍存在以下问题：

**代码规模庞大**是首要障碍。主流框架如 Clawdbot 的 43 万行代码意味着开发者需要花费大量时间才能理解系统全貌，这不仅增加了维护成本，也使得定制化开发变得困难重重。对于希望进行学术研究或教学演示的开发者而言，学习曲线过于陡峭。

**部署复杂度高**是第二个痛点。许多框架需要复杂的依赖管理、多步骤的配置流程，甚至需要专门的服务器基础设施。这导致许多潜在用户在初次尝试时就遭遇挫折，无法快速体验 AI 助手的便利。

**资源消耗过大**也是一个现实问题。大型框架往往需要较高的计算资源和内存占用，这对于个人用户或边缘设备来说并不友好。轻量级解决方案的缺失限制了 AI 助手的普及范围。

### 1.3 目标用户群体

nanobot 面向以下几类用户群体：

**个人用户**可以通过简单的一键部署命令，在几分钟内拥有一个专属的 AI 助手。无论是日常问答、任务管理还是信息检索，nanobot 都能提供及时有效的支持。

**研究者和教育工作者**可以借助 nanobot 简洁的代码结构，深入理解 AI Agent 的核心工作原理。清晰的模块划分和详尽的注释使得学术研究和教学演示变得轻而易举。

**开发者**可以将 nanobot 作为构建更复杂应用的起点。其开放的架构设计和丰富的扩展接口，为二次开发和功能定制提供了坚实的基础。

## 二、nanobot 是什么

### 2.1 核心定义

nanobot 是一个**超轻量级个人 AI 助手框架**，仅用约 4000 行代码实现了传统框架 99% 的核心功能。作为 Clawdbot 的轻量级衍生项目，nanobot 在保持强大功能的同时，将代码规模压缩到了极致。

该框架采用现代化的 Python 异步架构，基于 Python 3.11+ 构建，充分利用了 asyncio 的并发能力。nanobot 的设计哲学强调**代码可读性**、**易于扩展**和**快速部署**，使其成为个人 AI 助手领域的理想选择。

### 2.2 主要特性与功能

nanobot 提供四大核心功能模块，覆盖个人用户的主要使用场景：

**实时市场分析助手**能够全天候监控市场动态，发现投资机会，提供数据洞察和市场趋势分析。用户可以通过自然语言查询获取专业的市场分析报告，无需复杂的金融知识背景。

**全栈软件工程师**具备完整的代码开发能力，可以完成从需求分析到代码编写、部署上线的全流程。用户只需描述需求，nanobot 就能自动生成符合规范的代码，并支持直接执行和调试。

**智能日程管理**帮助用户高效管理日常任务，支持定时提醒、周期性任务设置和自动化工作流。用户可以用自然语言添加和管理日程，nanobot 会准时发送提醒通知。

**个人知识助手**提供长期记忆存储和知识检索功能，能够学习用户偏好、记住重要信息，并在需要时快速检索相关知识。这使得 nanobot 能够提供越来越个性化的服务。

### 2.3 技术栈概述

nanobot 的技术栈经过精心选择，平衡了功能性和轻量性：

**核心语言**采用 Python 3.11+，利用其简洁的语法和丰富的生态系统。异步编程模型确保了系统的高并发处理能力，同时保持了代码的可读性。

**LLM 集成**通过 litellm 库实现，支持 OpenRouter、Anthropic、OpenAI、DeepSeek、Groq、Gemini 等多家主流 LLM 服务商。这种多 provider 架构让用户可以根据需求灵活切换模型，优化成本和效果。

**通信渠道**原生支持 Telegram、WhatsApp 和飞书三大平台。bridge 目录下的 TypeScript 实现提供了可靠的跨平台消息传递能力。

**工具系统**内置文件系统操作、Shell 命令执行、网页搜索和抓取、消息发送、定时任务、子代理等核心工具，形成完整的功能闭环。

### 2.4 与竞品的差异化

| 对比维度 | nanobot | Clawdbot | 其他主流框架 |
|---------|---------|----------|-------------|
| 代码行数 | ~4,000 行 | 430,000+ 行 | 通常数十万行 |
| 部署时间 | 2 分钟 | 30 分钟以上 | 10-60 分钟 |
| 资源占用 | 极低 | 高 | 中等偏高 |
| 代码可读性 | 优秀 | 一般 | 参差不齐 |
| 学习曲线 | 平缓 | 陡峭 | 中等 |
| 扩展难度 | 简单 | 复杂 | 中等 |

nanobot 的差异化优势在于其**极致的轻量化**与**完整的核心功能**之间的平衡。它不追求功能的堆砌，而是专注于提供最常用、最实用的功能，并通过简洁的代码让用户能够真正理解并掌控自己的 AI 助手。

## 三、如何使用 nanobot

### 3.1 快速开始

nanobot 的安装和使用非常简单，即使是没有任何技术背景的用户也能在几分钟内完成部署。

**方式一：从源码安装（推荐用于开发）**

```bash
git clone https://github.com/HKUDS/nanobot.git
cd nanobot
pip install -e .
```

**方式二：使用 uv 工具安装（稳定版）**

```bash
uv tool install nanobot-ai
```

**方式三：从 PyPI 安装（标准方式）**

```bash
pip install nanobot-ai
```

### 3.2 初始化配置

安装完成后，执行初始化命令创建配置文件和工作空间：

```bash
nanobot onboard
```

此命令会在用户主目录下创建 `~/.nanobot/config.json` 配置文件和 `~/nanobot-workspace/` 工作空间。配置文件采用 JSON 格式，包含 LLM 服务商配置、渠道设置和工具参数等。

**API 密钥配置示例**

```json
{
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-v1-xxx"
    }
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-5"
    }
  },
  "tools": {
    "web": {
      "search": {
        "apiKey": "BSA-xxx"
      }
    }
  }
}
```

API 密钥可以通过以下渠道获取：LLM 服务选择 OpenRouter（推荐，支持所有模型）或各服务商的官方平台；网页搜索功能可选配 Brave Search API。

### 3.3 核心使用方式

**命令行对话模式**

```bash
# 发送单条消息
nanobot agent -m "What is 2+2?"

# 进入交互式对话
nanobot agent
```

**启动网关服务**

启动网关服务后，nanobot 可以在后台持续运行，接收来自各渠道的消息：

```bash
nanobot gateway
```

**查看系统状态**

```bash
nanobot status
```

### 3.4 通信渠道配置

nanobot 支持三大主流即时通讯平台，用户可以根据需求选择启用。

**Telegram 渠道（推荐）**

1. 在 Telegram 中搜索 @BotFather机器人
2. 发送 `/newbot` 命令并按照提示创建新机器人
3. 复制机器人 Token
4. 配置 config.json：

```json
{
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "YOUR_BOT_TOKEN",
      "allowFrom": ["YOUR_USER_ID"]
    }
  }
}
```

**WhatsApp 渠道**

需要 Node.js 18+ 环境支持，通过扫码登录：

```bash
nanobot channels login
# 扫描二维码绑定设备
```

配置示例：

```json
{
  "channels": {
    "whatsapp": {
      "enabled": true,
      "allowFrom": ["+1234567890"]
    }
  }
}
```

**飞书渠道**

支持 WebSocket 长连接，无需公网 IP：

```bash
pip install nanobot-ai[feishu]
```

在飞书开放平台创建应用并配置权限后，填写 config.json：

```json
{
  "channels": {
    "feishu": {
      "enabled": true,
      "appId": "cli_xxx",
      "appSecret": "xxx",
      "allowFrom": []
    }
  }
}
```

### 3.5 本地模型支持

nanobot 支持通过 vLLM 部署本地模型，实现完全离线运行：

**步骤一：启动 vLLM 服务**

```bash
vllm serve meta-llama/Llama-3.1-8B-Instruct --port 8000
```

**步骤二：配置本地 provider**

```json
{
  "providers": {
    "vllm": {
      "apiKey": "dummy",
      "apiBase": "http://localhost:8000/v1"
    }
  },
  "agents": {
    "defaults": {
      "model": "meta-llama/Llama-3.1-8B-Instruct"
    }
  }
}
```

### 3.6 定时任务管理

nanobot 内置 Cron 定时任务支持，可以设置周期性自动化任务：

```bash
# 添加每日任务（每天上午9点发送问候）
nanobot cron add --name "daily" --message "Good morning!" --cron "0 9 * * *"

# 添加周期性任务（每3600秒执行一次）
nanobot cron add --name "hourly" --message "Check status" --every 3600

# 列出所有任务
nanobot cron list

# 删除任务
nanobot cron remove <job_id>
```

### 3.7 Docker 部署

nanobot 支持 Docker 容器化部署，适合需要隔离环境或快速迁移的场景：

```bash
# 构建镜像
docker build -t nanobot .

# 初始化配置
docker run -v ~/.nanobot:/root/.nanobot --rm nanobot onboard

# 运行网关
docker run -v ~/.nanobot:/root/.nanobot -p 18790:18790 nanobot gateway

# 执行单次命令
docker run -v ~/.nanobot:/root/.nanobot --rm nanobot agent -m "Hello!"
```

### 3.8 最佳实践建议

**配置优化**方面，建议根据实际使用频率选择合适的 LLM 模型。对于日常对话场景，可以选择性价比高的模型如 MiniMax 或 DeepSeek；对于代码生成等复杂任务，则可以使用更强大的 Claude 或 GPT 系列模型。

**安全设置**方面，务必保护好 API 密钥，不要将 config.json 上传到公开代码库。建议使用环境变量或密钥管理服务来存储敏感信息。

**性能调优**方面，可以通过调整 `max_iterations` 参数控制 Agent 的最大思考迭代次数，在效果和响应速度之间取得平衡。对于资源受限的环境，可以降低最大迭代次数以减少计算开销。

**扩展开发**方面，nanobot 的工具系统支持插件式扩展，开发者可以参照内置工具的实现方式，添加自定义功能模块。技能系统（Skills）则提供了更高级的扩展机制，支持通过 Markdown 文件定义复杂的功能模板。

---

**参考链接**

- GitHub 仓库：https://github.com/HKUDS/nanobot
- PyPI 包：https://pypi.org/project/nanobot-ai/
- 官方文档：见项目 README
