# Remotion 架构图集

## 1. 系统整体架构图

### 核心架构概览

```mermaid
graph TB
    subgraph 用户层 User Layer
        UI[Remotion Studio UI]
        CLI[命令行工具]
        API[Node.js API]
        Player[Remotion Player]
    end

    subgraph 开发层 Development Layer
        React[React Components]
        Composition[Composition 定义]
        Props[Props 模式]
        Hooks[Remotion Hooks]
    end

    subgraph 渲染层 Rendering Layer
        Browser[Chromium Browser]
        FFmpeg[FFmpeg Engine]
        Rust[Rust Optimized Core]
        Canvas[Canvas/WebGL]
    end

    subgraph 部署层 Deployment Layer
        Local[本地渲染]
        Lambda[AWS Lambda]
        CloudRun[Google Cloud Run]
    end

    UI --> React
    CLI --> React
    API --> React
    Player --> React

    React --> Composition
    Composition --> Props
    Hooks --> React

    React --> Browser
    Browser --> Canvas
    FFmpeg --> Rust
    Rust --> FFmpeg

    Browser --> Local
    Lambda --> Local
    CloudRun --> Local
```

### 详细组件架构

```mermaid
graph LR
    subgraph 项目结构 Project Structure
        Config[remotion.config.ts]
        Compositions[compositions/]
        Components[components/]
        Assets[assets/]
        Utils[utils/]
    end

    subgraph 核心模块 Core Modules
        Runtime[@remotion/runtime]
        Player[@remotion/player]
        Studio[@remotion/studio]
        Renderer[@remotion/renderer]
    end

    subgraph 扩展包 Extension Packages
        Shapes[@remotion/shapes]
        Rive[@remotion/rive]
        Tailwind[@remotion/tailwind]
        Lambda[@remotion/lambda]
    end

    Compositions --> Runtime
    Components --> Runtime
    Utils --> Runtime

    Runtime --> Player
    Runtime --> Studio
    Runtime --> Renderer

    Config --> Renderer
    Assets --> Renderer
```

## 2. 核心数据流图

### 视频渲染数据流

```mermaid
flowchart TD
    A[开始渲染请求] --> B{渲染模式选择}

    B -->|本地渲染| C[初始化浏览器实例]
    B -->|Lambda 渲染| D[部署到 AWS Lambda]
    B -->|Cloud Run| E[部署到 Cloud Run]

    C --> F[加载 React 应用]
    D --> G[创建 Lambda 函数]
    E --> H[创建容器实例]

    F --> I[创建 Composition]
    G --> I
    H --> I

    I --> J[解析组件结构]
    J --> K[计算 Props 和元数据]

    K --> L{数据类型}
    L -->|静态数据| M[直接渲染]
    L -->|动态数据| N[calculateMetadata]
    N --> O[获取外部数据]
    O --> M

    M --> P[渲染帧序列]
    P --> Q[处理动画和过渡]

    Q --> R[应用样式和特效]
    R --> S[生成帧图像]

    S --> T{输出格式}
    T -->|视频| U[FFmpeg 编码]
    T -->|图像| V[图像格式转换]

    U --> W[生成视频文件]
    V --> X[生成静态图像]

    W --> Y[输出文件]
    X --> Y

    Y --> Z[渲染完成]
```

### Props 数据流

```mermaid
flowchart LR
    A[外部输入数据] --> B{Zod Schema 验证}
    B -->|验证通过| C[类型转换]
    B -->|验证失败| D[错误处理]

    C --> E[传递到组件]
    E --> F[渲染计算]

    F --> G[UI 更新]
    G --> H[最终输出]

    D --> I[显示错误信息]
    I --> J[阻止渲染]
```

### Studio 交互数据流

```mermaid
sequenceDiagram
    participant U as 用户
    participant S as Remotion Studio
    participant P as Props 编辑器
    participant R as 渲染引擎

    U->>S: 打开项目
    S->>U: 显示时间线和预览

    U->>S: 修改 props
    S->>P: 更新 props 值
    P->>S: 验证数据

    S->>U: 实时预览更新
    U->>S: 点击渲染按钮

    S->>R: 发起渲染任务
    R-->>S: 返回进度
    R-->>S: 渲染完成

    S->>U: 显示输出文件
```

## 3. 模块依赖关系图

### 核心包依赖关系

```mermaid
graph TB
    subgraph 核心 Core
        main[remotion 主包]
        runtime[@remotion/runtime]
        renderer[@remotion/renderer]
    end

    subgraph 工具 Utilities
        player[@remotion/player]
        studio[@remotion/studio]
        cli[@remotion/cli]
    end

    subgraph 媒体 Media
        audio[音频处理]
        video[视频处理]
        images[图像处理]
    end

    subgraph 云服务 Cloud
        lambda[@remotion/lambda]
        cloudrun[@remotion/cloudrun]
    end

    main --> runtime
    main --> cli
    runtime --> renderer

    player --> main
    studio --> main

    audio --> renderer
    video --> renderer
    images --> renderer

    lambda --> main
    cloudrun --> main
```

### 组件依赖关系

