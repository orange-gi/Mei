# Supermemory 架构图集

## 系统整体架构图

### 整体架构概览

Supermemory采用现代化的单体仓库（Monorepo）架构设计，由Turborepo进行统一的构建管理和发布编排。整个系统由多个独立的应用程序（Apps）和共享功能包（Packages）组成，形成清晰的分层结构和模块边界。这种架构既保证了开发效率（代码共享、类型统一），又维护了各应用之间的独立性和可部署性。

```mermaid
flowchart TB
    subgraph 用户层["用户接入层"]
        BrowserExt["浏览器扩展"]
        RaycastExt["Raycast扩展"]
        WebApp["Web应用"]
        MCPClient["MCP客户端"]
    end

    subgraph 应用层["应用服务层"]
        WebServer["Web服务器\nRemix + Hono"]
        MCPServer["MCP服务器\nCloudflare Workers"]
        DocsServer["文档服务器"]
        GraphPlayground["图谱 Playground"]
    end

    subgraph 共享包层["共享功能包"]
        UI["UI组件库\n@supermemory/ui"]
        Lib["核心库\n@supermemory/lib"]
        AI["AI SDK\n@supermemory/ai-sdk"]
        MemoryGraph["记忆图谱\n@supermemory/memory-graph"]
        Hooks["React Hooks\n@supermemory/hooks"]
        Tools["工具函数\n@supermemory/tools"]
        Validation["数据验证\n@supermemory/validation"]
    end

    subgraph 基础设施层["基础设施层"]
        PG["PostgreSQL\nDrizzle ORM"]
        CF_KV["Cloudflare KV"]
        CF_DO["Cloudflare Durable Objects\n向量引擎"]
        AI_Services["AI服务商\nOpenAI/Anthropic/Google"]
        Auth["认证服务\nbetter-auth"]
        Storage["对象存储"]
    end

    用户层 --> 应用层
    应用层 --> 共享包层
    共享包层 --> 基础设施层
```

### 技术栈分布图

不同层次的技术选型反映了Supermemory在性能和开发效率之间的平衡考量。前端采用Remix框架充分利用服务端渲染的优势，后端服务部署在Cloudflare Workers上实现边缘计算能力，数据库层选择PostgreSQL保证数据一致性和查询能力。

```mermaid
flowchart LR
    subgraph Frontend["前端技术栈"]
        F1["Remix"]
        F2["React"]
        F3["TailwindCSS"]
        F4["Vanilla Extract"]
    end

    subgraph Backend["后端技术栈"]
        B1["Hono"]
        B2["Cloudflare Workers"]
        B3["Cloudflare Pages"]
    end

    subgraph Database["数据层"]
        D1["PostgreSQL"]
        D2["Drizzle ORM"]
        D3["Cloudflare KV"]
        D4["Durable Objects\n向量存储"]
    end

    subgraph AI["AI集成"]
        A1["Vercel AI SDK"]
        A2["OpenAI SDK"]
        A3["Anthropic SDK"]
        A4["Google GenAI"]
    end

    subgraph Infrastructure["基础设施"]
        I1["Cloudflare"]
        I2["PostgreSQL"]
        I3["Sentry"]
        I4["PostHog"]
    end

    Frontend --> Backend
    Backend --> Database
    Backend --> AI
    Infrastructure --> Frontend
    Infrastructure --> Backend
```

## 核心数据流图

### 记忆添加流程

用户添加记忆内容时，系统会经过一系列处理步骤将原始内容转换为可检索的向量表示。这一流程涉及内容解析、向量化、索引存储等多个环节，每个环节都经过优化以确保处理效率和存储空间的合理利用。

```mermaid
sequenceDiagram
    participant User as 用户
    participant Web as Web应用
    participant Parse as 内容解析器
    participant Vector as 向量引擎
    participant DB as PostgreSQL
    participant KV as Cloudflare KV

    User->>Web: 添加记忆（文本/链接/文件）
    Web->>Parse: 解析原始内容
    Parse-->>Web: 提取纯文本和元数据

    Note over Web,Vector: 向量化处理
    Web->>Vector: 调用嵌入模型生成向量
    Vector-->>Web: 返回向量 embedding

    Note over DB,KV: 存储阶段
    Web->>DB: 保存原始内容、元数据、时间戳
    Web->>DB: 存储向量ID关联记录
    Web->>KV: 缓存向量数据

    Web-->>User: 添加完成确认
```

