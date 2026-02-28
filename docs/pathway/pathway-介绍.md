### 项目概览（Why–What–How）

本项目是 **Pathway：Live Data Framework**。它提供 **Python API** 来描述数据管道（ETL/实时分析/LLM & RAG 管道等），并由 **可扩展的 Rust 引擎**执行，底层基于 Timely/Differential Dataflow 做 **增量计算**，统一支持 **批处理与流处理**。

### Why：为什么要做这个项目

- **统一批与流**：同一套 Python 代码既能在本地/CI 做批处理，也能在生产中持续处理流式更新与回放（replay）。
- **把“实时”做成默认能力**：当输入数据发生变化（新增、修改、延迟到达、乱序），系统以增量方式更新下游结果，而不是全量重算。
- **Python 生态 + 引擎级性能**：用户用 Python 写业务逻辑/调用 ML/LLM 工具链；执行层交给 Rust 引擎实现多线程/多进程/分布式与更稳定的内存/吞吐表现。
- **端到端可部署**：提供连接器（Kafka、文件系统、对象存储、DB、HTTP 等）、持久化（状态保存与恢复）、监控等，使“从数据源到下游系统”的闭环更直接。

### What：这个项目提供什么能力

从“用户看到的产品形态”看，Pathway 主要由以下能力组成：

- **Python DSL / API（表、列、表达式、算子）**
  - 核心抽象是 `Table`/`Schema`/`ColumnExpression` 等，用于描述数据流图（dataflow graph）。
  - 提供常见算子：`filter`、`join`、`groupby/reduce`、window/temporal join、排序/去重等。
- **连接器（Connectors）**
  - 输入：CSV/JSONLines、Kafka/Redpanda、S3/MinIO、GDrive、数据库 CDC/写入、HTTP/Websocket 等（以具体实现为准）。
  - 输出：文件/对象存储/数据库/HTTP 等（以具体实现为准）。
- **UDF 与异步调用**
  - 支持把任意 Python 函数作为变换逻辑，也支持异步 UDF（常用于调用外部 API/LLM/Embedding）。
- **持久化与恢复（Persistence / Recovery）**
  - 保存计算状态与输入快照，支持更新或故障后恢复，避免从零重跑。
- **监控与可观测性**
  - 运行时指标、连接器延迟/吞吐等（可通过 dashboard/导出指标等方式呈现，具体以配置为准）。
- **可选扩展（xpacks）**
  - 例如 LLM/RAG 工具链（解析器、切分器、向量索引/检索、与 LangChain/LlamaIndex 的集成等）。
- **Rust 引擎**
  - 承担算子执行、状态管理、时间/水位线语义、连接器读取与背压、持久化、分布式/多进程协调等。

### How：它是如何工作的（从写代码到跑起来）

#### 用户侧的“写法”（Python）

典型管道结构：

- **定义 schema（可选）**
- **从连接器读入数据 → 得到 `Table`**
- **在 `Table` 上做变换与聚合**
- **把结果写出到下游**
- **调用 `pw.run()` 启动**

（示例可参考仓库根目录 `README.md` 中的代码片段。）

#### 系统内部的“运行路径”（Python 编排层 → Rust 执行层）

高层流程可以概括为：

- **Python 层构建逻辑计划**：用户调用 API 时，形成表达式与算子图（logical dataflow）。
- **Python ↔ Rust 绑定层**：项目通过 `maturin` 构建 `pathway.engine`（PyO3 扩展模块），将逻辑图与运行配置交给 Rust。
- **Rust 引擎执行与增量维护**：
  - 连接器持续产生“数据变更事件”（insert/update/delete 或更一般的差分）。
  - 引擎以增量方式推进算子，维护中间状态，并持续更新输出。
  - 若开启持久化，引擎周期性保存状态/快照，以支持恢复。
- **多线程/多进程/分布式**：
  - CLI/运行参数可配置线程与进程（例如环境变量 `PATHWAY_THREADS`、`PATHWAY_PROCESSES` 等由 `pathway` CLI 注入）。

### 系统边界（System Boundary）

为了避免“框架做什么/不做什么”混淆，建议按如下边界理解：

- **系统内（Pathway 负责）**
  - **计算语义与执行**：算子执行、增量更新、时间语义/一致性处理、状态管理。
  - **连接器运行时**：从外部系统读写、背压与缓冲、格式解析（如 CSV/JSON/Parquet 等）、错误处理与重试（以实现为准）。
  - **状态持久化与恢复**：状态/快照的写入、恢复流程、相关配置。
  - **可观测性**：指标采集与导出、运行日志、（可选）dashboard 服务端点。
  - **扩展包（xpacks）**：如 LLM/RAG 组件的封装与集成能力（可选安装）。

- **系统外（Pathway 不负责/不直接控制）**
  - **外部数据源与下游系统本身**：Kafka/DB/S3/LLM 服务的可用性、容量、权限、网络延迟等。
  - **数据治理/建模策略**：数据质量、主键设计、事件时间字段选择、业务一致性规则等需要由使用方定义。
  - **基础设施运维**：Kubernetes/容器平台、服务网格、VPC/防火墙、证书等由部署环境负责。

### 模块边界（Module Boundary）

下面以仓库目录为主线，描述“模块职责、对外接口、依赖方向”。（括号内为关键目录/文件线索）

