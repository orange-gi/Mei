# PentAGI 项目介绍

## 一、为什么存在（Why）

### 1.1 项目背景与问题陈述

在当今数字化时代，网络安全威胁日益复杂和多样化。传统的渗透测试方法面临诸多挑战：人工渗透测试不仅耗时耗力，而且高度依赖测试人员的经验和技能水平。随着网络攻击技术的不断演进，安全专业人员需要一种更高效、更智能的解决方案来应对日益增长的安全测试需求。

**现有方案的局限性**主要体现在以下几个方面：

- **效率低下**：人工渗透测试需要投入大量时间和人力资源，从信息收集到漏洞利用，每个环节都需要专业人员进行操作
- **技能依赖**：测试质量高度依赖于测试人员的经验和知识积累，不同水平的测试人员会产生差异显著的测试结果
- **覆盖不足**：人工测试难以全面覆盖所有攻击向量和漏洞类型，容易遗漏潜在的安全问题
- **成本高昂**：专业的渗透测试服务费用昂贵，中小型企业难以负担频繁的安全评估成本
- **知识孤岛**：每次测试的经验和发现难以有效复用，后续测试需要重新开始

### 1.2 项目目标与愿景

PentAGI（Penetration Testing Artificial General Intelligence）应运而生，旨在通过人工智能技术彻底改变渗透测试的方式。该项目的核心目标包括：

- **自动化渗透测试**：利用AI代理实现完全自动化的渗透测试流程，减少对人工干预的依赖
- **智能化决策**：通过大型语言模型（LLM）实现智能化的测试策略规划和漏洞利用决策
- **知识积累**：建立长期记忆系统，积累每次测试的经验和发现，形成可复用的知识库
- **专业工具集成**：集成20+专业安全工具，确保测试的专业性和全面性

### 1.3 目标用户群体

PentAGI 主要面向以下用户群体：

- **安全研究人员**：需要高效工具辅助进行安全研究和漏洞发现的专业人员
- **渗透测试工程师**：希望自动化常规测试流程，专注于复杂安全问题的专家
- **安全爱好者**：对网络安全感兴趣，希望学习和实践渗透测试技术的爱好者
- **企业安全团队**：需要定期进行安全评估，但资源有限的组织
- **DevSecOps团队**：希望在开发周期中集成自动化安全测试的开发团队

## 二、是什么（What）

### 2.1 项目核心定义

PentAGI 是一个创新性的自动化安全测试工具，它将尖端人工智能技术与专业渗透测试工具深度融合。该系统采用多代理架构，通过专门化的AI代理协同工作，完成从目标分析到漏洞利用的完整渗透测试流程。

作为一个自托管解决方案，PentAGI 完全运行在用户自己的基础设施上，确保敏感测试数据的完全控制和隐私保护。用户可以选择多种大型语言模型提供商，包括OpenAI、Anthropic、Ollama、AWS Bedrock、Google AI等，满足不同场景的需求。

### 2.2 核心功能特性

PentAGI 提供了一系列强大而全面的功能特性：

**安全隔离与保护**

- 所有操作均在沙盒化的Docker环境中执行，确保完全隔离
- 容器化设计防止测试活动对宿主机造成影响
- 支持分布式双节点架构，将测试操作隔离在独立服务器上

**AI驱动自动化**

- 完全自主的AI代理系统，自动确定并执行渗透测试步骤
- 多代理协作系统，包含研究者、开发者、执行者等专门化角色
- 智能上下文管理，通过链式摘要技术处理长对话

**专业工具套件**

- 内置20+专业安全工具，包括nmap、metasploit、sqlmap等
- 自动Docker镜像选择，根据任务需求智能选择最合适的工具环境
- 沙盒化执行环境，确保工具使用的安全性

**智能记忆系统**

- 长期存储研究结果和成功方法，供将来使用
- 向量数据库存储，支持语义搜索和相似性查询
- 知识图谱集成（Graphiti + Neo4j），追踪实体间的语义关系

**搜索与情报**