```mermaid
graph LR
    subgraph 基础组件 Basic
        Composition[Composition]
        useVideoConfig[useVideoConfig]
        useCurrentFrame[useCurrentFrame]
    end

    subgraph 动画组件 Animation
        useSpring[useSpring]
        useVelocity[useVelocity]
        animate[animate function]
    end

    subgraph 媒体组件 Media
        Video[Video]
        OffthreadVideo[OffthreadVideo]
        Audio[Audio]
        Image[Image]
    end

    subgraph 布局组件 Layout
        AbsoluteFill[AbsoluteFill]
        Sequence[Sequence]
        Loop[Loop]
        Series[Series]
    end

    useVideoConfig --> Composition
    useCurrentFrame --> Composition

    useSpring --> useVelocity
    animate --> useSpring

    Video --> OffthreadVideo
    Audio --> Video
    Image --> Video

    AbsoluteFill --> Sequence
    Sequence --> Loop
    Series --> Sequence
```

## 4. 用户交互流程图

### 创建新项目流程

```mermaid
flowchart TD
    A[用户执行 create-video] --> B{包管理器选择}

    B -->|npm| C[npx create-video@latest]
    B -->|pnpm| D[pnpm create video]
    B -->|yarn| E[yarn create video]
    B -->|bun| F[bun create video]

    C --> G[下载模板]
    D --> G
    E --> G
    F --> G

    G --> H[选择模板类型]
    H -->|Hello World| I[基础模板]
    H -->|Blank| J[空白模板]
    H -->|其他模板| K[指定模板]

    I --> L[安装依赖]
    J --> L
    K --> L

    L --> M[初始化完成]
    M --> N[运行 npm run dev]
    N --> O[启动 Remotion Studio]
```

### 视频制作交互流程

```mermaid
flowchart TD
    A[打开 Remotion Studio] --> B{操作选择}

    B -->|编辑代码| C[修改组件源码]
    B -->|调整参数| D[编辑 Props]
    B -->|预览效果| E[播放预览]
    B -->|调整时间| F[拖动时间轴]

    C --> G{自动刷新}
    D --> G
    E --> H[更新预览]
    F --> I[跳转到帧]

    G --> H
    I --> H
    H --> J{满意?}

    J -->|否| B
    J -->|是| K[点击渲染]

    K --> L[配置渲染选项]
    L --> M[开始渲染]

    M --> N{渲染成功?}
    N -->|是| O[输出视频文件]
    N -->|否| P[查看错误信息]
    P --> Q[修复问题]
    Q --> M
```

### Lambda 部署流程

```mermaid
flowchart TD
    A[准备 Lambda 部署] --> B[配置区域和内存]
    B --> C[部署函数]

    C --> D{部署方式}
    D -->|首次部署| E[npx remotion lambda deploy]
    D -->|更新代码| F[npx remotion lambda update]

    E --> G[上传代码到 S3]
    F --> G

    G --> H[创建/更新 Lambda 函数]
    H --> I[等待部署完成]

    I --> J[触发远程渲染]
    J --> K[返回渲染结果]

    K --> L[下载输出文件]
```

## 5. 部署架构图

### 多平台部署架构

```mermaid
graph TB
    subgraph 本地开发环境 Local Dev
        Dev[开发者的机器]
        Studio[Remotion Studio]
        Browser[Chromium 实例]
    end

    subgraph AWS 云环境 AWS Cloud
        S3[S3 存储桶]
        Lambda[AWS Lambda 函数]
        CloudWatch[CloudWatch 日志]
    end

    subgraph Google Cloud
        CloudRun[Cloud Run 服务]
        Artifact[Artifact Registry]
    end

    subgraph 输出 Output
        VideoFiles[视频文件]
        Images[静态图像]
        CDN[CDN 加速]
    end

    Dev -->|运行开发| Studio
    Studio -->|预览| Browser

    Dev -->|本地渲染| Browser

    Browser -->|编码视频| S3
    S3 -->|存储输出| VideoFiles

    Lambda -->|远程渲染| S3
    CloudWatch -->|日志| Lambda

    CloudRun -->|容器渲染| S3
    Artifact -->|镜像| CloudRun

    S3 -->|分发| CDN
    VideoFiles --> CDN
    Images --> CDN
```

### 无服务器渲染架构

```mermaid
graph LR
    subgraph 客户端 Client
        Request[渲染请求]
        Config[渲染配置]
    end

    subgraph AWS Lambda
        API[API Gateway]
        Router[路由处理]
        Renderer[渲染函数]
        Storage[S3 存储]
    end

    subgraph 渲染进程
        Chrome[Chrome 实例]
        FFmpeg[FFmpeg 引擎]
        Rust[Rust 核心]
    end

    Request --> API
    Config --> API

    API --> Router
    Router --> Renderer

    Renderer --> Chrome
    Chrome --> FFmpeg
    FFmpeg --> Rust

    Renderer --> Storage

    Storage --> Output[输出文件]
```

## 6. 性能优化架构

### 渲染性能优化

```mermaid
graph TB
    subgraph 优化前 Before
        A[渲染请求] --> B[单线程处理]
        B --> C[重复解码]
        C --> D[性能瓶颈]
    end

    subgraph 优化机制 Optimization
        E[OffthreadVideo]
        F[Rust 加速]
        G[帧缓存]
        H[并行处理]
    end

    subgraph 优化后 After
        I[渲染请求] --> J[优化处理]
        J --> K[性能提升]
        K --> L[2倍速度提升]
    end

    A -.-> E
    A -.-> F
    A -.-> G
    A -.-> H

    E --> J
    F --> J
    G --> J
    H --> J
```

---

**文档说明**：
- 以上架构图使用 Mermaid 语法编写，可直接在支持 Mermaid 的 Markdown 查看器中渲染
- 图表展示了 Remotion 从开发到部署的完整架构流程
- 涵盖了核心组件、数据流、用户交互和部署架构

**作者**: Matrix Agent
**最后更新**: 2026-01-23
