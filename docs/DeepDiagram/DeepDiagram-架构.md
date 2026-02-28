# DeepDiagram AI 架构图集

## Mermaid 架构图文档

---

## 1. 系统整体架构图

### 1.1 高级架构概览

```mermaid
graph TB
    subgraph User Layer [用户交互层]
        WebBrowser[Web 浏览器]
        MobileApp[移动设备]
        APIEndpoints[API 端点]
    end

    subgraph Frontend [React 前端]
        UI[React 19 UI]
        StateMgmt[Zustand 状态管理]
        Canvas[交互式画布]
        ChatPanel[聊天面板]
    end

    subgraph Backend [FastAPI 后端]
        APIGateway[API 网关]
        Auth[认证模块]
        SessionMgr[会话管理]
        AgentOrchestrator[Agent 编排器]
    end

    subgraph Agent Layer [多代理层]
        Router[智能路由器]

        subgraph Specialized Agents [专业 Agent]
            MindMap[🧠 MindMap Agent]
            Flow[🧜‍♂️ Flowchart Agent]
            Charts[📊 Charts Agent]
            Mermaid[🧜‍♀️ Mermaid Agent]
            Drawio[✏️ Draw.io Agent]
            Info[🎨 Infographic Agent]
            General[🤖 General Agent]
        end
    end

    subgraph AI Layer [AI 能力层]
        LLM[大语言模型]
        LangGraph[LangGraph 编排]
        LangChain[LangChain 工具链]
    end

    subgraph Data Layer [数据存储层]
        PostgreSQL[(PostgreSQL)]
        Redis[(Redis 缓存)]
        SSE[Server-Sent Events]
    end

    User Layer --> Frontend
    Frontend <-->|SSE/WebSocket| Backend
    Backend --> Agent Layer
    Agent Layer <--> AI Layer
    Backend --> Data Layer
```

### 1.2 技术栈架构

```mermaid
graph LR
    subgraph Frontend Tech [前端技术栈]
        React19[React 19]
        TS[TypeScript]
        Vite[Vite]
        Tailwind[TailwindCSS]
        Zustand[Zustand]
    end

    subgraph Chart Engines [图表引擎]
        RF[React Flow]
        ME[Mind-elixir]
        EC[Apache ECharts]
        MD[Mermaid.js]
        DI[Draw.io XML]
        AI[AntV Info]
    end

    subgraph Backend Tech [后端技术栈]
        FastAPI[FastAPI]
        Python[Python 3.10+]
        LG[LangGraph]
        LC[LangChain]
    end

    subgraph LLM Support [LLM 支持]
        OpenAI[OpenAI]
        DeepSeek[DeepSeek]
    end

    subgraph Database [数据存储]
        PG[(PostgreSQL)]
        Redis[(Redis)]
    end

    React19 --> RF
    React19 --> ME
    React19 --> EC
    React19 --> MD
    React19 --> DI
    React19 --> AI

    FastAPI --> LG
    LG --> LC
    LC --> OpenAI
    LC --> DeepSeek

    FastAPI --> PG
    FastAPI --> Redis
```

---

## 2. 核心数据流图

### 2.1 用户请求处理流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant FE as 前端
    participant API as FastAPI
    participant Router as 智能路由器
    selected Agent as 目标 Agent
    Tool as 图表工具
    LLM as 大语言模型
    FE2 as 前端渲染

    User->>FE: 输入自然语言请求
    FE->>FE: 打包消息（支持多模态）
    FE->>API: POST /api/chat

    API->>API: 创建/恢复会话状态
    API->>Router: 路由分析

    Router->>Router: 解析用户意图
    Router->>Router: 检查 @agent 显式指定
    Router->>Router: 分析上下文历史
    Router-->>API: 返回目标 Agent

    API->>selected Agent: 分发请求
    selected Agent->>LLM: 思考+工具调用
    LLM->>Tool: 调用图表生成工具

    Tool->>Tool: 生成图表代码
    Tool-->>LLM: 返回结果

    LLM-->>selected Agent: 最终响应
    selected Agent-->>API: 流式响应

    API->>FE: SSE 流式推送
    FE->>FE2: 实时渲染图表

    FE2-->>User: 展示结果