- 内置浏览器用于从Web资源获取最新信息
- 支持多种外部搜索系统集成：
  - Tavily AI搜索引擎
  - Traversaal AI搜索引擎
  - Perplexity AI搜索
  - DuckDuckGo
  - Google自定义搜索
  - Searxng元搜索引擎

**监控与分析**

- 详细的日志记录和监控面板
- Grafana/Prometheus集成，实时监控系统状态
- Langfuse集成，监控AI代理性能和调用追踪
- OpenTelemetry支持，分布式追踪和可观测性

**报告与可视化**

- 生成详细的漏洞报告，包含利用指南
- 现代化的Web UI界面，便于管理和监控
- REST和GraphQL API支持，便于与外部系统集成

### 2.3 技术架构概述

PentAGI 采用微服务架构设计，支持水平扩展。系统主要由以下核心组件构成：

**前端服务**

- React + TypeScript 构建的现代化Web界面
- 响应式设计，支持多种设备和屏幕尺寸

**后端服务**

- Go语言实现的REST和GraphQL API
- 高性能处理能力，支持并发任务执行

**数据存储**

- PostgreSQL + pgvector 向量存储
- 语义搜索和记忆存储
- Neo4j图数据库（知识图谱）
- Redis缓存和速率限制
- MinIO对象存储

**任务队列**

- 异步任务处理系统
- 可靠的任务调度和执行

**AI代理系统**

- 多代理协作框架
- 专门化的代理角色分工
- 智能任务委派机制

## 三、如何使用（How）

### 3.1 快速开始指南

PentAGI 提供多种安装方式，推荐使用交互式安装程序进行快速部署。

**系统要求**

- Docker和Docker Compose
- 最少2个vCPU
- 最少4GB内存
- 20GB可用磁盘空间
- 网络连接（用于下载镜像和更新）

**使用安装程序（推荐）**

对于Linux amd64系统：

```bash
# 创建安装目录
mkdir -p pentagi && cd pentagi

# 下载安装程序
wget -O installer.zip https://pentagi.com/downloads/linux/amd64/installer-latest.zip

# 解压
unzip installer.zip

# 运行交互式安装程序
sudo ./installer
```

安装程序将引导完成以下步骤：

1. 系统检查：验证Docker、网络连接和系统要求
2. 环境配置：创建和配置。env文件
3. 提供商设置：配置LLM提供商（OpenAI、Anthropic、Gemini等）
4. 搜索引擎配置：设置DuckDuckGo、Google、Tavily等
5. 安全加固：生成安全凭据和配置SSL证书
6. 部署：使用docker-compose启动PentAGI

**手动安装**

```bash
# 创建工作目录
mkdir pentagi && cd pentagi

# 复制环境变量示例文件
curl -o .env https://raw.githubusercontent.com/vxcontrol/pentagi/master/.env.example

# 复制提供商配置示例
curl -o example.custom.provider.yml https://raw.githubusercontent.com/vxcontrol/pentagi/master/examples/configs/custom-openai.provider.yml
curl -o example.ollama.provider.yml https://raw.githubusercontent.com/vxcontrol/pentagi/master/examples/configs/ollama-llama318b.provider.yml

# 编辑。env文件，填写必要的API密钥
vim .env

# 启动PentAGI栈
curl -O https://raw.githubusercontent.com/vxcontrol/pentagi/master/docker-compose.yml
docker compose up -d
```

### 3.2 LLM提供商配置

PentAGI 支持多种大型语言模型提供商，配置灵活：

**OpenAI配置**

```bash
OPEN_AI_KEY=your_openai_api_key
OPEN_AI_SERVER_URL=https://api.openai.com/v1
```

**Anthropic（Claude）配置**

```bash
ANTHROPIC_API_KEY=your_anthropic_api_key
ANTHROPIC_SERVER_URL=https://api.anthropic.com/v1
```

**Google AI（Gemma）配置**

```bash
GEMINI_API_KEY=your_gemini_api_key
GEMINI_SERVER_URL=https://generativelanguage.googleapis.com
```

**AWS Bedrock配置**

