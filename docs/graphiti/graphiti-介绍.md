# Graphiti 项目介绍

## 为什么（Why）

### 传统 RAG 方法的局限性

在人工智能应用开发领域，检索增强生成（Retrieval-Augmented Generation，RAG）已成为提升大语言模型回答准确性和时效性的关键技术。然而，传统 RAG 方法在实际应用中面临诸多挑战，这些挑战严重制约了 AI 系统在动态环境中的表现。传统 RAG 依赖批处理模式，这意味着新数据的集成往往需要重新处理整个知识库，无法实现实时更新。当用户的对话历史、业务数据或外部信息发生变化时，系统难以及时反映这些变化，导致生成的回答可能包含过时或不一致的信息。此外，传统 RAG 方法通常采用静态数据摘要的方式，无法有效捕捉实体之间复杂的动态关系变化，也无法精确回答涉及时间维度的问题，例如「用户三个月前的偏好是什么」或「某关系何时开始何时结束」等。

在多轮对话场景中，传统 RAG 方法的局限性更为明显。AI 代理需要维护连贯的对话上下文，理解对话中提及的实体和关系如何随时间演变。传统方法难以追踪这些动态变化，无法判断当前信息是否与历史信息存在矛盾或冲突。例如，当用户在对话中先说「我喜欢苹果」，后来又说「我对苹果过敏」时，传统 RAG 系统无法智能地处理这种信息更新和矛盾检测，而是简单地存储所有信息而不进行时间维度的推理。

### Graphiti 的诞生背景

Graphiti 是由 Zep 团队开发并开源的时序知识图谱框架，旨在解决上述传统 RAG 方法的痛点。Zep 团队在开发 AI 记忆和上下文工程平台的过程中，深入研究了代理记忆（Agent Memory）的最佳实践，并发表了学术论文《Zep：一种面向代理记忆的时序知识图谱架构》。该论文提出了 Graphiti 的核心设计理念和技术架构，为构建智能、自适应的 AI 代理记忆系统提供了理论基础和实践指南。

Graphiti 的设计动机源于真实的 AI 应用开发需求。在构建会话 AI 助手、个性化推荐系统和自动化工作流代理时，开发者普遍面临如何高效管理和查询动态知识的难题。Graphiti 应运而生，它不仅是一个知识图谱构建库，更是一套完整的解决方案，帮助开发者构建能够实时学习、适应和演进的 AI 系统。通过将知识表示为图结构，Graphiti 能够自然地表达实体之间的复杂关系，同时通过时序建模保留历史信息的完整性和可追溯性。

### 核心问题解决

Graphiti 致力于解决以下几个核心问题。首先是**实时增量更新问题**，Graphiti 支持即时集成新的数据片段（Episode），无需重新计算整个知识图谱，大大降低了维护成本并提高了系统的响应速度。其次是**双时间模型问题**，Graphiti 实现了事件发生时间（valid_at）和数据摄入时间（created_at）的显式追踪，使得系统能够精确回答「某关系在特定时间点是否成立」这样的时序查询。第三是**高效混合检索问题**，Graphiti 结合了语义向量搜索、关键词 BM25 搜索和图遍历三种检索方式，实现了低延迟、高准确度的知识检索，且不依赖 LLM 进行结果汇总。第四是**矛盾检测与处理问题**，当新数据与历史数据存在冲突时，Graphiti 能够自动检测并通过时间边失效机制处理这些矛盾，确保知识的准确性和一致性。

## 是什么（What）

### 项目概述

Graphiti 是一个专为 AI 代理设计的时序知识图谱构建和查询框架。它能够持续整合用户交互、结构化和非结构化企业数据以及外部信息，构建一个连贯的、可查询的知识图谱。与传统知识图谱不同，Graphiti 特别强调对动态变化数据的处理能力，支持增量更新、高效检索和精确的历史时序查询，同时保持了知识图谱的灵活性和可扩展性。

从技术实现角度看，Graphiti 是一个 Python 库，提供了完整的 API 用于创建、操作和查询知识图谱。它采用异步架构设计，支持高并发操作，并提供了丰富的配置选项以适应不同的应用场景。Graphiti 的核心代码托管在 GitHub 上，采用 Apache 2.0 开源许可证，版本号当前为 0.26.3，已在生产环境中得到验证。

