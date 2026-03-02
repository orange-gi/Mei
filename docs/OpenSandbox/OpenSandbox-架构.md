# OpenSandbox 架构详解

本文档通过 Mermaid 图表从多个角度展示 OpenSandbox 的系统架构，帮助开发者深入理解各组件之间的关系和数据流转方式。

## 系统整体架构图

OpenSandbox 采用四层架构设计，从上到下依次为 SDK 层、规范层、运行时层和沙箱实例层。各层之间通过 HTTP API 进行通信，职责边界清晰，便于独立扩展和维护。

```mermaid
flowchart TB
    subgraph SDKs["SDKs 层 - 多语言客户端库"]
        direction TB
        Python["Python SDK"]
        Kotlin["Java/Kotlin SDK"]
        TypeScript["JavaScript/TypeScript SDK"]
        CSharp["C#/.NET SDK"]
    end

    subgraph Specs["规范层 - OpenAPI 定义"]
        direction TB
        Lifecycle["Sandbox Lifecycle API<br/>/specs/sandbox-lifecycle.yml"]
        Execution["Sandbox Execution API<br/>/specs/execd-api.yaml"]
    end

    subgraph Runtime["运行时层 - 服务器实现"]
        direction TB
        Server["FastAPI Server<br/>/server/"]
        Docker["Docker Runtime"]
        Kubernetes["Kubernetes Runtime"]
        Router["Sandbox Router"]
    end

    subgraph Instances["沙箱实例层 - 运行中的容器"]
        direction TB
        Execd["execd 守护进程<br/>/components/execd/"]
        Jupyter["Jupyter Server"]
        UserProcess["用户进程"]
    end

    SDKs -->|HTTP API| Runtime
    Specs -->|定义契约| Runtime
    Runtime -->|创建和管理| Instances
    Execd -->|WebSocket| Jupyter
    Execd -->|exec| UserProcess

    style SDKs fill:#e1f5fe,stroke:#01579b
    style Specs fill:#e8f5e9,stroke:#2e7d32
    style Runtime fill:#fff3e0,stroke:#e65100
    style Instances fill:#fce4ec,stroke:#c2185b
```

## 核心数据流图

OpenSandbox 中的数据流主要分为三类：沙箱创建流程、代码执行流程和文件操作流程。这些流程展示了系统内部各组件如何协同工作。

### 沙箱创建流程

当用户通过 SDK 发起创建沙箱请求时，系统会经历一系列操作来完成沙箱实例的初始化。以下流程图展示了从请求创建到沙箱就绪的完整过程：

```mermaid
sequenceDiagram
    participant Client as 客户端 SDK
    participant Server as Sandbox Server
    participant Docker as Docker Runtime
    participant Container as Sandbox Container
    participant Execd as execd 守护进程
    participant Jupyter as Jupyter Server

    Client->>Server: POST /sandboxes<br/>创建沙箱请求
    Server->>Docker: 1. 拉取容器镜像
    Docker-->>Server: 镜像拉取完成
    Server->>Docker: 2. 注入 execd 二进制文件
    Server->>Docker: 3. 创建容器<br/>修改 entrypoint
    Docker->>Container: 启动容器
    Container->>Execd: 4. 启动 execd 守护进程
    Execd->>Jupyter: 5. 启动 Jupyter Server
    Container->>Container: 6. 执行用户入口进程
    Execd-->>Server: 沙箱就绪
    Server-->>Client: 返回沙箱实例<br/>包含执行端点
```

### 代码执行流程

代码执行是 OpenSandbox 的核心功能之一。通过 Code Interpreter SDK，用户可以在沙箱中执行多语言代码。以下流程图展示了代码从提交到返回结果的完整过程：

