# Hermes Agent 架构文档

## 系统整体架构

### 核心架构概览

```mermaid
graph TB
    subgraph 用户层["用户交互层"]
        CLI["CLI 终端<br/>(TUI)"]
        Gateway["消息网关<br/>(Gateway)"]
        MCP["MCP 服务器"]
    end

    subgraph 核心层["Agent 核心"]
        AIAgent["AIAgent<br/>主代理循环"]
        ContextEngine["Context Engine<br/>上下文引擎"]
        MemoryManager["Memory Manager<br/>记忆管理器"]
        PromptBuilder["Prompt Builder<br/>提示词构建器"]
        ContextCompressor["Context Compressor<br/>上下文压缩器"]
    end

    subgraph 模型层["模型适配层"]
        OpenAI["OpenAI Adapter"]
        Anthropic["Anthropic Adapter"]
        OpenRouter["OpenRouter Adapter"]
        Bedrock["AWS Bedrock"]
        Gemini["Google Gemini"]
        Custom["Custom Endpoint"]
    end

    subgraph 工具层["工具系统"]
        ToolRegistry["Tool Registry<br/>工具注册表"]
        ToolSets["Tool Sets<br/>工具集"]

        subgraph 工具集["工具集分类"]
            WebTools["Web Tools<br/>网络工具"]
            FileTools["File Tools<br/>文件工具"]
            TerminalTools["Terminal Tools<br/>终端工具"]
            BrowserTools["Browser Tools<br/>浏览器工具"]
            SkillTools["Skill Tools<br/>技能工具"]
            CronTools["Cron Tools<br/>定时工具"]
            MiscTools["Misc Tools<br/>其他工具"]
        end
    end

    subgraph 存储层["存储与配置"]
        Config["Config<br/>配置管理"]
        Skills["Skills Store<br/>技能存储"]
        Memory["Memory Store<br/>记忆存储"]
        Sessions["Sessions<br/>会话管理"]
    end

    CLI --> AIAgent
    Gateway --> AIAgent
    MCP --> AIAgent

    AIAgent --> ContextEngine
    AIAgent --> MemoryManager
    AIAgent --> PromptBuilder

    ContextEngine --> ContextCompressor
    ContextCompressor --> ModelAdapters["Model Adapters"]

    PromptBuilder --> ModelAdapters
    MemoryManager --> Memory

    ModelAdapters --> OpenAI
    ModelAdapters --> Anthropic
    ModelAdapters --> OpenRouter
    ModelAdapters --> Bedrock
    ModelAdapters --> Gemini
    ModelAdapters --> Custom

    AIAgent --> ToolRegistry
    ToolRegistry --> ToolSets

    ToolSets --> WebTools
    ToolSets --> FileTools
    ToolSets --> TerminalTools
    ToolSets --> BrowserTools
    ToolSets --> SkillTools
    ToolSets --> CronTools
    ToolSets --> MiscTools

    ToolRegistry --> Config
    ToolRegistry --> Skills
    ToolRegistry --> Sessions
```

### 分层架构详解

```mermaid
graph LR
    subgraph Layer1["第一层: 用户接口层"]
        TUI["TUI 终端"]
        Gateway["多平台网关"]
        API["API 接口"]
    end

    subgraph Layer2["第二层: 命令处理层"]
        Commands["命令解析"]
        Hooks["钩子系统"]
        Profiles["配置文件"]
    end

    subgraph Layer3["第三层: Agent 核心"]
        AgentLoop["Agent 循环"]
        Context["上下文管理"]
        Memory["记忆系统"]
    end

    subgraph Layer4["第四层: 模型抽象层"]
        Adapters["适配器集合"]
        Router["路由选择"]
    end

    subgraph Layer5["第五层: 工具执行层"]
        Registry["工具注册"]
        Executor["执行器"]
        Security["安全检查"]
    end

    TUI --> Commands
    Gateway --> Commands
    API --> Commands

    Commands --> Hooks
    Hooks --> AgentLoop

    AgentLoop --> Context
    AgentLoop --> Memory
    AgentLoop --> Adapters

    Adapters --> Registry
    Registry --> Executor
    Executor --> Security
```

---

## 核心数据流图

### 主代理执行流程