### 记忆检索与对话流程

当用户与记忆库进行对话时，系统需要协调向量检索、提示词构造、AI模型调用等多个组件。这一流程的设计目标是最大化检索相关性、最小化响应延迟，同时控制token使用成本。

```mermaid
sequenceDiagram
    participant User as 用户
    participant Chat as 对话服务
    participant Vector as 向量引擎
    participant DB as PostgreSQL
    participant LLM as AI模型
    participant Context as 上下文构造

    User->>Chat: 发送问题/查询
    Chat->>Vector: 计算查询向量
    Vector-->>Chat: 返回Top-K相似向量

    Loop 检索结果处理
        Chat->>DB: 获取向量对应原文
        DB-->>Chat: 返回原始内容片段
    end

    Chat->>Context: 构造系统提示词
    Context->>Context: 插入记忆片段
    Context->>Context: 添加用户设定

    Context->>LLM: 发送完整提示词
    LLM-->>Context: 返回生成结果
    Context-->>Chat: 返回带引用的回答

    Chat-->>User: 展示回答和引用来源
```

### MCP工具调用流程

当AI工具（如Claude Desktop、Cursor）通过MCP协议调用Supermemory时，系统需要处理工具调用请求、验证权限、执行操作并返回结果。这一流程遵循MCP协议规范，确保与各种AI客户端的兼容性。

```mermaid
sequenceDiagram
    participant AI as AI客户端
    participant MCP as MCP服务器
    participant Auth as 认证服务
    participant Tool as 工具处理器
    participant Vector as 向量引擎
    participant DB as 数据库

    AI->>MCP: 工具调用请求
    MCP->>Auth: 验证用户身份

    alt 未认证
        Auth-->>MCP: 返回认证错误
        MCP-->>AI: 要求认证
    else 已认证
        Auth-->>MCP: 确认用户ID

        Note over MCP,Tool: 分发到对应工具
        MCP->>Tool: 路由到指定工具

        alt search_memories
            Tool->>Vector: 执行向量检索
            Vector-->>Tool: 返回记忆片段
            Tool->>Tool: 格式化结果
        else add_memory
            Tool->>Tool: 解析请求内容
            Tool->>DB: 存储新记忆
            Tool->>Vector: 生成向量索引
        else getProfile
            Tool->>DB: 查询用户配置
            Tool-->>Tool: 格式化偏好设置
        end

        Tool-->>MCP: 返回处理结果
        MCP-->>AI: 响应工具调用结果
    end
```

## 类与模块关系图

### 核心模块依赖关系

Supermemory的代码组织遵循清晰的依赖规则：应用程序（Apps）依赖共享功能包（Packages），Packages之间也存在明确的依赖关系。这种分层架构确保了代码的高内聚低耦合，便于维护和测试。

```mermaid
flowchart TB
    subgraph Apps["应用程序"]
        WebApp["web\nRemix应用"]
        BrowserExt["browser-extension\n浏览器扩展"]
        MCPServer["mcp\nMCP服务器"]
        Docs["docs\n文档站点"]
        Playground["memory-graph-playground\n图谱实验"]
        Raycast["raycast-extension\nRaycast扩展"]
    end

    subgraph Packages["共享包"]
        UI["ui\nUI组件库"]
        Lib["lib\n核心功能库"]
        AI["ai-sdk\nAI集成SDK"]
        MemoryGraph["memory-graph\n记忆图谱"]
        Hooks["hooks\nReact Hooks"]
        Tools["tools\n工具函数"]
        Validation["validation\n数据验证"]
    end

    WebApp --> UI
    WebApp --> Lib
    WebApp --> AI
    WebApp --> MemoryGraph
    WebApp --> Hooks

    BrowserExt --> UI
    BrowserExt --> Lib

    MCPServer --> AI
    MCPServer --> MemoryGraph
    MCPServer --> Lib

    Docs --> UI

    AI --> Lib
    MemoryGraph --> Lib
    Hooks --> Lib
    Tools --> Lib
    Validation --> Lib
```

### 数据库实体关系图

Supermemory的数据模型围绕「记忆」核心概念构建，包含用户、会话、记忆内容、向量索引等实体。实体之间通过外键关联形成完整的数据图谱，支持复杂的查询和检索需求。