### 核心功能特性

Graphiti 提供了四大核心功能特性，每个特性都针对 AI 应用开发的实际需求进行了精心设计。

**实时增量更新**是 Graphiti 的首要特性。当新的对话片段、数据记录或外部信息到达时，Graphiti 能够立即将其整合到现有知识图谱中，而无需重新处理历史数据。这一特性对于需要实时响应的 AI 应用至关重要，例如客服系统需要即时学习用户的最新偏好，或者金融应用需要实时更新市场信息。Graphiti 的增量更新机制确保了系统能够以最小的计算开销保持知识的时效性。

**双时间数据模型**是 Graphiti 的核心创新。每一段数据（Episode）都有两个时间戳：valid_at 表示事件实际发生的时间，created_at 表示数据被摄入系统的时间。这种设计使得 Graphiti 能够回答复杂的时序查询，例如「用户在 2024 年 6 月的偏好是什么」或「某关系从何时开始有效」。双时间模型还为矛盾检测提供了基础，当新数据声称某关系在某个时间点成立，但历史数据显示该关系在更早时间点已经结束时，系统能够准确识别并处理这种冲突。

**高效混合检索**是 Graphiti 在性能方面的关键优势。传统的知识图谱检索往往依赖向量相似度搜索，但单一方法难以应对所有查询场景。Graphiti 结合了三种检索方法：语义向量搜索（基于 embeddings 的相似度匹配）、关键词搜索（BM25 算法）和图遍历（基于图结构的广度优先搜索）。通过混合检索，Graphiti 能够在不同类型的查询中都表现出色。此外，Graphiti 提供了多种重排序策略，包括 RRF（Reciprocal Rank Fusion）、MMR（Max Marginal Relevance）和 Cross-Encoder 重排序，进一步提升了检索结果的质量。

**自定义实体定义**体现了 Graphiti 的灵活性。开发者可以通过 Pydantic 模型定义自己的实体类型和关系类型，这使得 Graphiti 能够适应各种垂直领域的知识表示需求。例如，医疗应用可以定义「诊断」「药物」「症状」等实体类型及其之间的关系；电商应用可以定义「商品」「用户」「订单」等实体类型。这种可扩展性大大拓宽了 Graphiti 的应用范围。

### 支持的图数据库后端

Graphiti 的另一个重要特性是支持多种图数据库后端，这为开发者提供了极大的部署灵活性。目前 Graphiti 支持以下图数据库：

Neo4j 是最广泛使用的图数据库，Graphiti 对其提供了完整的原生支持。Neo4j 的 Cypher 查询语言功能强大，社区活跃，文档完善，是生产环境的首选。对于已经使用 Neo4j 的团队，Graphiti 可以无缝集成到现有技术栈中。

FalkorDB 是一个高性能的图数据库，基于 Redis 构建，提供了极快的查询速度和良好的可扩展性。FalkorDB 的优势在于其内存优先的设计，适合对延迟敏感的应用场景。Graphiti 提供了对 FalkorDB 的完整支持，包括专用的 Python 客户端。

Kuzu 是一个嵌入式图数据库，以其简单易用和高效性能著称。Kuzu 无需独立的服务器进程，可以直接嵌入到应用程序中运行，这简化了部署和运维。对于希望简化基础设施的团队，Kuzu 是一个不错的选择。

Amazon Neptune 是 AWS 托管的图数据库服务，支持属性图（Property Graph）和资源描述框架（RDF）两种模型。Neptune 与 AWS 生态系统深度集成，适合已经使用 AWS 的团队。Graphiti 还支持将 Neptune 与 Amazon OpenSearch Serverless 结合使用，实现全文搜索功能。

### 与竞品的对比

为了更好地理解 Graphiti 的定位，以下是 Graphiti 与相关技术的详细对比：

**Graphiti 与 GraphRAG**代表了两种不同的技术路线。GraphRAG 是微软提出的图增强 RAG 方法，主要用于静态文档的摘要生成，采用批处理模式，适合对固定文档库进行索引和检索。相比之下，Graphiti 专注于动态数据管理，支持持续增量更新，采用时序边无效机制处理矛盾，查询延迟通常在亚秒级别，而 GraphRAG 可能需要数秒到数十秒。GraphRAG 不支持自定义实体类型，而 Graphiti 提供了完整的 Pydantic 模型扩展能力。

