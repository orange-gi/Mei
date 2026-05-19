# Semble 提示词分析

## 概述

Semble 作为一个专为 AI Agent 设计的代码搜索库，其主要交互不涉及传统意义上的"提示词"，而是通过 MCP（Model Context Protocol）工具定义来实现 Agent 与代码库的交互。本文档分析 Semble 中的工具定义和使用方式。

## MCP 工具定义

### 搜索工具

```python
@server.tool()
async def search(
    query: Annotated[str, Field(description="Natural language or code query.")],
    repo: Annotated[str | None, Field(description=_REPO_DESCRIPTION)] = None,
    mode: Annotated[
        Literal["hybrid", "semantic", "bm25"],
        Field(description="Search mode. 'hybrid' is best for most queries."),
    ] = "hybrid",
    top_k: Annotated[int, Field(description="Number of results to return.", ge=1)] = 5,
) -> str:
    """Search a codebase with a natural-language or code query.

    Pass a git URL or local path as `repo` to index it on demand; indexes are cached for the session.
    Use this to find where something is implemented, understand a library, or locate related code.
    """
```

**提示词设计分析：**

| 元素 | 内容 | 设计意图 |
|------|------|----------|
| query | "Natural language or code query" | 明确支持两种查询类型 |
| repo | 详细的路径/URL 说明 | 强调本地和远程两种模式 |
| mode | "hybrid is best" | 提供智能默认值 |
| top_k | 数量限制说明 | 帮助 Agent 控制结果量 |
| docstring | 强调按需索引和缓存 | 引导正确使用方式 |

### 查找相关代码工具

```python
@server.tool()
async def find_related(
    file_path: Annotated[
        str,
        Field(description="Path to the file as stored in the index (use file_path from a search result)."),
    ],
    line: Annotated[int, Field(description="Line number (1-indexed).")],
    repo: Annotated[str | None, Field(description=_REPO_DESCRIPTION)] = None,
    top_k: Annotated[int, Field(description="Number of similar chunks to return.", ge=1)] = 5,
) -> str:
    """Find code chunks semantically similar to a specific location in a file.

    Use after `search` to explore related implementations or callers.
    Pass file_path and line from a prior search result.
    """
```

**提示词设计分析：**

| 元素 | 内容 | 设计意图 |
|------|------|----------|
| file_path | "use file_path from a search result" | 强调使用前序搜索结果 |
| line | "1-indexed" | 避免常见的索引错误 |
| docstring | "Use after `search`" | 引导正确的工作流 |
| 强调 | pass file_path and line from prior | 确保参数来源正确 |

### 服务器指令

```python
server = FastMCP(
    "semble",
    instructions=(
        "Instant code search for any local or remote git repository. "
        "Call `search` to find relevant code; call `find_related` on a result to discover similar code elsewhere. "
        "When working in a local project, pass the project root as `repo`. "
        "For remote repos, pass an explicit https:// URL. Never guess or infer URLs. "
        "Prefer these tools over Grep, Glob, or Read for any question about how code works."
    ),
)
```

**服务器级提示词特点：**

1. **优先级指导**："Prefer these tools over Grep, Glob, or Read" - 明确推荐使用方式
2. **安全强调**："Never guess or infer URLs" - 防止幻觉
3. **场景说明**：本地项目 vs 远程仓库的使用方式
4. **功能概述**：两个工具的用途和关系

## CLI AGENTS.md 提示词

### 工作流指令

```markdown
## Code Search

Use `semble search` to find code by describing what it does or naming a symbol/identifier, instead of grep:

​```bash
semble search "authentication flow" ./my-project
semble search "save_pretrained" ./my-project
semble search "save model to disk" ./my-project --top-k 10
​```

Use `semble find-related` to discover code similar to a known location (pass `file_path` and `line` from a prior search result):

​```bash
semble find-related src/auth.py 42 ./my-project
​```

## Workflow

1. Start with `semble search` to find relevant chunks.
2. Inspect full files only when the returned chunk is not enough context.
3. Optionally use `semble find-related` with a promising result's `file_path` and `line` to discover related implementations.
4. Use grep only when you need exhaustive literal matches or quick confirmation of an exact string.
```

