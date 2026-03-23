# DeerFlow 架构详解

## 系统整体架构

DeerFlow 采用分层架构设计，将智能体运行时与应用程序层严格分离，确保核心框架的可复用性和稳定性。

```mermaid
graph TB
    subgraph 前端层["前端层"]
        FE[Next.js Web UI<br/>端口 3000]
    end

    subgraph 网关层["网关层"]
        NGINX[Nginx 反向代理<br/>端口 2026]
    end

    subgraph 应用层["应用层 (app/)"]
        GATEWAY[Gateway API<br/>FastAPI REST<br/>端口 8001]
        CHANNELS[IM 渠道服务<br/>Telegram/Slack/飞书]
    end

    subgraph 核心层["核心层 (harness/)"]
        LG[LangGraph Server<br/>端口 2024]

        subgraph AGENT["智能体系统"]
            LEAD[Lead Agent<br/>主智能体]
            MIDDLEWARE[中间件链<br/>10 个组件]
            SUBAGENTS[子智能体系统<br/>并行执行]
        end

        subgraph TOOLS["工具生态"]
            SANDBOX[沙箱工具]
            BUILTIN[内置工具]
            MCP[MCP 协议工具]
            COMMUNITY[社区工具]
        end

        subgraph MEMORY["记忆系统"]
            EXTRACT[记忆提取]
            QUEUE[更新队列]
            STORAGE[持久化存储]
        end
    end

    subgraph 沙箱层["隔离执行层"]
        DOCKER[Docker 容器]
        LOCAL[本地文件系统]
        K8S[Kubernetes Pods]
    end

    FE --> NGINX
    NGINX -->|"/api/langgraph/*"| LG
    NGINX -->|"/api/*"| GATEWAY
    NGINX -->|"/*"| FE

    GATEWAY --> CHANNELS
    GATEWAY --> LG

    LG --> LEAD
    LEAD --> MIDDLEWARE
    LEAD --> SUBAGENTS
    LEAD --> TOOLS
    LEAD --> MEMORY

    SANDBOX --> DOCKER
    SANDBOX --> LOCAL
    SANDBOX --> K8S
```

---

## 核心数据流

### 用户请求处理流程

当用户发送一条消息时，系统会经过以下处理流程：

```mermaid
sequenceDiagram
    participant USER as 用户
    participant NGINX as Nginx
    participant LG as LangGraph Server
    participant MIDDLEWARE as 中间件链
    participant AGENT as Lead Agent
    participant SANDBOX as 沙箱
    participant SUBAGENT as 子智能体
    participant MEMORY as 记忆系统

    USER->>NGINX: 发送消息
    NGINX->>LG: 路由到 LangGraph

    Note over LG: 创建/获取 Thread

    LG->>MIDDLEWARE: 依次执行 10 个中间件
    MIDDLEWARE->>SANDBOX: 4. SandboxMiddleware<br/>获取隔离执行环境

    MIDDLEWARE->>MEMORY: 7. MemoryMiddleware<br/>注入用户记忆上下文

    MIDDLEWARE->>AGENT: 中间件处理完成

    AGENT->>AGENT: LLM 推理决策

    alt 需要子智能体
        AGENT->>SUBAGENT: 委托并行任务
        SUBAGENT->>SANDBOX: 子智能体执行
        SANDBOX-->>SUBAGENT: 返回结果
        SUBAGENT-->>AGENT: 汇总结果
    end

    alt 需要文件操作
        AGENT->>SANDBOX: 读写文件
        SANDBOX-->>AGENT: 返回文件内容
    end

    AGENT-->>MIDDLEWARE: 生成响应
    MIDDLEWARE->>MEMORY: 8. MemoryMiddleware<br/>排队更新记忆

    LG-->>NGINX: SSE 流式响应
    NGINX-->>USER: 实时流式输出
```

---

## 中间件链详解

DeerFlow 的 Lead Agent 包含 10 个按顺序执行的中间件，每个中间件负责特定的横切关注点：

```mermaid
flowchart LR
    subgraph 中间件链["中间件执行顺序"]
        direction TB
        M1[1. ThreadDataMiddleware<br/>线程数据初始化]
        M2[2. UploadsMiddleware<br/>文件注入]
        M3[3. SandboxMiddleware<br/>沙箱获取]
        M4[4. DanglingToolCallMiddleware<br/>工具调用补全]
        M5[5. SummarizationMiddleware<br/>上下文压缩]
        M6[6. TodoListMiddleware<br/>任务清单跟踪]
        M7[7. TitleMiddleware<br/>标题生成]
        M8[8. MemoryMiddleware<br/>记忆管理]
        M9[9. ViewImageMiddleware<br/>图像注入]
        M10[10. ClarificationMiddleware<br/>澄清拦截]
    end

    INPUT[用户输入] --> M1
    M1 --> M2
    M2 --> M3
    M3 --> M4
    M4 --> M5
    M5 --> M6
    M6 --> M7
    M7 --> M8
    M8 --> M9
    M9 --> M10
    M10 --> OUTPUT[LLM 推理]
```