#### Python 层（对用户暴露的 API 与编排层）

- **公共 API 聚合层（入口）**：`python/pathway/__init__.py`
  - **职责**：对外导出稳定 API（`Table`、`Schema`、`pw.io.*`、`pw.run()`、`pw.reducers.*`、`pw.stdlib.*` 等），隐藏内部实现细节。
  - **依赖方向**：依赖 `pathway.internals`、`pathway.io`、`pathway.stdlib`、`pathway.xpacks`（可选）。

- **核心编排与语义实现**：`python/pathway/internals/`
  - **职责**：
    - 表/列/表达式系统：列引用、表达式树、类型/Schema、表达式求值与打印等。
    - 算子与图执行：join/groupby/reduce、图解析与 runner、运行配置、错误处理与监控埋点等。
    - SQL 支持与转换（可选）：`internals/sql/`
    - UDF 执行框架：同步/异步、缓存/重试等（`internals/udfs/`）。
  - **边界原则**：这里是“框架语义核心”，应尽量不掺杂具体业务；对外 API 通过 `__init__.py` 暴露。

- **连接器 Python 封装**：`python/pathway/io/`
  - **职责**：提供 `pw.io.*` 的用户接口、参数校验、schema 映射、以及与引擎/运行时的对接。
  - **依赖方向**：依赖 `internals`；底层可能委托 Rust 连接器实现。

- **标准库（可组合能力）**：`python/pathway/stdlib/`
  - **职责**：在核心算子之上提供更高层的可复用组件与算法：
    - `temporal/`：窗口、as-of join、时间行为等。
    - `indexing/`：向量/全文/混合检索索引与 retriever。
    - `ml/`：一些 ML 辅助能力与 smart ops。
    - `viz/`：表可视化/交互展示等。
  - **边界原则**：stdlib 更偏“能力组合”，不直接承担底层执行引擎职责。

- **扩展包（可选安装）**：`python/pathway/xpacks/`
  - **职责**：将某些领域能力做成可选组件（例如 `xpacks/llm/` 提供 LLM/RAG 相关工具、MCP server 等）。
  - **边界原则**：与核心 API 解耦，通过 extra 依赖启用（参见 `pyproject.toml` 的 optional-dependencies）。

- **第三方集成代码（随仓库分发）**：`python/pathway/third_party/`
  - **职责**：对某些外部生态（如 Airbyte serverless）的适配与封装。

- **CLI（运行入口之一）**：`python/pathway/cli.py`
  - **职责**：提供 `pathway spawn ...` 等命令，管理多进程/多线程启动、录制与回放相关环境变量注入等。

#### Rust 层（执行引擎与运行时）

- **Rust crate 总入口**：`src/lib.rs`
  - **职责**：汇总暴露引擎、连接器、持久化、Python API 绑定等模块。

- **执行引擎**：`src/engine/`
  - **职责**：dataflow 执行、算子实现、时间推进、状态与一致性、对外输出、监控/遥测等。
  - **关键边界**：这里决定“性能与正确性”的下限，应避免掺入 Python 层的便捷逻辑。

- **连接器（Rust 实现）**：`src/connectors/`
  - **职责**：与外部系统交互、格式解析、背压/缓冲、offset/同步等。

- **持久化**：`src/persistence/`
  - **职责**：状态/快照的存取、不同后端（文件/S3/Azure 等）的适配、恢复所需元数据管理。

- **Python 绑定层**：`src/python_api/` 与 `src/python_api.rs`
  - **职责**：作为 PyO3 扩展模块桥接 Python 编排层与 Rust 引擎（对应 `pyproject.toml` 的 `module-name = "pathway.engine"`）。

#### 外部依赖与示例/测试

- **外部底层依赖（vendored）**：`external/`
  - **职责**：包含 Timely/Differential Dataflow 等相关代码（用于引擎能力构建与定制）。
  - **边界原则**：一般不作为 Pathway 公共 API 的直接延伸，而是引擎内部依赖。

- **示例与模板**：`examples/`、`docs/2.developers/7.templates/`
  - **职责**：提供可运行的端到端样例（ETL、实时分析、RAG 等），面向用户学习与快速落地。

- **测试**：`python/pathway/tests/`、`integration_tests/`、`tests/`（Rust）
  - **职责**：分别覆盖 Python API 语义、跨组件集成、Rust 引擎单元/集成等。

### 依赖方向（建议的“不可逆箭头”）

为了保持边界清晰，推荐的依赖方向是：

- `python/pathway/__init__.py` → `internals` / `io` / `stdlib` / `xpacks`
- `io` / `internals` → `pathway.engine`（Rust 扩展模块）/ 运行时
- `stdlib` → `internals`（以组合为主，避免反向依赖）
- `xpacks` → `internals` / `io` / `stdlib`（可选能力不反向侵入核心）
- Rust `engine` → Rust `connectors`/`persistence`/`external_integration`

### 这份文档的使用方式

- **给新同学**：先读 Why/What/How，建立“Python 写逻辑、Rust 跑执行”的心智模型。
- **给开发者**：按模块边界定位改动点：新增算子多半在 `internals` + `src/engine`；新增连接器通常在 `python/pathway/io` + `src/connectors`；新增领域能力优先放 `stdlib` 或 `xpacks`。
