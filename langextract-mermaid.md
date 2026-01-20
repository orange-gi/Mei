# LangExtract 项目架构与数据流分析

## 一、系统架构总览

### 1.1 架构设计理念

LangExtract采用了分层架构设计，将用户接口层、任务编排层、模型适配层和数据持久层清晰地分离。这种分层设计的核心优势在于关注点分离——用户接口层专注于提供简洁的API，任务编排层专注于处理复杂的提取逻辑，模型适配层专注于与不同LLM提供商的交互，数据持久层专注于结果的序列化和可视化。

在架构风格上，LangExtract采用了插件化设计模式，通过Provider Registry机制支持多种LLM提供商的动态接入。这种设计确保了核心功能的稳定性，同时提供了良好的扩展性。当需要支持新的LLM提供商时，只需实现相应的Provider接口并注册到Registry中，无需修改核心代码。

### 1.2 系统层次结构

```
┌─────────────────────────────────────────────────────────────────┐
│                     用户接口层 (User Interface)                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   lx.extract()                            │  │
│  │                   lx.io.save_annotated_documents()        │  │
│  │                   lx.visualize()                          │  │
│  └───────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                    任务编排层 (Task Orchestration)               │
│  ┌───────────────┬───────────────┬───────────────┬───────────┐  │
│  │  文本分块器    │  并行调度器    │  多轮处理器   │ 结果聚合器 │  │
│  │  TextChunker  │ Parallelizer  │ MultiPasser   │ Aggregator │  │
│  └───────────────┴───────────────┴───────────────┴───────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                    模型适配层 (Model Adapter)                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   Model Factory                            │  │
│  │         ┌─────────────────────────────────────────┐        │  │
│  │         │          Provider Registry              │        │  │
│  │         │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐       │        │  │
│  │         │  │Gemini│ │OpenAI│ │Ollama│ │Vertex│ ...  │        │  │
│  │         │  └─────┘ └─────┘ └─────┘ └─────┘       │        │  │
│  │         └─────────────────────────────────────────┘        │  │
│  └───────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                    数据持久层 (Data Persistence)                 │
│  ┌───────────────────┐                    ┌───────────────────┐  │
│  │  JSONL 序列化器    │                    │  HTML 可视化器    │  │
│  │  JSONL Serializer │                    │  HTML Visualizer │  │
│  └───────────────────┘                    └───────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## 二、核心组件详解

### 2.1 文本分块器（TextChunker）

文本分块器负责将长文档拆分为适合LLM处理的块。分块策略直接影响提取质量：过大的块可能导致信息遗漏，过小的块可能破坏上下文连贯性。

```mermaid
graph TD
    A[输入文档] --> B[文档长度检查]
    B -->|小于阈值| C[单块处理]
    B -->|大于阈值| D[分块处理]
    
    D --> E[计算分块数量]
    E --> F[滑动窗口分块]
    
    F --> G[块大小控制]
    G --> H[重叠区域设置]
    
    H --> I[生成文本块列表]
    I --> J[添加上下文标记]
    
    style A fill:#e1f5fe
    style I fill:#c8e6c9
    style C fill:#fff9c4
```

**分块算法特点**：
- 滑动窗口：确保每个区域都被覆盖
- 重叠区域：处理块边界处的跨块引用
- 智能截断：在句子或段落边界切分，保持语义完整性

### 2.2 并行调度器（Parallelizer）

并行调度器利用多线程或异步机制，同时处理多个文本块，以提高整体吞吐量。

```mermaid
flowchart LR
    subgraph 任务队列
    A[文本块1] --> Q[任务队列]
    B[文本块2] --> Q
    C[文本块N] --> Q
    end
    
    subgraph Worker池
    Q --> W1[Worker 1]
    Q --> W2[Worker 2]
    Q --> W3[Worker N]
    end
    
    W1 --> R1[结果1]
    W2 --> R2[结果2]
    W3 --> Rn[结果N]
    
    R1 --> 聚合[结果聚合器]
    R2 --> 聚合
    Rn --> 聚合
    
    style Q fill:#ffecb3
    style 聚合 fill:#c8e6c9
```

**并行策略**：
- Worker数量可配置（`max_workers`参数）
- 支持异步API调用的并发执行
- 错误处理：单块失败不影响整体处理

### 2.3 多轮处理器（MultiPasser）

多轮处理器通过多次提取轮次来提高召回率。每一轮的提取结果会影响后续轮次的提示词。

```mermaid
flowchart TD
    A[输入文本] --> B[第1轮提取]
    B --> C{轮次 < 目标?}
    C -->|是| D[分析遗漏]
    D --> E[增强提示词]
    E --> B
    C -->|否| F[最终结果]
    
    B -.-> P1[结果1]
    E -.-> P2[补充提取]
    
    style B fill:#bbdefb
    style E fill:#fff9c4
    style F fill:#c8e6c9
