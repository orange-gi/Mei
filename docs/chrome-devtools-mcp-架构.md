# Chrome DevTools MCP 架构文档

## 一、系统整体架构

### 1.1 架构概览

Chrome DevTools MCP采用了分层架构设计，从上到下依次为：MCP客户端层、MCP服务器层、业务逻辑层和浏览器控制层。这种分层设计使得各层之间的职责清晰，便于维护和扩展。

```mermaid
flowchart TB
    subgraph MCP客户端层["MCP 客户端层"]
        A1[Claude Code]
        A2[Cursor]
        A3[Copilot]
        A4[Gemini CLI]
        A5[其他MCP客户端]
    end

    subgraph MCP服务器层["MCP 服务器层"]
        B1[MCP Server SDK]
        B2[工具注册与管理]
        B3[请求路由]
        B4[响应处理]
    end

    subgraph 业务逻辑层["业务逻辑层"]
        C1[工具定义层]
        C2[工具处理层]
        C3[上下文管理]
        C4[响应格式化]
    end

    subgraph 浏览器控制层["浏览器控制层"]
        D1[Puppeteer控制]
        D2[Chrome DevTools协议]
        D3[浏览器实例管理]
        D4[页面管理]
    end

    subgraph 浏览器["Chrome 浏览器"]
        E1[浏览器实例]
        E2[页面上下文]
        E3[DevTools协议连接]
    end

    A1 --> B1
    A2 --> B1
    A3 --> B1
    A4 --> B1
    A5 --> B1

    B1 --> B2
    B2 --> B3
    B3 --> B4

    B4 --> C1
    C1 --> C2
    C2 --> C3
    C3 --> C4

    C4 --> D1
    D1 --> D2
    D2 --> D3
    D3 --> D4

    D4 --> E1
    D1 --> E1
    E2 --> E3
```

### 1.2 核心组件职责

Chrome DevTools MCP项目由多个核心组件构成，每个组件都有明确的职责范围。入口文件`src/index.ts`负责整个服务器的生命周期管理，包括初始化MCP服务器、创建浏览器上下文、注册工具、处理请求等核心功能。

**入口层组件**

`src/index.ts`作为整个应用的入口点，负责协调各个模块之间的工作。它解析命令行参数，初始化MCP服务器实例，创建浏览器上下文，并根据配置注册相应的工具。该文件还负责处理工具请求的路由和错误处理，确保整个系统能够稳定运行。

**浏览器管理层**

`src/browser.ts`文件封装了所有与浏览器相关的操作。该模块负责启动Chrome浏览器实例、连接到已运行的浏览器，维护浏览器会话，以及处理各种浏览器级别的配置选项。它支持多种连接方式，包括直接启动、远程调试端口连接、WebSocket连接等。

**上下文管理层**

`src/McpContext.ts`是核心的上下文管理组件，负责维护与浏览器会话相关的所有状态信息。它管理页面生命周期、跟踪打开的标签页、提供页面选择和切换功能，以及处理DevTools窗口检测。该组件是连接MCP服务器和浏览器的关键桥梁。

**响应处理层**

`src/McpResponse.ts`负责格式化工具执行结果，将其转换为MCP协议要求的格式。该组件处理文本响应、结构化内容、图像数据等不同类型的返回结果，并提供丰富的格式化选项。

**工具定义层**

`src/tools/`目录包含了所有可用工具的定义和实现。每个工具都被封装为独立的模块，包括输入模式定义、处理逻辑、参数验证等功能。这种模块化设计使得添加新工具变得非常简单。

## 二、核心数据流

### 2.1 工具调用完整流程

当AI编码助手调用Chrome DevTools MCP工具时，请求会经过一系列处理步骤。理解这个流程有助于更好地理解系统的工作原理，以及在遇到问题时进行有效的故障排查。

