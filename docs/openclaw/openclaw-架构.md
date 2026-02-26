# OpenClaw 架构文档

## 1. 系统整体架构图

### 1.1 核心架构概览

```mermaid
graph TB
    subgraph "用户设备层"
        subgraph "macOS 平台"
            MenuBar["菜单栏应用\nOpenClaw.app"]
            VoiceWake["Voice Wake\n语音唤醒"]
            TalkMode["Talk Mode\n对话模式"]
            Canvas["Live Canvas\n可视化画布"]
        end

        subgraph "iOS 平台"
            IOSNode["iOS 节点"]
            IOSVoice["语音唤醒"]
            IOSCamera["相机/屏幕录制"]
        end

        subgraph "Android 平台"
            AndroidNode["Android 节点"]
            AndroidVoice["语音唤醒"]
            AndroidCamera["相机/屏幕录制"]
        end
    end

    subgraph "Gateway 控制平面"
        GW["Gateway 服务\nws://127.0.0.1:18789"]
        CLI["CLI 界面\nopenclaw 命令"]
        WebUI["Web UI\nControl UI + WebChat"]
    end

    subgraph "AI 运行时层"
        PI["Pi Agent 运行时\nRPC 模式"]
        Models["模型提供商\nAnthropic/OpenAI"]
        Skills["Skills 平台\nClawHub"]
    end

    subgraph "通道层 Channels"
        WhatsApp["WhatsApp\nBaileys"]
        Telegram["Telegram\ngrammY"]
        Slack["Slack\nBolt"]
        Discord["Discord\ndiscord.js"]
        GoogleChat["Google Chat\nChat API"]
        Signal["Signal\nsignal-cli"]
        IMessage["iMessage\nBlueBubbles"]
        Teams["Microsoft Teams\nBot Framework"]
        Matrix["Matrix\nSDK"]
        Zalo["Zalo\nSDK"]
    end

    subgraph "工具层 Tools"
        Browser["Browser Control\nChrome CDP"]
        CanvasTools["Canvas 工具\nA2UI"]
        NodeTools["Node 工具\n相机/屏幕/位置"]
        Cron["Cron 定时任务"]
        Webhook["Webhook 触发器"]
        Sessions["会话管理\nsessions_*"]
    end

    subgraph "数据层"
        Config["配置文件\n~/.openclaw/openclaw.json"]
        SessionsData["会话数据\n~/.openclaw/sessions/"]
        Credentials["凭证存储\n~/.openclaw/credentials/"]
        Workspace["工作区\n~/.openclaw/workspace/"]
        SkillsStore["技能存储\n~/.openclaw/workspace/skills/"]
    end

    MenuBar --> GW
    VoiceWake --> GW
    TalkMode --> GW
    Canvas --> GW

    IOSNode --> GW
    AndroidNode --> GW

    CLI --> GW
    WebUI --> GW

    GW --> PI
    PI --> Models
    PI --> Skills

    GW --> WhatsApp
    GW --> Telegram
    GW --> Slack
    GW --> Discord
    GW --> GoogleChat
    GW --> Signal
    GW --> IMessage
    GW --> Teams
    GW --> Matrix
    GW --> Zalo

    GW --> Browser
    GW --> CanvasTools
    GW --> NodeTools
    GW --> Cron
    GW --> Webhook
    GW --> Sessions

    GW --> Config
    GW --> SessionsData
    GW --> Credentials
    GW --> Workspace
    GW --> SkillsStore

    style GW fill:#e1f5fe
    style PI fill:#f3e5f5
    style Models fill:#fff3e0
```

### 1.2 消息流向架构

```mermaid
sequenceDiagram
    participant User as "用户"
    participant Channel as "消息通道"
    participant Gateway as "Gateway"
    participant Router as "消息路由器"
    participant Agent as "Pi Agent"
    participant Tools as "工具层"
    participant Response as "响应处理器"

    Note over User,Channel: 用户发送消息

    User->>Channel: 发送消息/语音/文件

    Channel->>Gateway: 接收消息事件

    Gateway->>Router: 路由到对应会话

    Router->>Router: 应用安全策略\n(dmPolicy/allowFrom)

    Router->>Agent: 转发消息到 Agent

    Note over Agent: Agent 处理消息

    Agent->>Tools: 调用工具\n(browser/canvas/camera等)

    Tools-->>Agent: 工具执行结果

    Agent->>Response: 生成响应

    Response->>Router: 处理响应\n(流式/非流式)

    Router->>Gateway: 路由响应

    Gateway->>Channel: 发送响应消息

    Channel->>User: 用户收到响应

    Note over User,Channel: 完成交互
```