**Graphiti 与 Zep**的关系是开源与商业的关系。Zep 是 Graphiti 上构建的完整上下文工程平台，提供了开箱即用的用户体验、内置的用户和会话管理、预配置的高性能检索、仪表板和调试工具、企业级支持和服务等级协议。Graphiti 则是开源的核心引擎，开发者需要自行构建周围系统，包括用户管理、API 接口和运维监控。对于需要快速上线且重视稳定性的团队，Zep 是更好的选择；对于需要深度定制或希望避免供应商锁定的团队，Graphiti 提供了更大的灵活性。

## 怎么做（How）

### 安装与配置

Graphiti 的安装过程简单直接，通过 pip 或 uv 包管理器即可完成。首先确保 Python 版本不低于 3.10，然后执行以下命令：

```bash
pip install graphiti-core
```

或者使用更现代的 uv 包管理器：

```bash
uv add graphiti-core
```

如果需要使用 FalkorDB 作为图数据库后端，需要安装额外的依赖：

```bash
pip install graphiti-core[falkordb]
uv add graphiti-core[falkordb]
```

使用 Kuzu 后端：

```bash
pip install graphiti-core[kuzu]
uv add graphiti-core[kuzu]
```

使用 Amazon Neptune 后端：

```bash
pip install graphiti-core[neptune]
uv add graphiti-core[neptune]
```

Graphiti 还支持可选的 LLM 提供商，例如 Anthropic、Groq 和 Google Gemini：

```bash
pip install graphiti-core[anthropic,groq,google-genai]
```

### 快速开始示例

以下是一个完整的快速开始示例，展示了如何使用 Graphiti 构建和查询知识图谱：

```python
import asyncio
from datetime import datetime
from graphiti_core import Graphiti

async def main():
    # 初始化 Graphiti 实例
    graphiti = Graphiti(
        uri="bolt://localhost:7687",
        user="neo4j",
        password="password"
    )

    # 构建索引和约束
    await graphiti.build_indices_and_constraints()

    # 添加对话片段（Episode）
    result = await graphiti.add_episode(
        name="user-intro",
        episode_body="user: 我叫张三，今年28岁，在北京工作。",
        source_description="用户自我介绍",
        reference_time=datetime.utcnow()
    )
    print(f"添加了 {len(result.nodes)} 个实体节点")
    print(f"添加了 {len(result.edges)} 个关系边")

    # 添加更多对话
    await graphiti.add_episode(
        name="user-preference",
        episode_body="user: 我喜欢旅游，尤其是去海边。",
        source_description="用户偏好分享",
        reference_time=datetime.utcnow()
    )

    # 搜索相关知识
    search_results = await graphiti.search(
        query="张三喜欢什么"
    )
    print(f"搜索到 {len(search_results)} 条相关知识")

    # 关闭连接
    await graphiti.close()

asyncio.run(main())
```

### 核心 API 详解

Graphiti 的核心 API 围绕知识图谱的三个主要操作展开：添加（Add）、检索（Search）和更新（Update）。

**添加 Episode**是 Graphiti 最常用的操作之一。每个 Episode 代表一个数据片段，可以是对话消息、JSON 数据或自由文本。`add_episode` 方法会自动从 Episode 中提取实体和关系，将其与现有知识图谱进行去重，然后保存到数据库中。该方法返回 `AddEpisodeResults` 对象，包含新增的节点、边和社区信息。`add_episode_bulk` 方法支持批量添加多个 Episode，适合初始化场景或批量数据导入。

**检索知识**提供了多种方法以满足不同的查询需求。`search` 方法执行混合搜索，结合语义相似度、关键词匹配和图遍历，返回与查询相关的知识边。`search_` 方法是更高级的搜索接口，返回完整的图对象（包含节点和边），支持更精细的配置选项。检索结果会自动根据相关性进行重排序，确保最重要的信息排在前面。