```mermaid
erDiagram
    User ||--o{ Memory : creates
    User ||--o{ Session : has
    User ||--o{ Tag : owns

    Memory ||--o{ MemoryVector : has
    Memory ||--o{ MemoryTag : tagged_with
    Memory ||--o{ Source : originates_from

    Session ||--o{ Message : contains

    MemoryVector }o--|| VectorEngine : stored_in

    User {
        uuid id PK
        string email
        string name
        json preferences
        timestamp created_at
    }

    Memory {
        uuid id PK
        uuid user_id FK
        string content
        string source_type
        string source_url
        json metadata
        boolean is_archived
        timestamp created_at
    }

    Session {
        uuid id PK
        uuid user_id FK
        string title
        timestamp created_at
    }

    Message {
        uuid id PK
        uuid session_id FK
        string role
        string content
        json memory_references
        timestamp created_at
    }

    MemoryVector {
        uuid id PK
        uuid memory_id FK
        vector embedding
        string model_name
        int token_count
    }

    Tag {
        uuid id PK
        uuid user_id FK
        string name
        string color
    }
```

## 用户交互流程图

### Web应用用户旅程

用户首次使用Supermemory Web应用的典型交互流程涵盖账号创建、内容添加、对话检索等核心环节。每个环节都经过精心设计，确保用户能够快速上手并获得价值。

```mermaid
flowchart TD
    Start([开始]) --> Login{已有账号?}

    Login -->|是| SignIn[登录账号]
    Login -->|否| SignUp[创建账号]

    SignIn --> Dashboard[进入主界面]
    SignUp --> Onboarding[引导流程]
    Onboarding --> Dashboard

    Dashboard --> AddMemory[添加记忆]
    AddMemory -->|文本| InputText[输入文本内容]
    AddMemory -->|链接| FetchUrl[抓取网页内容]
    AddMemory -->|文件| UploadFile[上传文件]

    InputText --> Process[内容处理]
    FetchUrl --> Process
    UploadFile --> Process

    Process --> Vectorize[向量化存储]
    Vectorize --> Success[添加成功]

    Success --> SearchMemories{需要检索?}

    SearchMemories -->|是| OpenChat[打开对话界面]
    OpenChat --> Chat[输入问题]
    Chat --> Retrieve[检索记忆]
    Retrieve --> Generate[AI生成回答]
    Generate --> ShowResult[展示结果]
    ShowResult --> MoreChat{继续对话?}

    MoreChat -->|是| Chat
    MoreChat -->|否| BackToDashboard[返回主界面]

    SearchMemories -->|否| BackToDashboard

    BackToDashboard --> End([结束])
```

### 浏览器扩展交互流程

浏览器扩展提供了一种更加便捷的记忆保存方式，用户可以在浏览任意网页时快速捕捉感兴趣的内容。这一流程最大化了「即时捕获」的用户体验，减少了信息流失的可能性。

```mermaid
flowchart LR
    subgraph Extension["浏览器扩展交互"]
        A[浏览网页] --> B{发现感兴趣内容?}
        B -->|否| A
        B -->|是| C[点击扩展图标]

        C --> D[选择保存方式]
        D --> E{保存类型?}

        E -->|整个页面| F[自动抓取页面内容]
        E -->|选定内容| G[获取选定文本]
        E -->|右键菜单| H[右键菜单保存]

        F --> I[显示预览界面]
        G --> I
        H --> I

        I --> J[补充标签和描述]
        J --> K[确认保存]

        K --> L[上传到Supermemory]
        L --> M[显示保存成功]
        M --> N{继续浏览?}

        N -->|是| A
        N -->|否| O[关闭扩展]
    end
```

### MCP集成配置流程

将Supermemory集成到外部AI工具中需要完成MCP服务器的配置。这一流程涉及获取配置信息、在AI工具中添加服务器、验证连接成功等步骤。

```mermaid
flowchart TD
    Start([开始配置MCP]) --> Login[登录Supermemory]
    Login --> Navigate[进入「Connect to AI」页面]

    Navigate --> Select[选择目标AI工具]

    Select -->|Claude Desktop| Step1[复制MCP服务器URL]
    Select -->|Cursor| Step2[获取JSON配置]
    Select -->|Windsurf| Step3[获取服务器地址]
    Select -->|VS Code| Step4[配置MCP设置]

    Step1 --> OpenApp[打开AI工具设置]
    Step2 --> OpenApp
    Step3 --> OpenApp
    Step4 --> OpenApp

    OpenApp --> AddServer[添加MCP服务器]
    AddServer --> Paste[粘贴配置信息]
    Paste --> Save[保存配置]

    Save --> Connect[建立连接]
    Connect --> Verify{连接成功?}

    Verify -->|是| Test[测试工具调用]
    Verify -->|否| Retry[检查配置重试]

    Test --> Search[执行搜索测试]
    Search --> Result[确认返回记忆]
    Result --> Success([配置完成])

    Retry --> AddServer
```