## 2. 核心数据流图

### 2.1 消息处理数据流

```mermaid
flowchart LR
    subgraph "输入处理"
        Input["消息输入\n文本/语音/文件/链接"]
        Parse["消息解析\n提取内容/元数据"]
        Validate["安全验证\n配对检查/白名单"]
    end

    subgraph "会话处理"
        Session["会话管理\n查找/创建/更新"]
        Context["上下文管理\n历史记录/摘要"]
        Model["模型调用\n思考/生成"]
    end

    subgraph "工具执行"
        ToolsRouter["工具路由\n分发到各类工具"]
        Browser["浏览器控制"]
        Canvas["画布操作"]
        Media["媒体处理"]
        System["系统操作"]
        External["外部API"]
    end

    subgraph "输出处理"
        OutputGen["响应生成\n格式化/渲染"]
        Stream["流式处理\n分块发送"]
        Deliver["消息投递\n路由到通道"]
    end

    Input --> Parse
    Parse --> Validate
    Validate --> Session
    Session --> Context
    Context --> Model
    Model --> ToolsRouter
    ToolsRouter --> Browser
    ToolsRouter --> Canvas
    ToolsRouter --> Media
    ToolsRouter --> System
    ToolsRouter --> External
    Browser --> OutputGen
    Canvas --> OutputGen
    Media --> OutputGen
    System --> OutputGen
    External --> OutputGen
    OutputGen --> Stream
    Stream --> Deliver

    style Input fill:#e3f2fd
    style Session fill:#f3e5f5
    style ToolsRouter fill:#fff3e0
    style Deliver fill:#e8f5e9
```

### 2.2 媒体处理管道

```mermaid
flowchart TB
    subgraph "输入阶段"
        ImageIn["图片输入\nWhatsApp/Telegram/上传"]
        AudioIn["音频输入\n语音消息/录音"]
        VideoIn["视频输入\n视频消息/屏幕录制"]
    end

    subgraph "处理阶段"
        subgraph "图片处理"
            ImageParse["图片解析\nsharp 库"]
            ImageResize["尺寸调整\n压缩/缩放"]
            ImageOCR["OCR 提取\n(可选)"]
        end

        subgraph "音频处理"
            AudioParse["音频解析\n格式转换"]
            AudioTranscribe["语音转文字\n(可选)"]
            AudioTTS["语音合成\nElevenLabs/edge-tts"]
        end

        subgraph "视频处理"
            VideoParse["视频解析\n帧提取"]
            VideoThumb["缩略图生成"]
            VideoTranscribe["视频转文字\n(可选)"]
        end
    end

    subgraph "输出阶段"
        MediaStore["临时文件存储\n生命周期管理"]
        AIInput["输入到 AI\n作为上下文"]
        DeliverOut["分发给用户\n保持原始格式"]
    end

    ImageIn --> ImageParse
    ImageParse --> ImageResize
    ImageResize --> ImageOCR
    ImageOCR --> MediaStore

    AudioIn --> AudioParse
    AudioParse --> AudioTranscribe
    AudioTranscribe --> AudioTTS
    AudioTTS --> MediaStore

    VideoIn --> VideoParse
    VideoParse --> VideoThumb
    VideoParse --> VideoTranscribe
    VideoThumb --> MediaStore
    VideoTranscribe --> MediaStore

    MediaStore --> AIInput
    MediaStore --> DeliverOut

    style MediaStore fill:#fff8e1
```

## 3. 模块关系图

### 3.1 Gateway 核心模块依赖