**Saga 支持**是 Graphiti 管理长对话的高级功能。Saga 代表一个连续的主题或会话序列，Episode 可以通过 `saga` 参数关联到特定的 Saga。Graphiti 会自动为 Saga 内的 Episode 创建 `NEXT_EPISODE` 边和 `HAS_EPISODE` 边，形成完整的对话历史链。这使得系统能够理解对话的演进过程，并在检索时考虑上下文连续性。

### 多后端配置

Graphiti 的数据库驱动设计遵循工厂模式，开发者可以通过多种方式配置图数据库连接。

对于 Neo4j，除了传统的 URI、用户名和密码方式，还可以使用自定义数据库名称：

```python
from graphiti_core import Graphiti
from graphiti_core.driver.neo4j_driver import Neo4jDriver

driver = Neo4jDriver(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="password",
    database="my_custom_database"
)

graphiti = Graphiti(graph_driver=driver)
```

对于 FalkorDB：

```python
from graphiti_core import Graphiti
from graphiti_core.driver.falkordb_driver import FalkorDriver

driver = FalkorDriver(
    host="localhost",
    port=6379,
    username="falkor_user",
    password="falkor_password",
    database="my_custom_graph"
)

graphiti = Graphiti(graph_driver=driver)
```

对于 Kuzu（嵌入式）：

```python
from graphiti_core import Graphiti
from graphiti_core.driver.kuzu_driver import KuzuDriver

driver = KuzuDriver(db="/tmp/graphiti.kuzu")
graphiti = Graphiti(graph_driver=driver)
```

### 使用 Docker 快速启动服务

Graphiti 提供了 Docker Compose 配置，可以快速启动依赖服务：

```bash
# 启动 Neo4j
docker compose up

# 或者启动 FalkorDB
docker compose --profile falkordb up
```

默认的 docker-compose.yml 配置了 Neo4j 5.26 版本，端口映射为 7474（HTTP）和 7687（Bolt）。首次访问 Neo4j Browser 时需要设置新密码。

### LLM 提供商配置

Graphiti 默认使用 OpenAI 进行 LLM 推理和 Embedding 生成。通过配置环境变量或传入自定义客户端，可以切换到其他 LLM 提供商。

**使用 Google Gemini**：

```python
from graphiti_core import Graphiti
from graphiti_core.llm_client.gemini_client import GeminiClient, LLMConfig
from graphiti_core.embedder.gemini import GeminiEmbedder, GeminiEmbedderConfig
from graphiti_core.cross_encoder.gemini_reranker_client import GeminiRerankerClient

api_key = "<your-google-api-key>"

graphiti = Graphiti(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="password",
    llm_client=GeminiClient(
        config=LLMConfig(api_key=api_key, model="gemini-2.0-flash")
    ),
    embedder=GeminiEmbedder(
        config=GeminiEmbedderConfig(api_key=api_key, embedding_model="embedding-001")
    ),
    cross_encoder=GeminiRerankerClient(
        config=LLMConfig(api_key=api_key, model="gemini-2.5-flash-lite")
    )
)
```

**使用 Ollama（本地 LLM）**：

```python
from graphiti_core import Graphiti
from graphiti_core.llm_client.config import LLMConfig
from graphiti_core.llm_client.openai_generic_client import OpenAIGenericClient
from graphiti_core.embedder.openai import OpenAIEmbedder, OpenAIEmbedderConfig
from graphiti_core.cross_encoder.openai_reranker_client import OpenAIRerankerClient

llm_config = LLMConfig(
    api_key="ollama",
    model="deepseek-r1:7b",
    small_model="deepseek-r1:7b",
    base_url="http://localhost:11434/v1"
)

llm_client = OpenAIGenericClient(config=llm_config)

graphiti = Graphiti(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="password",
    llm_client=llm_client,
    embedder=OpenAIEmbedder(
        config=OpenAIEmbedderConfig(
            api_key="ollama",
            embedding_model="nomic-embed-text",
            embedding_dim=768,
            base_url="http://localhost:11434/v1",
        )
    ),
    cross_encoder=OpenAIRerankerClient(client=llm_client, config=llm_config)
)
```

使用 Ollama 时，需要确保 Ollama 服务正在运行，并已拉取所需的模型：

```bash
ollama pull deepseek-r1:7b
ollama pull nomic-embed-text
```

### 自定义实体类型

Graphiti 支持通过 Pydantic 模型定义自定义实体类型，这使得知识图谱能够适应特定领域的需求：

