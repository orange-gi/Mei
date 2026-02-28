# Browser-Use 架构图分析

> **作者**: Matrix Agent
> **项目地址**: https://github.com/browser-use/browser-use
> **分析时间**: 2026-01-24

---

## 一、系统整体架构图

### 1.1 核心架构概览

```mermaid
graph TB
    subgraph 用户层
        User[用户代码]
    end

    subgraph Browser-Use 核心库
        Agent[Agent 代理模块]
        Tools[Tools 工具系统]
        Controller[Controller 控制器]
    end

    subgraph LLM 集成层
        LLM[LLM 接口抽象层]
        OpenAI[ChatOpenAI]
        Anthropic[ChatAnthropic]
        Google[ChatGoogle]
        BrowserUse[ChatBrowserUse]
        Ollama[ChatOllama]
        Others[其他 LLM...]
    end

    subgraph 浏览器控制层
        Browser[Browser/BrowserSession]
        CDP[CDP Client]
        Playwright[Playwright 底层]
    end

    subgraph 辅助系统
        DOM[DOM 服务]
        FileSystem[文件系统]
        Telemetry[遥测系统]
        EventBus[事件总线]
    end

    User --> Agent
    User --> Tools

    Agent --> LLM
    Agent --> Browser
    Agent --> Tools
    Agent --> FileSystem

    Tools --> Controller
    Controller --> Browser

    LLM --> OpenAI
    LLM --> Anthropic
    LLM --> Google
    LLM --> BrowserUse
    LLM --> Ollama

    Browser --> CDP
    CDP --> Playwright

    Browser --> EventBus
    EventBus --> DOM
    EventBus --> Telemetry
```

### 1.2 模块职责说明

```mermaid
graph LR
    A[Agent] --> B[任务规划]
    A --> C[状态管理]
    A --> D[LLM 调用]
    A --> E[历史记录]

    F[Browser] --> G[页面导航]
    F --> H[元素交互]
    F --> I[截图捕获]
    F --> J[标签页管理]

    K[Tools] --> L[内置动作]
    K --> M[自定义工具]
    K --> N[动作注册]

    O[LLM] --> P[提示词生成]
    O --> Q[响应解析]
    O --> R[多模型支持]
```

---

## 二、核心数据流图

### 2.1 代理执行主流程

```mermaid
sequenceDiagram
    participant User as 用户代码
    participant Agent as Agent
    participant Prompt as SystemPrompt
    participant LLM as LLM
    participant Browser as Browser
    participant DOM as DOM Service
    participant Tools as Tools

    User->>Agent: 初始化并运行任务
    Agent->>Browser: 获取页面状态
    Browser->>DOM: 获取 DOM 快照
    DOM->>DOM: 简化 DOM 树
    DOM-->>Browser: 页面结构
    Browser-->>Agent: 页面状态 + 截图
    Agent->>Prompt: 生成系统提示词
    Agent->>AgentMessagePrompt: 生成用户消息
    Agent->>LLM: 发送完整提示词
    LLM-->>Agent: 返回动作序列
    Agent->>Tools: 执行动作
    Tools->>Browser: 派发浏览器事件
    Browser-->>Agent: 动作结果
    Agent->>Agent: 更新历史记录
    Agent->>Agent: 检查是否完成
    loop 任务循环
        Agent->>Browser: 获取新页面状态
        Browser-->>Agent: 页面变化
        Agent->>LLM: 发送新状态
        LLM-->>Agent: 下一动作
    end
    Agent-->>User: 返回执行历史
```

### 2.2 消息流转详情

```mermaid
flowchart TB
    subgraph 消息构造
        SysPrompt[系统提示词]
        UserPrompt[用户提示词]
        History[历史记录]
        Screenshot[截图]
        DOMTree[DOM 结构]
    end

    subgraph 消息组合
        Combine[消息组合器]
        Cache[消息缓存]
    end

    subgraph LLM 交互
        API[LLM API]
        Response[响应解析]
        Actions[动作序列]
    end

    SysPrompt --> Combine
    UserPrompt --> Combine
    History --> Combine
    Screenshot --> Combine
    DOMTree --> Combine

    Combine --> Cache
    Cache --> API
    API --> Response
    Response --> Actions
```

---

## 三、类/模块关系图

### 3.1 Agent 核心类图

