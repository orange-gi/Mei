# Qwen-Agent 架构分析

## 一、系统整体架构

Qwen-Agent采用了分层架构设计，从底层到顶层依次为：模型层、Agent层、工具层和应用层。这种设计确保了各模块之间的松耦合和高内聚，便于扩展和维护。

```mermaid
graph TB
    subgraph 应用层
        A1[Gradio UI]
        A2[Web Server]
        A3[BrowserQwen]
    end

    subgraph Agent层
        B1[BasicAgent]
        B2[FnCallAgent]
        B3[Assistant]
        B4[ReActChat]
        B5[GroupChat]
        B6[Router]
    end

    subgraph 工具层
        C1[Code Interpreter]
        C2[RAG]
        C3[Web Search]
        C4[Weather]
        C5[Image Gen]
        C6[Doc Parser]
        C7[MCP Tools]
    end

    subgraph 模型层
        D1[DashScope]
        D2[OpenAI]
        D3[Azure OpenAI]
        D4[Ollama]
        D5[Transformers]
        D6[OpenVINO]
    end

    A1 --> B1
    A2 --> B1
    A3 --> B2

    B1 --> B2
    B2 --> B3
    B2 --> B4
    B2 --> B5
    B5 --> B6

    B3 --> C1
    B3 --> C2
    B3 --> C3
    B3 --> C4
    B3 --> C5
    B3 --> C6
    B3 --> C7

    D1 <--> B1
    D2 <--> B1
    D3 <--> B1
    D4 <--> B1
    D5 <--> B1
    D6 <--> B1
```

### 架构层次说明

**模型层（LLM）**：负责与各种大语言模型服务进行通信，包括阿里云DashScope、OpenAI、Azure等云端服务，以及Ollama、Transformers等本地部署方案。模型层抽象了底层API的差异，提供统一的调用接口。

**Agent层**：实现了各种类型的智能体，每个Agent都有其特定的工作流程。BasicAgent是最基础的LLM封装，FnCallAgent增加了函数调用能力，Assistant在FnCallAgent基础上集成了RAG功能，ReActChat采用ReAct推理模式，GroupChat支持多智能体协作。

**工具层**：提供Agent可以调用的各种工具，包括代码执行、文档检索、网页搜索、图像生成等。工具通过统一的BaseTool接口进行封装，便于扩展新的工具。

**应用层**：提供了面向用户的应用示例，包括Web服务、Gradio图形界面和BrowserQwen浏览器助手。

## 二、核心模块关系

### 2.1 Agent类继承关系

Qwen-Agent的Agent体系采用了继承设计，从基类Agent派生出多种专用Agent类型。这种设计使得代码复用最大化，同时保持了各类型Agent的独立性。

```mermaid
classDiagram
    class Agent {
        <<abstract>>
        +llm: BaseChatModel
        +function_map: Dict
        +system_message: str
        +run(messages) Iterator
        +_run(messages) Iterator
        +_call_llm(messages) Iterator
        +_call_tool(tool_name, tool_args) str
    }

    class BasicAgent {
        +_run(messages) Iterator
    }

    class FnCallAgent {
        +mem: Memory
        +_run(messages) Iterator
        +_call_tool(tool_name, tool_args) str
    }

    class ReActChat {
        +_run(messages) Iterator
        +_prepend_react_prompt(messages) List[Message]
    }

    class Assistant {
        +_run(messages) Iterator
        +_prepend_knowledge_prompt(messages) List[Message]
    }

    class GroupChat {
        +agents: List[Agent]
        +host: Agent
        +_run(messages) Iterator
        +_select_agent(messages) Agent
    }

    Agent <|-- BasicAgent
    Agent <|-- FnCallAgent
    Agent <|-- GroupChat
    FnCallAgent <|-- ReActChat
    FnCallAgent <|-- Assistant
```

### 2.2 模块职责说明

**Agent基类**：定义了所有Agent的通用接口和行为规范，包括LLM调用、工具调用、消息处理等核心功能。

**BasicAgent**：最基础的Agent实现，仅包含LLM调用，没有任何工具增强。适用于简单的对话场景。

**FnCallAgent**：在Agent基础上增加了函数调用能力，支持工具注册、工具调用、结果处理等完整的工作流程。