### 中间件职责说明

| 序号 | 中间件名称 | 核心功能 |
|-----|----------|---------|
| 1 | ThreadDataMiddleware | 为每个线程创建隔离的目录结构（workspace、uploads、outputs） |
| 2 | UploadsMiddleware | 将上传的文件注入对话上下文 |
| 3 | SandboxMiddleware | 获取沙箱执行环境，存储 sandbox_id |
| 4 | DanglingToolCallMiddleware | 补全因用户中断而缺失的工具调用响应 |
| 5 | SummarizationMiddleware | 当接近 token 限制时压缩上下文（可选）|
| 6 | TodoListMiddleware | 在计划模式下跟踪多步骤任务（可选）|
| 7 | TitleMiddleware | 首轮交互后自动生成会话标题 |
| 8 | MemoryMiddleware | 将对话排队等待异步记忆更新 |
| 9 | ViewImageMiddleware | 为支持视觉的模型注入图像数据（条件触发）|
| 10 | ClarificationMiddleware | 拦截澄清请求并中断执行（必须最后）|

---

## 子智能体系统

### 子智能体执行架构

DeerFlow 支持复杂的并行任务分解，主智能体可以将任务委托给子智能体并行执行：

```mermaid
graph TB
    subgraph 主控["主智能体 (Lead Agent)"]
        DECIDER[任务分解器]
        COORD[结果协调器]
    end

    subgraph 线程池["执行线程池"]
        THREAD1[线程 1]
        THREAD2[线程 2]
        THREAD3[线程 3]
    end

    subgraph 子智能体["子智能体实例"]
        SA1["子智能体 1<br/>general-purpose"]
        SA2["子智能体 2<br/>bash"]
        SA3["子智能体 N<br/>..."]
    end

    subgraph 沙箱隔离["隔离环境"]
        SB1[沙箱 1]
        SB2[沙箱 2]
        SB3[沙箱 N]
    end

    DECIDER -->|任务委托| THREAD1
    DECIDER -->|任务委托| THREAD2
    DECIDER -->|任务委托| THREAD3

    THREAD1 --> SA1
    THREAD2 --> SA2
    THREAD3 --> SA3

    SA1 --> SB1
    SA2 --> SB2
    SA3 --> SB3

    THREAD1 -.->|5秒轮询| SA1
    THREAD2 -.->|5秒轮询| SA2
    THREAD3 -.->|5秒轮询| SA3

    SA1 --->|结果返回| COORD
    SA2 --->|结果返回| COORD
    SA3 --->|结果返回| COORD
```

### 内置子智能体类型

| 类型 | 工具集 | 适用场景 |
|-----|-------|---------|
| general-purpose | 全部工具（除 task）| 复杂多步骤任务 |
| bash | 仅 bash 命令 | 快速命令执行 |

### 约束限制

- 最大并发数：3 个子智能体/轮次
- 超时时间：15 分钟
- 调度线程池：3 个工作线程
- 执行线程池：3 个工作线程

---

## 沙箱系统架构

### 虚拟路径映射

DeerFlow 使用虚拟路径系统，智能体看到的是统一的容器内路径：

```mermaid
graph LR
    subgraph 虚拟路径["智能体视角"]
        V_USER[/mnt/user-data/]
        V_WORKSPACE[/mnt/user-data/workspace/]
        V_UPLOADS[/mnt/user-data/uploads/]
        V_OUTPUTS[/mnt/user-data/outputs/]
        V_SKILLS[/mnt/skills/]
    end

    subgraph 物理路径["物理文件系统"]
        P_THREAD1["backend/.deer-flow/threads/{thread_id}/user-data/"]
        P_SKILLS["deer-flow/skills/"]
    end

    V_USER -->|"线程隔离"| P_THREAD1
    V_SKILLS -->|"只读共享"| P_SKILLS
```

### 沙箱提供者

```mermaid
classDiagram
    class SandboxProvider {
        <<abstract>>
        +acquire() Sandbox
        +get(sandbox_id) Sandbox
        +release(sandbox_id)
    }

    class LocalSandboxProvider {
        +singleton 实例
        +execute_command()
        +read_file()
        +write_file()
        +list_dir()
    }

    class AioSandboxProvider {
        +provisioner_url
        +execute_command()
        +read_file()
        +write_file()
        +list_dir()
    }

    SandboxProvider <|-- LocalSandboxProvider
    SandboxProvider <|-- AioSandboxProvider
```

---

## 记忆系统架构

### 记忆更新流程