```mermaid
classDiagram
    class Agent~Context, AgentStructuredOutput~ {
        -task: str
        -llm: BaseChatModel
        -browser: Browser
        -tools: Tools
        -message_manager: MessageManager
        +__init__()
        +run() AgentHistoryList
        +run_step() AgentStepResult
        +_execute_action()
        +_get_browser_state()
    }

    class MessageManager {
        -message_history: list
        +add_message()
        +get_messages()
        +reset()
    }

    class SystemPrompt {
        -prompt_template: str
        +get_system_message() SystemMessage
        +_load_prompt_template()
    }

    class AgentMessagePrompt {
        -browser_state: BrowserStateSummary
        -agent_history: str
        -task: str
        +get_user_message() UserMessage
    }

    class Tools~Context~ {
        -registry: Registry
        +register_action()
        +get_actions()
    }

    class Browser {
        -browser_session: BrowserSession
        -event_bus: EventBus
        +navigate()
        +click()
        +input()
        +screenshot()
        +get_state()
    }

    class BrowserSession {
        -id: str
        -browser_profile: BrowserProfile
        +connect()
        +close()
        +new_tab()
    }

    Agent "1" *-- "1" MessageManager : 包含
    Agent "1" *-- "1" SystemPrompt : 使用
    Agent "1" *-- "1" Tools : 使用
    Agent "1" *-- "1" Browser : 控制
    Browser "1" *-- "1" BrowserSession : 管理
```

### 3.2 浏览器会话类图

```mermaid
classDiagram
    class BrowserSession {
        <<event-driven>>
        +id: str
        +browser_profile: BrowserProfile
        +event_bus: EventBus
        +connect()
        +close()
        +navigate()
        +click()
        +input()
        +scroll()
        +switch_tab()
        +extract()
    }

    class BrowserProfile {
        +headless: bool
        +user_data_dir: Path
        +locale: str
        +proxy_settings: ProxySettings
        +timeout: int
    }

    class EventBus {
        +dispatch()
        +subscribe()
        +publish()
    }

    class CDPClient {
        +connect()
        +send_command()
        +on_event()
    }

    class Tab {
        +target_id: TargetID
        +url: str
        +title: str
        +navigate()
        +screenshot()
    }

    BrowserSession "1" *-- "1" EventBus : 使用
    BrowserSession "1" *-- "1" BrowserProfile : 配置
    BrowserSession "1" *-- "n" Tab : 管理
    EventBus ..> CDPClient : 事件处理
```

### 3.3 工具系统类图

```mermaid
classDiagram
    class Tools~Context~ {
        -registry: Registry
        +search()
        +navigate()
        +click()
        +input_text()
        +scroll()
        +extract()
        +done()
        +switch_tab()
    }

    class Registry~Context~ {
        -actions: dict
        +register_action()
        +get_action()
        +list_actions()
    }

    class ActionModel {
        +action_names: list[str]
        +action_params: dict
        +model_validate()
    }

    class ActionResult {
        +extracted_content: str
        +error: str
        +success: bool
    }

    class DoneAction {
        +success: bool
        +text: str
        +files_to_display: list
    }

    class ExtractAction {
        +query: str
        +schema: dict
    }

    Tools "1" *-- "1" Registry : 使用
    Registry "1" *-- "n" ActionModel : 注册
    Tools --> DoneAction : 内置动作
    Tools --> ExtractAction : 内置动作
```

---

## 四、用户交互流程图

### 4.1 典型使用流程

```mermaid
flowchart TD
    A[开始] --> B[创建 Browser 实例]
    B --> C[创建 LLM 实例]
    C --> D[可选: 创建 Tools 实例]
    D --> E[创建 Agent 实例]
    E --> F[调用 agent.run()]
    F --> G{任务是否完成?}
    G -->|否| H[执行下一步动作]
    H --> I[获取新页面状态]
    I --> F
    G -->|是| J[返回执行历史]
    J --> K[结束]
```

### 4.2 动作执行流程

```mermaid
flowchart LR
    subgraph 执行步骤
        A[LLM 响应] --> B[解析动作]
        B --> C[验证动作参数]
        C --> D[查找动作处理器]
        D --> E[执行动作]
        E --> F[返回结果]
    end

    subgraph 动作类型
        click --> navigate
        navigate --> input
        input --> scroll
        scroll --> extract
        extract --> done
    end
```

### 4.3 错误恢复流程

```mermaid
flowchart TD
    A[执行动作] --> B{成功?}
    B -->|是| C{任务完成?}
    B -->|否| D{失败次数 < max?}
    C -->|是| E[返回结果]
    C -->|否| F[继续下一动作]
    D -->|是| G[尝试恢复]
    G --> H[重新执行动作]
    H --> B
    D -->|否| I[记录最终错误]
    I --> J[调用 done 报告失败]
```

---

## 五、部署架构图

### 5.1 本地部署架构

```mermaid
graph TB
    subgraph 开发环境
        User[用户代码]
        Python[Python 进程]
        BrowserUse[Browser-Use 库]
    end

    subgraph 本地浏览器
        Chromium[Chromium 浏览器]
        CDP[CDP 端口 9222]
    end

    subgraph 外部服务
        LLM[LLM API 服务]
        Cloud[Browser Use Cloud 可选]
    end

    User --> Python
    Python --> BrowserUse
    BrowserUse --> Chromium
    Chromium --> LLM
    BrowserUse -.-> Cloud
```