**ReActChat**：采用ReAct（Reasoning + Acting）模式的Agent，通过显式的推理步骤来增强工具调用能力。

**Assistant**：在FnCallAgent基础上集成了RAG功能，可以管理文档、处理检索、生成带上下文的回答。

**GroupChat**：多智能体管理器，支持创建多个专业Agent，协调发言顺序、管理群组对话。

## 三、数据流图

### 3.1 单Agent数据流

以下展示了单个Agent处理用户请求时的完整数据流，从接收消息到返回响应涉及多个处理阶段。

```mermaid
flowchart TD
    A[User Message] --> B[Preprocessing]
    B --> C{Agent Type}
    C -->|BasicAgent| D[Direct LLM Call]
    C -->|FnCallAgent| E[Function Detection]
    C -->|Assistant| F[RAG Retrieval]
    C -->|ReActChat| G[ReAct Reasoning]

    D --> H[LLM Response]

    E --> I{Tool Call?}
    I -->|Yes| J[Execute Tool]
    I -->|No| H
    J --> K[Process Result]
    K --> E

    F --> L[Inject Knowledge]
    L --> E

    G --> M[Reasoning Step]
    M --> I
    I -->|Yes| J
    I -->|No| H

    H --> N[Post-processing]
    N --> O[Return Response]
```

### 3.2 函数调用详细流程

函数调用是Agent的核心能力之一，下图详细展示了从LLM生成函数调用请求到执行工具并返回结果的完整流程。

```mermaid
sequenceDiagram
    participant U as User
    participant A as Agent
    participant LLM as LLM
    participant T as Tool
    participant M as Memory

    U->>A: User Message
    A->>M: Check Files/Knowledge
    M-->>A: Retrieval Result
    A->>LLM: Send Message+Tool Desc
    LLM-->>A: Function Call Request
    A->>T: Execute Tool
    T-->>A: Tool Result
    A->>LLM: Result+Continue
    LLM-->>A: Final Response
    A-->>U: Return Response
```

### 3.3 多智能体协作流程

GroupChat支持多个Agent协同工作，通过自动或手动的发言顺序管理，实现复杂任务的分解和协作处理。

```mermaid
flowchart TD
    A[User Message] --> B[Group Manager]
    B --> C{Select Mode}
    C -->|auto| D[Host Agent Select]
    C -->|round_robin| E[Round Robin]
    C -->|random| F[Random Select]
    C -->|manual| G[Manual Select]

    D --> H[Execute Selected Agent]
    E --> H
    F --> H
    G --> H

    H --> I{Multi-round?}
    I -->|Yes| J[Update Order]
    J --> C
    I -->|No| K[Return Response]

    K --> L{Check End}
    L -->|Continue| C
    L -->|End| M[Complete]
```

## 四、LLM模块架构

### 4.1 模型适配器设计

Qwen-Agent通过统一的适配器接口支持多种LLM后端，这种设计使得添加新的模型支持变得非常简单。

```mermaid
classDiagram
    class BaseChatModel {
        <<abstract>>
        +model: str
        +model_type: str
        +chat(messages, functions) Iterator
        +_chat(messages) Iterator
    }

    class DashScopeLLM {
        +chat(messages) Iterator
    }

    class OpenAILLM {
        +chat(messages) Iterator
    }

    class AzureLLM {
        +chat(messages) Iterator
    }

    class OllamaLLM {
        +chat(messages) Iterator
    }

    class TransformersLLM {
        +chat(messages) Iterator
    }

    class QwenVLLM {
        +chat(messages) Iterator
    }

    class QwenAudioLLM {
        +chat(messages) Iterator
    }

    BaseChatModel <|-- DashScopeLLM
    BaseChatModel <|-- OpenAILLM
    BaseChatModel <|-- AzureLLM
    BaseChatModel <|-- OllamaLLM
    BaseChatModel <|-- TransformersLLM
    DashScopeLLM <|-- QwenVLLM
    DashScopeLLM <|-- QwenAudioLLM
```

### 4.2 函数调用提示词处理

函数调用功能的核心在于如何将工具描述和调用格式有效地传递给LLM。Qwen-Agent实现了多种提示词模板来处理不同模型的函数调用格式。

