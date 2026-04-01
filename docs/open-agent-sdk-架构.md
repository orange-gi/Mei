# Open Agent SDK 架构文档

## 1. 系统整体架构

### 1.1 核心架构概述

Open Agent SDK 采用进程内架构，将完整的 Claude Code 引擎直接嵌入到应用程序中。架构分为三个主要层次：入口层、引擎层和服务层。

```mermaid
flowchart TB
    subgraph Entry["入口层 (Entrypoint)"]
        SDK["SDK API<br/>createAgent / query"]
        CLI["CLI 入口<br/>codeany 命令"]
    end

    subgraph Engine["引擎层 (Core Engine)"]
        Agent["Agent 类<br/>状态管理、会话控制"]
        QueryEngine["QueryEngine<br/>核心查询循环"]
        QueryLoop["Query Loop<br/>多轮对话循环"]
    end

    subgraph Services["服务层 (Services)"]
        API["API Client<br/>Anthropic API 调用"]
        Tools["Tool System<br/>26 个内置工具"]
        MCP["MCP Client<br/>MCP 协议支持"]
        Memory["Memory System<br/>自动记忆管理"]
        Compact["Context Compact<br/>上下文压缩"]
        Permission["Permission System<br/>权限控制"]
    end

    subgraph External["外部组件"]
        Anthropic["Anthropic API"]
        MCPServers["MCP Servers"]
        FileSystem["文件系统"]
    end

    SDK --> Agent
    CLI --> Agent
    Agent --> QueryEngine
    QueryEngine --> QueryLoop
    QueryLoop <--> API
    QueryLoop <--> Tools
    QueryLoop <--> Memory
    QueryLoop <--> Compact
    QueryLoop <--> Permission
    API <--> Anthropic
    QueryLoop <--> MCP
    MCP <--> MCPServers
    Tools <--> FileSystem

    style Entry fill:#e1f5fe
    style Engine fill:#fff3e0
    style Services fill:#e8f5e9
    style External fill:#fce4ec
```

### 1.2 模块目录结构

项目源代码位于 `src/` 目录，包含多个功能模块：