```mermaid
sequenceDiagram
    participant Client as MCP客户端
    participant Server as MCP服务器
    participant Registry as 工具注册表
    participant Context as McpContext
    participant Handler as 工具处理器
    participant Response as McpResponse
    participant Browser as Puppeteer
    participant Chrome as Chrome浏览器

    Client->>Server: 工具调用请求
    Server->>Registry: 查找工具定义
    Registry-->>Server: 返回工具元数据

    Server->>Context: 获取浏览器上下文
    alt 浏览器未启动
        Context->>Browser: 启动Chrome实例
        Browser-->>Context: 浏览器句柄
    end

    Context-->>Server: 上下文就绪

    Server->>Handler: 执行工具处理函数
    Handler->>Browser: 发送浏览器命令
    Browser->>Chrome: 通过DevTools协议通信
    Chrome-->>Browser: 执行结果
    Browser-->>Handler: 返回执行结果

    Handler->>Response: 格式化响应
    Response-->>Server: 格式化后的响应

    Server-->>Client: 返回调用结果
```

### 2.2 浏览器启动与连接流程

Chrome DevTools MCP支持多种浏览器连接方式，以适应不同的使用场景。理解这些连接方式有助于选择最适合具体需求的配置方案。

```mermaid
flowchart LR
    subgraph 连接方式选择
        A{选择连接方式}
    end

    subgraph 方式一["自动启动（默认）"]
        B1[MCP服务器]
        B2[启动Chrome进程]
        B3[等待连接就绪]
        B4[创建Puppeteer会话]
    end

    subgraph 方式二["远程调试端口"]
        C1[MCP服务器]
        C2[已有Chrome运行中]
        C3[通过端口连接]
        C4[创建Puppeteer会话]
    end

    subgraph 方式三["自动连接 Chrome 144+"]
        D1[MCP服务器]
        D2[用户启动Chrome]
        D3[自动检测可用实例]
        D4[建立连接]
    end

    subgraph 方式四["WebSocket连接"]
        E1[MCP服务器]
        E2[WebSocket端点]
        E3[自定义头连接]
        E4[创建会话]
    end

    A -->|默认配置| B1
    A -->|--browser-url| C1
    A -->|--autoConnect| D1
    A -->|--wsEndpoint| E1
```

### 2.3 性能分析数据流

性能分析是Chrome DevTools MCP的核心功能之一。该功能涉及多个组件之间的协作，包括性能跟踪记录、跟踪数据处理、CrUX数据获取等步骤。

```mermaid
flowchart TB
    subgraph 性能分析请求
        A[AI助手请求性能分析]
    end

    subgraph 跟踪记录阶段
        B1[performance_start_trace]
        B2[导航到目标页面]
        B3[启用DevTools性能追踪]
        B4[等待页面加载]
        B5[performance_stop_trace]
        B6[收集跟踪数据]
    end

    subgraph 数据处理阶段
        C1[Trace Processing模块]
        C2[提取性能指标]
        C3[分析渲染数据]
        C4[计算核心Web指标]
    end

    subgraph 可选CrUX数据
        D1[提取页面URL]
        D2[发送到CrUX API]
        D3[获取真实用户数据]
    end

    subgraph 结果生成
        E1[生成性能洞察]
        E2[格式化响应]
        E3[返回AI助手]
    end

    A --> B1
    B1 --> B2
    B2 --> B3
    B3 --> B4
    B4 --> B5
    B5 --> B6

    B6 --> C1
    C1 --> C2
    C2 --> C3
    C3 --> C4

    C4 --> D1
    D1 --> D2
    D2 --> D3

    D3 --> E1
    C4 --> E1
    E1 --> E2
    E2 --> E3
```

## 三、模块关系图

### 3.1 核心类依赖关系

Chrome DevTools MCP项目中的各个核心类之间存在明确的依赖关系。理解这些依赖关系有助于理解代码结构，以及在进行功能扩展时确保正确的设计决策。