```mermaid
graph TB
    subgraph "Gateway 核心"
        GWMain["gateway/index.ts\n入口文件"]
        GWServer["WebSocket 服务器\nws://127.0.0.1:18789"]
        GWConfig["配置管理\nConfig 模块"]
    end

    subgraph "会话管理"
        Sessions["sessions/ 会话模块"]
        SessionModel["会话模型\nmain/群组/激活模式"]
        SessionStore["会话存储\nJSONL 文件"]
    end

    subgraph "消息通道"
        Channels["channels/ 通道模块"]
        Routing["routing/ 路由模块"]
        ChannelBase["通道基类"]
        ChannelImpl["各平台实现"]
    end

    subgraph "Agent 集成"
        Agents["agents/ Agent 模块"]
        PiRuntime["Pi Agent 运行时"]
        ToolsDef["工具定义\nSchema"]
        Streaming["流式处理"]
    end

    subgraph "工具系统"
        Tools["tools/ 工具模块"]
        Browser["browser/ 浏览器"]
        Canvas["canvas-host/ 画布"]
        Nodes["nodes/ 设备节点"]
        Cron["cron/ 定时任务"]
    end

    subgraph "辅助功能"
        CLI["cli/ 命令行"]
        Web["web/ Web 服务"]
        TUI["tui/ 终端 UI"]
        Media["media/ 媒体处理"]
    end

    GWMain --> GWServer
    GWMain --> GWConfig
    GWServer --> Sessions
    GWServer --> Channels
    GWServer --> Agents
    GWServer --> Tools
    GWServer --> CLI
    GWServer --> Web
    GWServer --> TUI
    GWServer --> Media

    Sessions --> SessionModel
    Sessions --> SessionStore

    Channels --> Routing
    Channels --> ChannelBase
    Channels --> ChannelImpl

    Agents --> PiRuntime
    Agents --> ToolsDef
    Agents --> Streaming

    Tools --> Browser
    Tools --> Canvas
    Tools --> Nodes
    Tools --> Cron

    style GWMain fill:#bbdefb
    style GWServer fill:#bbdefb
    style Sessions fill:#f8bbd9
    style Channels fill:#c8e6c9
    style Agents fill:#ffe0b2
    style Tools fill:#b2dfdb
```

### 3.2 通道模块结构

```mermaid
graph TB
    subgraph "核心抽象"
        BaseChannel["通道基类\nChannel.ts"]
        ChannelRouter["通道路由器\nrouting/index.ts"]
        ConfigSchema["配置 Schema"]
    end

    subgraph "内置通道 Core"
        WhatsApp["whatsapp/\nBaileys Web"]
        Telegram["telegram/\ngrammY"]
        Slack["slack/\nBolt"]
        Discord["discord/\ndiscord.js"]
        GoogleChat["web/\nGoogle Chat"]
        Signal["signal/\nsignal-cli"]
        IMessage["imessage/\nBlueBubbles"]
    end

    subgraph "扩展通道 Extensions"
        MSTeams["extensions/msteams/\nMicrosoft Teams"]
        Matrix["extensions/matrix/\nMatrix"]
        Zalo["extensions/zalo/\nZalo"]
        ZaloUser["extensions/zalouser/\nZalo Personal"]
        VoiceCall["extensions/voice-call/\n语音通话"]
    end

    subgraph "通用功能"
        Allowlist["allowFrom/白名单"]
        Pairing["配对管理\npairing/"]
        DMControl["DM 策略\ndmPolicy"]
        Chunking["消息分块\nchunk/"]
    end

    BaseChannel --> WhatsApp
    BaseChannel --> Telegram
    BaseChannel --> Slack
    BaseChannel --> Discord
    BaseChannel --> GoogleChat
    BaseChannel --> Signal
    BaseChannel --> IMessage

    ChannelRouter --> BaseChannel
    ChannelRouter --> MSTeams
    ChannelRouter --> Matrix
    ChannelRouter --> Zalo
    ChannelRouter --> ZaloUser
    ChannelRouter --> VoiceCall

    WhatsApp --> Allowlist
    Telegram --> Allowlist
    Slack --> Allowlist
    Discord --> Allowlist

    Allowlist --> Pairing
    Allowlist --> DMControl

    GoogleChat --> Chunking
    Slack --> Chunking

    style BaseChannel fill:#e3f2fd
    style ChannelRouter fill:#fff3e0
    style WhatsApp fill:#e8f5e9
    style Telegram fill:#e8f5e9
```

## 4. 用户交互流程图

### 4.1 典型用户交互流程