### 5.2 云端部署架构

```mermaid
graph TB
    subgraph 客户端
        User[用户代码]
        LocalBrowser[本地 Browser 实例]
    end

    subgraph Browser Use Cloud
        LoadBalancer[负载均衡]
        AgentPool[Agent 池]
        StealthBrowser[隐身浏览器集群]
        Auth[认证服务]
    end

    subgraph 外部服务
        LLM[LLM API]
    end

    User --> LocalBrowser
    LocalBrowser --> LoadBalancer
    LoadBalancer --> AgentPool
    AgentPool --> StealthBrowser
    StealthBrowser --> LLM
    LoadBalancer --> Auth
```

### 5.3 多代理并行架构

```mermaid
graph LR
    subgraph 任务队列
        Queue[任务队列]
    end

    subgraph 代理池
        Agent1[Agent 1]
        Agent2[Agent 2]
        Agent3[Agent 3]
        AgentN[Agent N]
    end

    subgraph 共享资源
        BrowserPool[浏览器池]
        LLM[LLM 服务]
    end

    Queue --> Agent1
    Queue --> Agent2
    Queue --> Agent3
    Queue --> AgentN

    Agent1 --> BrowserPool
    Agent2 --> BrowserPool
    Agent3 --> BrowserPool
    AgentN --> BrowserPool

    Agent1 --> LLM
    Agent2 --> LLM
    Agent3 --> LLM
    AgentN --> LLM
```

---

## 六、数据模型图

### 6.1 核心数据模型

```mermaid
erDiagram
    Agent ||--|| BrowserSession : 控制
    Agent ||--|| LLM : 调用
    Agent ||--|| Tools : 使用
    Agent ||--|{ AgentHistory : 记录

    BrowserSession ||--|| BrowserProfile : 配置
    BrowserSession ||--|{ Tab : 管理

    LLM ||--|{ Message : 发送
    Message ||--|| ActionModel : 包含

    ActionModel ||--|{ ActionResult : 生成

    Tab ||--|| BrowserState : 持有
```

### 6.2 关键数据结构

```mermaid
classDiagram
    class AgentHistory {
        +step_history: list~AgentStep~
        +final_result: AgentOutput
        +total_steps: int
        +success: bool
    }

    class AgentStep {
        +step_number: int
        +action: str
        +result: ActionResult
        +screenshot: str
        +timestamp: datetime
    }

    class BrowserStateSummary {
        +url: str
        +title: str
        +tabs: list~TabInfo~
        +dom_state: DOMState
        +screenshot: str
    }

    class ActionModel {
        +thinking: str
        +evaluation_previous_goal: str
        +memory: str
        +next_goal: str
        +action: list~dict~
    }

    class ActionResult {
        +extracted_content: str
        +error: str | None
        +long_term_memory: str | None
        +success: bool
    }
```

---

## 七、事件驱动架构

### 7.1 事件总线流程

```mermaid
flowchart TB
    subgraph 事件发布者
        Agent[Agent]
        Tools[Tools]
        User[用户代码]
    end

    subgraph EventBus
        Dispatcher[事件分发器]
        Subscribers[订阅者注册表]
    end

    subgraph 事件处理
        CDPClient[CDP 客户端]
        Browser[浏览器控制]
        Screenshot[截图服务]
        Logging[日志服务]
    end

    Agent --> Dispatcher
    Tools --> Dispatcher
    User --> Dispatcher

    Dispatcher --> Subscribers
    Subscribers --> CDPClient
    Subscribers --> Browser
    Subscribers --> Screenshot
    Subscribers --> Logging
```

### 7.2 浏览器事件流

```mermaid
sequenceDiagram
    participant Tools as Tools
    participant EventBus as EventBus
    participant CDP as CDP Client
    participant Browser as Browser
    participant Page as Page

    Tools->>EventBus: 派发 ClickEvent
    EventBus->>CDP: 发送 CDP 命令
    CDP->>Page: 执行点击
    Page-->>CDP: 返回结果
    CDP-->>EventBus: 事件完成
    EventBus-->>Tools: 返回 ActionResult

    Note over Page: 页面状态变化
    Page->>CDP: 触发 DOM 变化事件
    CDP->>EventBus: 派发 StateChangeEvent
    EventBus->>Browser: 更新状态
```

---

## 八、总结

Browser-Use 的架构设计体现了以下核心原则：

1. **分层架构**：清晰的层次划分，从用户接口到底层浏览器控制
2. **事件驱动**：使用 EventBus 实现松耦合的组件通信
3. **可扩展性**：通过 Tools 和 Registry 机制支持自定义动作
4. **多模型支持**：统一的 LLM 接口适配多种模型
5. **容错设计**：内置重试机制和错误恢复

这种架构使得 Browser-Use 既能处理简单的单步操作，也能支持复杂的多代理并行场景。