```mermaid
flowchart TB
    subgraph 对话层
        USER[用户消息]
        AI[AI 响应]
    end

    subgraph 记忆中间件
        FILTER[MemoryMiddleware]
        QUEUE[更新队列]
    end

    subgraph 记忆处理
        EXTRACT[LLM 提取]
        DEDUP[去重检查]
        VALIDATE[验证]
    end

    subgraph 存储层
        FACTS[事实库]
        CONTEXT[用户上下文]
        HISTORY[历史记录]
    end

    USER --> FILTER
    AI --> FILTER
    FILTER --> QUEUE

    QUEUE -->|30秒防抖| EXTRACT
    EXTRACT --> DEDUP
    DEDUP --> VALIDATE
    VALIDATE --> FACTS
    VALIDATE --> CONTEXT
    VALIDATE --> HISTORY

    FACTS -->|下次对话| FILTER
    CONTEXT -->|下次对话| FILTER
```

### 记忆数据结构

```json
{
  "userContext": {
    "workContext": "工作背景摘要",
    "personalContext": "个人信息摘要",
    "topOfMind": "当前最关注的事项"
  },
  "history": {
    "recentMonths": "近期对话摘要",
    "earlierContext": "早期对话背景",
    "longTermBackground": "长期知识积累"
  },
  "facts": [
    {
      "id": "唯一标识",
      "content": "事实内容",
      "category": "preference|knowledge|context|behavior|goal",
      "confidence": 0.85,
      "createdAt": "2026-03-01",
      "source": "thread_id"
    }
  ]
}
```

---

## 工具生态系统

### 工具分类

```mermaid
mindmap
    root((工具生态))
        内置工具
            present_files
            ask_clarification
            view_image
            task
        沙箱工具
            bash
            ls
            read_file
            write_file
            str_replace
        MCP 工具
            GitHub
            Filesystem
            Custom Servers
        社区工具
            Tavily 搜索
            Jina AI 网页抓取
            Firecrawl 爬虫
            DuckDuckGo 图片搜索
```

### 工具加载流程

```mermaid
flowchart TB
    START[get_available_tools] --> GROUPS[加载工具组配置]
    GROUPS --> MCP[加载 MCP 服务器]
    MCP --> BUILTIN[加载内置工具]
    BUILTIN --> SUBAGENT{子智能体启用?}
    SUBAGENT -->|是| TASK[添加 task 工具]
    SUBAGENT -->|否| VISION{视觉模型?}
    TASK --> VISION
    VISION -->|是| VIEW[添加 view_image]
    VISION -->|否| RETURN[返回工具列表]
    VIEW --> RETURN
```

---

## Gateway API 路由

```mermaid
graph TB
    GATEWAY["Gateway API<br/>FastAPI"]

    subgraph 路由模块
        MODELS["/api/models"]
        MCP["/api/mcp/config"]
        SKILLS["/api/skills"]
        MEMORY["/api/memory"]
        UPLOADS["/api/threads/{id}/uploads"]
        ARTIFACTS["/api/threads/{id}/artifacts"]
    end

    GATEWAY --> MODELS
    GATEWAY --> MCP
    GATEWAY --> SKILLS
    GATEWAY --> MEMORY
    GATEWAY --> UPLOADS
    GATEWAY --> ARTIFACTS
```

| 路由 | 方法 | 功能 |
|-----|------|-----|
| /api/models | GET | 获取可用模型列表 |
| /api/mcp/config | GET/PUT | 获取/更新 MCP 配置 |
| /api/skills | GET/PUT | 获取/更新技能状态 |
| /api/skills/install | POST | 从 .skill 包安装技能 |
| /api/memory | GET | 获取记忆数据 |
| /api/memory/reload | POST | 强制重载记忆 |
| /api/threads/{id}/uploads | POST | 上传文件 |
| /api/threads/{id}/artifacts | GET | 获取生成产物 |

---

## IM 渠道集成

### 消息流架构

```mermaid
flowchart TB
    subgraph 外部平台
        TELEGRAM[Telegram]
        SLACK[Slack]
        FEISHU[飞书]
    end

    subgraph 渠道服务
        CHANNEL[Channel 实现类]
        BUS[MessageBus 消息总线]
        MANAGER[ChannelManager]
    end

    subgraph LangGraph
        LG[LangGraph Server]
        CLIENT[langgraph-sdk 客户端]
    end

    TELEGRAM --> CHANNEL
    SLACK --> CHANNEL
    FEISHU --> CHANNEL

    CHANNEL --> BUS
    BUS --> MANAGER
    MANAGER --> CLIENT
    CLIENT --> LG

    LG -->|"runs.stream()"| FEISHU
    LG -->|"runs.wait()"| TELEGRAM
    LG -->|"runs.wait()"| SLACK

    FEISHU -->|"流式卡片更新"| FEISHU
    TELEGRAM -->|"最终响应"| TELEGRAM
    SLACK -->|"最终响应"| SLACK
```