```mermaid
flowchart TD
    Start([用户开始交互])

    subgraph "认证阶段"
        Auth["用户认证\n(首次/重新认证)"]
        Credentials["凭证管理\n~/.openclaw/credentials/"]
        Paired["检查配对状态"]
    end

    subgraph "消息接收"
        Receive["接收消息\n(任意通道)"]
        Parse["解析消息内容\n文本/语音/文件/链接"]
        Route["路由到会话\n应用安全策略"]
    end

    subgraph "Agent 处理"
        Context["加载会话上下文\n历史/摘要"]
        Thinking["思考阶段\n(可选)"]
        Generate["生成响应\n工具调用"]
        Execute["执行工具\n浏览器/画布/相机等"]
        Format["格式化输出"]
    end

    subgraph "消息发送"
        Deliver["投递响应\n流式/非流式"]
        Update["更新会话状态\nToken 计数/日志"]
        Store["持久化存储"]
    end

    End([交互完成])

    Start --> Auth
    Auth --> Credentials
    Credentials --> Paired

    Paired -->|未配对| Receive
    Paired -->|已配对| Receive

    Receive --> Parse
    Parse --> Route
    Route --> Context

    Context --> Thinking
    Thinking --> Generate
    Generate --> Execute
    Execute --> Format

    Format --> Deliver
    Deliver --> Update
    Update --> Store
    Store --> End

    style Auth fill:#e3f2fd
    style Context fill:#f3e5f5
    style Execute fill:#fff3e0
    style Deliver fill:#e8f5e9
```

### 4.2 多设备协同流程

```mermaid
sequenceDiagram
    participant Mac as "macOS 设备\n菜单栏应用"
    participant Gateway as "Gateway 服务"
    participant Pi as "Pi Agent"
    participant iOS as "iOS 节点"
    participant Android as "Android 节点"

    Note over Mac, Gateway: 设备发现与连接

    Mac->>Gateway: 连接 Gateway WS\n(启动时自动)
    iOS->>Gateway: Bonjour 配对\n(首次连接)
    Android->>Gateway: Bridge 配对\n(首次连接)

    Gateway->>Mac: 返回节点列表\n(node.list)
    Gateway->>iOS: 返回节点列表
    Gateway->>Android: 返回节点列表

    Note over Mac, Pi: 语音唤醒流程

    Mac->>Mac: Voice Wake 检测\n("Hey OpenClaw")
    Mac->>Pi: 转发语音\n(openclaw-mac agent --message "...")

    Pi->>Pi: 处理请求
    Pi->>Gateway: 工具调用\n(node.invoke: system.run)

    Gateway->>Mac: 执行系统命令\n(返回 stdout/stderr)
    Mac-->>Pi: 命令结果
    Pi-->>Mac: 对话响应

    Note over iOS, Gateway: 相机调用流程

    iOS->>Gateway: 节点可用事件\n(node.describe)
    Pi->>Gateway: 工具调用\n(node.invoke: camera.snap)
    Gateway->>iOS: 转发相机请求
    iOS->>iOS: 拍摄照片
    iOS-->>Gateway: 返回图片
    Gateway-->>Pi: 图片数据
    Pi-->>iOS: 处理结果

    Note over Android, Gateway: 屏幕录制流程

    Android->>Gateway: 屏幕录制事件
    Pi->>Gateway: 工具调用\n(node.invoke: screen.record)
    Gateway->>Android: 开始录制
    Android-->>Gateway: 视频流
    Gateway-->>Pi: 视频数据
```

### 4.3 群组消息处理流程

```mermaid
flowchart TD
    subgraph "群组消息触发"
        GroupMsg["群组消息\n收到@提及/回复"]
        CheckMention["检查是否需要提及"]
        CheckReply["检查是否回复Bot"]
    end

    subgraph "激活决策"
        Activation["激活策略\nactivation Mode"]
        MentionReq["mention|always\n(策略决定)"]
        Ignore["忽略消息\n(不需要响应)"]
        Process["处理消息\n(进入对话流程)"]
    end

    subgraph "消息处理"
        Extract["提取消息内容\n(清理@/标签)"]
        RouteToAgent["路由到 Agent"]
        SendResponse["发送响应\n(回复/新消息)"]
    end

    subgraph "特殊规则"
        OwnerOnly["仅所有者命令\n/restart/配置"]
        GroupRules["群组规则\n(各平台差异)"]
    end

    GroupMsg --> CheckMention
    CheckMention -->|需要提及| CheckReply
    CheckMention -->|不需要提及| Ignore

    CheckReply -->|是回复| Process
    CheckReply -->|不是回复| Activation

    Activation --> MentionReq
    MentionReq -->|always| Process
    MentionReq -->|mention| CheckMention

    Process --> Extract
    Extract --> RouteToAgent
    RouteToAgent --> SendResponse

    SendResponse --> OwnerOnly

    Process --> GroupRules
    OwnerOnly -.->|管理员命令| Process

    style Process fill:#e8f5e9
    style Activation fill:#fff3e0
```

## 5. 部署架构图

### 5.1 本地部署架构（推荐）