## 部署架构图

### 生产环境部署架构

Supermemory的生产环境采用多区域部署策略，充分利用Cloudflare的全球边缘网络提供低延迟服务访问。核心数据存储在PostgreSQL数据库中，AI相关计算通过API调用外部服务完成，实现了计算与存储的分离。

```mermaid
flowchart TB
    subgraph Global["Cloudflare全球网络"]
        subgraph Edge["边缘节点"]
            Workers1["Web Workers\nWeb应用"]
            Workers2["MCP Workers\nMCP服务"]
            Workers3["API Workers\nREST API"]
        end

        subgraph Storage["分布式存储"]
            KV["KV存储\n键值缓存"]
            DO["Durable Objects\n向量引擎"]
            R2["R2存储\n文件对象"]
        end
    end

    subgraph Data["数据中心"]
        PG["PostgreSQL\n主数据库"]
        Replica["只读副本\n读写分离"]
    end

    subgraph External["外部服务"]
        OpenAI["OpenAI API"]
        Anthropic["Anthropic API"]
        Google["Google AI"]
    end

    subgraph Users["用户端"]
        Browser["浏览器"]
        Desktop["桌面应用"]
        Mobile["移动设备"]
    end

    Users --> Cloudflare

    Cloudflare --> Workers1
    Cloudflare --> Workers2
    Cloudflare --> Workers3

    Workers1 --> KV
    Workers1 --> DO
    Workers2 --> DO
    Workers3 --> PG

    PG --> Replica

    Workers1 --> OpenAI
    Workers2 --> Anthropic
    Workers3 --> Google

    DO --> R2
```

### 开发环境架构

开发环境采用与生产环境相似的架构，但使用本地服务和模拟组件替代云端资源。这一设计确保开发者在本地能够完成大部分开发和测试工作，同时保持代码在环境间的一致性。

```mermaid
flowchart TB
    subgraph Dev["开发环境"]
        subgraph Local["本地服务"]
            Node["Node.js\n开发服务器"]
            DB_Local["PostgreSQL\n本地数据库"]
            Minio["MinIO\n对象存储模拟"]
        end

        subgraph Config["配置管理"]
            Env[".env.local\n环境变量"]
            Turbo["Turborepo\n构建管理"]
        end
    end

    subgraph Remote["远程开发资源"]
        CF_Dev["Cloudflare Wrangler\n本地模拟"]
        Docker["Docker\n容器服务"]
    end

    subgraph External["外部服务"]
        AI_API["AI API\n代理/直连"]
    end

    Developer["开发者"] --> Local
    Local --> Remote
    Local --> External
```

### CI/CD流水线架构

Supermemory采用自动化的持续集成和持续部署流程，确保代码变更能够快速、安全地部署到生产环境。流水线涵盖代码检查、构建测试、预览部署、生产发布等多个阶段。

```mermaid
flowchart LR
    subgraph Source["代码管理"]
        Git["GitHub仓库"]
        Branch["分支策略\nmain/develop/feature"]
    end

    subgraph CI["持续集成"]
        Check["代码检查\nBiome Lint"]
        Test["单元测试"]
        Type["类型检查\nTypeScript"]
        Build["应用构建"]
    end

    subgraph CD["持续部署"]
        Preview["预览环境\nCloudflare Pages"]
        Staging["预发布环境"]
        Production["生产环境"]
    end

    subgraph Monitor["监控反馈"]
        Sentry["错误追踪"]
        Analytics["使用分析"]
        Alert["告警通知"]
    end

    Git --> Check
    Check --> Test
    Test --> Type
    Type --> Build

    Build --> Preview
    Preview -->|通过| Staging
    Staging -->|验证| Production

    Production --> Monitor
    Monitor -->|反馈| Git
```

---

以上架构图集全面展示了Supermemory系统的设计理念和技术实现细节。从整体架构到模块关系，从数据流到用户交互，从开发环境到生产部署，每个层面都经过精心设计以支持系统的极速、可扩展目标。
