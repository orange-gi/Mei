# VibeTunnel 架构文档

## 1. 系统整体架构图

VibeTunnel 采用三层架构设计，各层之间通过明确定义的接口进行通信。这种架构确保了各组件的独立性和可维护性，同时提供了流畅的用户体验。

```mermaid
graph TB
    subgraph 用户层["用户层"]
        Browser["浏览器"]
        Mobile["移动设备"]
        Terminal["终端命令行"]
    end

    subgraph macOS应用层["macOS 菜单栏应用层"]
        MenuBar["菜单栏 UI"]
        ServerManager["服务器管理器"]
        SessionMonitor["会话监控器"]
        TTYForwardManager["TTY 转发管理器"]
        Settings["设置面板"]
    end

    subgraph 服务层["Node.js/Bun 服务器层"]
        HTTPServer["HTTP 服务器"]
        WebSocketServer["WebSocket 服务器"]

        subgraph 核心服务["核心服务"]
            AuthService["认证服务"]
            SessionManager["会话管理器"]
            PTYManager["PTY 管理器"]
            TerminalManager["终端管理器"]
        end

        subgraph 路由层["API 路由层"]
            SessionRoutes["会话路由"]
            AuthRoutes["认证路由"]
            ConfigRoutes["配置路由"]
            RemoteRoutes["远程访问路由"]
            FileRoutes["文件路由"]
            GitRoutes["Git 路由"]
            WorktreeRoutes["工作树路由"]
            PushRoutes["推送通知路由"]
        end

        subgraph 远程服务["远程访问服务"]
            TailscaleService["Tailscale 服务"]
            NgrokService["Ngrok 服务"]
            CloudflareService["Cloudflare 服务"]
            MDNSService["mDNS 服务"]
        end
    end

    subgraph 数据层["数据与系统层"]
        UnixSocket["UNIX 套接字"]
        PTY["伪终端"]
        ControlDir["控制目录 ~/.vibetunnel"]
        SystemAuth["系统认证"]
        VAPIDKeys["VAPID 密钥"]
    end

    Browser --> HTTPServer
    Mobile --> HTTPServer
    Terminal -->|vt 命令| TTYForwardManager

    MenuBar --> ServerManager
    MenuBar --> SessionMonitor
    MenuBar --> Settings

    ServerManager --> HTTPServer
    ServerManager --> TailscaleService

    HTTPServer --> AuthRoutes
    HTTPServer --> SessionRoutes
    HTTPServer --> ConfigRoutes
    HTTPServer --> WebSocketServer

    WebSocketServer --> SessionManager
    WebSocketServer --> PTYManager

    SessionManager --> PTYManager
    PTYManager --> PTY

    AuthRoutes --> AuthService
    AuthService --> SystemAuth

    ConfigRoutes --> VAPIDKeys
    PushRoutes --> VAPIDKeys

    TailscaleService -->|HTTPS 代理| HTTPServer
    NgrokService -->|隧道| HTTPServer
    CloudflareService -->|隧道| HTTPServer
    MDNSService -->|服务发现| Browser

    classDef macos fill:#007AFF,stroke:#333,stroke-width:2px,color:#fff
    classDef server fill:#339933,stroke:#333,stroke-width:2px,color:#fff
    classDef service fill:#FF9500,stroke:#333,stroke-width:2px,color:#fff
    classDef data fill:#5856D6,stroke:#333,stroke-width:2px,color:#fff
    classDef user fill:#34C759,stroke:#333,stroke-width:2px,color:#fff

    class MenuBar,ServerManager,SessionMonitor,TTYForwardManager,Settings macos
    class HTTPServer,WebSocketServer server
    class AuthService,SessionManager,PTYManager,TerminalManager,TailscaleService,NgrokService,CloudflareService,MDNSService service
    class UnixSocket,PTY,ControlDir,SystemAuth,VAPIDKeys data
    class Browser,Mobile,Terminal user
```