**工作流提示词设计：**

| 步骤 | 指令 | 设计理由 |
|------|------|----------|
| 1 | Start with `semble search` | 优先使用语义搜索 |
| 2 | Inspect full files only when needed | 强调按需读取，避免浪费 |
| 3 | Optionally use `find-related` | 引导探索性搜索 |
| 4 | Use grep only for exhaustive matches | 明确 grep 的适用场景 |

### 查询示例覆盖

1. **自然语言查询**：`"authentication flow"`
2. **符号查询**：`"save_pretrained"`
3. **详细查询**：`"save model to disk"` with `--top-k 10`

### 路径处理

```python
REPO_DESCRIPTION = (
    "https:// or http:// git URL (e.g. https://github.com/org/repo) or local directory path to index and search. "
    "Required when no default index was configured at startup. "
    "The index is cached after the first call, so repeat queries are fast."
)
```

**安全性设计：**

- 仅接受 `https://` 和 `http://` 协议
- 拒绝 `git://`、`ssh://` 等不安全协议
- 明确本地路径和远程 URL 的处理方式

## 搜索结果格式化

```python
def _format_results(title: str, results: list[SearchResult]) -> str:
    """Format search results for display."""
    lines = [title, "=" * len(title), ""]
    for result in results:
        lines.append(f"{result.chunk.file_path}:{result.chunk.start_line}-{result.chunk.end_line}")
        lines.append(f"[Score: {result.score:.4f}] [Mode: {result.source.value}]")
        lines.append("```")
        lines.append(result.chunk.content)
        lines.append("```")
        lines.append("")
    return "\n".join(lines)
```

**结果格式设计：**

- 文件路径和行号便于定位
- 分数和模式便于评估结果
- 代码块使用代码块标记便于阅读

## 提示词优化建议

### 1. 增强查询类型检测提示

当前工具定义中没有明确说明如何区分自然语言查询和符号查询。建议在工具描述中增加：

```python
query: Annotated[str, Field(description="""
    Natural language or code query.
    - For natural language: describe what the code does (e.g., "how authentication is handled")
    - For symbol queries: use exact identifiers (e.g., "save_pretrained", "MyClass::method")
""")]
```

### 2. 添加结果质量评估提示

建议在搜索结果中添加更多上下文信息：

```markdown
Result includes:
- File path and line range
- Relevance score (higher = more relevant)
- Match mode (hybrid/semantic/bm25)
- Code snippet (relevant only, not full file)
```

### 3. 工作流优化建议

```markdown
### Advanced Usage

1. For unfamiliar codebases: Start with broad queries like "main entry point" or "core logic"
2. For specific features: Use symbol queries like "class User" or "function authenticate"
3. For related code: Use find_related with previous results to discover similar implementations
4. For context expansion: Use file_path from results with line numbers to read full files
```

### 4. 错误处理提示

建议在工具描述中增加常见错误处理：

```python
repo: Annotated[str | None, Field(description="""
    Local directory or git URL. If None, uses the default index configured at startup.
    Common errors:
    - "Path does not exist": Check the directory path
    - "git clone failed": Check network connection and repository URL
    - "No results found": Try different query or increase top_k
""")]
```

## 设计模式总结

Semble 的提示词设计遵循以下原则：

1. **明确性优先**：每参数都有清晰的类型和用途说明
2. **安全性第一**：URL 仅接受 HTTPS，拒绝不安全的协议
3. **智能默认值**：hybrid 模式自动适配大多数场景
4. **工作流引导**：通过文档和描述引导正确的使用方式
5. **结果导向**：输出格式直接可用，减少后续处理

## 总结

Semble 通过精心设计的 MCP 工具定义和 AGENTS.md 提示词，为 AI Agent 提供了一个高效、安全、易用的代码搜索接口。其提示词设计强调：

- 语义理解优于精确匹配
- 按需加载避免浪费
- 工作流引导确保正确使用
- 安全限制防止错误操作