```

### 2.2 智能路由决策流程

```mermaid
flowchart TD
    A[用户输入] --> B{包含 @agent?}
    B -->|是| C[解析显式指定]
    B -->|否| D[LLM 意图分析]

    C --> E[清理消息内容]
    D --> F[检查历史上下文]

    E --> G[返回目标 Agent]
    F --> G

    G --> H{Agent 类型}
    H -->|mindmap| I[MindMap Agent]
    H -->|flow| J[Flowchart Agent]
    H -->|charts| K[Charts Agent]
    H -->|mermaid| L[Mermaid Agent]
    H -->|drawio| M[Draw.io Agent]
    H -->|infographic| N[Infographic Agent]
    H -->|general| O[General Agent]
```

### 2.3 ReAct 代理循环流程

```mermaid
flowchart LR
    subgraph Agent [Agent 节点]
        Think[思考] --> Decide[决策]
        Decide --> Tool{调用工具?}
        Tool -->|是| Execute[执行工具]
        Execute --> Result[获取结果]
        Result --> Think
        Tool -->|否| Final[返回最终结果]
    end

    subgraph Tool [工具节点]
        CreateMermaid[create_mermaid]
        CreateFlow[create_flow]
        CreateChart[create_chart]
        CreateDrawio[render_drawio_xml]
        CreateInfo[create_infographic]
    end

    Execute -.->|工具调用| CreateMermaid
    Execute -.->|工具调用| CreateFlow
    Execute -.->|工具调用| CreateChart
    Execute -.->|工具调用| CreateDrawio
    Execute -.->|工具调用| CreateInfo
```

---

## 3. Agent 模块关系图

### 3.1 LangGraph 工作流编排

```mermaid
graph TB
    Start([用户请求]) --> Router[路由器节点]

    Router -->|intent: mindmap| MindAgent[MindMap Agent]
    Router -->|intent: flowchart| FlowAgent[Flowchart Agent]
    Router -->|intent: charts| ChartsAgent[Charts Agent]
    Router -->|intent: mermaid| MermaidAgent[Mermaid Agent]
    Router -->|intent: drawio| DrawioAgent[Draw.io Agent]
    Router -->|intent: infographic| InfoAgent[Infographic Agent]
    Router -->|intent: general| GeneralAgent[General Agent]

    MindAgent -->|继续?| Check1{有工具调用?}
    FlowAgent -->|继续?| Check2{有工具调用?}
    ChartsAgent -->|继续?| Check3{有工具调用?}
    MermaidAgent -->|继续?| Check4{有工具调用?}
    DrawioAgent -->|继续?| Check5{有工具调用?}
    InfoAgent -->|继续?| Check6{有工具调用?}

    Check1 -->|是| MindTools[MindMap 工具]
    Check2 -->|是| FlowTools[Flowchart 工具]
    Check3 -->|是| ChartsTools[Charts 工具]
    Check4 -->|是| MermaidTools[Mermaid 工具]
    Check5 -->|是| DrawioTools[Draw.io 工具]
    Check6 -->|是| InfoTools[Infographic 工具]

    Check1 -->|否| End1([结束])
    Check2 -->|否| End2([结束])
    Check3 -->|否| End3([结束])
    Check4 -->|否| End4([结束])
    Check5 -->|否| End5([结束])
    Check6 -->|否| End6([结束])

    MindTools --> MindAgent
    FlowTools --> FlowAgent
    ChartsTools --> ChartsAgent
    MermaidTools --> MermaidAgent
    DrawioTools --> DrawioAgent
    InfoTools --> InfoAgent

    GeneralAgent --> End7([结束])
```

### 3.2 Agent 依赖关系

```mermaid
graph TD
    subgraph Core [核心模块]
        Config[配置管理]
        LLM[LLM 封装]
        DB[数据库]
        State[状态管理]
    end

    subgraph Dispatcher [路由模块]
        Router[路由器]
        IntentParser[意图解析]
        HistoryMgr[历史管理]
    end

    subgraph Agents [Agent 模块]
        MindMap[🧠 MindMap]
        Flow[🧜‍♂️ Flowchart]
        Charts[📊 Charts]
        Mermaid[🧜‍♀️ Mermaid]
        Drawio[✏️ Draw.io]
        Info[🎨 Infographic]
    end

    Core --> Dispatcher
    Dispatcher --> Agents

    LLM -.->|流式调用| Agents
    State -.->|上下文传递| Agents
    DB -.->|持久化| Dispatcher
