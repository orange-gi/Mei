# DeerFlow 项目介绍

## 项目概述

DeerFlow（Deep Exploration and Efficient Research Flow，深度探索与高效研究流）是由字节跳动开源的超级智能体框架。它通过编排子智能体、记忆系统和沙箱环境，让 AI 能够完成几乎任何复杂任务——这一切都通过可扩展的技能系统驱动。DeerFlow 2.0 是从零开始的重写版本，基于 LangGraph 和 LangChain 构建。

2026 年 2 月 28 日，DeerFlow 2.0 发布后登顶 GitHub Trending 第一名，充分证明了社区对该项目的认可与热情。

---

## Why：为什么需要 DeerFlow

### 传统深度研究的局限

传统的 AI 辅助研究存在以下核心问题：

- **上下文窗口限制**：长对话会超出模型上下文限制，导致早期信息丢失
- **工具能力单一**：缺乏文件系统、代码执行、Web 搜索等综合能力
- **无法协作分解**：复杂任务只能串行处理，缺乏并行子任务执行能力
- **记忆缺失**：每次对话结束即遗忘，无法跨会话积累知识
- **执行环境不安全**：直接执行用户代码存在安全风险

### DeerFlow 的解决之道

DeerFlow 不再只是一个聊天机器人，而是一个完整的**超级智能体运行平台**：

- **子智能体并行执行**：复杂任务自动分解，多个子智能体同时工作
- **隔离沙箱执行**：代码在 Docker 容器中安全运行，环境完全隔离
- **持久化记忆系统**：跨会话记住用户偏好、工作背景和知识积累
- **渐进式技能加载**：只加载当前任务需要的技能，保持上下文精简
- **可扩展工具生态**：内置工具、MCP 协议工具、社区工具按需组合

---

## What：DeerFlow 是什么

### 核心定位

DeerFlow 是一个**超级智能体线束（Super Agent Harness）**，它提供了智能体运行所需的一切基础设施：

- 文件系统访问
- 持久化记忆
- 技能系统
- 沙箱执行环境
- 子智能体规划与调度

### 核心特性

| 特性类别 | 详细说明 |
|---------|---------|n| **智能体编排** | 基于 LangGraph 的主智能体 + 子智能体系统，支持并行任务执行 |
| **沙箱执行** | 支持本地、Docker、Kubernetes 三种隔离执行模式 |
| **记忆系统** | LLM 驱动的持久化上下文，跨会话积累用户知识 |
| **技能生态** | 17 个内置技能，支持自定义技能扩展 |
| **工具集成** | 内置工具 + MCP 协议 + 社区工具（ Tavily、Jina AI、Firecrawl 等）|
| **多模态支持** | 支持图像理解、视频理解、语音处理 |
| **多渠道接入** | 支持 Web、 Telegram、Slack、飞书多渠道交互 |

### 技能系统

DeerFlow 的技能是结构化的能力模块，通过 Markdown 文件定义工作流程，最佳实践和资源引用。内置技能包括：

| 技能名称 | 功能描述 |
|---------|---------|n| deep-research | 系统化网络研究方法论 |
| consulting-analysis | 麦肯锡/BCG 风格的专业研究报告生成 |
| data-analysis | Excel/CSV 数据分析与 SQL 查询 |
| github-deep-research | GitHub 仓库深度研究 |
| ppt-generation | PPT/幻灯片生成 |
| image-generation | 图像生成 |
| video-generation | 视频生成 |
| podcast-generation | 播客音频生成 |
| chart-visualization | 数据可视化图表生成 |
| frontend-design | 前端界面设计与实现 |
| bootstrap | 项目初始化与引导 |

---

## How：如何使用 DeerFlow

### 快速开始

#### 环境要求

- Python 3.12+
- Node.js 22+
- Docker（可选，用于沙箱隔离）

#### 安装步骤

**第一步：克隆仓库**

```bash
git clone https://github.com/bytedance/deer-flow.git
cd deer-flow
```

**第二步：生成配置文件**

```bash
make config
```

**第三步：配置模型**

编辑 `config.yaml`，添加至少一个 LLM 模型配置：

```yaml
models:
  - name: gpt-4
    display_name: GPT-4
    use: langchain_openai:ChatOpenAI
    model: gpt-4
    api_key: $OPENAI_API_KEY
    max_tokens: 4096
    temperature: 0.7
```

**第四步：设置 API Key**

在 `.env` 文件中配置：

```bash
OPENAI_API_KEY=your-api-key-here
TAVILY_API_KEY=your-tavily-api-key
```

#### 运行方式

**Docker 方式（推荐）**

```bash
make docker-init  # 拉取沙箱镜像（仅首次）
make docker-start # 启动服务
```

**本地开发方式**

```bash
make check        # 检查环境依赖
make install      # 安装前后端依赖
make dev          # 启动所有服务
```

访问地址：`http://localhost:2026`

### 高级配置

#### 沙箱模式

DeerFlow 支持三种沙箱执行模式：