该架构图展示了 VibeTunnel 的核心设计理念：用户通过多种渠道（浏览器、移动设备、终端命令行）与系统交互，由 macOS 菜单栏应用统一管理，通过高性能 Node.js 服务器处理请求，最终连接到系统层的伪终端和认证系统。各层之间通过清晰定义的接口通信，保证了系统的模块化和可扩展性。

## 2. 核心数据流图

数据在 VibeTunnel 系统中以多种形式流转，包括用户输入、终端输出、认证令牌、配置信息等。理解这些数据流对于掌握系统工作原理至关重要。

```mermaid
sequenceDiagram
    participant User as 用户
    participant Browser as 浏览器
    participant WS as WebSocket 服务器
    participant PTY as PTY 管理器
    participant Terminal as 伪终端进程
    participant Auth as 认证服务
    participant Remote as 远程访问服务

    Note over User,Remote: 会话创建流程
    User->>Browser: 1. 访问 http://localhost:4020
    Browser->>Auth: 2. 认证请求
    Auth-->>Browser: 3. 认证结果

    User->>Browser: 4. 创建新会话请求
    Browser->>WS: 5. POST /api/sessions
    WS->>PTY: 6. 创建伪终端
    PTY-->>WS: 7. 会话 ID
    WS-->>Browser: 8. 会话信息

    Note over User,Terminal: 终端交互流程
    User->>Browser: 9. 输入命令
    Browser->>WS: 10. WebSocket 消息
    WS->>PTY: 11. 写入输入
    PTY->>Terminal: 12. 转发输入

    loop 终端输出
        Terminal->>PTY: 13. 输出数据
        PTY->>WS: 14. 转发输出
        WS->>Browser: 15. WebSocket 消息
        Browser->>User: 16. 渲染终端
    end

    Note over User,Remote: 远程访问流程（可选）
    User->>Remote: 17. 访问公共 URL
    Remote-->>User: 18. 重定向到本地
    Remote->>WS: 19. 代理请求
    WS-->>Remote: 20. 响应数据
    Remote-->>User: 21. 返回结果

    Note over User,WS: 会话结束流程
    User->>Browser: 22. 关闭会话
    Browser->>WS: 23. DELETE /api/sessions/:id
    WS->>PTY: 24. 终止进程
    PTY-->>WS: 25. 退出码
    WS->>Auth: 26. 记录日志
    WS-->>Browser: 27. 关闭确认
```

该数据流图清晰地展示了用户与 VibeTunnel 系统交互的完整生命周期。从最初的认证请求开始，到会话创建、终端交互、远程访问（可选），最后到会话结束，每个步骤都标注了对应的 API 调用和组件交互。这种端到端的流程展示有助于开发者理解整个系统的工作机制。

## 3. 类与模块关系图

VibeTunnel 的代码库包含多个核心模块，它们之间存在复杂的依赖关系。以下图表展示了主要模块及其相互关系。

