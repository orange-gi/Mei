# PentAGI 架构文档

## 一、系统整体架构

PentAGI 采用微服务架构设计，所有组件通过Docker容器化部署，实现完全隔离和易于扩展。系统架构分为多个层次，每个层次负责特定的功能域。

### 1.1 系统上下文图

以下是PentAGI与外部系统交互的整体视图：

```mermaid
flowchart TB
    classDef person fill:#08427B,stroke:#073B6F,color:#fff
    classDef system fill:#1168BD,stroke:#0B4884,color:#fff
    classDef external fill:#666666,stroke:#0B4884,color:#fff

    pentester["安全工程师
    (系统用户)"]

    pentagi["PentAGI
    (自主渗透测试系统)"]

    target["目标系统
    (被测试系统)"]
    llm["LLM提供商
    (OpenAI/Anthropic/Ollama/Bedrock/Gemini)"]
    search["搜索系统
    (Google/DuckDuckGo/Tavily/Traversaal/Perplexity/Searxng)"]
    langfuse["langfuse
    (LLM可观测性仪表板)"]
    grafana["grafana
    (系统监控仪表板)"]

    pentester --> |使用HTTPS| pentagi
    pentester --> |监控AI调用| langfuse
    pentester --> |监控系统| grafana
    pentagi --> |测试各种协议| target
    pentagi --> |查询| llm
    pentagi --> |搜索| search
    pentagi --> |报告| langfuse
    pentagi --> |报告| grafana

    class pentester person
    class pentagi system
    class target,llm,search,langfuse,grafana external
```

**架构说明**：

- **安全工程师（用户）**：通过HTTPS与PentAGI交互，监控AI行为和系统状态
- **PentAGI核心系统**：处理用户请求，协调各AI代理，执行渗透测试任务
- **目标系统**：被测试的系统，PentAGI通过各种协议进行安全测试
- **LLM提供商**：为AI代理提供推理能力，支持多种模型提供商
- **搜索系统**：用于信息收集和情报获取
- **监控工具**：Langfuse用于AI行为监控，Grafana用于系统监控

### 1.2 容器架构图

以下是PentAGI的详细容器架构，展示了所有微服务组件及其交互关系：

```mermaid
graph TB
    subgraph 核心服务
        UI["前端UI
        React + TypeScript"]
        API["后端API
        Go + GraphQL"]
        DB[(向量存储
        PostgreSQL + pgvector)]
        MQ[任务队列
        异步处理]
        Agent[AI代理
        多代理系统]
    end

    subgraph 知识图谱
        Graphiti[Graphiti
        知识图谱API]
        Neo4j[(Neo4j
        图数据库)]
    end

    subgraph 监控栈
        Grafana[Grafana
        仪表板]
        VictoriaMetrics[VictoriaMetrics
        时序数据库]
        Jaeger[Jaeger
        分布式追踪]
        Loki[Loki
        日志聚合]
        OTEL[OpenTelemetry
        数据收集]
    end

    subgraph 分析平台
        Langfuse[Langfuse
        LLM分析]
        ClickHouse[ClickHouse
        分析数据库]
        Redis[Redis
        缓存+速率限制]
        MinIO[MinIO
        S3存储]
    end

    subgraph 安全工具
        Scraper[网页抓取器
        隔离浏览器]
        PenTest[安全工具
        20+专业工具
        沙盒执行]
    end

    UI --> |HTTP/WS| API
    API --> |SQL| DB
    API --> |事件| MQ
    MQ --> |任务| Agent
    Agent --> |命令| PenTest
    Agent --> |查询| DB
    Agent --> |知识| Graphiti
    Graphiti --> |图| Neo4j

    API --> |遥测| OTEL
    OTEL --> |指标| VictoriaMetrics
    OTEL --> |追踪| Jaeger
    OTEL --> |日志| Loki

    Grafana --> |查询| VictoriaMetrics
    Grafana --> |查询| Jaeger
    Grafana --> |查询| Loki

    API --> |分析| Langfuse
    Langfuse --> |存储| ClickHouse
    Langfuse --> |缓存| Redis
    Langfuse --> |文件| MinIO

    classDef 核心 fill:#f9f,stroke:#333,stroke-width:2px,color:#000
    classDef 知识 fill:#ffa,stroke:#333,stroke-width:2px,color:#000
    classDef 监控 fill:#bbf,stroke:#333,stroke-width:2px,color:#000
    classDef 分析 fill:#bfb,stroke:#333,stroke-width:2px,color:#000
    classDef 工具 fill:#fbb,stroke:#333,stroke-width:2px,color:#000

    class UI,API,DB,MQ,Agent 核心
    class Graphiti,Neo4j 知识
    class Grafana,VictoriaMetrics,Jaeger,Loki,OTEL 监控
    class Langfuse,ClickHouse,Redis,MinIO 分析
    class Scraper,PenTest 工具
```