```mermaid
classDiagram
    class McpServer {
        +name: string
        +version: string
        +registerTool()
        +setRequestHandler()
    }

    class McpContext {
        -browser: Browser
        -pages: McpPage[]
        -selectedPage: McpPage
        +from()
        +getSelectedMcpPage()
        +getPageById()
        +detectOpenDevToolsWindows()
    }

    class McpPage {
        -page: Page
        -id: string
        +evaluate()
        +screenshot()
        +click()
    }

    class McpResponse {
        -serverArgs: Arguments
        -textContent: string
        -structuredContent: object
        +setPage()
        +addText()
        +addImage()
        +addResource()
        +handle()
    }

    class BrowserManager {
        +ensureBrowserLaunched()
        +ensureBrowserConnected()
    }

    class ToolDefinition {
        +name: string
        +description: string
        +schema: object
        +annotations: object
        +handler: Function
    }

    McpServer --> McpContext: 创建
    McpServer --> ToolDefinition: 注册
    McpContext --> McpPage: 管理
    McpPage --> BrowserManager: 交互
    McpResponse --> McpPage: 格式化数据
    McpServer --> McpResponse: 处理响应
```

### 3.2 工具分类与组织

Chrome DevTools MCP将29个工具分为6个主要类别，每个类别负责特定功能域。这种分类方式使得工具易于理解和使用，同时也便于按需启用或禁用特定类别的工具。

```mermaid
flowchart TB
    subgraph 工具分类
        A[工具总览]
    end

    subgraph 输入自动化["输入自动化 (9个工具)"]
        A1[click]
        A2[drag]
        A3[fill]
        A4[fill_form]
        A5[handle_dialog]
        A6[hover]
        A7[press_key]
        A8[type_text]
        A9[upload_file]
    end

    subgraph 导航自动化["导航自动化 (6个工具)"]
        B1[close_page]
        B2[list_pages]
        B3[navigate_page]
        B4[new_page]
        B5[select_page]
        B6[wait_for]
    end

    subgraph 仿真["仿真 (2个工具)"]
        C1[emulate]
        C2[resize_page]
    end

    subgraph 性能["性能 (4个工具)"]
        D1[performance_analyze_insight]
        D2[performance_start_trace]
        D3[performance_stop_trace]
        D4[take_memory_snapshot]
    end

    subgraph 网络["网络 (2个工具)"]
        E1[get_network_request]
        E2[list_network_requests]
    end

    subgraph 调试["调试 (6个工具)"]
        F1[evaluate_script]
        F2[get_console_message]
        F3[lighthouse_audit]
        F4[list_console_messages]
        F5[take_screenshot]
        F6[take_snapshot]
    end

    A --> 输入自动化
    A --> 导航自动化
    A --> 仿真
    A --> 性能
    A --> 网络
    A --> 调试
```

### 3.3 工具处理流程

每个工具在执行时都遵循统一的处理流程，包括参数验证、上下文获取、实际执行、结果格式化等步骤。这种标准化流程确保了工具行为的一致性和可预测性。

```mermaid
flowchart LR
    subgraph 工具请求处理
        A[接收工具调用] --> B{参数验证}

        B -->|验证失败| C[返回错误]
        B -->|验证通过| D[获取页面上下文]

        D --> E[执行工具逻辑]
        E --> F{执行结果}

        F -->|失败| G[捕获异常]
        F -->|成功| H[格式化结果]

        G --> I[返回错误信息]
        H --> J[返回成功响应]
    end
```

## 四、用户交互流程

### 4.1 典型使用场景流程

Chrome DevTools MCP的主要使用场景是AI编码助手代表用户执行浏览器操作。以下流程图展示了一个典型场景的完整交互过程。

