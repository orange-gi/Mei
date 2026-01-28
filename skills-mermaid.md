# Anthropic Skills 架构图

## 一、系统整体架构

### 1.1 架构概览

Anthropic Skills 系统采用分层架构设计，从上至下依次为：接入层、集成层、技能层和规格层。这种分层设计遵循了软件工程的最佳实践，实现了关注点分离和高内聚低耦合的目标。每一层都有明确的职责边界，层与层之间通过标准化接口进行通信，确保了系统的可维护性和可扩展性。

架构设计的核心理念是「技能即封装」。每个技能都是一个自包含的功能单元，包含执行特定任务所需的全部指令、脚本和资源。这种封装使得技能的开发和测试可以独立进行，也使得技能的复用和组合变得简单直观。用户可以根据自己的需求选择性地加载所需技能，实现个性化的 AI 助手能力。

整个架构围绕 Claude 核心引擎构建，技能系统作为扩展层存在。当用户发起请求时，系统会分析请求意图并激活相关的技能，激活的技能将其指令注入到 Claude 的上下文中，从而影响 AI 的行为和输出。这种动态注入机制是 Skills 系统区别于传统提示词工程的关键创新点。

### 1.2 分层架构图

以下架构图展示了 Skills 系统的整体分层结构和各层之间的交互关系：

```mermaid
graph TB
    subgraph "接入层"
        A1["Claude Code CLI"]
        A2["Claude.ai Web"]
        A3["Claude API"]
    end

    subgraph "集成层"
        I1["技能管理器"]
        I2["插件系统"]
        I3["API 接口"]
    end

    subgraph "技能层"
        S1["创意与设计技能"]
        S2["开发与技术技能"]
        S3["企业与通信技能"]
        S4["文档处理技能"]
    end

    subgraph "规格层"
        P1["技能规格定义"]
        P2["元数据规范"]
        P3["指令模板标准"]
    end

    A1 --> I1
    A2 --> I1
    A3 --> I1
    I1 --> I2
    I2 --> S1
    I2 --> S2
    I2 --> S3
    I2 --> S4
    S1 --> P1
    S2 --> P1
    S3 --> P1
    S4 --> P1
    P1 --> P2
    P1 --> P3
```

**架构说明**：

- **接入层**是用户与系统交互的入口点，支持三种使用方式：命令行工具（Claude Code）、网页应用（Claude.ai）和编程接口（API）
- **集成层**负责处理技能的管理和激活，包括技能管理器、插件系统和 API 接口三个核心组件
- **技能层**包含具体的技能实现，按照功能领域分为四个类别
- **规格层**定义了技能的标准规范，确保所有技能遵循统一的结构

### 1.3 技能仓库结构

Skills 仓库本身的目录结构反映了系统的设计理念：

```mermaid
graph TB
    subgraph "技能仓库根目录"
        R1[".claude-plugin/"]
        R2["skills/"]
        R3["spec/"]
        R4["template/"]
        R5["README.md"]
        R6[".gitignore"]
    end

    subgraph "skills 子目录"
        S1["creative/"]
        S2["development/"]
        S3["enterprise/"]
        S4["document/"]
    end

    R2 --> S1
    R2 --> S2
    R2 --> S3
    R2 --> S4

    subgraph "技能文件结构示例"
        F1["SKILL.md"]
        F2["scripts/"]
        F3["resources/"]
        F4["docx-js.md"]
        F5["ooxml.md"]
    end

    S4 --> F1
    S4 --> F2
    S4 --> F3
    S4 --> F4
    S4 --> F5
```

**结构说明**：