```mermaid
classDiagram
    %% 服务器核心类
    class Server {
        +startVibeTunnelServer()
        +createApp()
        -config: Config
        -server: http.Server
        +shutdown()
    }

    class AuthService {
        +authenticate()
        +validateToken()
        +createSession()
        +destroySession()
    }

    class SessionManager {
        +createSession()
        +getSession()
        +listSessions()
        +deleteSession()
        +cleanupOldVersions()
    }

    class PTYManager {
        +initialize()
        +spawnPty()
        +writeToPty()
        +resizePty()
        +killPty()
        +setSessionMonitor()
    }

    class TerminalManager {
        +createTerminal()
        +resizeTerminal()
        +cleanup()
    }

    class ConfigService {
        +startWatching()
        +stopWatching()
        +getConfig()
        +updateConfig()
    }

    %% 路由类
    class SessionRoutes {
        +createSessionRoutes()
        -GET /sessions
        -POST /sessions
        -DELETE /sessions/:id
    }

    class AuthRoutes {
        +createAuthRoutes()
        -POST /login
        -POST /logout
        -GET /me
    }

    class RemoteRoutes {
        +createRemoteRoutes()
        -GET /remotes
        -POST /remotes/:id/connect
    }

    class WorktreeRoutes {
        +createWorktreeRoutes()
        -POST /worktrees
        -GET /worktrees
        -DELETE /worktrees/:path
    }

    %% WebSocket 相关
    class WsV3Hub {
        +handleClientConnection()
        +handleMessage()
        +broadcast()
        +castOutput()
    }

    class ApiSocketServer {
        +start()
        +stop()
        +setServerInfo()
    }

    %% 远程服务类
    class TailscaleServeService {
        +start()
        +stop()
        +getStatus()
        +isRunning()
    }

    class NgrokService {
        +start()
        +stop()
        +getPublicUrl()
    }

    class CloudflareService {
        +start()
        +stop()
        +getPublicUrl()
    }

    class PushNotificationService {
        +initialize()
        +sendNotification()
        +subscribe()
        +cleanupInactive()
    }

    %% 依赖关系
    Server --> AuthService
    Server --> SessionRoutes
    Server --> AuthRoutes
    Server --> RemoteRoutes
    Server --> ConfigService
    Server --> PTYManager
    Server --> TerminalManager
    Server --> WsV3Hub
    Server --> ApiSocketServer
    Server --> TailscaleServeService
    Server --> NgrokService
    Server --> CloudflareService
    Server --> PushNotificationService

    SessionRoutes --> SessionManager
    SessionRoutes --> PTYManager
    SessionRoutes --> TerminalManager

    PTYManager --> SessionManager
    WsV3Hub --> PTYManager
    WsV3Hub --> TerminalManager
```

该类图展示了 VibeTunnel 服务器端的核心类结构。Server 类作为入口点，协调所有服务和路由。AuthService、SessionManager、PTYManager 和 TerminalManager 构成了核心业务逻辑层。各种路由类（SessionRoutes、AuthRoutes 等）处理 HTTP API 请求。WsV3Hub 和 ApiSocketServer 处理 WebSocket 通信。远程服务类（TailscaleServeService、NgrokService 等）提供可选的远程访问功能。

## 4. 用户交互流程图

用户与 VibeTunnel 的交互涉及多个步骤，包括首次安装、日常使用和配置调整等场景。

```mermaid
flowchart TB
    subgraph 安装流程["安装阶段"]
        A1[开始安装] --> A2{安装方式}
        A2 -->|macOS 应用| A3[下载 .dmg 文件]
        A2 -->|npm 包| A4[npm install -g vibetunnel]
        A3 --> A5[拖动到应用程序文件夹]
        A5 --> A6[首次启动]
        A6 --> A7[权限请求]
        A7 --> A8[菜单栏图标出现]
        A4 --> A8
    end

    subgraph 认证流程["认证阶段"]
        B1[打开浏览器] --> B2{认证模式}
        B2 -->|系统认证| B3[输入系统用户名密码]
        B2 -->|SSH 密钥| B4[使用 SSH 密钥认证]
        B2 -->|无认证| B5[直接访问]
        B3 --> B6[验证凭证]
        B4 --> B6
        B6 --> B7[生成会话令牌]
        B7 --> B8[进入仪表板]
    end

    subgraph 会话流程["日常使用阶段"]
        C1[在终端中执行命令] --> C2{vt 命令}
        C2 -->|转发命令| C3[创建新会话]
        C3 --> C4[显示在仪表板]
        C2 -->|打开 Shell| C5[交互式会话]
        C5 --> C4
        C4 --> C6[查看实时输出]
        C6 --> C7{需要远程访问?}
        C7 -->|是| C8[配置远程访问]
        C7 -->|否| C9[继续本地使用]
        C8 --> C10[通过 Tailscale/ngrok 访问]
    end

    subgraph 配置流程["配置阶段"]
        D1[打开设置面板] --> D2{配置选项}
        D2 -->|认证设置| D3[选择认证模式]
        D2 -->|远程访问| D4[配置 Tailscale/ngrok]
        D2 -->|通知设置| D5[配置推送通知]
        D2 -->|会话设置| D6[配置录制选项]
        D3 --> D7[保存配置]
        D4 --> D7
        D5 --> D7
        D6 --> D7
    end

    subgraph 结束流程["结束使用"]
        E1[关闭浏览器] --> E2[会话保持运行]
        E2 --> E3{需要完全退出?}
        E3 -->|否| E4[稍后继续]
        E3 -->|是| E5[点击菜单栏退出]
        E5 --> E6[停止服务器]
        E6 --> E7[进程退出]
    end

    A8 --> B1
    B8 --> C1
    C9 --> D1
    C10 --> D1
    E4 --> C1

    style A1 fill:#34C759
    style B1 fill:#007AFF
    style C1 fill:#FF9500
    style D1 fill:#5856D6
    style E1 fill:#FF3B30
```