```mermaid
graph TB
    subgraph "用户设备"
        subgraph "macOS 主机"
            App["OpenClaw.app\n菜单栏应用"]
            Gateway["Gateway 服务\n本地 127.0.0.1:18789"]
            Data["本地数据\n~/.openclaw/"]
        end

        Node1["iOS 设备\n(可选)"]
        Node2["Android 设备\n(可选)"]
    end

    subgraph "云服务"
        Claude["Anthropic Claude\nAPI"]
        ChatGPT["OpenAI ChatGPT\nAPI"]
        Tailscale["Tailscale\n(可选远程访问)"]
        ClawHub["ClawHub\n技能注册表"]
    end

    subgraph "消息平台"
        WA["WhatsApp"]
        TG["Telegram"]
        Slack["Slack"]
        Discord["Discord"]
    end

    App --> Gateway
    Gateway --> Claude
    Gateway --> ChatGPT
    Gateway --> Tailscale

    Gateway --> WA
    Gateway --> TG
    Gateway --> Slack
    Gateway --> Discord

    Gateway --> Data

    Node1 -->|Bonjour| Gateway
    Node2 -->|Bridge| Gateway

    Tailscale -->|远程访问| Gateway
    Gateway --> ClawHub

    style Gateway fill:#bbdefb
    style Data fill:#f3e5f5
```

### 5.2 服务器远程部署架构

```mermaid
graph TB
    subgraph "服务器环境"
        Linux["Linux 服务器\n(Ubuntu/Debian)"]
        Docker["Docker 容器"]
        Gateway["Gateway 服务\n0.0.0.0:18789"]
        Data["持久化数据\n volumes/"]
    end

    subgraph "访问层"
        Tailscale["Tailscale Serve\n(推荐)"]
        SSH["SSH 隧道\n(备选)"]
        Web["Web UI\n via Gateway"]
    end

    subgraph "客户端"
        Mac["远程 macOS\n(菜单栏应用)"]
        Browser["浏览器\n(访问 Web UI)"]
    end

    subgraph "消息平台"
        WA["WhatsApp"]
        TG["Telegram"]
        Slack["Slack"]
        Discord["Discord"]
    end

    subgraph "AI 服务"
        Claude["Anthropic API"]
        OpenAI["OpenAI API"]
    end

    Linux --> Docker
    Docker --> Gateway
    Gateway --> Data

    Gateway --> Tailscale
    Gateway --> SSH
    Gateway --> Web

    Tailscale --> Mac
    SSH --> Mac
    Web --> Browser

    Gateway --> WA
    Gateway --> TG
    Gateway --> Slack
    Gateway --> Discord

    Gateway --> Claude
    Gateway --> OpenAI

    style Linux fill:#e0e0e0
    style Docker fill:#bbdefb
    style Gateway fill:#bbdefb
```

### 5.3 Docker 沙箱部署架构

```mermaid
graph TB
    subgraph "主机环境"
        Gateway["Gateway 主服务"]
        DockerEngine["Docker Engine"]
        HostData["主机数据\n~/.openclaw/"]
    end

    subgraph "主会话容器"
        MainSession["主会话容器\n(bash 权限)"]
        HostTools["主机工具\n(gateway/browser/canvas)"]
    end

    subgraph "沙箱会话容器"
        subgraph "沙箱 1"
            Sandbox1["沙箱容器 1"]
            DockerBash["Docker bash"]
            DenyBrowser["禁止: browser"]
            DenyCanvas["禁止: canvas"]
        end

        subgraph "沙箱 2"
            Sandbox2["沙箱容器 2"]
            AllowList["允许: read/write"]
            AllowSessions["允许: sessions_*"]
        end
    end

    subgraph "安全策略"
        PolicyMain["主会话: 无沙箱\n(完整权限)"]
        PolicySandbox["群组会话: 沙箱\n(受限权限)"]
        DenyList["禁止工具清单\ndiscord/gateway/cron等"]
    end

    Gateway --> HostData

    Gateway --> MainSession
    MainSession --> HostTools
    MainSession -.->|安全策略| PolicyMain

    Gateway --> Sandbox1
    Gateway --> Sandbox2

    Sandbox1 --> DockerBash
    Sandbox2 --> DockerBash

    DockerBash -.->|应用| PolicySandbox

    PolicySandbox -.-> DenyBrowser
    PolicySandbox -.-> DenyCanvas
    PolicySandbox -.-> DenyList

    style Gateway fill:#bbdefb
    style MainSession fill:#c8e6c9
    style Sandbox1 fill:#fff3e0
    style Sandbox2 fill:#fff3e0
```