```

**多轮提取逻辑**：
- 第一轮：基础提取，识别主要实体
- 后续轮次：针对性补充，处理遗漏和模糊情况
- 终止条件：达到指定轮数或结果收敛

### 2.4 结果聚合器（Aggregator）

结果聚合器负责合并来自多个块和多轮处理的提取结果，并解决核心指代问题。

```mermaid
flowchart TD
    A[多块结果] --> B[去重处理]
    C[多轮结果] --> B
    B --> D[指代消解]
    D --> E[位置映射]
    E --> F[属性合并]
    F --> G[最终结果]
    
    subgraph 指代消解规则
    D --> D1[同一实体检测]
    D --> D2[引用关系识别]
    D --> D3[上下文传递]
    end
    
    style G fill:#c8e6c9
```

## 三、数据流分析

### 3.1 端到端数据流

```mermaid
flowchart TD
    subgraph 输入阶段
        U[用户]
        P[提示词定义]
        E[示例准备]
        T[源文本]
    end
    
    subgraph 处理阶段
        TC[文本分块]
        PM[提示词格式化]
        MC[模型调用]
        RP[结果解析]
        RA[结果聚合]
    end
    
    subgraph 输出阶段
        J[JSONL存储]
        H[HTML可视化]
        R[最终结果]
    end
    
    U --> P
    U --> E
    U --> T
    P --> PM
    E --> PM
    T --> TC
    PM --> MC
    TC --> MC
    MC --> RP
    RP --> RA
    RA --> J
    RA --> H
    RA --> R
    
    style PM fill:#e3f2fd
    style MC fill:#ffecb3
    style RA fill:#c8e6c9
```

### 3.2 提示词格式化数据流

提示词格式化是连接用户意图和模型理解的关键环节。

```mermaid
flowchart LR
    subgraph 原始输入
        A[基础提示词]
        B[示例列表]
        C[提取定义]
    end
    
    subgraph 格式化处理
        D[模板渲染]
        E[示例注入]
        F[格式约束]
    end
    
    subgraph 模型输入
        G[完整提示词]
    end
    
    A --> D
    B --> E
    C --> D
    
    D --> F
    E --> F
    
    F --> G
    
    style G fill:#c8e6c9
```

**提示词模板结构**：
```
[系统指令]
[提取任务描述]
[格式要求]
[示例1]
[示例2]
...
[待提取文本]
```

### 3.3 批处理数据流（Vertex AI）

Vertex AI批处理模式针对大规模离线处理场景。

```mermaid
flowchart TD
    A[输入数据] --> B[数据分片]
    B --> C{数据量评估}
    
    C -->|小批量| D[实时API]
    C -->|大批量| E[批处理API]
    
    D --> F[直接返回]
    E --> G[上传到GCS]
    G --> H[提交批处理作业]
    H --> I{作业状态}
    I -->|进行中| J[轮询等待]
    I -->|完成| K[从GCS下载]
    I -->|失败| L[重试或错误处理]
    
    J --> I
    
    F --> M[结果合并]
    K --> M
    L --> M
    
    M --> N[最终输出]
    
    style E fill:#ffecb3
    style K fill:#c8e6c9
```

### 3.4 多语言处理数据流

LangExtract内置了多语言分词支持，确保在不同语言环境下的准确提取。

```mermaid
flowchart TD
    A[输入文本] --> B[语言检测]
    B --> C{Unicode分词?}
    
    C -->|是| D[UnicodeTokenizer]
    C -->|否| E[RegexTokenizer]
    C -->|自定义| F[自定义Tokenizer]
    
    D --> G[分块处理]
    E --> G
    F --> G
    
    G --> H[LLM处理]
    H --> I[结果重组]
    
    I --> J[输出]
    
    style D fill:#e3f2fd
    style F fill:#fff9c4
```

## 四、模型适配层架构

### 4.1 Provider Registry 机制

Provider Registry采用模式匹配和优先级解析机制，支持动态发现和注册LLM提供商。

```mermaid
classDiagram
    class ProviderRegistry {
        +providers: Dict[str, Provider]
        +register(name: str, priority: int) decorator
        +get_provider(model_id: str) Provider
        +list_providers() List[str]
    }
    
    class ModelFactory {
        +create_model(config: ModelConfig) BaseModel
        +get_default_provider() str
    }
    
    class BaseModel {
        <<interface>>
        +infer(prompt: str) ExtractionResult
        +supports_schema() bool
    }
    
    class GeminiProvider {
        +infer(prompt: str)
        +supports_schema() True
    }
    
    class OpenAIProvider {
        +infer(prompt: str)
        +supports_schema() False
    }
    
    class OllamaProvider {
        +infer(prompt: str)
        +supports_schema() False
    }
    
    ProviderRegistry --> ModelFactory
    ModelFactory --> BaseModel
    BaseProvider <|-- GeminiProvider
    BaseProvider <|-- OpenAIProvider
    BaseProvider <|-- OllamaProvider