- **agent.ts**: Agent 主类，状态管理
- **query.ts**: 核心查询循环引擎
- **Tool.ts**: 工具接口定义
- **tools/**: 26 个内置工具实现
- **services/**: API 调用、记忆管理等服务
- **mcp/**: MCP 协议客户端
- **context/**: 上下文管理系统
- **state/**: 状态管理
- **hooks/**: 权限和生命周期钩子
- **components/**: UI 组件

## 2. 核心数据流

### 2.1 Agent 查询流程

```mermaid
sequenceDiagram
    participant User as 用户代码
    participant Agent as Agent 类
    participant QueryEngine as QueryEngine
    participant QueryLoop as Query Loop
    participant Tools as Tool System
    participant API as Anthropic API
    participant Memory as Memory System

    User->>Agent: createAgent(options)
    Agent->>Agent: 初始化状态、工具、MCP
    User->>Agent: agent.prompt(query)
    Agent->>QueryEngine: ask({ prompt, tools, ... })
    QueryEngine->>QueryLoop: 启动查询循环

    loop 多轮对话循环
        QueryLoop->>Memory: 加载相关记忆
        Note over QueryLoop: 构建请求消息
        QueryLoop->>API: callModel(messages, tools)
        API-->>QueryLoop: 流式响应

        alt 工具调用
            loop 工具执行
                QueryLoop->>Tools: 执行工具
                Tools-->>QueryLoop: 工具结果
            end
        end

        QueryLoop->>Memory: 保存上下文快照
    end

    QueryLoop-->>Agent: 完成响应
    Agent-->>User: QueryResult
```

### 2.2 工具执行流程

工具执行支持并发和串行两种模式。

```mermaid
flowchart LR
    subgraph Input["输入处理"]
        Request["工具请求"]
        Validate["输入验证"]
        Permission["权限检查"]
    end

    subgraph Execution["执行层"]
        ReadOnly["只读工具<br/>并发执行"]
        WriteOp["写操作工具<br/>串行执行"]
    end

    subgraph Output["输出处理"]
        Result["结果转换"]
        Render["渲染消息"]
    end

    Request --> Validate
    Validate --> Permission

    Permission -->|isConcurrencySafe| ReadOnly
    Permission -->|需要串行| WriteOp

    ReadOnly --> Result
    WriteOp --> Result

    Result --> Render

    style Input fill:#e3f2fd
    style Execution fill:#fff8e1
    style Output fill:#e8f5e9
```

### 2.3 MCP 集成数据流

MCP（Model Context Protocol）允许连接外部工具和服务。

```mermaid
flowchart TB
    subgraph Agent["Agent 内部"]
        MCPLoader["MCP Client<br/>连接管理器"]
        ToolPool["Tool Pool<br/>工具池"]
        Request["请求分发"]
    end

    subgraph Protocol["MCP 协议层"]
        STDIO["stdio 传输"]
        SSE["SSE 传输"]
        HTTP["HTTP 传输"]
    end

    subgraph Servers["MCP 服务器"]
        FS["文件系统服务器"]
        Web["Web 搜索服务器"]
        Custom["自定义服务器"]
    end

    MCPLoader --> Protocol
    ToolPool --> Request
    Request --> MCPLoader

    Protocol --> STDIO
    Protocol --> SSE
    Protocol --> HTTP

    STDIO --> FS
    SSE --> Web
    HTTP --> Custom

    style Agent fill:#e1f5fe
    style Protocol fill:#fff3e0
    style Servers fill:#e8f5e9
```

## 3. 类/模块关系图

### 3.1 Agent 核心类图

Agent 类是 SDK 的主要入口。

```mermaid
classDiagram
    class Agent {
        +options: AgentOptions
        +appState: AppState
        +readFileCache: FileStateCache
        +mutableMessages: Message[]
        +tools: Tools
        +resolvedModel: string
        +mcpClients: MCPServerConnection[]

        +createAgent(options): Agent
        +query(prompt): AsyncGenerator~SDKMessage~
        +prompt(text): Promise~QueryResult~
        +getMessages(): Message[]
        +clear(): void
        -_init(): Promise~void~
        -resolveEnvOptions(): void
    }

    class QueryEngine {
        +ask(params): AsyncGenerator~SDKMessage~
    }

    class Tool {
        +name: string
        +inputSchema: z.ZodType
        +call(args, context): Promise~ToolResult~
        +description(): Promise~string~
        +isReadOnly(): boolean
        +isConcurrencySafe(): boolean
    }

    Agent --> QueryEngine : uses
    Agent --> Tool : uses
```

### 3.2 工具系统架构

工具系统采用策略模式，每个工具实现统一的 Tool 接口。

```mermaid
classDiagram
    class Tool~Input, Output~ {
        <<interface>>
        +name: string
        +inputSchema: Input
        +call(args, context): ToolResult~Output~
        +description(): string
        +isReadOnly(): boolean
        +isConcurrencySafe(): boolean
    }

    class ReadTool {
        +name = "Read"
    }

    class WriteTool {
        +name = "Write"
    }

    class EditTool {
        +name = "Edit"
    }

    class BashTool {
        +name = "Bash"
    }

    Tool <|.. ReadTool
    Tool <|.. WriteTool
    Tool <|.. EditTool
    Tool <|.. BashTool
```

### 3.3 权限系统类图

权限系统实现四层验证管道。

```mermaid
classDiagram
    class PermissionSystem {
        +checkPermission(tool, input): PermissionResult
        +validateInput(tool, input): ValidationResult
    }

    class RuleEngine {
        +alwaysAllowRules: ToolPermissionRulesBySource
        +alwaysDenyRules: ToolPermissionRulesBySource
        +evaluate(input): PermissionResult
    }

    class AIClassifier {
        +classifyRisk(tool, input): RiskLevel
        +circuitBreaker: CircuitBreaker
    }

    class PermissionContext {
        +mode: PermissionMode
        +additionalWorkingDirectories: Map
    }

    PermissionSystem --> RuleEngine
    PermissionSystem --> AIClassifier
    PermissionSystem --> PermissionContext
```

## 4. 用户交互流程

### 4.1 创建和使用 Agent 的完整流程

```mermaid
flowchart TB
    subgraph Setup["初始化阶段"]
        A[安装 SDK] --> B[设置 API Key]
        B --> C[createAgent]
        C --> D[初始化 MCP]
        D --> E[加载工具]
    end

    subgraph Usage["使用阶段"]
        E --> F{选择模式}
        F -->|流式| G[query 方法]
        F -->|阻塞| H[prompt 方法]
        G --> I[遍历消息]
        H --> J[等待结果]
        I --> K{消息类型}
        J --> K
        K -->|assistant| L[处理文本]
        K -->|tool_use| M[执行工具]
        K -->|result| N[完成任务]
        M --> O[工具结果]
        O --> G
        L --> G
    end

    subgraph Cleanup["清理阶段"]
        N --> P[释放资源]
        P --> Q[关闭 MCP]
    end

    style Setup fill:#e3f2fd
    style Usage fill:#fff8e1
    style Cleanup fill:#e8f5e9
```

### 4.2 子 Agent 协作流程

```mermaid
flowchart LR
    subgraph Main["主 Agent"]
        M1[接收任务] --> M2[分解任务]
        M2 --> M3[派生子 Agent]
        M3 --> M4[收集结果]
        M4 --> M5[整合响应]
    end

    subgraph Sub1["子 Agent 1"]
        S1A[代码审查] --> S1B[扫描代码]
        S1B --> S1C[发现问题]
        S1C --> S1D[返回报告]
    end

    subgraph Sub2["子 Agent 2"]
        S2A[文档生成] --> S2B[分析代码]
        S2B --> S2C[生成文档]
        S2C --> S2D[返回文档]
    end

    M3 -->|code-reviewer| Sub1
    M3 -->|doc-writer| Sub2
    S1D --> M4
    S2D --> M4

    style Main fill:#e1f5fe
    style Sub1 fill:#e8f5e9
    style Sub2 fill:#e8f5e9
```

## 5. 部署架构

### 5.1 典型部署场景

Open Agent SDK 支持多种部署场景。

```mermaid
flowchart TB
    subgraph Clients["客户端"]
        Web["Web 应用"]
        Mobile["移动应用"]
        Desktop["桌面应用"]
    end

    subgraph Deployment["部署场景"]
        Server["服务器部署"]
        Serverless["Serverless 函数"]
        Docker["Docker 容器"]
        K8s["Kubernetes Pod"]
        CI["CI/CD 流水线"]
    end

    subgraph Infrastructure["基础设施"]
        API["Anthropic API"]
        Cache["Redis/Memcached"]
        Storage["持久化存储"]
    end

    Web --> Server
    Mobile --> Server
    Desktop --> Server

    Server --> API
    Serverless --> API
    Docker --> API
    K8s --> API

    Serverless --> Cache
    Server --> Cache
    K8s --> Storage

    style Clients fill:#e1f5fe
    style Deployment fill:#fff8e1
    style Infrastructure fill:#e8f5e9
```

## 6. 上下文管理架构

### 6.1 上下文生命周期

```mermaid
flowchart TB
    subgraph Creation["创建阶段"]
        New[新建会话]
        Load[加载项目上下文]
        Inject[注入系统提示]
    end

    subgraph Maintenance["维护阶段"]
        Append[追加消息]
        Trim[剪裁过长消息]
        Compress[上下文压缩]
        Dream[后台记忆整理]
    end

    subgraph Optimization["优化阶段"]
        Microcompact["微压缩"]
        Autocompact["自动压缩"]
        Snip["剪裁压缩"]
        Collapse["上下文折叠"]
    end

    subgraph Cleanup["清理阶段"]
        Archive["归档旧消息"]
        Persist["持久化"]
    end

    New --> Load --> Inject --> Append
    Append --> Trim
    Trim --> Compress
    Compress --> Dream
    Dream --> Microcompact
    Microcompact --> Autocompact
    Autocompact --> Snip
    Snip --> Collapse
    Collapse --> Archive
    Archive --> Persist

    style Creation fill:#e3f2fd
    style Maintenance fill:#fff8e1
    style Optimization fill:#e8f5e9
    style Cleanup fill:#fce4ec
```

### 6.2 自动压缩流程

当上下文超过阈值时，系统自动进行压缩。

```mermaid
flowchart TB
    A[检测上下文长度] --> B{是否超过阈值?}
    B -->|否| Z[继续使用]
    B -->|是| C{检查压缩类型}

    C -->|微压缩| D[提取关键信息]
    C -->|自动压缩| E[生成摘要]
    C -->|剪裁压缩| F[删除低价值消息]

    D --> G[替换原始消息]
    E --> H[保留摘要+关键片段]
    F --> I[保留最近消息]

    G --> J[验证压缩后长度]
    H --> J
    I --> J

    J --> K{验证通过?}
    K -->|是| Z
    K -->|否| L[进一步压缩]

    L --> C

    style A fill:#e1f5fe
    style D fill:#e8f5e9
    style E fill:#e8f5e9
    style F fill:#e8f5e9
    style L fill:#ffcdd2
```

## 7. 错误处理和恢复

### 7.1 错误恢复机制

```mermaid
flowchart TB
    A[发生错误] --> B{错误类型}
    B -->|API 错误| C[重试逻辑]
    B -->|权限错误| D[请求权限]
    B -->|上下文过长| E[压缩重试]
    B -->|工具失败| F[降级处理]

    C --> G{重试次数}
    G -->|可重试| H[指数退避]
    G -->|不可重试| I[报告错误]

    D --> J{用户响应}
    J -->|授权| K[重试操作]
    J -->|拒绝| I

    E --> L[压缩成功?]
    L -->|是| M[继续]
    L -->|否| I

    F --> N[记录日志]
    N --> O[返回部分结果]

    style I fill:#ffcdd2
    style M fill:#e8f5e9
```

## 总结

Open Agent SDK 的架构设计体现了以下核心原则：

1. **模块化**：各组件职责清晰，易于测试和维护
2. **可扩展性**：支持自定义工具和 MCP 集成
3. **容错性**：完善的错误处理和恢复机制
4. **性能优化**：上下文压缩、并发执行等优化策略
5. **安全性**：四层权限管道确保操作安全