```python
from pydantic import BaseModel
from datetime import datetime
from typing import Optional
from graphiti_core import Graphiti

class PersonEntity(BaseModel):
    name: str
    age: int
    occupation: str

class ProductEntity(BaseModel):
    product_name: str
    brand: str
    category: str

async def main():
    graphiti = Graphiti(uri="bolt://localhost:7687", user="neo4j", password="password")

    entity_types = {
        "Person": PersonEntity,
        "Product": ProductEntity
    }

    await graphiti.add_episode(
        name="product-mention",
        episode_body="user: 张三买了一部 iPhone 15 Pro",
        source_description="产品提及",
        reference_time=datetime.utcnow(),
        entity_types=entity_types
    )

    await graphiti.close()

asyncio.run(main())
```

### 搜索配置 recipes

Graphiti 提供了多种预定义的搜索配置（recipes），适用于不同的应用场景：

```python
from graphiti_core import Graphiti
from graphiti_core.search.search_config_recipes import (
    COMBINED_HYBRID_SEARCH_CROSS_ENCODER,
    EDGE_HYBRID_SEARCH_RRF,
    NODE_HYBRID_SEARCH_CROSS_ENCODER
)

async def search_example(graphiti: Graphiti):
    # 使用 RRF 重排序的边搜索
    results = await graphiti.search_(
        query="用户偏好",
        config=EDGE_HYBRID_SEARCH_RRF
    )

    # 使用 Cross-Encoder 重排序的综合搜索
    results = await graphiti.search_(
        query="张三的工作",
        config=COMBINED_HYBRID_SEARCH_CROSS_ENCODER
    )

    # 基于节点距离的重排序
    results = await graphiti.search_(
        query="相关产品",
        config=EDGE_HYBRID_SEARCH_NODE_DISTANCE,
        center_node_uuid="node-uuid-here"
    )
```

### 最佳实践建议

在实际项目中应用 Graphiti 时，以下最佳实践可以帮助获得最佳效果：

**并发控制**是使用 Graphiti 时需要重点关注的配置。Graphiti 的摄入管道设计为高并发模式，但默认并发度较低（SEMAPHORE_LIMIT=10），以避免触发 LLM 提供商的速率限制。如果遇到 429 错误，可以降低 SEMAPHORE_LIMIT 值；如果 LLM 提供商允许更高的吞吐量，可以增加该值以提升性能。

**批量操作**适合大规模数据导入场景。使用 `add_episode_bulk` 方法可以显著提高初始数据导入的效率。但需要注意，批量操作不支持边无效化功能，如果需要处理数据冲突，仍然应该使用单个 Episode 添加方式。

**图分区管理**通过 group_id 参数实现。每个 group_id 代表图的一个分区，不同分区的数据相互隔离。合理使用分组可以提高查询效率，并实现多租户数据隔离。

**监控与追踪**对于生产环境至关重要。Graphiti 集成了 OpenTelemetry 支持，可以输出详细的追踪数据用于性能分析和问题诊断。

## 应用场景

Graphiti 适用于多种 AI 应用场景，包括但不限于：

**AI 助手记忆系统**是最直接的应用场景。Graphiti 可以为对话式 AI 助手维护长期记忆，记住用户的偏好、历史交互和关键信息，使助手能够提供更加个性化和连贯的服务。

**知识管理和问答系统**可以利用 Graphiti 的混合检索能力，从结构化和非结构化数据中快速检索相关信息，支持复杂的多跳查询。

**工作流自动化代理**可以通过 Graphiti 跟踪任务状态、历史决策和上下文信息，实现更智能的任务规划和执行。

**个性化推荐系统**可以利用 Graphiti 的时序建模能力，理解用户兴趣的演变过程，提供更加精准的个性化推荐。

## 相关资源

Graphiti 的官方文档和资源可以在以下位置找到：项目仓库位于 GitHub（github.com/getzep/graphiti），详细的 API 文档和使用指南发布在 help.getzep.com/graphiti。开发者社区活跃于 Discord 频道，学术论文发表在 arXiv 上。Zep 团队还提供了 MCP Server 实现，使 Graphiti 能够与 Claude、Cursor 等 AI 客户端无缝集成。