```

### 3.3 提示词系统架构

```mermaid
graph TB
    subgraph Global [全局配置]
        ThinkingModes[思考模式配置]
        SystemPrompts[系统提示词]
    end

    subgraph MindMap Prompts
        M1[MINDMAP_SYSTEM_PROMPT]
        M2[mindmap_agent_node 系统提示词]
        M3[create_mindmap 工具提示词]
    end

    subgraph Flowchart Prompts
        F1[FLOW_SYSTEM_PROMPT]
        F2[flow_agent_node 系统提示词]
        F3[create_flow 工具提示词]
    end

    subgraph Charts Prompts
        C1[CHARTS_SYSTEM_PROMPT]
        C2[charts_agent_node 系统提示词]
        C3[create_chart 工具提示词]
    end

    subgraph Mermaid Prompts
        MD1[MERMAID_SYSTEM_PROMPT]
        MD2[mermaid_agent_node 系统提示词]
        MD3[create_mermaid 工具提示词]
    end

    subgraph Drawio Prompts
        D1[DRAWIO_SYSTEM_PROMPT]
        D2[drawio_agent_node 系统提示词]
        D3[render_drawio_xml 工具提示词]
    end

    subgraph Infographic Prompts
        I1[INFOGRAPHIC_SYSTEM_PROMPT]
        I2[infographic_agent_node 系统提示词]
        I3[create_infographic 工具提示词]
    end

    Global -->|影响所有| MindMap Prompts
    Global -->|影响所有| Flowchart Prompts
    Global -->|影响所有| Charts Prompts
    Global -->|影响所有| Mermaid Prompts
    Global -->|影响所有| Drawio Prompts
    Global -->|影响所有| Infographic Prompts
```

---

## 4. 用户交互流程图

### 4.1 完整用户会话流程

```mermaid
flowchart TD
    A([用户打开应用]) --> B[加载会话历史]
    B --> C{有历史会话?}
    C -->|是| D[恢复最近会话]
    C -->|否| E[创建新会话]

    D --> F[显示聊天面板]
    E --> F

    F --> G[用户输入/上传图片]
    G --> H[前端处理输入]

    H --> I[发送请求到后端]
    I --> J[后端处理请求]

    J --> K{请求类型?}
    K -->|新图表| L[智能路由选择 Agent]
    K -->|修改图表| M[上下文关联]
    K -->|一般对话| N[General Agent]

    L --> O[Agent 处理]
    M --> O

    O --> P[LLM 生成结果]
    P --> Q[工具生成图表]
    Q --> R[流式返回前端]

    R --> S[实时渲染图表]
    S --> T[用户查看结果]

    T --> U{需要修改?}
    U -->|是| V[输入修改指令]
    V --> G
    U -->|否| W{新图表?}

    W -->|是| X[输入新需求]
    X --> G
    W -->|否| Y([结束当前会话])

    Y --> Z[会话持久化]
```

### 4.2 多模态输入处理流程

```mermaid
flowchart LR
    subgraph Input [输入处理]
        TextInput[文本输入]
        ImageUpload[图片上传]
    end

    subgraph Processing [处理层]
        TextParser[文本解析]
        ImageOCR[图像识别]
        ContextCombiner[上下文合并]
    end

    subgraph Output [输出]
        MultimodalMsg[多模态消息]
        RouterDecision[路由决策]
    end

    TextInput --> TextParser
    ImageUpload --> ImageOCR

    TextParser --> ContextCombiner
    ImageOCR --> ContextCombiner

    ContextCombiner --> MultimodalMsg
    MultimodalMsg --> RouterDecision
```

### 4.3 图表导出流程

```mermaid
flowchart TD
    A[生成图表] --> B{图表类型?}

    B -->|React Flow| C[导出 JSON]
    B -->|ECharts| D[导出 JSON option]
    B -->|Mermaid| E[导出 Mermaid 语法]
    B -->|Draw.io| F[导出 XML]
    B -->|Mind Map| G[导出 Markdown]
    B -->|Infographic| H[导出 AntV DSL]

    C --> I[前端渲染]
    D --> I
    E --> I
    F --> I
    G --> I
    H --> I

    I --> J[导出工具栏]

    J --> K{导出格式?}
    K -->|PNG| L[Canvas 渲染]
    K -->|SVG| M[矢量导出]
    K -->|JSON| N[保留源文件]

    L --> O([下载文件])
    M --> O
    N --> O