```mermaid
sequenceDiagram
    participant User as 用户应用
    participant CIP SDK as Code Interpreter SDK
    participant Execd as execd 守护进程
    participant Jupyter as Jupyter Server
    participant Kernel as Jupyter Kernel<br/>(Python/Java/JS等)

    User->>CIP SDK: 1. 执行代码请求
    CIP SDK->>Execd: 2. POST /code/context<br/>创建执行会话
    Execd-->>CIP SDK: 会话创建成功
    CIP SDK->>Execd: 3. POST /code<br/>提交代码执行
    Execd->>Jupyter: 4. WebSocket 发送<br/>execute_request
    Jupyter->>Kernel: 5. 转发到内核
    Kernel->>Kernel: 6. 执行代码
    Kernel-->>Jupyter: 7. 执行结果<br/>stream 事件
    Jupyter-->>Execd: 8. WebSocket 消息
    Execd-->>CIP SDK: 9. SSE 事件流
    CIP SDK-->>User: 10. 返回执行结果
```

### 文件操作流程

文件操作通过 Filesystem SDK 提供，支持上传、下载、搜索等多种操作。以下是文件上传的流程：

```mermaid
flowchart LR
    subgraph Client["客户端"]
        FS_SDK["Filesystem SDK"]
    end

    subgraph Sandbox["沙箱实例"]
        Execd["execd 守护进程"]
        Filesystem["容器文件系统"]
    end

    FS_SDK -->|POST /files/upload| Execd
    Execd -->|写入文件| Filesystem
    Execd -->|设置权限| Filesystem

    style Client fill:#e1f5fe,stroke:#01579b
    style Sandbox fill:#fce4ec,stroke:#c2185b
```

## 类与模块关系图

OpenSandbox 的代码结构清晰，主要分为服务端、SDK、组件和沙箱实现几个大部分。以下图表展示了各模块之间的依赖关系：

```mermaid
flowchart TB
    subgraph Server["Server 模块 - Python FastAPI"]
        direction TB
        ServerMain["src/main.py<br/>主入口"]
        ServerAPI["src/api<br/>API 路由"]
        ServerRuntime["src/runtime<br/>运行时接口"]
        ServerDocker["src/runtime/docker.py<br/>Docker 实现"]
        ServerK8s["src/runtime/kubernetes.py<br/>K8s 实现"]
    end

    subgraph Components["核心组件"]
        direction TB
        Execd["components/execd<br/>执行守护进程"]
        Ingress["components/ingress<br/>入口网关"]
        Egress["components/egress<br/>出口控制"]
    end

    subgraph SDKs["多语言 SDK"]
        direction TB
        PySDK["sdks/sandbox/python<br/>Python SDK"]
        KtSDK["sdks/sandbox/kotlin<br/>Kotlin SDK"]
        TSSDK["sdks/sandbox/javascript<br/>TS SDK"]
    end

    subgraph Specs["API 规范"]
        direction TB
        LifecycleSpec["specs/sandbox-lifecycle.yml<br/>生命周期API"]
        ExecSpec["specs/execd-api.yaml<br/>执行API"]
    end

    ServerMain --> ServerAPI
    ServerAPI --> ServerRuntime
    ServerRuntime --> ServerDocker
    ServerRuntime --> ServerK8s
    ServerRuntime --> Execd

    SDKs -->|HTTP 调用| Server
    Specs -->|定义契约| Server
    Specs -->|定义契约| Execd

    style Server fill:#fff3e0,stroke:#e65100
    style Components fill:#e8f5e9,stroke:#2e7d32
    style SDKs fill:#e1f5fe,stroke:#01579b
    style Specs fill:#f3e5f5,stroke:#7b1fa2
```

## 用户交互流程图

不同的用户场景对应着不同的交互流程。以下图表展示了典型用户场景下，系统各组件如何响应用户操作。

### 基础沙箱操作流程

基础沙箱操作包括创建、使用和销毁三个阶段。以下流程图展示了完整的交互序列：