**组件功能说明**：

| 组件 | 技术栈 | 功能描述 |
|------|--------|----------|
| 前端UI | React + TypeScript | 用户交互界面，展示测试进度和结果 |
| 后端API | Go + GraphQL | 业务逻辑处理，API网关 |
| 向量存储 | PostgreSQL + pgvector | 语义搜索，记忆存储 |
| 任务队列 | 异步处理 | 可靠的任务调度和执行 |
| AI代理 | 多代理系统 | 协调各专业代理执行测试任务 |
| Graphiti | 知识图谱API | 语义关系追踪和上下文理解 |
| Neo4j | 图数据库 | 存储实体关系和时序上下文 |
| Grafana | 监控仪表板 | 系统指标可视化 |
| VictoriaMetrics | 时序数据库 | 高性能指标存储 |
| Jaeger | 分布式追踪 | 端到端调试和问题诊断 |
| Loki | 日志聚合 | 可扩展的日志分析 |
| OpenTelemetry | 数据收集 | 统一的可观测性数据收集 |
| Langfuse | LLM分析 | AI模型性能监控和追踪 |
| ClickHouse | 分析数据库 | 列式分析数据仓库 |
| Redis | 缓存 | 高速缓存和速率限制 |
| MinIO | 对象存储 | S3兼容的文件存储 |
| 网页抓取器 | 隔离浏览器 | 安全的Web信息收集 |
| 安全工具 | 20+专业工具 | 渗透测试工具集 |

## 二、核心数据流

PentAGI的核心数据流展示了信息如何在系统中流转，从用户发起测试请求到生成最终报告的完整过程。

### 2.1 主数据流程

```mermaid
flowchart LR
    subgraph 输入层
        User[用户请求] --> API[API网关]
    end

    subgraph 处理层
        API --> Flow[流程管理]
        Flow --> Task[任务分解]
        Task --> SubTask[子任务分配]
        SubTask --> Agent[AI代理执行]
    end

    subgraph 执行层
        Agent --> Tools{工具选择}
        Tools -->|安全工具| Pentest[渗透测试工具]
        Tools -->|搜索| Search[搜索工具]
        Tools -->|浏览器| Browser[浏览器工具]
        Tools -->|终端| Terminal[终端命令]
    end

    subgraph 存储层
        Agent --> Memory[记忆系统]
        Memory --> Vector[(向量数据库)]
        Memory --> Graph[知识图谱]
    end

    subgraph 输出层
        Agent --> Report[报告生成]
        Report --> User
    end
```

### 2.2 代理交互流程

PentAGI采用多代理协作模式，不同类型的代理各司其职，协同完成复杂的渗透测试任务：

```mermaid
sequenceDiagram
    participant O as 编排器
    participant R as 研究者
    participant D as 开发者
    participant E as 执行者
    participant VS as 向量存储
    participant KB as 知识库

    Note over O,KB: 流程初始化
    O->>VS: 查询相似任务
    VS-->>O: 返回经验
    O->>KB: 加载相关知识
    KB-->>O: 返回上下文

    Note over O,R: 研究阶段
    O->>R: 分析目标
    R->>VS: 搜索相似案例
    VS-->>R: 返回模式
    R->>KB: 查询漏洞
    KB-->>R: 返回已知问题
    R->>VS: 存储发现
    R-->>O: 研究结果

    Note over O,D: 规划阶段
    O->>D: 规划攻击
    D->>VS: 查询漏洞利用
    VS-->>D: 返回技术
    D->>KB: 加载工具信息
    KB-->>D: 返回能力
    D-->>O: 攻击计划

    Note over O,E: 执行阶段
    O->>E: 执行计划
    E->>KB: 加载工具指南
    KB-->>E: 返回程序
    E->>VS: 存储结果
    E-->>O: 执行状态
```

**代理角色说明**：

| 代理类型 | 功能职责 |
|----------|----------|
| 编排器（Orchestrator） | 协调整个测试流程，管理任务分解和代理协作 |
| 研究者（Researcher） | 收集目标信息，分析漏洞模式，研究攻击向量 |
| 开发者（Developer） | 规划攻击策略，选择合适的漏洞利用方法 |
| 执行者（Executor） | 执行具体的测试命令和工具调用 |