```

---

## 5. 部署架构图

### 5.1 Docker 部署架构

```mermaid
graph TB
    subgraph Docker Host [Docker 宿主机]
        subgraph Nginx [Nginx 反向代理]
            Static[静态资源]
            Proxy[API 代理]
        end

        subgraph Frontend Container [前端容器]
            ReactApp[React 应用]
        end

        subgraph Backend Container [后端容器]
            FastAPI[FastAPI 服务]
            LangGraph[LangGraph 编排]
            Agents[Agent 集群]
            LLM[LLM 接口]
        end

        subgraph Database Container [数据库容器]
            PostgreSQL[(PostgreSQL)]
        end
    end

    User((用户)) --> Internet
    Internet --> Nginx
    Nginx -->|80/443| ReactApp
    Nginx -->|/api/*| FastAPI

    ReactApp -->|SSE| FastAPI

    FastAPI --> PostgreSQL
    FastAPI --> LLM
    LLM -->|API 调用| OpenAI[OpenAI API]
    LLM -->|API 调用| DeepSeek[DeepSeek API]
```

### 5.2 环境配置架构

```mermaid
flowchart LR
    subgraph Configuration [配置管理]
        EnvFile[.env 文件]
        ConfigMap[配置映射]
        Secrets[密钥管理]
    end

    subgraph Environment Variables [环境变量]
        OPENAI[OPENAI_API_KEY]
        DEEPSEEK[DEEPSEEK_API_KEY]
        DB[数据库连接]
        MODEL[模型配置]
    end

    subgraph Services [服务配置]
        BackendConfig[后端配置]
        FrontendConfig[前端配置]
    end

    EnvFile --> Environment Variables
    Secrets --> Environment Variables

    Environment Variables --> BackendConfig
    Environment Variables --> FrontendConfig
```

---

## 6. 数据模型关系图

### 6.1 核心实体关系

```mermaid
erDiagram
    User ||--o{ Session : has
    Session ||--o{ Message : contains
    Session ||--o{ Diagram : generates
    Message ||--o{ ToolCall : includes

    User {
        string id PK
        string name
        string email
        datetime created_at
    }

    Session {
        string id PK
        string user_id FK
        string title
        string status
        datetime created_at
        datetime updated_at
    }

    Message {
        string id PK
        string session_id FK
        string role
        string content
        string agent_name
        datetime created_at
    }

    Diagram {
        string id PK
        string session_id FK
        string diagram_type
        string content
        string metadata
        datetime created_at
    }

    ToolCall {
        string id PK
        string message_id FK
        string tool_name
        string arguments
        string result
        datetime created_at
    }
```

---

## 7. 关键时序图

### 7.1 思维导图生成时序图

```mermaid
sequenceDiagram
    participant User as 用户
    participant Frontend as 前端
    participant Router as 路由器
    participant MindAgent as MindMap Agent
    participant LLM as LLM
    participant MindTool as create_mindmap
    participant Renderer as Mind-elixir

    User->>Frontend: "创建一个产品规划思维导图"
    Frontend->>Router: 路由请求
    Router->>Router: 识别为 mindmap
    Router-->>Frontend: 返回 mindmap_agent

    Frontend->>MindAgent: 发送消息
    MindAgent->>LLM: 流式调用

    LLM->>MindAgent: 思考过程
    MindAgent->>MindTool: 调用 create_mindmap

    MindTool->>LLM: 生成 Markdown
    LLM-->>MindTool: 返回结果

    MindTool-->>MindAgent: Markdown 内容
    MindAgent-->>Frontend: 流式响应

    Frontend->>Renderer: 渲染 Markmap
    Renderer-->>User: 展示思维导图
```

### 7.2 数据图表生成时序图

```mermaid
sequenceDiagram
    participant User as 用户
    participant FE as 前端
    participant Router as 路由器
    participant Charts as Charts Agent
    participant LLM as LLM
    participant ChartTool as create_chart
    participant ECharts as Apache ECharts

    User->>FE: "展示季度销售数据"
    FE->>Router: 路由请求
    Router->>Router: 识别为 charts
    Router-->>FE: 返回 charts_agent

    FE->>Charts: 发送消息
    Charts->>LLM: 流式调用

    LLM->>ChartTool: 调用 create_chart
    Note over ChartTool: 数据增强<br/>专业配色<br/>故事化标题

    ChartTool-->>LLM: ECharts option JSON
    LLM-->>Charts: 响应

    Charts-->>FE: 流式 JSON
    FE->>ECharts: 初始化图表
    ECharts-->>User: 渲染交互式图表
```

---

*文档生成时间: 2026-01-20*
*作者: Matrix Agent*