```mermaid
sequenceDiagram
    participant User as 用户
    participant AI as AI编码助手
    participant MCP as Chrome DevTools MCP
    participant Browser as Chrome浏览器
    participant Target as 目标网站

    User->>AI: "检查网站性能"

    AI->>MCP: 调用 performance_analyze_insight
    Note over MCP: 如果浏览器未启动<br/>自动启动Chrome

    MCP->>Browser: 启动/连接浏览器
    Browser-->>MCP: 连接就绪

    MCP->>Browser: navigate_page(网站URL)
    Browser->>Target: 访问网站
    Target-->>Browser: 返回页面内容

    MCP->>Browser: performance_start_trace
    MCP->>Browser: 等待页面加载
    MCP->>Browser: performance_stop_trace

    MCP->>Browser: 获取跟踪数据
    Browser-->>MCP: 跟踪数据

    MCP->>MCP: 分析性能数据
    MCP->>MCP: 获取CrUX数据(可选)

    MCP-->>AI: 返回分析结果

    AI-->>User: "发现以下性能问题：<br/>1. LCP时间过长<br/>2. 建议优化图片"
```

### 4.2 自动化操作流程

自动化操作是Chrome DevTools MCP的另一核心功能场景。以下流程展示了AI如何执行一系列复杂的用户交互操作。

```mermaid
sequenceDiagram
    participant AI as AI编码助手
    participant MCP as Chrome DevTools MCP
    participant WaitFor as WaitForHelper
    participant Browser as Chrome浏览器
    participant Page as 页面

    AI->>MCP: 执行测试流程

    rect rgb(240, 248, 255)
        Note over MCP,Browser: 步骤1：导航
    end

    MCP->>Browser: navigate_page(登录页)
    Browser->>Page: 加载页面
    Page-->>Browser: 页面就绪
    Browser-->>MCP: 导航完成

    rect rgb(240, 248, 255)
        Note over MCP,Browser: 步骤2：点击登录按钮
    end

    MCP->>Page: click(登录按钮选择器)
    WaitFor->>Page: 等待按钮可见
    Page-->>WaitFor: 按钮可见
    Page-->>Browser: 执行点击
    Browser-->>MCP: 点击完成

    rect rgb(240, 248, 255)
        Note over MCP,Browser: 步骤3：填写表单
    end

    MCP->>Page: fill_form(用户名, 密码)
    WaitFor->>Page: 等待表单出现
    Page-->>WaitFor: 表单就绪
    Page-->>Browser: 填写表单
    Browser-->>MCP: 表单填写完成

    rect rgb(240, 248, 255)
        Note over MCP,Browser: 步骤4：提交并验证
    end

    MCP->>Page: click(提交按钮)
    MCP->>WaitFor: wait_for(导航完成/元素出现)
    Page-->>WaitFor: 条件满足
    Browser-->>MCP: 提交成功

    MCP->>Browser: take_screenshot
    Browser-->>MCP: 返回截图

    MCP-->>AI: 返回执行结果
    AI-->>User: "测试完成，登录成功"
```

### 4.3 配置与初始化流程

在使用Chrome DevTools MCP之前，需要进行适当的配置。以下流程展示了从配置到首次使用的完整初始化过程。

```mermaid
flowchart TB
    subgraph 初始化阶段
        A[用户配置MCP客户端] --> B{选择连接方式}

        B -->|方式1| C[默认自动启动]
        B -->|方式2| D[远程调试端口]
        B -->|方式3| E[WebSocket连接]

        C --> F[MCP服务器启动]
        D --> F
        E --> F
    end

    subgraph 浏览器阶段
        F --> G{浏览器状态}

        G -->|未运行| H[启动Chrome]
        G -->|已运行| I[连接现有实例]

        H --> J[创建浏览器会话]
        I --> J

        J --> K[创建McpContext]
    end

    subgraph 工具注册阶段
        K --> L[加载工具定义]
        L --> M[注册到MCP服务器]
        M --> N[等待工具调用]
    end

    subgraph 使用阶段
        N --> O[AI助手发起请求]
        O --> P[执行工具]
        P --> Q[返回结果]
        Q --> O
    end
```

## 五、部署架构

### 5.1 单机部署架构

对于大多数使用场景，Chrome DevTools MCP采用单机部署架构。在这种模式下，MCP服务器和浏览器实例运行在同一台机器上，通过本地进程通信进行交互。

