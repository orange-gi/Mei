# System Prompts Leaks 架构分析

## 系统整体架构

```mermaid
graph TB
    subgraph "AI服务提供商"
        A1[Anthropic Claude]
        A2[OpenAI GPT]
        A3[Google Gemini]
        A4[xAI Grok]
        A5[Perplexity]
        A6[Proton]
    end

    subgraph "提示词收集层"
        C1[社区贡献]
        C2[逆向工程]
        C3[API分析]
        C4[用户分享]
    end

    subgraph "存储层"
        S1[Markdown文件]
        S2[XML文件]
        S3[文本文件]
    end

    subgraph "分析层"
        P1[版本对比]
        P2[结构分析]
        P3[模式提取]
        P4[变更追踪]
    end

    subgraph "应用层"
        U1[研究分析]
        U2[学习参考]
        U3[工具开发]
        U4[安全审计]
    end

    C1 --> S1
    C2 --> S1
    C3 --> S1
    C4 --> S1
    
    A1 --> C2
    A2 --> C2
    A3 --> C2
    A4 --> C2
    
    S1 --> P1
    S1 --> P2
    S1 --> P3
    S1 --> P4
    
    P1 --> U1
    P2 --> U2
    P3 --> U3
    P4 --> U4
```

## 核心数据流

```mermaid
flowchart LR
    subgraph "原始数据源"
        R1[AI系统响应]
        R2[API调用日志]
        R3[用户报告]
    end

    subgraph "提取过程"
        E1[提示词识别]
        E2[结构化处理]
        E3[格式转换]
        E4[验证确认]
    end

    subgraph "存储系统"
        T1[版本化存储]
        T2[分类索引]
        T3[元数据管理]
    end

    subgraph "消费场景"
        C1[本地分析]
        C2[在线查看]
        C3[程序调用]
        C4[版本比较]
    end

    R1 --> E1
    R2 --> E1
    R3 --> E1
    
    E1 --> E2
    E2 --> E3
    E3 --> E4
    
    E4 --> T1
    T1 --> T2
    T2 --> T3
    
    T1 --> C1
    T1 --> C2
    T1 --> C3
    T1 --> C4
```

## 提示词结构模式

```mermaid
classDiagram
    class SystemPrompt {
        +metadata: MetaInfo
        +coreInstructions: list
        +toolDefinitions: list
        +behavioralConstraints: list
        +safetyGuidelines: list
        +outputSpecifications: list
        +interactionStyle: StyleGuide
        +analyze()
        +extractPatterns()
        +compareVersions()
    }

    class MetaInfo {
        +knowledgeCutoff: datetime
        +currentDate: datetime
        +modelVersion: string
        +personality: string
    }

    class ToolDefinition {
        +toolName: string
        +description: string
        +parameters: dict
        +constraints: list
        +usageExamples: list
    }

    class SafetyGuideline {
        +category: string
        +rule: string
        +severity: enum
        +exceptions: list
    }

    class BehavioralConstraint {
        +constraintType: string
        +condition: string
        +action: string
        +priority: int
    }

    SystemPrompt "*" --> "1" MetaInfo
    SystemPrompt "*" --> "*" ToolDefinition
    SystemPrompt "*" --> "*" SafetyGuideline
    SystemPrompt "*" --> "*" BehavioralConstraint
```

## AI服务架构对比

```mermaid
graph TB
    subgraph "Anthropic Claude架构"
        AC1[系统提示词]
        AC2[工具层]
        AC3[记忆系统]
        AC4[搜索工具]
        AC5[文件操作]
    end

    subgraph "OpenAI GPT架构"
        OG1[系统提示词]
        OG2[工具定义]
        OG3[函数调用]
        OG4[附件处理]
        OG5[Web搜索]
    end

    subgraph "Google Gemini架构"
        GG1[系统指令]
        GG2[工具配置]
        GG3[API集成]
        GG4[多媒体处理]
    end

    AC1 --> AC2
    AC2 --> AC3
    AC2 --> AC4
    AC2 --> AC5
    
    OG1 --> OG2
    OG2 --> OG3
    OG3 --> OG4
    OG4 --> OG5
    
    GG1 --> GG2
    GG2 --> GG3
    GG3 --> GG4
```

## 提示词处理流程