```mermaid
flowchart TD
    Start([用户开始]) --> Create["创建沙箱<br/>Sandbox.create()"]

    Create -->|POST /sandboxes| Server{"Server 处理"}

    Server -->|拉取镜像| Docker["Docker Hub<br/>或私有仓库"]
    Docker -->|返回镜像| Server
    Server -->|创建容器| Docker
    Docker -->|容器运行| Running[沙箱运行中]

    Running --> Use["使用沙箱"]

    Use --> Commands["执行命令<br/>sandbox.commands.run()"]
    Commands -->|POST /command| Execd
    Execd -->|返回结果| Commands

    Use --> Files["文件操作<br/>sandbox.files.read/write"]
    Files -->|POST /files/*| Execd
    Execd -->|返回结果| Files

    Use --> Code["代码执行<br/>interpreter.codes.run()"]
    Code -->|POST /code| Execd
    Execd -->|Jupyter协议| Jupyter["Jupyter Server"]
    Jupyter -->|返回结果| Execd
    Execd -->|SSE流| Code

    Use --> Destroy["销毁沙箱<br/>sandbox.kill()"]
    Destroy -->|DELETE /sandboxes/{id}| Server
    Server -->|清理容器| Docker

    Destroy --> End([流程结束])

    style Start fill:#e3f2fd
    style Running fill:#c8e6c9
    style End fill:#ffcdd2
```

### AI Agent 集成场景

OpenSandbox 广泛用于 AI Agent 的代码执行场景。以下图表展示了典型的集成模式：

```mermaid
flowchart LR
    subgraph AI_System["AI Agent 系统"]
        Agent["AI Agent<br/>(Claude/GPT/Gemini)"]
        Tool["Agent Tools<br/>工具层"]
    end

    subgraph OpenSandbox["OpenSandbox 平台"]
        SDK["Sandbox SDK"]
        Server["Sandbox Server"]
        Container["沙箱容器"]
    end

    Agent -->|"生成代码"| Tool
    Tool -->|"调用工具"| SDK
    SDK -->|"API 调用"| Server
    Server -->|"管理"| Container

    Container -->|"执行结果"| Server
    Server -->|"返回结果"| SDK
    SDK -->|"结果解析"| Tool
    Tool -->|"执行反馈"| Agent

    style AI_System fill:#e8eaf6,stroke:#3f51b5
    style OpenSandbox fill:#e0f2f1,stroke:#00695c
```

## 部署架构图

OpenSandbox 支持多种部署模式，从本地开发到大规模生产环境都有完善的解决方案。

### Docker 本地部署架构

对于本地开发和测试环境，Docker 部署是最简单的方式：

```mermaid
flowchart TB
    subgraph Local["本地开发环境"]
        Developer["开发者机器"]

        subgraph DockerHost["Docker Host"]
            DockerEngine["Docker Engine"]

            subgraph Containers["容器"]
                SandboxServer["opensandbox-server<br/>沙箱服务器"]
                Sandbox1["沙箱实例 1<br/>(code-interpreter)"]
                Sandbox2["沙箱实例 2<br/>(chrome)"]
                SandboxN["沙箱实例 N"]
            end

            DockerEngine --> SandboxServer
            DockerEngine --> Sandbox1
            DockerEngine --> Sandbox2
            DockerEngine --> SandboxN
        end

        Developer -->|启动| SandboxServer
        Developer -->|使用| Sandbox1
    end

    style Local fill:#f5f5f5,stroke:#616161
    style DockerHost fill:#fff8e1,stroke:#ff8f00
    style Containers fill:#ffecb3,stroke:#ff6f00
```

### Kubernetes 分布式部署架构

对于大规模生产环境，Kubernetes 部署提供了水平扩展和高可用能力：

```mermaid
flowchart TB
    subgraph K8s["Kubernetes 集群"]
        subgraph ControlPlane["控制平面"]
            APIServer["Kubernetes API Server"]
        end

        subgraph WorkerNodes["工作节点"]
            Node1["Node 1"]
            Node2["Node 2"]
            NodeN["Node N"]
        end

        subgraph Services["Service 层"]
            LB["Load Balancer"]
            Ingress["Ingress Controller"]
        end

        subgraph Workloads["工作负载"]
            ServerPod["opensandbox-server<br/>Deployment"]
            SandboxPods["沙箱 Pod 们<br/>Dynamic Provisioning"]
        end

        LB --> Ingress
        Ingress --> ServerPod
        ServerPod -->|动态创建| SandboxPods

        APIServer -->|管理| ServerPod
        APIServer -->|管理| SandboxPods

        Node1 -.->|运行| ServerPod
        Node2 -.->|运行| SandboxPods
        NodeN -.->|运行| SandboxPods
    end

    Client["外部客户端"] --> LB

    style K8s fill:#fce4ec,stroke:#c2185b
    style ControlPlane fill:#e1f5fe,stroke:#01579b
    style WorkerNodes fill:#e8f5e9,stroke:#2e7d32
    style Services fill:#fff3e0,stroke:#e65100
    style Workloads fill:#f3e5f5,stroke:#7b1fa2
```