```mermaid
sequenceDiagram
    participant User as 用户
    participant CLI as CLI/Gateway
    participant Agent as AIAgent
    participant Context as ContextEngine
    participant Prompt as PromptBuilder
    participant Model as ModelAdapter
    participant Tools as ToolRegistry
    participant Memory as MemoryManager

    User->>CLI: 输入消息/命令
    CLI->>Agent: 传递用户输入

    Note over Agent: 开始一轮对话

    Agent->>Memory: 获取会话记忆
    Agent->>Context: 加载上下文文件
    Agent->>Prompt: 构建系统提示词

    Memory->>Prompt: 返回记忆上下文
    Context->>Prompt: 返回项目上下文

    Prompt->>Agent: 返回完整提示词

    Agent->>Model: 发送 API 请求

    alt 模型返回工具调用
        Model->>Agent: 返回函数调用
        Agent->>Tools: 请求工具执行

        loop 工具执行循环
            Tools->>Tools: 执行工具
            Tools->>Agent: 返回结果

            alt 需要更多工具调用
                Agent->>Model: 发送工具结果
                Model->>Agent: 下一轮调用
            end
        end

        Agent->>Memory: 更新记忆
        Agent->>Context: 更新上下文
    else 模型返回文本
        Model->>Agent: 返回文本响应
    end

    Agent->>CLI: 返回最终结果
    CLI->>User: 显示响应

    Note over Agent: 一轮对话结束
```

### 消息网关数据流

```mermaid
flowchart LR
    subgraph 消息源["消息来源平台"]
        TG["Telegram"]
        DC["Discord"]
        SL["Slack"]
        WA["WhatsApp"]
        SG["Signal"]
        MX["Matrix"]
    end

    subgraph 网关处理["Gateway 处理"]
        Receiver["消息接收器"]
        Parser["消息解析器"]
        Router["消息路由"]
        Session["会话管理"]
    end

    subgraph Agent["Hermes Agent"]
        Core["Agent Core"]
        Memory["记忆系统"]
    end

    TG --> Receiver
    DC --> Receiver
    SL --> Receiver
    WA --> Receiver
    SG --> Receiver
    MX --> Receiver

    Receiver --> Parser
    Parser --> Router
    Router --> Session
    Session --> Core

    Core --> Memory
    Memory --> Core

    Core --> Sender
    Sender --> TG
    Sender --> DC
    Sender --> SL
    Sender --> WA
    Sender --> SG
    Sender --> MX
```

---

## 模块关系图

### Agent 核心模块

```mermaid
classDiagram
    class AIAgent {
        +conversation: list
        +model_adapter: BaseAdapter
        +memory_manager: MemoryManager
        +tool_registry: ToolRegistry

        +run(user_input: str) str
        +_build_system_prompt() str
        +_call_model(messages: list) dict
        +_handle_tool_call(call: dict) str
        +_update_memory(result: str)
    }

    class ContextEngine {
        +load_project_context(cwd: Path) str
        +load_agents_md(path: Path) str
        +scan_threats(content: str) bool
        +resolve_references(text: str) str
    }

    class MemoryManager {
        +session_memory: dict
        +persistent_memory: dict
        +user_profile: UserProfile

        +get_memory_context() str
        +add_to_memory(event: str)
        +search_sessions(query: str) list
        +summarize_old_turns() str
    }

    class PromptBuilder {
        +identity_block: str
        +platform_hints: str
        +skills_index: str
        +context_files: str

        +build_system_prompt(config: Config) str
        +_assemble_identity() str
        +_assemble_skills() str
        +_assemble_context() str
    }

    class ContextCompressor {
        +compression_threshold: int
        +summary_model: BaseAdapter

        +should_compress(messages: list) bool
        +compress(messages: list) list
        +generate_summary(content: str) str
    }

    AIAgent --> ContextEngine
    AIAgent --> MemoryManager
    AIAgent --> PromptBuilder
    AIAgent --> ContextCompressor

    ContextEngine --> PromptBuilder
    MemoryManager --> PromptBuilder
```

### 模型适配器体系

```mermaid
classDiagram
    class BaseAdapter {
        <<abstract>>
        +model_name: str
        +api_key: str

        +chat(messages: list) dict
        +stream(messages: list) Generator
        +count_tokens(text: str) int
    }

    class OpenAIAdapter {
        +client: OpenAIClient

        +chat(messages: list) dict
        +_build_request() dict
    }

    class AnthropicAdapter {
        +client: AnthropicClient

        +chat(messages: list) dict
        +_map_messages() list
    }

    class OpenRouterAdapter {
        +client: httpxClient

        +chat(messages: list) dict
        +_route_to_model(model: str)
    }

    class BedrockAdapter {
        +client: boto3Client

        +chat(messages: list) dict
        +_use_converse_api()
    }

    class GeminiAdapter {
        +client: GeminiClient

        +chat(messages: list) dict
        +_convert_schema() dict
    }

    BaseAdapter <|-- OpenAIAdapter
    BaseAdapter <|-- AnthropicAdapter
    BaseAdapter <|-- OpenRouterAdapter
    BaseAdapter <|-- BedrockAdapter
    BaseAdapter <|-- GeminiAdapter
```

---

## 用户交互流程图

### CLI 交互流程

