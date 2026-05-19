# Semble 项目介绍

## Why - 为什么需要 Semble

### 现有代码搜索方案的痛点

在软件开发过程中，开发者经常需要快速定位和理解代码库中的特定功能或实现。然而，传统方案存在以下问题：

1. **grep + 读取方式的低效性**
   - grep 只能进行基于字符串的精确匹配
   - 返回大量文件，需要人工逐一读取
   - 上下文理解能力差，无法理解语义

2. **AI Agent 的 Token 消耗问题**
   - 上下文窗口资源有限且昂贵
   - 读取整个仓库需要消耗大量 Token
   - 平均每个搜索请求浪费约 98% 的 Token

3. **现有方案的限制**
   - 基于 Transformer 的代码搜索需要 GPU
   - 需要 API 密钥或外部服务
   - 索引速度慢，无法实时响应

### Semble 的解决之道

Semble 专为 AI Agent 设计，通过以下方式解决上述问题：

- **语义搜索**：理解代码意图，而非简单字符串匹配
- **极低 Token 消耗**：仅返回相关代码片段，减少约 98% 的 Token 使用
- **极速响应**：CPU 上毫秒级查询，无需 GPU
- **零配置**：无需 API 密钥或外部依赖

## What - Semble 是什么

### 核心定位

Semble 是一个专为 AI Agent 设计的代码搜索库，能够即时返回精确的代码片段，Token 消耗比 grep+读取减少约 98%。

### 核心特性

| 特性 | 描述 |
|------|------|
| **快速** | 平均仓库索引约 250ms，查询响应约 1.5ms，全部在 CPU 上运行 |
| **准确** | NDCG@10 达到 0.854，与代码专用 Transformer 模型相当 |
| **Token 高效** | 仅返回相关片段，比 grep+读取节省约 98% 的 Token |
| **零配置** | 无需 API 密钥、GPU 或外部服务 |
| **MCP 服务器** | 支持 Claude Code、Cursor、Codex、OpenCode 等主流 Agent |
| **本地/远程** | 支持本地路径或 Git URL |

### 技术架构

Semble 采用混合搜索策略，结合两种互补的检索器：

1. **语义检索器** - 使用 Model2Vec 静态嵌入和代码专用模型 potion-code-16M
2. **词汇检索器** - 使用 BM25 进行标识符和 API 名称的精确匹配

两种结果通过 Reciprocal Rank Fusion (RRF) 融合，再通过代码感知信号重新排序。

### 使用场景

- **代码定位**：找到特定功能或实现的代码位置
- **代码理解**：快速获取功能相关的代码片段
- **代码发现**：通过已知代码片段找到相关的其他实现
- **Agent 集成**：作为 AI Coding Agent 的搜索后端

## How - 如何使用 Semble

### 安装

```bash
pip install semble       # pip 安装
uv tool install semble   # 或使用 uv 安装
```

### MCP 服务器模式（推荐）

#### Claude Code

```bash
claude mcp add semble -s user -- uvx --from "semble[mcp]" semble
```

#### Codex

在 `~/.codex/config.toml` 中添加：

```toml
[mcp_servers.semble]
command = "uvx"
args = ["--from", "semble[mcp]", "semble"]
```

#### Cursor

在 `~/.cursor/mcp.json` 中添加：

```json
{
  "mcpServers": {
    "semble": {
      "command": "uvx",
      "args": ["--from", "semble[mcp]", "semble"]
    }
  }
}
```

### CLI 命令行模式

#### 搜索代码

```bash
# 自然语言查询
semble search "authentication flow" ./my-project

# 符号/标识符查询
semble search "save_pretrained" ./my-project

# 限制结果数量
semble search "save model to disk" ./my-project --top-k 10

# 搜索远程仓库
semble search "save model to disk" https://github.com/MinishLab/model2vec
```

#### 查找相关代码

```bash
semble find-related src/auth.py 42 ./my-project
```

### Python 库使用

```python
from semble import SembleIndex

# 从本地目录创建索引
index = SembleIndex.from_path("./my-project")

# 从远程 Git 仓库创建索引
index = SembleIndex.from_git("https://github.com/MinishLab/model2vec")

# 执行搜索
results = index.search("save model to disk", top_k=3)

# 查找相关代码
related = index.find_related(results[0], top_k=3)

# 访问结果
result = results[0]
print(result.chunk.file_path)   # "model2vec/model.py"
print(result.chunk.start_line)  # 127
print(result.chunk.end_line)    # 150
print(result.chunk.content)     # "def save_pretrained(self, path: PathLike, ..."
```

### 搜索模式

| 模式 | 说明 |
|------|------|
| `hybrid` | 结合语义和 BM25（默认） |
| `semantic` | 仅使用语义嵌入 |
| `bm25` | 仅使用词汇匹配 |

### 查看节省统计

```bash
semble savings           # 摘要统计
semble savings --verbose # 详细统计
```

## 核心模块说明

### 索引模块 (`src/semble/index/`)

- **index.py**: SembleIndex 主类，管理索引和搜索
- **create.py**: 从路径创建索引
- **dense.py**: 语义嵌入索引
- **sparse.py**: BM25 稀疏索引
- **file_walker.py**: 文件遍历和过滤

### 搜索模块 (`src/semble/search.py`)

实现三种搜索模式：
- `search_semantic`: 纯语义搜索
- `search_bm25`: 纯词汇搜索
- `search_hybrid`: 混合搜索（默认）

### 排序模块 (`src/semble/ranking/`)

- **boosting.py**: 提升相关代码块的排名
- **penalties.py**: 对噪音文件（如测试文件）进行降权
- **weighting.py**: 自适应权重调整

### 分块模块 (`src/semble/chunking/`)

使用 tree-sitter 进行代码感知的分块：
- **core.py**: 核心分块逻辑
- **chunking.py**: 分块策略

### MCP 服务 (`src/semble/mcp.py`)

实现 MCP 服务器，支持：
- `search`: 自然语言或代码查询
- `find_related`: 查找相关代码

## 许可证

MIT License
