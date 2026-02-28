# DeepDiagram AI 项目深度分析

## Why-What-How 项目介绍

---

## 一、Why（为什么）

### 1.1 项目解决的问题

**核心痛点**：在日常工作、学习和项目管理中，人们经常需要创建各种图表（思维导图、流程图、数据图表、系统架构图等），但传统工具存在以下问题：

1. **学习成本高** - 专业绘图工具（如 Draw.io、Visio）功能复杂，入门门槛高
2. **效率低下** - 从零开始绘制图表需要大量时间和精力
3. **缺乏智能化** - 现有工具无法理解自然语言并自动生成图表
4. **多工具分散** - 不同类型的图表需要使用不同的工具，缺乏统一平台
5. **格式不兼容** - 不同工具生成的图表格式难以互相转换

### 1.2 项目背景与动机

DeepDiagram AI 诞生于 AI 技术快速发展的背景下，旨在将大语言模型的智能化能力与传统图表绘制需求相结合：

- **AI 赋能**：利用 LLM 的理解能力，将自然语言描述转换为专业的图表代码
- **多模态支持**：支持文本输入和图像上传，满足多样化的用户需求
- **自动化生成**：通过智能路由和 Agent 技术，自动选择最适合的图表类型

### 1.3 目标用户

1. **产品经理** - 快速绘制流程图、用户旅程图
2. **技术人员** - 生成系统架构图、API 流程图、数据库 ER 图
3. **数据分析师** - 可视化数据、创建专业图表
4. **项目经理** - 制作甘特图、时间线、项目规划图
5. **教育工作者** - 创建教学材料、信息图

### 1.4 为什么现有方案不够好

| 维度 | 传统方案 | DeepDiagram AI |
|------|----------|----------------|
| **交互方式** | 拖拽式操作 | 自然语言交互 |
| **智能程度** | 纯手动绘制 | AI 自动生成 |
| **多格式支持** | 单一工具 | 多引擎统一平台 |
| **学习曲线** | 陡峭 | 几乎为零 |
| **修改迭代** | 手动调整 | 自然语言指令修改 |
| **实时预览** | 有限 | SSE 流式实时预览 |

---

## 二、What（是什么）

### 2.1 项目概述

**DeepDiagram AI** 是一个开源的智能可视化平台，采用多代理（Multi-Agent）架构，利用 Agentic AI 技术将自然语言转换为专业的图表。它是一个全栈应用，前端使用 React 19 + TypeScript，后端使用 FastAPI + Python，通过 LangGraph 实现复杂的工作流编排。

### 2.2 核心功能

DeepDiagram AI 提供了六大专业图表生成引擎：

#### 🧠 Mind Map Agent（思维导图引擎）
- **技术栈**：`mind-elixir` + Markmap
- **功能**：生成层级结构清晰、交互式的思维导图
- **输出格式**：Markdown / Markmap
- **适用场景**：头脑风暴、知识整理、概念梳理

#### 🧜‍♂️ Flowchart Agent（流程图引擎）
- **技术栈**：`React Flow`
- **功能**：创建现代化、交互式的业务流程图
- **输出格式**：React Flow JSON
- **特点**：
  - 自动布局算法
  - 错误处理分支
  - 决策菱形节点
  - 专业的视觉样式

#### 📊 Data Chart Agent（数据图表引擎）
- **技术栈**：`Apache ECharts`
- **功能**：将数据或描述转换为专业的统计图表
- **支持类型**：
  - 柱状图（Bar）
  - 折线图（Line）
  - 饼图（Pie）
  - 雷达图（Radar）
  - 仪表盘（Gauge）
  - 漏斗图（Funnel）
  - 热力图（Heatmap）
- **特点**：数据增强、故事化标题、专业配色

#### ✏️ Draw.io Agent（专业架构图引擎）
- **技术栈**：Draw.io (Atlas Theme) + mxGraph XML
- **功能**：生成企业级系统架构图
- **输出格式**：Draw.io XML
- **特点**：
  - 专业的架构设计模式
  - 容器和泳道分组
  - 高保真样式设计
  - 云服务组件库

#### 🧜‍♀️ Mermaid Agent（通用图表引擎）
- **技术栈**：`Mermaid.js` + `react-zoom-pan-pinch`
- **功能**：生成多种技术文档图表
- **支持类型**：
  - 流程图（graph）
  - 时序图（sequenceDiagram）
  - 类图（classDiagram）
  - 状态图（stateDiagram）
  - 甘特图（gantt）
  - 用户旅程图（journey）
  - Git 图（gitGraph）
  - 饼图（pie）