- **skills/** 目录包含所有示例技能，按功能类别组织
- **spec/** 目录包含 Agent Skills 的规格定义文档
- **template/** 目录提供创建新技能的模板
- **.claude-plugin/** 目录定义 Claude Code 插件配置

## 二、核心数据流

### 2.1 技能激活数据流

当用户与 Claude 交互时，系统内部经历一系列步骤来确定是否需要激活技能以及激活哪些技能。以下流程图详细展示了这一过程：

```mermaid
sequenceDiagram
    participant User as 用户
    participant Claude as Claude 引擎
    participant Matcher as 技能匹配器
    participant Loader as 技能加载器
    participant Skills as 技能仓库
    participant Context as 执行上下文

    User->>Claude: 发起请求
    Claude->>Matcher: 请求意图分析
    Matcher->>Skills: 查询可用技能
    Skills-->>Matcher: 返回技能元数据
    Matcher-->>Claude: 匹配结果（技能列表）

    alt 找到匹配技能
        Claude->>Loader: 加载技能指令
        Loader->>Skills: 读取 SKILL.md
        Skills-->>Loader: 返回技能内容
        Loader-->>Context: 注入技能指令
        Context-->>Claude: 更新执行上下文
    else 未找到匹配技能
        Claude-->>User: 使用默认能力响应
    end

    Claude-->>User: 生成响应（使用激活的技能）
```

**数据流说明**：

1. 用户发起请求后，系统首先进行意图分析
2. 技能匹配器根据请求内容查询可用技能
3. 匹配结果返回给 Claude 引擎
4. 如果找到匹配技能，技能加载器读取并解析 SKILL.md 文件
5. 技能指令被注入到执行上下文中
6. Claude 使用更新后的上下文生成最终响应

### 2.2 文档处理数据流

以 DOCX 文档处理为例，展示完整的文档操作数据流：

```mermaid
flowchart TB
    subgraph "输入阶段"
        I1["原始 DOCX 文件"]
        I2["用户指令"]
    end

    subgraph "预处理阶段"
        P1["任务类型判断"]
        P2["工具选择"]
        P3["前置条件检查"]
    end

    subgraph "执行阶段"
        E1["解包文档"]
        E2["内容处理"]
        E3["重新打包"]
    end

    subgraph "输出阶段"
        O1["生成 DOCX 文件"]
        O2["响应摘要"]
    end

    I1 --> P1
    I2 --> P1
    P1 --> P2
    P2 --> P3
    P3 --> E1
    E1 --> E2
    E2 --> E3
    E3 --> O1
    O1 --> O2

    style P1 fill:#e1f5fe
    style P2 fill:#e1f5fe
    style P3 fill:#e1f5fe
    style E1 fill:#f3e5f5
    style E2 fill:#f3e5f5
    style E3 fill:#f3e5f5
```

**详细数据流说明**：

**输入阶段**：接收用户提供的 DOCX 文件和操作指令。文件路径需要验证存在性，指令需要解析操作类型。

**预处理阶段**：系统首先判断任务类型（创建、读取、编辑），然后根据类型选择合适的工具和脚本，最后检查前置条件（如必要的参考文档是否已阅读）。

**执行阶段**：对于编辑类任务，首先使用 unpack.py 脚本解压 DOCX 文件（ZIP 格式），然后根据具体操作修改 XML 内容，最后使用 pack.py 脚本重新打包为 DOCX 格式。

**输出阶段**：生成处理后的文件，并提供操作摘要供用户确认。

### 2.3 修订追踪工作流数据流

修订追踪（Redlining）是 DOCX 技能中最高级的工作流，以下是其完整的数据流转过程：

```mermaid
flowchart LR
    subgraph "阶段一：准备"
        A1["获取当前文档"]
        A2["转换为 Markdown"]
        A3["识别变更需求"]
    end

    subgraph "阶段二：分组"
        B1["分析变更类型"]
        B2["确定分组策略"]
        B3["划分变更批次"]
    end

    subgraph "阶段三：实现"
        C1["阅读参考文档"]
        C2["解压目标文档"]
        C3["逐批次实现变更"]
    end

    subgraph "阶段四：验证"
        D1["重新打包文档"]
        D2["转换验证"]
        D3["确认变更正确"]
    end

    A1 --> A2 --> A3 --> B1 --> B2 --> B3 --> C1 --> C2 --> C3 --> D1 --> D2 --> D3

    style A1 fill:#fff3e0
    style A2 fill:#fff3e0
    style A3 fill:#fff3e0
    style B1 fill:#e8f5e9
    style B2 fill:#e8f5e9
    style B3 fill:#e8f5e9
    style C1 fill:#e3f2fd
    style C2 fill:#e3f2fd
    style C3 fill:#e3f2fd
    style D1 fill:#fce4ec
    style D2 fill:#fce4ec
    style D3 fill:#fce4ec
```

**批次划分策略**：

修订追踪工作流的核心创新是批次划分。系统将大量变更分组为 3-10 个变更的小批次，每个批次可以独立测试和验证。批次划分可以基于以下维度：

- **按文档章节**：将同一章节的变更归为一批
- **按变更类型**：将同类变更（如日期修改、名称替换）归为一批
- **按复杂程度**：简单变更先行，复杂变更后处理
- **按页面位置**：将相邻页面的变更归为一批

## 三、模块关系图

### 3.1 技能内部模块关系

以 DOCX 技能为例，展示其内部各模块之间的依赖关系：

```mermaid
classDiagram
    class SkillCore {
        +name: string
        +description: string
        +workflow_tree: dict
        +execute(user_request)
    }

    class DecisionMaker {
        +classify_task(request)
        +select_workflow(task_type)
        +validate_prerequisites()
    }

    class DocumentHandler {
        +unpack_document(path)
        +pack_document(dir, output_path)
        +read_xml(file_path)
        +write_xml(file_path, content)
    }

    class RedlineProcessor {
        +batch_changes(changes)
        +apply_batch(batch)
        +validate_changes()
    }

    class ToolRegistry {
        +get_tool(tool_name)
        +list_tools()
        +validate_dependencies()
    }

    class ReferenceDocs {
        +load_docx_js_guide()
        +load_ooxml_guide()
        +get_api_reference()
    }

    SkillCore --> DecisionMaker
    SkillCore --> DocumentHandler
    SkillCore --> RedlineProcessor
    SkillCore --> ToolRegistry
    SkillCore --> ReferenceDocs
    DecisionMaker --> DocumentHandler
    RedlineProcessor --> DocumentHandler
    ToolRegistry --> ReferenceDocs
```

**模块职责说明**：

- **SkillCore**：技能核心类，负责协调各模块的工作
- **DecisionMaker**：决策模块，根据任务类型选择合适的工作流程
- **DocumentHandler**：文档处理模块，负责 DOCX 文件的解压、读取和打包
- **RedlineProcessor**：修订追踪模块，处理复杂的文档编辑操作
- **ToolRegistry**：工具注册表，管理技能所需的外部工具
- **ReferenceDocs**：参考文档模块，提供详细的技术文档访问

### 3.2 技能间协作关系

不同技能之间可以协作完成复杂任务，以下展示了技能间的协作关系：

```mermaid
graph TB
    subgraph "任务协调"
        T1["任务路由器"]
    end

    subgraph "企业通信技能"
        E1["品牌指南"]
        E2["邮件写作"]
        E3["报告生成"]
    end

    subgraph "文档处理技能"
        D1["DOCX 编辑"]
        D2["PDF 生成"]
        D3["PPTX 创建"]
    end

    subgraph "开发技术技能"
        C1["代码审查"]
        C2["测试生成"]
    end

    T1 --> E1
    T1 --> E2
    T1 --> E3
    T1 --> D1
    T1 --> D2
    T1 --> D3
    T1 --> C1
    T1 --> C2

    E1 --> D1
    E1 --> D2
    E1 --> D3
    E2 --> D1
    E3 --> D1
    E3 --> D3
```

**协作模式说明**：

- **任务路由**：任务路由器根据任务类型将请求分发给相应的技能
- **技能组合**：多个技能可以组合使用，例如「品牌指南」与「DOCX 编辑」组合确保生成的文档符合品牌规范
- **流水线执行**：某些任务需要按顺序执行多个技能，例如生成报告可能需要先由「报告生成」技能规划结构，再由「DOCX 编辑」技能创建文档

### 3.3 Claude 与 Skills 的交互关系

以下图表展示了 Claude 引擎与 Skills 系统之间的交互关系：

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as Claude 引擎
    participant S as 技能系统
    participant T as 工具执行器

    U->>C: 用户请求
    C->>S: 技能匹配请求
    S-->>C: 匹配技能列表
    C->>S: 加载技能指令
    S-->>C: 技能内容

    Note over C, S: 技能指令注入上下文

    C->>C: 基于技能指令生成响应
    C->>T: 工具调用（如需要）
    T-->>C: 工具执行结果
    C-->>U: 生成响应
```

## 四、用户交互流程

### 4.1 Claude Code 用户交互流程

以下是 Claude Code 用户使用 Skills 的完整交互流程：

```mermaid
flowchart TB
    subgraph "安装阶段"
        I1["注册仓库到插件市场"]
        I2["浏览可用技能"]
        I3["选择并安装技能"]
    end

    subgraph "激活阶段"
        A1["发起任务请求"]
        A2["隐式技能激活"]
        A3["显式技能激活"]
    end

    subgraph "执行阶段"
        E1["Claude 执行任务"]
        E2["调用技能工具"]
        E3["生成响应结果"]
    end

    subgraph "反馈阶段"
        F1["用户评估结果"]
        F2["迭代优化（如需要）"]
    end

    I1 --> I2 --> I3 --> A1
    A1 --> A2
    A1 --> A3
    A2 --> E1
    A3 --> E1
    E1 --> E2
    E2 --> E3
    E3 --> F1
    F1 --> F2
    F2 --> A1
```

**交互方式说明**：

**隐式激活**：Claude 根据请求内容自动匹配并激活相关技能，用户无需显式指定技能。例如，用户说「帮我创建一个专业的商业合同」，系统自动激活 DOCX 技能中的文档创建功能。

**显式激活**：用户可以显式指定使用某个技能。例如，「使用 PDF 技能从这个 DOCX 文件生成 PDF」。显式激活在用户清楚知道自己需要什么技能时更加高效。

### 4.2 技能创建流程

创建新技能遵循以下流程：

```mermaid
flowchart TB
    subgraph "规划阶段"
        P1["确定技能目标"]
        P2["定义使用场景"]
        P3["规划工作流程"]
    end

    subgraph "开发阶段"
        D1["创建技能目录"]
        D2["编写 SKILL.md"]
        D3["开发辅助脚本"]
        D4["准备资源文件"]
    end

    subgraph "测试阶段"
        T1["功能测试"]
        T2["集成测试"]
        T3["用户体验测试"]
    end

    subgraph "发布阶段"
        R1["代码审查"]
        R2["文档完善"]
        R3["提交到仓库"]
    end

    P1 --> P2 --> P3 --> D1 --> D2 --> D3 --> D4 --> T1 --> T2 --> T3 --> R1 --> R2 --> R3
```

### 4.3 修订追踪操作流程

修订追踪工作流的详细用户交互流程：

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as Claude
    participant D as 文档处理模块
    participant V as 验证模块

    U->>C: 上传文档，请求修订
    C->>D: 转换为 Markdown
    D-->>C: 当前状态 Markdown

    Note over C: 与用户确认变更需求

    U->>C: 提供变更需求清单
    C->>C: 分析并分组变更
    C-->>U: 展示批次划分方案
    U->>C: 确认批次方案

    loop 每个变更批次
        C->>C: 实现批次内变更
        C->>D: 打包中间文档
        D-->>C: 中间文档
        C->>V: 验证变更
        V-->>C: 验证结果
    end

    C->>D: 最终打包
    D-->>C: 最终文档
    C->>V: 最终验证
    V-->>C: 验证通过

    C-->>U: 返回修订后文档
```

## 五、部署架构

### 5.1 技能仓库部署架构

Skills 项目本身的部署架构相对简单，主要依赖于 GitHub 的基础设施：

```mermaid
graph TB
    subgraph "开发环境"
        D1["本地开发"]
        D2["测试环境"]
    end

    subgraph "版本控制"
        G["GitHub 仓库"]
        B["分支管理"]
        R["版本发布"]
    end

    subgraph "分发渠道"
        M["Claude Code 插件市场"]
        W["Claude.ai Web"]
        A["Claude API"]
    end

    subgraph "终端用户"
        U1["Claude Code 用户"]
        U2["Claude.ai 用户"]
        U3["API 开发者"]
    end

    D1 --> G
    D2 --> G
    G --> B
    B --> R
    R --> M
    R --> W
    R --> A
    M --> U1
    W --> U2
    A --> U3
```

### 5.2 运行时技能加载架构

技能在运行时被加载和激活的架构设计：

```mermaid
graph TB
    subgraph "客户端"
        C1["Claude Code"]
        C2["Claude.ai Web"]
        C3["API 客户端"]
    end

    subgraph "Anthropic 云服务"
        K["技能缓存"]
        L["技能加载器"]
        E["Claude 引擎"]
    end

    subgraph "技能仓库镜像"
        S["GitHub 镜像"]
        M["插件市场缓存"]
    end

    C1 --> K
    C2 --> K
    C3 --> K
    K --> L
    L --> E
    L --> S
    L --> M
```

**加载优化策略**：

- **缓存机制**：已安装的技能被缓存在本地，避免重复下载
- **按需加载**：技能在使用时才被加载到内存，未激活的技能不占用资源
- **增量更新**：技能更新时只下载变化的部分，减少带宽消耗

### 5.3 技能开发与部署工作流

完整的技能开发与部署工作流：

```mermaid
flowchart LR
    subgraph "本地开发"
        E1["编写 SKILL.md"]
        E2["开发脚本"]
        E3["本地测试"]
    end

    subgraph "代码审查"
        R1["Pull Request"]
        R2["代码审查"]
        R3["自动化测试"]
    end

    subgraph "集成发布"
        I1["合并到主分支"]
        I2["更新插件市场"]
        I3["版本标记"]
    end

    subgraph "用户获取"
        U1["自动更新"]
        U2["手动安装"]
        U3["API 上传"]
    end

    E1 --> E2 --> E3 --> R1 --> R2 --> R3 --> I1 --> I2 --> I3
    I2 --> U1
    I3 --> U2
    I3 --> U3
```

## 六、架构总结

### 6.1 设计特点

Anthropic Skills 架构展现了以下设计特点：

**模块化设计**：技能作为独立模块封装功能，模块之间通过标准化接口交互，实现了高内聚低耦合的目标。这种设计使得技能的开发和测试可以独立进行，也便于技能的复用和组合。

**分层架构**：系统采用接入层、集成层、技能层和规格层的分层结构，每层有明确的职责边界。分层设计使得系统易于理解和维护，也便于未来扩展。

**动态激活**：技能在运行时动态加载和激活，而不是硬编码到系统中。这种设计提供了极大的灵活性，用户可以根据需要选择性地加载技能。

**标准化规范**：所有技能遵循统一的规格标准，确保了一致性和互操作性。标准化也使得技能的分享和复用更加便捷。

### 6.2 可扩展性

架构设计支持多种扩展方式：

**横向扩展**：可以轻松添加新类别的技能，只需遵循规格标准即可。新技能可以与现有技能协作，无需修改核心系统。

**纵向扩展**：可以在现有技能基础上增加新功能，如添加新的辅助脚本、参考文档等，而不影响技能的核心结构。

**平台扩展**：架构支持在不同平台（CLI、Web、API）上使用，展示了良好的可移植性。