该流程图全面展示了用户与 VibeTunnel 交互的所有阶段。从安装开始，经过认证、日常会话使用、配置调整，到最后的退出，每个阶段都包含了关键决策点（如认证模式选择、远程访问配置等）和可能的分支路径。这种端到端的流程展示帮助用户理解完整的使用生命周期。

## 5. 部署架构图

VibeTunnel 支持多种部署方式，从本地开发到生产环境都有对应的最佳实践。

```mermaid
graph TB
    subgraph 本地部署["本地部署架构"]
        subgraph Mac设备["macOS 设备"]
            subgraph 菜单栏["菜单栏应用"]
                MenuApp["VibeTunnel App"]
            end

            subgraph 服务进程["后台服务"]
                NodeServer["Node.js 服务器"]
                PTYProc["PTY 进程池"]
                WSServer["WebSocket 服务器"]
            end

            subgraph 远程访问["远程访问选项"]
                Tailscale["Tailscale"]
                Ngrok["ngrok"]
                Cloudflare["Cloudflare"]
            end

            MenuApp --> NodeServer
            NodeServer --> PTYProc
            NodeServer --> WSServer
            NodeServer --> Tailscale
            NodeServer --> Ngrok
            NodeServer --> Cloudflare
        end

        subgraph 访问设备["访问设备"]
            LocalBrowser["本地浏览器"]
            MobileBrowser["移动浏览器"]
            RemoteBrowser["远程浏览器"]
        end

        LocalBrowser -->|localhost:4020| NodeServer
        MobileBrowser -->|Tailscale 网络| Tailscale
        RemoteBrowser -->|公网 URL| Ngrok
        RemoteBrowser -->|公网 URL| Cloudflare
    end

    subgraph npm部署["npm 包部署架构"]
        subgraph Linux服务器["Linux 服务器"]
            NodeServerNPM["vibetunnel 服务器"]
            SystemD["systemd 服务"]
            PTYProcNPM["PTY 进程"]

            subgraph 反向代理["反向代理"]
                Nginx["nginx/Caddy"]
                SSL["SSL 证书"]
            end

            NodeServerNPM --> PTYProcNPM
            SystemD --> NodeServerNPM
            NodeServerNPM --> Nginx
            Nginx --> SSL
        end

        AccessClients["远程客户端"] --> Nginx
    end

    subgraph HQ架构["HQ（总部）模式架构"]
        subgraph HQ服务器["HQ 服务器"]
            HQServer["VibeTunnel HQ"]
            RemoteReg["远程服务器注册表"]
            WebUI["HQ Web UI"]
        end

        subgraph 远程服务器["远程服务器"]
            Remote1["远程服务器 1"]
            Remote2["远程服务器 2"]
            RemoteN["远程服务器 N"]
        end

        Remote1 -->|注册连接| HQServer
        Remote2 -->|注册连接| HQServer
        RemoteN -->|注册连接| HQServer
        HQServer --> WebUI
        Admin["管理员"] --> WebUI
    end

    %% 样式定义
    classDef local fill:#34C759,stroke:#333,stroke-width:2px,color:#fff
    classDef npm fill:#339933,stroke:#333,stroke-width:2px,color:#fff
    classDef hq fill:#FF9500,stroke:#333,stroke-width:2px,color:#fff
    classDef process fill:#007AFF,stroke:#333,stroke-width:2px,color:#fff
    classDef storage fill:#5856D6,stroke:#333,stroke-width:2px,color:#fff

    class Mac设备,Linux服务器,HQ服务器 local
    class MenuApp,NodeServer,NodeServerNPM,HQServer process
    class PTYProc,PTYProcNPM process
    class WSServer,Nginx,SystemD process
    class Tailscale,Ngrok,Cloudflare,SSL process
    class RemoteReg,WebUI storage
```