#### 🎨 Infographic Agent（信息图引擎）
- **技术栈**：`AntV Infographic`
- **功能**：创建专业的数据海报和可视化摘要
- **输出格式**：AntV DSL
- **特点**：
  - 声明式 DSL 设计
  - 丰富的内置模板
  - 高质量 SVG 渲染
  - 时间线、对比图、金字塔等模板

### 2.3 智能路由系统

DeepDiagram AI 配备了智能意图识别系统，能够：

1. **自动路由**：根据用户描述自动选择最合适的图表类型
2. **显式指定**：支持 `@agent` 语法显式指定图表引擎
3. **上下文感知**：记住会话历史，支持"添加"、"修改"等上下文相关操作
4. **多模态理解**：支持图片上传，草图转数字化图表

### 2.4 技术差异化

| 特性 | DeepDiagram AI | 竞品对比 |
|------|----------------|----------|
| **架构模式** | Multi-Agent + LangGraph | 单体 AI 应用 |
| **LLM 支持** | OpenAI / DeepSeek 双支持 | 通常单一 |
| **实时预览** | SSE 流式传输 | 轮询或刷新 |
| **会话管理** | PostgreSQL 持久化 | 无或简单存储 |
| **消息分支** | 支持版本回溯 | 通常不支持 |
| **可扩展性** | Agent 插件化设计 | 硬编码 |

### 2.5 技术栈概览

#### 前端技术栈
```
React 19 + TypeScript + Vite
├── UI 框架: TailwindCSS
├── 状态管理: Zustand
├── 图表引擎:
│   ├── React Flow (流程图)
│   ├── Mind-elixir (思维导图)
│   ├── Apache ECharts (数据图表)
│   ├── Mermaid.js (通用图表)
│   ├── Draw.io XML (架构图)
│   └── AntV Infographic (信息图)
├── 交互组件: react-resizable-panels
└── 工具库: react-zoom-pan-pinch
```

#### 后端技术栈
```
Python 3.10+ + FastAPI
├── AI 框架: LangGraph + LangChain
├── LLM: ChatOpenAI (OpenAI / DeepSeek)
├── ORM: SQLModel
├── 数据库: PostgreSQL
├── 实时通信: Server-Sent Events (SSE)
└── 部署: Docker + Nginx
```

---

## 三、How（怎么做）

### 3.1 快速开始

#### 环境要求
- **Python**: 3.10 或更高版本
- **Node.js**: v20 或更高版本
- **Docker & Docker Compose**: 生产环境部署

#### 方式一：开发环境配置

**后端设置**：
```bash
cd backend
uv sync
bash start_backend.sh
```

**前端设置**：
```bash
cd frontend
npm install
npm run dev
```

访问 `http://localhost:5173`

#### 方式二：Docker 部署（推荐）

**配置环境变量**：

创建 `.env` 文件：
```env
OPENAI_API_KEY=your_key_here
OPENAI_BASE_URL=https://api.openai.com
MODEL_ID=claude-sonnet-3-7  # 可选，默认 claude-sonnet-3-7
DEEPSEEK_API_KEY=your_key_here
DEEPSEEK_BASE_URL=https://api.deepseek.com
```

**启动服务**：
```bash
docker-compose up -d
```

访问 `http://localhost`

### 3.2 核心使用方式

#### 自然语言交互

```
用户: "创建一个用户登录流程图"
系统: → 智能路由到 Flowchart Agent
      → 生成 React Flow JSON
      → 前端渲染交互式流程图

用户: "添加密码重置功能"
系统: → 识别上下文为 Flowchart
      → 在现有流程图中添加分支
```

#### 显式指定 Agent

```
用户: "@mermaid 创建一个时序图，展示用户注册流程"
系统: → 路由到 Mermaid Agent
      → 生成 Mermaid 语法

用户: "@chart 展示月度销售数据"
系统: → 路由到 Charts Agent
      → 生成 ECharts 配置
```

#### 多模态输入

```
用户: [上传手绘架构图]
系统: → 识别图像内容
      → 转换为 Draw.io XML
      → 生成专业架构图
```

### 3.3 关键配置说明