```bash
BEDROCK_REGION=us-east-1
BEDROCK_ACCESS_KEY_ID=your_aws_access_key
BEDROCK_SECRET_ACCESS_KEY=your_aws_secret_key
```

**Ollama本地模型配置**

```bash
OLLAMA_SERVER_URL=http://localhost:11434
OLLAMA_SERVER_MODEL=llama3.1:8b-instruct-q8_0
```

### 3.3 搜索引擎配置

为获得更好的测试结果，建议配置搜索引擎：

```bash
# 必需至少配置一个LLM提供商
DUCKDUCKGO_ENABLED=true
GOOGLE_API_KEY=your_google_key
GOOGLE_CX_KEY=your_google_cx
TAVILY_API_KEY=your_tavily_key
TRAVERSAAL_API_KEY=your_traversaal_key
PERPLEXITY_API_KEY=your_perplexity_key

# Searxng元搜索引擎
SEARXNG_URL=http://your-searxng-instance:8080
```

### 3.4 访问与使用

部署完成后，通过以下方式访问PentAGI：

- **Web UI**：访问 https://localhost:8443
- **默认凭据**：admin@pentagi.com / admin
- **首次登录后请立即修改默认密码**

### 3.5 高级功能配置

**Langfuse集成（AI监控）**

```bash
# 在。env中添加Langfuse配置
LANGFUSE_BASE_URL=http://langfuse-web:3000
LANGFUSE_PROJECT_ID=your_project_id
LANGFUSE_PUBLIC_KEY=your_public_key
LANGFUSE_SECRET_KEY=your_secret_key

# 启动Langfuse栈
curl -O https://raw.githubusercontent.com/vxcontrol/pentagi/master/docker-compose-langfuse.yml
docker compose -f docker-compose.yml -f docker-compose-langfuse.yml up -d
```

**知识图谱集成（Graphiti）**

```bash
# 在。env中启用Graphiti
GRAPHITI_ENABLED=true
GRAPHITI_URL=http://graphiti:8000
GRAPHITI_MODEL_NAME=gpt-5-mini

# Neo4j配置
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password
NEO4J_URI=bolt://neo4j:7687

# 启动Graphiti栈
curl -O https://raw.githubusercontent.com/vxcontrol/pentagi/master/docker-compose-graphiti.yml
docker compose -f docker-compose.yml -f docker-compose-graphiti.yml up -d
```

**监控与可观测性**

```bash
# 启用OpenTelemetry
OTEL_HOST=otelcol:8148

# 启动监控栈
curl -O https://raw.githubusercontent.com/vxcontrol/pentagi/master/docker-compose-observability.yml
docker compose -f docker-compose.yml -f docker-compose-observability.yml up -d

# 访问Grafana
# http://localhost:3000
```

### 3.6 最佳实践建议

**安全建议**

- 强烈建议使用分布式双节点架构，将测试操作隔离在独立服务器上
- 及时修改默认凭据，使用强密码
- 配置SSL证书启用HTTPS
- 定期更新Docker镜像获取最新安全修复
- 妥善保管API密钥，不要提交到版本控制系统

**性能优化**

- 根据实际需求选择合适的LLM模型，平衡性能和成本
- 对于本地部署，建议使用具有足够GPU内存的设备运行Ollama
- 合理配置知识图谱和向量数据库，定期清理不再需要的数据

**使用技巧**

- 充分利用知识图谱功能，积累测试经验
- 定期查看Langfuse监控面板，优化代理行为
- 使用ftester工具进行调试和问题诊断
- 关注官方Discord和Telegram社区，获取支持和更新信息

## 四、总结

PentAGI 代表了渗透测试领域的创新突破，通过将人工智能与专业安全工具的深度结合，为安全专业人员提供了强大而灵活的自动化测试解决方案。无论您是安全研究人员、渗透测试工程师还是安全爱好者，PentAGI都能帮助您更高效、更智能地完成安全评估工作。

作为一款开源自托管解决方案，PentAGI确保了数据的完全控制和隐私保护，让组织能够在其自己的基础设施上运行敏感的安全测试。通过持续的功能更新和社区支持，PentAGI正在不断演进，为网络安全领域带来更多创新价值。