```mermaid
flowchart TB
    Start["用户启动 hermes"] --> Choice{"选择模式"}

    Choice -->|CLI 模式| CLIStart["加载配置"]
    Choice -->|Gateway 模式| GatewayStart["启动网关"]

    CLIStart --> LoadConfig["加载配置文件"]
    LoadConfig --> InitAgent["初始化 Agent"]

    GatewayStart --> InitGateway["初始化网关"]
    InitGateway --> RegisterPlatforms["注册消息平台"]
    RegisterPlatforms --> WaitMessage["等待消息"]

    InitAgent --> ShowBanner["显示 Banner"]
    ShowBanner --> LoadHistory["加载历史会话"]
    LoadHistory --> AwaitInput["等待用户输入"]

    AwaitInput --> ParseInput{"解析输入"}

    ParseInput -->|斜杠命令| SlashCmd["执行斜杠命令"]
    ParseInput -->|普通消息| ProcessMsg["处理消息"]

    SlashCmd --> CommandResult["显示结果"]
    ProcessMsg --> AgentLoop["进入 Agent 循环"]

    CommandResult --> AwaitInput
    AgentLoop --> StreamOutput["流式输出"]

    StreamOutput --> CheckInterrupt{"检查中断?"}
    CheckInterrupt -->|是| Stop["停止执行"]
    CheckInterrupt -->|否| ToolLoop{"工具循环?"}

    ToolLoop -->|需要工具| ExecuteTool["执行工具"]
    ExecuteTool --> StreamOutput

    ToolLoop -->|完成| FinalOutput["输出最终结果"]

    FinalOutput --> CheckCompress{"需要压缩?"}
    CheckCompress -->|是| Compress["压缩上下文"]
    CheckCompress -->|否| SaveHistory["保存历史"]

    Compress --> SaveHistory
    SaveHistory --> AwaitInput

    Stop --> AwaitInput
    WaitMessage --> ProcessMsg

    subgraph Agent循环["Agent 核心循环"]
        direction TB
        SendModel["发送模型"]
        GetResponse["获取响应"]
        CheckType{"类型检查"}
        ToolCall["工具调用"]
        TextOutput["文本输出"]
    end

    ProcessMsg --> SendModel
    SendModel --> GetResponse
    GetResponse --> CheckType
    CheckType -->|工具调用| ToolCall
    CheckType -->|文本| TextOutput
    ToolCall --> SendModel
```

### 技能创建流程

```mermaid
flowchart LR
    A["复杂任务完成"] --> B{"自动检测?"}

    B -->|是| C["提示创建技能"]
    B -->|否| D["手动触发"]

    C --> E["用户提供技能名"]
    D --> E

    E --> F["提取任务步骤"]
    F --> G["生成技能模板"]

    G --> H["添加 YAML 元数据"]
    H --> I["保存到 skills/"]

    I --> J["注册到索引"]
    J --> K["技能创建完成"]

    K --> L{"首次使用?"}
    L -->|是| M["观察执行"]
    L -->|否| N["完成"]

    M --> O{"需要优化?"}
    O -->|是| P["调整技能"]
    O -->|否| N

    P --> Q["自我优化"]
    Q --> M
```

---

## 部署架构图

### 多种部署模式

```mermaid
flowchart TB
    subgraph Local["本地部署"]
        LocalCLI["Hermes CLI"]
        LocalEnv["本地环境"]
    end

    subgraph Docker["Docker 部署"]
        DockerContainer["Docker 容器"]
        DockerVolume["持久卷"]
    end

    subgraph SSH["SSH 远程部署"]
        SSHClient["本地客户端"]
        SSHServer["远程服务器"]
    end

    subgraph Daytona["Daytona 无服务器"]
        DaytonaCloud["Daytona Cloud"]
        DaytonaHibernate["休眠/唤醒"]
    end

    subgraph Modal["Modal 无服务器"]
        ModalCloud["Modal Cloud"]
        ModalGPU["GPU 竞价实例"]
    end

    subgraph 共享资源["共享后端服务"]
        ModelProviders["模型提供商"]
        Redis["缓存服务"]
        DB["数据库"]
    end

    Local --> ModelProviders
    Docker --> ModelProviders
    SSH --> ModelProviders
    Daytona --> ModelProviders
    Modal --> ModelProviders

    Docker --> DockerVolume
    Daytona --> Redis
    Modal --> Redis
```

---

## 总结

本文档通过多种 Mermaid 图表展示了 Hermes Agent 的完整架构：

1. **系统整体架构**：展示从用户接口到模型适配的完整分层
2. **核心数据流**：展示主代理循环、消息网关、工具执行的详细流程
3. **模块关系图**：展示核心类之间的依赖关系
4. **用户交互流程**：展示 CLI 和 Gateway 的用户交互模式
5. **部署架构**：展示多种部署选项

这些图表可以帮助开发者快速理解 Hermes Agent 的设计理念和实现细节。