#### LLM 配置（backend/app/core/config.py）

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `OPENAI_API_KEY` | OpenAI API 密钥 | 必填 |
| `OPENAI_BASE_URL` | OpenAI API 地址 | `https://api.openai.com` |
| `MODEL_ID` | 模型名称 | `claude-sonnet-3-7` |
| `DEEPSEEK_API_KEY` | DeepSeek API 密钥 | 可选 |
| `DEEPSEEK_BASE_URL` | DeepSeek API 地址 | `https://api.deepseek.com` |
| `MAX_TOKENS` | 最大生成 Token 数 | 4096 |
| `THINKING_VERBOSITY` | 思考详细程度 | `normal` |

#### 思考模式配置

```python
# llm.py 中的三种模式
verbosity = "concise"   # 简洁模式 - 只输出关键推理
verbosity = "normal"    # 标准模式 - 使用模型默认
verbosity = "verbose"   # 详细模式 - 探索所有可能性
```

### 3.4 最佳实践建议

#### 1. 充分利用智能路由

- 描述需求时使用**具体关键词**（如"流程"、"架构"、"时序图"）
- 需要特定图表类型时使用 **@agent 语法**
- 复杂需求可以**分步描述**，系统会保持上下文

#### 2. 善用上下文修改

```
初始: "创建一个用户管理流程图"
修改: "添加角色权限分配步骤"
修改: "增加异常处理分支"
```

系统会自动记住之前的图表状态，支持增量修改。

#### 3. 利用会话分支功能

- 当生成结果不理想时，使用 **Retry** 功能探索不同方案
- 通过 **版本历史** 回溯到满意的结果
- 支持在同一会话中对比不同方案

#### 4. 数据图表优化

- 提供具体**数据指标**和**时间范围**
- 说明**分析目的**（如"展示增长趋势"、"对比各产品线"）
- 使用**专业术语**（如"同比增长率"、"市场份额"）

#### 5. 架构图设计

- 明确系统**规模和复杂度**
- 说明**技术栈偏好**
- 提及**安全、扩展性**等非功能需求

### 3.5 项目结构

```
DeepDiagram/
├── backend/                    # 后端服务
│   ├── app/
│   │   ├── agents/            # Agent 核心模块
│   │   │   ├── mindmap.py     # 思维导图 Agent
│   │   │   ├── flow.py        # 流程图 Agent
│   │   │   ├── charts.py      # 数据图表 Agent
│   │   │   ├── mermaid.py     # Mermaid Agent
│   │   │   ├── drawio.py      # Draw.io Agent
│   │   │   ├── infographic.py # 信息图 Agent
│   │   │   ├── dispatcher.py  # 智能路由器
│   │   │   └── graph.py       # LangGraph 工作流
│   │   ├── api/               # API 路由
│   │   ├── core/              # 核心配置
│   │   ├── services/          # 业务逻辑
│   │   └── state/             # 状态管理
│   └── main.py
├── frontend/                   # 前端应用
│   ├── src/
│   │   ├── components/        # React 组件
│   │   │   ├── agents/        # 各 Agent 渲染组件
│   │   │   ├── CanvasPanel.tsx
│   │   │   └── ChatPanel.tsx
│   │   ├── store/             # 状态管理
│   │   └── types/             # TypeScript 类型
│   └── vite.config.ts
├── docs/                       # 文档
├── docker-compose.yml          # Docker 编排
└── README.md
```

---

## 四、Roadmap 与未来规划

### 已完成 ✅

- [x] MVP 版本（3 个核心 Agent）
- [x] Draw.io 集成
- [x] 独立 Mermaid Agent
- [x] 可调整布局面板
- [x] 持久化会话管理
- [x] 消息分支与版本控制

### 规划中 🚧

- [ ] 一键刷新会话（新聊天功能）
- [ ] 扩展多模态支持（PDF、Docx 解析）
- [ ] 更多图表模板
- [ ] 团队协作功能
- [ ] 导出格式扩展

---

## 五、总结

DeepDiagram AI 是一个创新性的 AI 驱动的可视化平台，通过 Multi-Agent 架构和智能路由系统，将自然语言转换为各种专业图表。其核心优势在于：

1. **零学习成本** - 用自然语言描述需求即可
2. **多引擎统一** - 六大图表类型一站式解决
3. **智能上下文** - 支持增量修改和版本管理
4. **开源可扩展** - 插件化 Agent 设计，易于扩展
5. **企业级部署** - Docker 容器化，支持 PostgreSQL

**官方网站**: http://deepd.cturing.cn/

**GitHub 仓库**: https://github.com/orange-gi/DeepDiagram

---

*文档生成时间: 2026-01-20*
*作者: Matrix Agent*