```mermaid
flowchart LR
    subgraph Input
        A[Raw Messages]
        B[Function Definitions]
    end

    subgraph Processing
        C[Preprocess]
        D{Model Type}
        E[Qwen Template]
        F[Nous Template]
        G[Base Template]
    end

    subgraph Output
        H[Formatted Messages]
        I[Function Call Result]
    end

    A --> C
    B --> C
    C --> D
    D -->|qwen| E
    D -->|nous| F
    D -->|other| G
    E --> H
    F --> H
    G --> H
    H --> LLM[LLM Call]
    LLM --> I
```

## 五、工具系统架构

### 5.1 工具注册与调用

Qwen-Agent提供了统一的工具注册和管理机制，所有工具都继承自BaseTool基类，通过TOOL_REGISTRY进行注册。

```mermaid
classDiagram
    class BaseTool {
        <<abstract>>
        +name: str
        +function: Dict
        +file_access: bool
        +call(tool_args, files) Any
    }

    class CodeInterpreter {
        +call(tool_args, files) str
    }

    class PythonExecutor {
        +call(tool_args) str
    }

    class Retrieval {
        +call(tool_args) List[str]
    }

    class WebSearch {
        +call(tool_args) str
    }

    class DocParser {
        +call(tool_args) Dict
    }

    class MCPTool {
        +call(tool_args) Any
    }

    BaseTool <|-- CodeInterpreter
    BaseTool <|-- PythonExecutor
    BaseTool <|-- Retrieval
    BaseTool <|-- WebSearch
    BaseTool <|-- DocParser
    BaseTool <|-- MCPTool
```

### 5.2 MCP工具集成

Qwen-Agent支持MCP（Model Context Protocol）协议，可以接入更多第三方工具和服务，扩展了框架的能力边界。

```mermaid
flowchart TD
    A[MCP Config] --> B[MCPManager]
    B --> C[Tool Discovery]
    C --> D[Tool Registration]
    D --> E[TOOL_REGISTRY]

    F[User Request] --> G[Agent]
    G --> H{Need Tool?}
    H -->|Yes| I[Find Tool]
    I --> J[TOOL_REGISTRY]
    J --> K{MCP Tool?}
    K -->|Yes| L[MCP Server Call]
    K -->|No| M[Local Execute]
    L --> N[Return Result]
    M --> N
    H -->|No| O[Direct LLM Response]
    N --> G
    O --> P[Return Response]
```

## 六、部署架构

### 6.1 服务部署模式

Qwen-Agent支持多种部署方式以满足不同场景的需求，从简单的本地运行到生产级别的服务部署。

```mermaid
graph TB
    subgraph Client
        A[Web Frontend]
        B[Mobile App]
        C[Third-party Service]
    end

    subgraph Gateway
        D[Load Balancer]
    end

    subgraph Service
        E[Qwen-Agent Server]
        F[Auth Service]
    end

    subgraph Infrastructure
        G[Docker Container]
        H[Model Service]
        I[File System]
    end

    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    E --> G
    G --> H
    G --> I
```

### 6.2 BrowserQwen架构

BrowserQwen是基于Qwen-Agent构建的浏览器助手，展示了如何将Agent能力与浏览器自动化相结合。

```mermaid
flowchart TD
    A[User Request] --> B[BrowserQwen Agent]
    B --> C{Browser Operation?}
    C -->|Yes| D[Browser Controller]
    D --> E[Execute Operation]
    E --> F[Get Page Content]
    F --> B
    C -->|No| G[Normal LLM Process]
    G --> H[Return Response]
```

## 七、扩展性设计

### 7.1 自定义Agent

Qwen-Agent的设计充分考虑了扩展性，开发者可以轻松创建自定义的Agent类型。

```mermaid
flowchart LR
    A[Extend Agent Base Class] --> B[Implement _run Method]
    B --> C[Define Workflow]
    C --> D[Register to System]
    D --> E[Use Custom Agent]
```

### 7.2 自定义工具

创建自定义工具只需继承BaseTool类并实现call方法。

```mermaid
flowchart LR
    A[Extend BaseTool] --> B[Define function Property]
    B --> C[Implement call Method]
    C --> D[Register to TOOL_REGISTRY]
    D --> E[Use in Agent]
```