## 6. 技术栈依赖图

```mermaid
graph TB
    subgraph "运行时层"
        Node["Node.js 22+"]
        Bun["Bun (开发/测试)"]
        Pnpm["pnpm 10+"]
    end

    subgraph "核心框架"
        TypeScript["TypeScript 5.x"]
        Hono["Hono 4.x\nWeb 框架"]
        Express["Express 5.x\n备用 Web"]
        Ws["ws 8.x\nWebSocket"]
    end

    subgraph "AI 集成"
        PiCore["@mariozechner/pi-agent-core"]
        PiAI["@mariozechner/pi-ai"]
        PiCoding["@mariozechner/pi-coding-agent"]
        PiTUI["@mariozechner/pi-tui"]
    end

    subgraph "消息通道 SDK"
        grammY["grammY\nTelegram"]
        SlackBolt["@slack/bolt\nSlack"]
        DiscordJS["discord.js\nDiscord"]
        Baileys["@whiskeysockets/baileys\nWhatsApp"]
        SignalUtils["signal-utils\nSignal"]
    end

    subgraph "工具库"
        Playwright["playwright-core\n浏览器控制"]
        Sharp["sharp\n图片处理"]
        Croner["croner\n定时任务"]
        Puppeteer["(备用) Chrome CDP"]
    end

    subgraph "测试框架"
        Vitest["Vitest 4.x\n单元/集成测试"]
        Coverage["V8 Coverage\n70% 阈值"]
    end

    subgraph "代码质量"
        Oxlint["Oxlint\nLinting"]
        Oxfmt["Oxfmt\n格式化"]
        SwiftLint["SwiftLint\nSwift 代码"]
        PreCommit["Pre-commit Hooks"]
    end

    Node --> TypeScript
    Bun --> TypeScript
    Pnpm --> Node

    TypeScript --> Hono
    TypeScript --> Express
    TypeScript --> Ws

    Hono --> grammY
    Hono --> SlackBolt
    Hono --> DiscordJS
    Express --> Baileys

    TypeScript --> PiCore
    TypeScript --> PiAI
    PiAI --> PiCoding
    PiAI --> PiTUI

    TypeScript --> Playwright
    TypeScript --> Sharp
    TypeScript --> Croner

    TypeScript --> Vitest
    Vitest --> Coverage

    TypeScript --> Oxlint
    TypeScript --> Oxfmt
    Oxlint --> PreCommit

    style Node fill:#e3f2fd
    style TypeScript fill:#bbdefb
    style PiCore fill:#f3e5f5
    style PiAI fill:#f3e5f5
```

## 7. 安全架构图

```mermaid
graph TB
    subgraph "安全边界"
        subgraph "可信区域 (用户设备)"
            Gateway["Gateway 服务"]
            LocalData["本地数据"]
            Agent["Pi Agent"]
            TrustedTools["可信工具\nbrowser/canvas/camera"]
        end

        subgraph "半可信区域 (Docker 沙箱)"
            Sandbox["沙箱容器"]
            LimitedTools["受限工具\nbash/process/read"]
            DeniedTools["禁止工具\nbrowser/gateway/cron"]
        end

        subgraph "外部区域 (不可信)"
            MessageChannels["消息通道\nDM/群组消息"]
            ExternalAPIs["外部 API\n模型提供商"]
            UntrustedInput["用户输入\n任意内容"]
        end
    end

    subgraph "安全策略"
        Auth["认证\nOAuth/API Key"]
        Pairing["配对控制\ndmPolicy"]
        Allowlist["白名单\nallowFrom"]
        SandboxMode["沙箱模式\nagents.defaults.sandbox.mode"]
        ToolPolicy["工具策略\nallowlist/denylist"]
    end

    MessageChannels --> Auth
    MessageChannels --> Pairing
    Pairing --> Allowlist

    UntrustedInput -->|输入验证| Gateway
    Gateway -->|策略检查| ToolPolicy

    ToolPolicy -->|主会话| TrustedTools
    ToolPolicy -->|群组会话| Sandbox

    Sandbox --> LimitedTools
    Sandbox -.->|阻止| DeniedTools

    ExternalAPIs -->|加密传输| Gateway

    style Gateway fill:#bbdefb
    style Sandbox fill:#fff3e0
    style Auth fill:#c8e6c9
    style Pairing fill:#c8e6c9
```