```mermaid
flowchart LR
    subgraph 用户机器
        subgraph 应用层
            A1[MCP客户端<br/>Claude/Cursor/Copilot]
        end

        subgraph ChromeDevToolsMCP
            A2[MCP服务器进程]
            A3[Node.js运行时]
        end

        subgraph 浏览器层
            A4[Chrome浏览器实例]
            A5[用户数据目录]
        end

        A1 <-->|MCP协议| A2
        A2 <-->|Puppeteer API| A4
        A4 <--> A5
    end
```

### 5.2 远程浏览器部署架构

在某些特殊场景下，可能需要让MCP服务器连接到远程运行的Chrome浏览器实例。这种架构主要用于沙箱环境、多用户共享浏览器资源等情况。

```mermaid
flowchart TB
    subgraph 用户机器
        A1[MCP客户端]
        A2[MCP服务器]
    end

    subgraph 远程机器
        B1[Chrome浏览器]
        B2[远程调试端口<br/>9222]
        B3[用户数据目录]
    end

    A1 <-->|MCP协议| A2
    A2 <-.->|网络连接| B2
    B2 <-->|DevTools协议| B1
    B1 <--> B3
```

### 5.3 多实例架构

在团队协作或高并发场景下，可能需要运行多个Chrome DevTools MCP实例。这种架构需要注意用户数据目录的管理，以避免实例之间的冲突。

```mermaid
flowchart TB
    subgraph 共享资源
        A1[Chrome浏览器安装]
    end

    subgraph 实例1
        B1[MCP服务器#1]
        B2[用户数据目录#1]
    end

    subgraph 实例2
        C1[MCP服务器#2]
        C2[用户数据目录#2]
    end

    subgraph 实例N
        D1[MCP服务器#N]
        D2[用户数据目录#N]
    end

    A1 --> B1
    A1 --> C1
    A1 --> D1

    B1 --> B2
    C1 --> C2
    D1 --> D2
```

## 六、数据存储与状态管理

### 6.1 用户数据目录管理

Chrome DevTools MCP使用Chrome的用户数据目录来存储浏览器配置、缓存、扩展等数据。理解用户数据目录的管理机制有助于优化使用体验和解决相关问题。

```mermaid
flowchart TB
    subgraph 用户数据目录
        A[~/.cache/chrome-devtools-mcp/]
    end

    subgraph 按渠道分离
        A --> B1[chrome-profile-stable/]
        A --> B2[chrome-profile-canary/]
        A --> B3[chrome-profile-beta/]
        A --> B4[chrome-profile-dev/]
    end

    subgraph 每个渠道的内容
        B1 --> C1[Default/]
        B1 --> C2[Local Storage/]
        B1 --> C3[Session Storage/]
        B1 --> C4[Cache/]
        B1 --> C5[Extensions/]
    end
```

### 6.2 临时隔离模式

使用`--isolated`参数时，系统会创建临时用户数据目录，并在浏览器关闭后自动清理。这种模式适合需要隔离性的使用场景。

```mermaid
flowchart LR
    A[启动MCP服务器<br/>--isolated=true] --> B[创建临时目录<br/>/tmp/chrome-devtools-xxx]
    B --> C[启动Chrome<br/>--user-data-dir=临时目录]
    C --> D[执行浏览器操作]
    D --> E[浏览器关闭]
    E --> F[清理临时目录]
```

## 七、总结

Chrome DevTools MCP的架构设计体现了现代微服务设计的最佳实践。通过清晰的层次划分、模块化的组件设计、标准化的接口定义，项目实现了高内聚、低耦合的目标。这种架构不仅便于维护和扩展，也为社区贡献新功能和修复问题提供了良好的基础。

理解项目的架构对于有效使用和定制Chrome DevTools MCP至关重要。无论是选择合适的连接方式、配置性能选项，还是排查问题，都需要建立在对系统架构的清晰认识之上。希望本文档能够帮助读者建立这种认识，更好地利用Chrome DevTools MCP提升开发效率。