## 三、实体关系图

PentAGI的数据模型清晰定义了各实体之间的关系，以下是核心实体及其关联：

```mermaid
erDiagram
    Flow ||--o{ Task : 包含
    Task ||--o{ SubTask : 包含
    SubTask ||--o{ Action : 包含
    Action ||--o{ Artifact : 产生
    Action ||--o{ Memory : 存储

    Flow {
        string id PK
        string name "流程名称"
        string description "流程描述"
        string status "active/completed/failed"
        json parameters "流程参数"
        timestamp created_at
        timestamp updated_at
    }

    Task {
        string id PK
        string flow_id FK
        string name "任务名称"
        string description "任务描述"
        string status "pending/running/done/failed"
        json result "任务结果"
        timestamp created_at
        timestamp updated_at
    }

    SubTask {
        string id PK
        string task_id FK
        string name "子任务名称"
        string description "子任务描述"
        string status "queued/running/completed/failed"
        string agent_type "researcher/developer/executor"
        json context "代理上下文"
        timestamp created_at
        timestamp updated_at
    }

    Action {
        string id PK
        string subtask_id FK
        string type "command/search/analyze/etc"
        string status "success/failure"
        json parameters "动作参数"
        json result "动作结果"
        timestamp created_at
    }

    Artifact {
        string id PK
        string action_id FK
        string type "file/report/log"
        string path "存储路径"
        json metadata "附加信息"
        timestamp created_at
    }

    Memory {
        string id PK
        string action_id FK
        string type "observation/conclusion"
        vector embedding "向量表示"
        text content "记忆内容"
        timestamp created_at
    }
```

**实体说明**：

| 实体 | 描述 | 关键属性 |
|------|------|----------|
| Flow | 完整的渗透测试流程 | 名称、描述、状态、参数 |
| Task | 流程中的一个主要任务 | 名称、描述、状态、结果 |
| SubTask | 任务分解的子任务 | 名称、状态、代理类型、上下文 |
| Action | 子任务执行的具体动作 | 类型、状态、参数、结果 |
| Artifact | 动作产生的文件/报告 | 类型、路径、元数据 |
| Memory | 存储的记忆/学习内容 | 类型、内容、向量化表示 |

## 四、记忆系统架构

PentAGI的智能记忆系统是其核心竞争优势之一，通过多层记忆架构实现持续学习和知识积累。

### 4.1 记忆系统架构图

```mermaid
graph TB
    subgraph "长期记忆"
        VS[(向量存储
        嵌入数据库)]
        KB[知识库
        领域专业知识]
        Tools[工具知识
        使用模式]
    end

    subgraph "工作记忆"
        Context[当前上下文
        任务状态]
        Goals[活跃目标
         objectives]
        State[系统状态
        资源]
    end

    subgraph "情景记忆"
        Actions[历史动作
        命令历史]
        Results[动作结果
        输出]
        Patterns[成功模式
        最佳实践]
    end

    Context --> |查询| VS
    VS --> |检索| Context

    Goals --> |咨询| KB
    KB --> |指导| Goals

    State --> |记录| Actions
    Actions --> |学习| Patterns
    Patterns --> |存储| VS

    Tools --> |通知| State
    Results --> |更新| Tools

    VS --> |增强| KB
    KB --> |索引| VS

    classDef 长期 fill:#f9f,stroke:#333,stroke-width:2px,color:#000
    classDef 工作 fill:#bbf,stroke:#333,stroke-width:2px,color:#000
    classDef 情景 fill:#bfb,stroke:#333,stroke-width:2px,color:#000

    class VS,KB,Tools 长期
    class Context,Goals,State 工作
    class Actions,Results,Patterns 情景
```

### 4.2 链式摘要系统

PentAGI使用链式摘要技术管理不断增长的对话上下文，防止超出LLM的令牌限制：

```mermaid
flowchart TD
    A[输入链] --> B{需要摘要?}
    B -->|否| C[返回原始链]
    B -->|是| D[转换为ChainAST]
    D --> E[应用分段摘要]
    E --> F[处理过大配对]
    F --> G[管理最后分段大小]
    G --> H[应用问答摘要]
    H --> I[重建链与摘要]
    I --> J{新链更小?}
    J -->|是| K[返回优化链]
    J -->|否| C

    classDef 处理 fill:#bbf,stroke:#333,stroke-width:2px,color:#000
    classDef 决策 fill:#bfb,stroke:#333,stroke-width:2px,color:#000
    classDef 输出 fill:#fbb,stroke:#333,stroke-width:2px,color:#000

    class A,D,E,F,G,H,I 处理
    class B,J 决策
    class C,K 输出
```