```mermaid
flowchart TD
    A[开始处理] --> B[读取提示词文件]
    
    B --> C{文件格式}
    
    C -->|Markdown| D[解析结构]
    C -->|XML| E[XML解析]
    C -->|Text| F[文本提取]
    
    D --> G[提取元信息]
    E --> G
    F --> G
    
    G --> H[识别主要部分]
    H --> I[系统角色定义]
    H --> J[工具列表]
    H --> K[行为规则]
    H --> L[安全准则]
    H --> M[输出格式]
    
    I --> N[结构化存储]
    J --> N
    K --> N
    L --> N
    M --> N
    
    N --> O{需要分析}
    
    O -->|版本对比| P[加载历史版本]
    O -->|模式提取| Q[分析重复模式]
    O -->|安全审计| R[检查约束条件]
    O -->|性能优化| S[评估指令复杂度]
    
    P --> T[生成对比报告]
    Q --> U[提取最佳实践]
    R --> V[输出安全建议]
    S --> W[优化建议]
    
    T --> X[结束]
    U --> X
    V --> X
    W --> X
```

## 用户交互流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant W as Web界面
    participant G as GitHub仓库
    participant L as 本地文件
    
    U->>W: 访问项目页面
    W->>U: 展示项目结构
    
    U->>W: 选择AI服务商
    W->>U: 显示可用模型列表
    
    U->>W: 选择特定模型
    W->>U: 展示完整提示词
    
    U->>W: 比较不同版本
    W->>G: 获取版本历史
    G->>W: 返回版本列表
    W->>U: 展示版本对比
    
    U->>W: 下载提示词
    W->>L: 读取文件
    L->>W: 返回文件内容
    W->>U: 提供下载链接
    
    U->>W: 贡献新内容
    W->>U: 展示贡献指南
    U->>G: 提交Pull Request
    G->>W: 通知审核状态
    W->>U: 显示贡献结果
```

## 部署架构

```mermaid
graph TB
    subgraph "内容存储"
        F1[GitHub仓库]
        F2[文件版本控制]
        F3[分支管理]
    end

    subgraph "访问层"
        W1[GitHub Web界面]
        W2[Git命令行]
        W3[API接口]
    end

    subgraph "分析工具"
        A1[本地脚本]
        A2[在线分析器]
        A3[版本比较器]
    end

    subgraph "社区协作"
        C1[Issue跟踪]
        C2[Pull Request]
        C3[讨论论坛]
    end

    F1 --> W1
    F1 --> W2
    F1 --> W3
    
    W1 --> A2
    W2 --> A1
    W3 --> A3
    
    A1 --> C1
    A2 --> C2
    A3 --> C3
```

## 数据统计概览

```mermaid
pie title 提示词提供商分布
    "Anthropic Claude" : 35
    "OpenAI GPT" : 40
    "Google Gemini" : 10
    "xAI Grok" : 5
    "其他" : 10
```

## 文件格式支持

```mermaid
graph LR
    subgraph "输入格式"
        I1[原始系统响应]
        I2[API日志]
        I3[用户贡献]
    end

    subgraph "处理引擎"
        P1[格式检测]
        P2[内容提取]
        P3[结构验证]
        P4[元数据生成]
    end

    subgraph "输出格式"
        O1[Markdown .md]
        O2[XML .xml]
        O3[Text .txt]
    end

    I1 --> P1
    I2 --> P1
    I3 --> P1
    
    P1 --> P2
    P2 --> P3
    P3 --> P4
    
    P4 --> O1
    P4 --> O2
    P4 --> O3
```

## 关键组件说明

### 1. 提示词收集器

负责从多个渠道获取系统提示词:

- **社区贡献渠道**: 用户提交的提示词内容
- **逆向工程方法**: 通过特定技术手段获取
- **API分析**: 从API响应中提取系统消息
- **官方文档**: 参考官方发布的信息

### 2. 存储管理系统

提供版本化的文件存储:

- **文件版本控制**: 跟踪提示词的历史变化
- **分类索引**: 按提供商和模型分类
- **元数据管理**: 记录文件属性和来源信息

### 3. 分析引擎

支持多种分析功能:

- **版本对比**: 比较不同时期的提示词差异
- **结构分析**: 解析提示词的内部结构
- **模式提取**: 识别常见的提示词设计模式
- **变更追踪**: 监控提示词的更新情况

### 4. 访问接口

提供多种访问方式:

- **Web界面**: GitHub项目页面浏览
- **命令行**: Git工具克隆和查询
- **API访问**: 程序化接口调用
- **本地分析**: 下载后进行离线分析