该部署架构图展示了 VibeTunnel 支持的三种主要部署模式：

1. **本地部署**：macOS 设备上运行菜单栏应用和 Node.js 服务器，支持多种远程访问方式（Tailscale、ngrok、Cloudflare），可从本地或远程浏览器访问。

2. **npm 包部署**：在 Linux 服务器上通过 npm 安装运行，可以配置 systemd 服务实现开机自启动，配合反向代理（nginx/Caddy）和 SSL 证书提供生产级别的 Web 服务。

3. **HQ 模式部署**：中心化的总部架构，多个远程服务器注册到 HQ 服务器，管理员通过 HQ Web UI 统一管理和监控所有远程服务器。

## 6. 组件依赖关系图

了解各组件之间的依赖关系对于系统维护和故障排查至关重要。

```mermaid
graph LR
    subgraph 核心依赖["核心组件依赖"]
        Server["server.ts\n(服务器入口)"]
        Auth["认证中间件"]
        Routes["API 路由"]
        Services["核心服务"]
    end

    subgraph 工具库["工具库"]
        Logger["日志工具"]
        Config["配置工具"]
        Version["版本信息"]
    end

    subgraph 外部依赖["外部依赖"]
        Express["Express.js"]
        WS["WebSocket"]
        Chalk["Chalk"]
        Helmet["Helmet"]
        Compression["Compression"]
        Fs["Node.js fs"]
        Path["Node.js path"]
        OS["Node.js os"]
        UUID["UUID"]
    end

    Server --> Auth
    Server --> Routes
    Server --> Services
    Server --> Logger
    Server --> Config
    Server --> Version

    Auth -->|使用| Express
    Auth -->|使用| Fs

    Routes -->|使用| Express
    Services -->|使用| WS
    Services -->|使用| Fs
    Services -->|使用| Path

    Logger -->|使用| Chalk
    Logger -->|使用| Fs

    Config -->|使用| Fs
    Config -->|使用| Path

    Express -->|依赖| Helmet
    Express -->|依赖| Compression

    Services -->|使用| UUID

    classDef core fill:#007AFF,stroke:#333,stroke-width:2px,color:#fff
    classDef util fill:#5856D6,stroke:#333,stroke-width:2px,color:#fff
    classDef external fill:#FF9500,stroke:#333,stroke-width:2px,color:#fff

    class Server,Auth,Routes,Services core
    class Logger,Config,Version util
    class Express,WS,Chalk,Helmet,Compression,Fs,Path,OS,UUID external
```

该依赖图展示了 server.ts 文件如何组织其依赖关系。核心组件（认证中间件、API 路由、核心服务）直接由服务器入口文件管理和协调。工具库（Logger、Config、Version）提供通用的辅助功能。外部依赖包括 Node.js 内置模块（fs、path、os）和第三方库（Express、WebSocket、Chalk 等）。

## 7. 会话生命周期状态图

每个终端会话在 VibeTunnel 中都有明确的状态生命周期，理解这些状态有助于调试和监控。