**摘要配置参数**：

| 参数 | 环境变量 | 默认值 | 说明 |
|------|----------|--------|------|
| 保留最后 | SUMMARIZER_PRESERVE_LAST | true | 是否保留最后部分的所有消息 |
| 使用问答对 | SUMMARIZER_USE_QA | true | 是否使用问答对摘要策略 |
| 最后分段大小 | SUMMARIZER_LAST_SEC_BYTES | 51200 | 最后分段的最大字节数(50KB) |
| 最大体配对大小 | SUMMARIZER_MAX_BP_BYTES | 16384 | 单个体配对的最大字节数(16KB) |
| 最大问答分段 | SUMMARIZER_MAX_QA_SECTIONS | 10 | 保留的最大问答分段数 |

## 五、部署架构

PentAGI支持多种部署模式，从简单的单节点部署到生产环境推荐的双节点架构。

### 5.1 单节点部署

适用于开发和测试环境，所有服务部署在单一服务器上：

```mermaid
graph LR
    subgraph 服务器
        Docker[Docker Engine]

        subgraph PentAGI网络
            UI[前端UI:8443]
            API[后端API:8080]
            DB[(PostgreSQL)]
            Agent[AI代理]
        end

        subgraph 监控网络
            Grafana[Grafana:3000]
            Langfuse[Langfuse:4000]
        end

        DockerSocket[Docker Socket]
    end

    User --> |HTTPS| UI
    UI --> API
    API --> DB
    API --> Agent
    Agent --> DockerSocket
```

### 5.2 双节点部署（推荐生产环境）

将工作节点隔离在独立服务器上，提供更强的安全边界：

```mermaid
graph TB
    subgraph 主节点
        Control[控制平面]
        DB[(数据库)]
        Monitor[监控服务]
    end

    subgraph 工作节点
        Worker[工作节点]
        Pentest[渗透测试容器]
        Docker[Docker in Docker]
    end

    Control --> |API| Worker
    Worker --> |执行| Pentest
    Pentest --> |容器| Docker

    style 主节点 fill:#e1f5fe
    style 工作节点 fill:#fce4ec
```

**双节点架构优势**：

- 隔离执行：工作容器运行在专用硬件上
- 网络隔离：渗透测试的网络边界分离
- 安全边界：Docker-in-Docker与TLS认证
- OOB攻击支持：专用的外带技术端口范围

## 六、监控与可观测性

PentAGI提供全面的监控和可观测性支持，帮助用户了解系统状态和AI代理行为。

### 6.1 监控数据流

```mermaid
flowchart LR
    subgraph 采集层
        OTEL[OpenTelemetry]
        App[应用程序]
        System[系统指标]
    end

    subgraph 存储层
        VM[VictoriaMetrics]
        Jaeger[Jaeger]
        Loki[Loki]
    end

    subgraph 展示层
        Grafana[Grafana仪表板]
    end

    OTEL --> |指标| VM
    OTEL --> |追踪| Jaeger
    OTEL --> |日志| Loki

    VM --> Grafana
    Jaeger --> Grafana
    Loki --> Grafana
```

### 6.2 Langfuse集成

Langfuse提供AI模型的可观测性，包括追踪、评估和调试：

```mermaid
flowchart TB
    subgraph PentAGI
        Agent[AI代理]
    end

    subgraph Langfuse
        Web[Web界面:3000]
        API[API服务器]
        ClickHouse[(ClickHouse)]
        Redis[(Redis)]
        MinIO[(MinIO)]
    end

    Agent --> |追踪数据| API
    API --> Web
    API --> |存储| ClickHouse
    API --> |缓存| Redis
    API --> |文件| MinIO
```

## 七、总结

PentAGI的架构设计体现了现代化云原生应用的最佳实践：

- **微服务架构**：各组件独立部署，易于扩展和维护
- **容器化隔离**：所有操作在隔离的容器中执行，确保安全
- **多代理协作**：专业化的AI代理协同工作，提高测试效率
- **智能记忆**：多层记忆系统实现持续学习和知识复用
- **全面监控**：端到端的可观测性支持问题诊断和性能优化
- **灵活部署**：支持从开发环境到生产环境的多种部署模式

这种架构设计使PentAGI成为一个强大、灵活且安全的自动化渗透测试平台。