---

## 部署架构

### Docker 开发环境

```mermaid
graph TB
    subgraph Docker 网络
        NGINX["nginx<br/>端口 2026"]

        subgraph 服务容器
            FRONTEND["frontend<br/>端口 3000"]
            LANGGRAPH["langgraph<br/>端口 2024"]
            GATEWAY["gateway<br/>端口 8001"]
            PROVISIONER["provisioner<br/>端口 8002<br/>(可选)"]
        end

        subgraph 沙箱容器
            SANDBOX["sandbox<br/>Docker 镜像"]
        end
    end

    subgraph 宿主机
        CONFIG["config.yaml"]
        DATA["backend/.deer-flow/"]
    end

    NGINX -->|"/api/langgraph/*"| LANGGRAPH
    NGINX -->|"/api/*"| GATEWAY
    NGINX -->|"/*"| FRONTEND

    LANGGRAPH --> GATEWAY
    LANGGRAPH -->|"sandbox"| SANDBOX
    GATEWAY --> SANDBOX

    CONFIG -.->|"挂载"| LANGGRAPH
    CONFIG -.->|"挂载"| GATEWAY
    DATA -.->|"挂载"| SANDBOX
```

### 本地开发模式

```mermaid
graph LR
    subgraph 终端 1
        LG_DEV["make dev<br/>LangGraph Server<br/>端口 2024"]
    end

    subgraph 终端 2
        GW_DEV["make gateway<br/>Gateway API<br/>端口 8001"]
    end

    subgraph 终端 3
        FE_DEV["make dev<br/>Next.js<br/>端口 3000"]
    end

    subgraph 终端 N
        NGINX_DEV["nginx<br/>端口 2026"]
    end

    FE_DEV --> NGINX_DEV
    LG_DEV --> NGINX_DEV
    GW_DEV --> NGINX_DEV
```

---

## 项目目录结构

```
deer-flow/
├── Makefile                    # 根目录命令
├── config.yaml                 # 主配置文件
├── extensions_config.json      # MCP 和技能扩展配置
│
├── backend/                    # 后端应用
│   ├── app/                   # 应用程序层
│   │   ├── gateway/          # FastAPI 网关
│   │   └── channels/         # IM 渠道集成
│   │
│   ├── packages/harness/      # 核心框架（可发布为 deerflow-harness）
│   │   └── deerflow/
│   │       ├── agents/        # 智能体系统
│   │       │   ├── lead_agent/# 主智能体
│   │       │   ├── middlewares/# 中间件链
│   │       │   └── memory/    # 记忆系统
│   │       ├── sandbox/       # 沙箱执行
│   │       ├── subagents/     # 子智能体
│   │       ├── tools/         # 工具
│   │       ├── mcp/           # MCP 协议
│   │       ├── models/        # 模型工厂
│   │       └── skills/        # 技能系统
│   │
│   ├── docs/                  # 后端文档
│   └── tests/                 # 测试套件
│
├── frontend/                   # Next.js 前端
│   ├── src/
│   │   ├── app/              # 页面
│   │   ├── components/        # 组件
│   │   └── lib/              # 工具库
│   └── public/
│
├── skills/                     # 技能目录
│   ├── public/                # 内置技能
│   │   ├── deep-research/
│   │   ├── consulting-analysis/
│   │   ├── data-analysis/
│   │   └── ...
│   └── custom/                 # 自定义技能（gitignore）
│
├── docker/                     # Docker 配置
│   ├── Dockerfile.frontend
│   ├── Dockerfile.backend
│   └── docker-compose.yaml
│
└── scripts/                    # 辅助脚本
```

---

## 配置系统

### config.yaml 结构

```yaml
# 模型配置
models:
  - name: gpt-4
    display_name: GPT-4
    use: langchain_openai:ChatOpenAI
    model: gpt-4
    api_key: $OPENAI_API_KEY
    supports_thinking: true
    supports_vision: true

# 工具配置
tools:
  - name: tavily
    use: deerflow.community.tavily: TavilySearch
    group: community

tool_groups:
  community:
    - tavily
    - jina_ai
    - firecrawl

# 沙箱配置
sandbox:
  use: deerflow.sandbox.local:LocalSandboxProvider

# 技能配置
skills:
  path: skills/
  container_path: /mnt/skills

# 子智能体配置
subagents:
  enabled: true
  max_concurrent: 3

# 记忆配置
memory:
  enabled: true
  storage_path: backend/.deer-flow/memory.json
  debounce_seconds: 30
  max_facts: 100
  fact_confidence_threshold: 0.7
```

### extensions_config.json 结构

```json
{
  "mcpServers": {
    "github": {
      "enabled": true,
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    }
  },
  "skills": {
    "pdf-processing": {
      "enabled": true
    }
  }
}
```