```mermaid
stateDiagram-v2
    [*] --> 等待创建: 用户请求创建会话

    等待创建 --> 创建中: PTYManager.spawnPty()

    创建中 --> 运行中: 伪终端成功创建
    创建中 --> 错误: 伪终端创建失败

    运行中 --> 活跃: 检测到用户活动
    运行中 --> 空闲: 无用户活动

    活跃 --> 运行中: 活动结束
    空闲 --> 活跃: 检测到新输入

    运行中 --> 暂停: 用户暂停会话
    暂停 --> 运行中: 用户恢复会话

    运行中 --> 终止中: 用户请求终止
    运行中 --> 终止中: 命令执行完成
    运行中 --> 终止中: 进程异常退出

    终止中 --> 清理中: 记录退出状态

    清理中 --> 已终止: 清理资源

    错误 --> 已终止: 记录错误日志

    已终止 --> [*]: 完成清理

    state 运行中 {
        [*] --> 交互模式
        交互模式 --> 脚本执行: 执行脚本
        脚本执行 --> 交互模式: 脚本结束
    }

    state 活跃 {
        [*] --> 输入等待
        输入等待 --> 输出处理: 收到输入
        输出处理 --> 输入等待: 输出完成
    }
```

该状态图详细描述了会话的完整生命周期。从「等待创建」开始，经过「创建中」到达「运行」状态。在运行状态下，会话可以在「活跃」和「空闲」之间切换。最终，会话会进入「终止中」和「清理中」状态，最终完成清理并标记为「已终止」。状态图还展示了运行状态的子状态（交互模式、脚本执行）和活跃状态的子状态（输入等待、输出处理）。

## 8. 远程访问架构对比图

VibeTunnel 支持多种远程访问方案，每种方案都有其适用场景和技术特点。

```mermaid
graph TB
    subgraph 本地环境["本地环境"]
        Mac["macOS 设备"]
        Server["VibeTunnel 服务器\nlocalhost:4020"]
    end

    subgraph Tailscale方案["Tailscale 方案（推荐）"]
        subgraph Tailscale网络["Tailscale 私有网络"]
            Mac2["macOS 设备"]
            iPad["iPad"]
            Phone["手机"]
        end

        subgraph Tailscale云["Tailscale 云服务"]
            WireGuard["WireGuard 隧道"]
            DNS["Tailscale DNS"]
        end

        iPad -->|HTTPS| WireGuard
        Phone -->|HTTPS| WireGuard
        WireGuard -->|代理| Server
        DNS -->|服务发现| iPad
        DNS -->|服务发现| Phone
    end

    subgraph Ngrok方案["Ngrok 方案"]
        User["远程用户"]
        Ngrok云["Ngrok 云服务"]
        Tunnel["ngrok 隧道"]

        User -->|HTTPS| Ngrok云
        Ngrok云 -->|隧道| Tunnel
        Tunnel -->|代理| Server
    end

    subgraph Cloudflare方案["Cloudflare 方案"]
        User2["远程用户"]
        Cloudflare["Cloudflare 网络"]
        QuickTunnel["Quick Tunnel"]

        User2 -->|HTTPS| Cloudflare
        Cloudflare -->|隧道| QuickTunnel
        QuickTunnel -->|代理| Server
    end

    Tailscale网络 -->|加密隧道| Mac2
    Tailscale方案 -->|零配置| Ngrok方案
    Ngrok方案 -->|临时共享| Cloudflare方案

    classDef local fill:#34C759,stroke:#333,stroke-width:2px,color:#fff
    classDef recommended fill:#007AFF,stroke:#333,stroke-width:2px,color:#fff
    classDef cloud fill:#FF9500,stroke:#333,stroke-width:2px,color:#fff

    class Mac,Server local
    class Tailscale网络,Tailscale云 recommended
    class Ngrok云,Cloudflare cloud
```

该架构对比图直观地展示了三种远程访问方案的网络拓扑和访问路径。Tailscale 方案通过创建加密的 WireGuard 隧道实现点对点通信，是最推荐的方式。Ngrok 和 Cloudflare 方案都通过云服务中转流量，适用于临时共享或没有 Tailscale 网络的场景。每种方案的安全性、配置复杂度和适用场景都有所不同，用户可以根据实际需求选择。

---

以上架构文档全面展示了 VibeTunnel 系统的设计理念、组件关系、数据流和部署模式。通过这些图表，开发者可以快速理解系统的整体架构，在进行代码修改或故障排查时能够快速定位相关组件和依赖关系。