```yaml
# config.yaml
sandbox:
  # 模式一：本地执行
  use: deerflow.sandbox.local:LocalSandboxProvider

  # 模式二：Docker 执行
  use: deerflow.community.aio_sandbox:AioSandboxProvider
  image: deerflow/sandbox:latest

  # 模式三：Kubernetes 执行（需要 provisioner）
  use: deerflow.community.aio_sandbox:AioSandboxProvider
  provisioner_url: http://localhost:8002
```

#### MCP 服务器

支持通过 MCP（Model Context Protocol）扩展工具能力：

```json
// extensions_config.json
{
  "mcpServers": {
    "github": {
      "enabled": true,
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    }
  }
}
```

#### IM 渠道集成

```yaml
# config.yaml
channels:
  telegram:
    enabled: true
    bot_token: $TELEGRAM_BOT_TOKEN

  slack:
    enabled: true
    bot_token: $SLACK_BOT_TOKEN
    app_token: $SLACK_APP_TOKEN

  feishu:
    enabled: true
    app_id: $FEISHU_APP_ID
    app_secret: $FEISHU_APP_SECRET
```

### 编程式使用

DeerFlow 也可以作为 Python 库直接使用：

```python
from deerflow.client import DeerFlowClient

client = DeerFlowClient()

# 同步对话
response = client.chat("分析这篇论文", thread_id="my-thread")

# 流式响应
for event in client.stream("hello"):
    if event.type == "messages-tuple" and event.data.get("type") == "ai":
        print(event.data["content"])

# 配置管理
models = client.list_models()
skills = client.list_skills()
```

---

## 架构概览

### 系统组件

```
┌─────────────────────────────────────────────────────────┐
│                    Nginx (端口 2026)                    │
│                  统一反向代理入口                         │
└─────────────┬───────────────────────┬───────────────────┘
              │                       │
    /api/langgraph/*                  │  /api/*
              ▼                       ▼
┌─────────────────────┐   ┌──────────────────────────────┐
│   LangGraph Server  │   │        Gateway API (8001)     │
│      (端口 2024)     │   │        FastAPI REST          │
│                      │   │                              │
│  ┌────────────────┐  │   │  模型配置 / MCP / 技能 /     │
│  │   Lead Agent   │  │   │  记忆管理 / 文件上传 /        │
│  │  ┌──────────┐  │  │   │  工件服务                    │
│  │  │Middleware│  │  │   └──────────────────────────────┘
│  │  │  Chain   │  │  │
│  │  └──────────┘  │  │
│  │  ┌──────────┐  │  │
│  │  │  Tools   │  │  │
│  │  └──────────┘  │  │
│  │  ┌──────────┐  │  │
│  │  │Subagents │  │  │
│  │  └──────────┘  │  │
│  └────────────────┘  │
└─────────────────────┘
```

### 核心技术栈

| 层级 | 技术选型 |
|-----|---------|n| **智能体框架** | LangGraph 1.0.6+、LangChain 1.2.3+ |
| **后端服务** | FastAPI 0.115.0+、Python 3.12+ |
| **前端界面** | Next.js、TypeScript、TailwindCSS |
| **沙箱隔离** | Docker、agent-sandbox |
| **文档转换** | markitdown（PDF/PPT/Excel/Word 转 Markdown）|
| **网络工具** | tavily-python、firecrawl-py、Jina AI |

---

## 应用场景

### 深度研究

使用 `deep-research` 技能进行系统性网络研究，适用于：

- 技术趋势分析
- 市场调研报告
- 竞品分析
- 学术文献综述

### 专业报告生成

使用 `consulting-analysis` 技能生成麦肯锡/BCG 风格的专业研究报告：

- 行业分析报告
- 消费者洞察报告
- 品牌策略分析
- 投资尽职调查

### 数据分析

使用 `data-analysis` 技能对上传的 Excel/CSV 数据进行 SQL 分析：

- 销售数据分析
- 用户行为分析
- 财务指标分析

### 内容创作

结合多种技能完成复杂内容创作任务：

- PPT 演示文稿生成
- 前端页面设计与实现
- 图像与视频生成
- 播客脚本与音频生成

---

## 与竞品对比

| 特性 | DeerFlow | LangChain Agents | AutoGPT | CrewAI |
|-----|----------|------------------|---------|--------|
| 子智能体并行 | 支持 | 需自行实现 | 不支持 | 支持 |
| 沙箱隔离 | 原生支持 | 需集成 | 基础支持 | 需集成 |
| 持久记忆 | 内置 | 需配置 | 基础 | 需配置 |
| 技能系统 | 渐进式加载 | 工具集 | 插件系统 | 角色系统 |
| MCP 协议 | 原生支持 | 部分支持 | 不支持 | 不支持 |
| IM 渠道 | 4 种渠道 | 不支持 | 不支持 | 不支持 |
| 开源许可 | MIT | MIT | MIT | MIT |

---

## 总结

DeerFlow 代表了新一代 AI 智能体平台的发展方向——不再是一个简单的对话界面，而是一个完整的**智能体运行基础设施**。通过其模块化设计、强大的工具生态和灵活的扩展机制，开发者可以快速构建和部署复杂的 AI 应用。

无论是个人用户还是企业团队，DeerFlow 都提供了足够的灵活性和功能性来满足各种复杂任务的自动化需求。