## 网络架构图

OpenSandbox 的网络架构设计考虑了安全性和灵活性，提供了入口网关和出口控制两层的网络策略。

### 整体网络拓扑

```mermaid
flowchart LR
    subgraph External["外部网络"]
        User["用户/AI Agent"]
        Internet["互联网"]
    end

    subgraph DMZ["DMZ 区域"]
        Ingress["Ingress Gateway<br/>/components/ingress/"]
        Router["Sandbox Router<br/>路由服务"]
    end

    subgraph Secure["安全区域"]
        Server["Sandbox Server"]
    end

    subgraph SandboxNet["沙箱网络"]
        Sandbox1["沙箱实例 1"]
        Sandbox2["沙箱实例 2"]
        Sandbox3["沙箱实例 3"]
    end

    subgraph Egress["出口控制"]
        EgressCtrl["Egress Controller<br/>/components/egress/"]
    end

    User -->|HTTPS| Internet
    Internet -->|请求| Ingress
    Ingress -->|路由| Router
    Router -->|转发| Sandbox1
    Router -->|转发| Sandbox2
    Router -->|转发| Sandbox3

    Server -->|管理| Sandbox1
    Server -->|管理| Sandbox2
    Server -->|管理| Sandbox3

    Sandbox1 -->|受限出口| EgressCtrl
    Sandbox2 -->|受限出口| EgressCtrl
    Sandbox3 -->|受限出口| EgressCtrl

    EgressCtrl -->|允许的流量| Internet

    style External fill:#e3f2fd,stroke:#1976d2
    style DMZ fill:#fff8e1,stroke:#ff8f00
    style Secure fill:#e8f5e9,stroke:#388e3c
    style SandboxNet fill:#fce4ec,stroke:#c2185b
    style Egress fill:#f3e5f5,stroke:#7b1fa2
```

## 技术栈概览

OpenSandbox 在技术选型上采用了多种编程语言和框架，以充分发挥各技术的优势：

```mermaid
flowchart LR
    subgraph Frontend["前端/客户端"]
        Python["Python<br/>SDK"]
        Kotlin["Kotlin<br/>SDK"]
        TypeScript["TypeScript<br/>SDK"]
        CSharp["C#<br/>SDK"]
    end

    subgraph Backend["后端服务"]
        FastAPI["Python FastAPI<br/>服务器"]
        Go["Go 1.24+<br/>execd 守护进程"]
    end

    subgraph Infrastructure["基础设施"]
        Docker["Docker"]
        K8s["Kubernetes"]
    end

    Python --> FastAPI
    Kotlin --> FastAPI
    TypeScript --> FastAPI
    CSharp --> FastAPI

    FastAPI --> Docker
    Go --> Docker
    Docker --> K8s

    style Frontend fill:#e1f5fe,stroke:#01579b
    style Backend fill:#fff3e0,stroke:#e65100
    style Infrastructure fill:#e8f5e9,stroke:#2e7d32
```

## 总结

通过以上多个维度的架构图表，我们可以清晰地看到 OpenSandbox 的设计理念和实现细节。四层架构确保了各组件之间的职责分离，协议优先的设计使得多语言 SDK 和自定义运行时成为可能，完善的数据流设计保证了代码执行、文件操作等核心功能的稳定可靠。无论是本地开发测试还是大规模生产部署，OpenSandbox 都能提供合适的解决方案。