```

### 4.2 模型调用时序图

```mermaid
sequenceDiagram
    participant User as 用户
    participant Extract as lx.extract()
    participant Factory as ModelFactory
    participant Provider as LLM Provider
    participant LLM as 大语言模型
    
    User->>Extract: extract(text, prompt, examples, model_id)
    Extract->>Factory: create_model(config)
    Factory->>ProviderRegistry: get_provider(model_id)
    ProviderRegistry-->>Factory: 返回Provider实例
    Factory-->>Extract: 返回Model实例
    
    Extract->>TextChunker: chunk_text(text)
    TextChunker-->>Extract: 文本块列表
    
    loop 每轮提取
        loop 每个文本块
            Extract->>Provider: format_prompt(prompt, examples, chunk)
            Provider-->>Extract: 格式化提示词
            Extract->>LLM: generate(提示词)
            LLM-->>Provider: 原始输出
            Provider->>Provider: parse_output(原始输出)
            Provider-->>Extract: 结构化结果
        end
        Extract->>Aggregator: aggregate(当前轮次结果)
        Aggregator-->>Extract: 聚合结果
    end
    
    Extract-->>User: 最终结果
```

## 五、自定义Provider开发架构

### 5.1 插件接口规范

自定义Provider需要实现以下接口规范。

```mermaid
flowchart TD
    A[自定义Provider开发] --> B[实现BaseModel接口]
    B --> C[注册到ProviderRegistry]
    C --> D[配置入口点]
    D --> E[测试验证]
    
    subgraph 接口要求
        F[infer()方法]
        G[supports_schema()方法]
        H[parse_output()方法]
    end
    
    B --> F
    B --> G
    B --> H
    
    style A fill:#c8e6c9
    style E fill:#c8e6c9
```

### 5.2 数据模型关系图

```mermaid
erDiagram
    ExtractedDocument ||--|| Document : represents
    ExtractedDocument ||--|{ Extraction : contains
    ExampleData ||--|{ Extraction : demonstrates
    Extraction }|--|| ExtractionClass : belongs_to
    Extraction }|--|| SourceSpan : maps_to
    
    ExtractedDocument {
        string text
        list extractions
        dict metadata
    }
    
    Extraction {
        string extraction_class
        string extraction_text
        SourceSpan source_span
        dict attributes
    }
    
    SourceSpan {
        int start
        int end
        string text
    }
    
    ExampleData {
        string text
        list extractions
    }
    
    ExtractionClass {
        string name
        dict attributes_schema
    }
```

## 六、可视化与持久化架构

### 6.1 结果持久化架构

```mermaid
flowchart TD
    A[ExtractedDocument] --> B[序列化器]
    
    subgraph JSONL序列化
        B --> C[JSON Lines格式]
        C --> D[文件写入]
        D --> E[.jsonl文件]
    end
    
    subgraph HTML可视化
        B --> F[模板渲染]
        F --> G[交互组件注入]
        G --> H[样式注入]
        H --> I[.html文件]
    end
    
    style E fill:#e3f2fd
    style I fill:#fff9c4
```

### 6.2 HTML可视化架构

```mermaid
flowchart TD
    subgraph 数据源
        A[JSONL文件] --> B[解析器]
    end
    
    subgraph 处理层
        B --> C[数据转换]
        C --> D[分类聚合]
    end
    
    subgraph 渲染层
        D --> E[主文档视图]
        D --> F[侧边栏导航]
        D --> G[搜索过滤]
    end
    
    subgraph 交互层
        E --> H[悬停高亮]
        E --> I[点击跳转]
        F --> J[分类筛选]
        G --> K[关键词搜索]
    end
    
    style E fill:#c8e6c9
    style H fill:#e1f5fe
```

## 七、配置与扩展点

### 7.1 可配置参数映射

| 功能模块 | 参数名称 | 类型 | 默认值 | 说明 |
|---------|---------|------|-------|------|
| 提取配置 | `extraction_passes` | int | 1 | 提取轮数 |
| 并行配置 | `max_workers` | int | 1 | 并行工作线程数 |
| 分块配置 | `max_char_buffer` | int | 40000 | 单块最大字符数 |
| 上下文配置 | `context_window_chars` | int | 1000 | 跨块上下文大小 |
| 模型配置 | `model_id` | str | "gemini-2.5-flash" | 模型标识 |
| 批处理配置 | `batch.enabled` | bool | False | 启用批处理 |

### 7.2 扩展点列表

LangExtract提供了多个扩展点以支持自定义需求。

**Provider扩展**：通过实现BaseModel接口并注册到ProviderRegistry，可以添加新的LLM提供商支持。

**Tokenizer扩展**：通过实现自定义Tokenizer并注册，可以支持特定语言或领域的分词策略。

**输出格式扩展**：通过自定义结果解析器，可以支持不同的输出格式和验证规则。

**可视化扩展**：通过修改HTML模板，可以定制可视化界面的样式和交互行为。
