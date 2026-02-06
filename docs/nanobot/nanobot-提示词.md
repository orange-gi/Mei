# nanobot 提示词与系统配置分析

## 一、概述

nanobot 作为一款超轻量级的个人 AI 助手框架，其提示词系统采用了**动态构建**与**模板化配置**相结合的设计模式。与传统的固定提示词模板不同，nanobot 的系统提示由多个可配置的组件动态组装而成，这种设计既保证了核心行为的一致性，又提供了灵活的自定义能力。

从技术实现角度来看，nanobot 的提示词系统包含三个层次：**硬编码的核心身份定义**、**可配置的引导文件**以及**运行时动态生成的内容**。这种分层设计使得系统管理员可以根据需求调整 Agent 的行为特征，而无需修改核心代码。

需要特别说明的是，nanobot 本身并非一个"提示词项目"，而是一个完整的 AI Agent 框架。其提示词相关功能主要体现在系统提示的构建机制上，而非预设的提示词模板。因此，本文档将从系统提示构建机制的角度进行分析，而非简单地罗列提示词文本。

## 二、系统提示词构建机制

### 2.1 构建入口与流程

nanobot 的系统提示词由 `ContextBuilder` 类负责构建，该类位于 `nanobot/agent/context.py` 文件中。构建流程遵循清晰的优先级顺序，从核心身份信息开始，逐步叠加记忆上下文、技能定义等组件。

```python
class ContextBuilder:
    """
    Builds the system prompt from bootstrap files, memory, and skills.
    """

    BOOTSTRAP_FILES = ["AGENTS.md", "SOUL.md", "USER.md", "TOOLS.md", "IDENTITY.md"]

    def build_system_prompt(self, skill_names: list[str] | None = None) -> str:
        """
        Build the system prompt from bootstrap files, memory, and skills.
        """
        parts = []

        # 第一步：添加核心身份信息
        parts.append(self._get_identity())

        # 第二步：加载引导文件
        bootstrap = self._load_bootstrap_files()
        if bootstrap:
            parts.append(bootstrap)

        # 第三步：加载记忆上下文
        memory = self.memory.get_memory_context()
        if memory:
            parts.append(f"# Memory\n\n{memory}")

        # 第四步：加载 Always Skills（始终加载的技能）
        always_skills = self.skills.get_always_skills()
        if always_skills:
            always_content = self.skills.load_skills_for_context(always_skills)
            if always_content:
                parts.append(f"# Active Skills\n\n{always_content}")

        # 第五步：生成可用技能摘要
        skills_summary = self.skills.build_skills_summary()
        if skills_summary:
            parts.append(f"# Skills\n\n{skills_summary}")

        return "\n\n---\n\n".join(parts)
```

### 2.2 核心身份提示词

nanobot 的核心身份信息是系统提示词中唯一硬编码的部分，位于 `_get_identity()` 方法中。这部分内容定义了 Agent 的基本角色、可用工具和工作空间结构。

```python
def _get_identity(self) -> str:
    """Get the core identity section."""
    from datetime import datetime
    now = datetime.now().strftime("%Y-%m-%d %H:%M (%A)")
    workspace_path = str(self.workspace.expanduser().resolve())

    return f"""# nanobot 🐈

You are nanobot, a helpful AI assistant. You have access to tools that allow you to:
- Read, write, and edit files
- Execute shell commands
- Search the web and fetch web pages
- Send messages to users on chat channels
- Spawn subagents for complex background tasks

## Current Time
{now}

## Workspace
Your workspace is at: {workspace_path}
- Memory files: {workspace_path}/memory/MEMORY.md
- Daily notes: {workspace_path}/memory/YYYY-MM-DD.md
- Custom skills: {workspace_path}/skills/{{skill-name}}/SKILL.md

IMPORTANT: When responding to direct questions or conversations, reply directly with your text response.
Only use the 'message' tool when you need to send a message to a specific chat channel (like WhatsApp).
For normal conversation, just respond with text - do not call the message tool.

Always be helpful, accurate, and concise. When using tools, explain what you're doing.
When remembering something, write to {workspace_path}/memory/MEMORY.md"""
```

**设计特点分析：**

1. **时间感知**：`{now}` 变量动态注入当前时间，使 Agent 能够进行时间相关的推理。

2. **工作空间定义**：明确指定工作空间路径和重要子目录的位置，帮助 Agent 理解文件组织结构。

3. **工具使用指导**：明确说明何时应该直接回复、何时应该使用 message 工具发送消息，避免 Agent 误用工具。

4. **记忆指导**：强调重要信息应写入 MEMORY.md 文件，引导 Agent 进行长期记忆。

### 2.3 引导文件加载机制

引导文件（Bootstrap Files）是 nanobot 自定义 Agent 行为的主要方式。这些文件位于工作空间的 `bootstrap/` 目录下，用户可以通过编辑这些文件来调整 Agent 的详细行为配置。

```python
def _load_bootstrap_files(self) -> str:
    """Load all bootstrap files from workspace."""
    parts = []

    for filename in self.BOOTSTRAP_FILES:
        file_path = self.workspace / filename
        if file_path.exists():
            content = file_path.read_text(encoding="utf-8")
            parts.append(f"## {filename}\n\n{content}")

    return "\n\n".join(parts) if parts else ""
```

**引导文件说明：**

| 文件名 | 用途 | 配置内容 |
|--------|------|----------|
| `AGENTS.md` | Agent 行为模式定义 | Agent 的思考方式、响应风格等 |
| `SOUL.md` | 价值观和行为准则 | Agent 的核心价值观、伦理准则 |
| `USER.md` | 用户信息 | 用户的偏好、背景信息 |
| `TOOLS.md` | 工具详细说明 | 工具的详细使用说明和示例 |
| `IDENTITY.md` | 身份扩展定义 | 额外的身份信息或角色设定 |

这种设计允许用户在不修改代码的情况下，完全自定义 Agent 的行为特征。

## 三、记忆系统提示词

### 3.1 记忆存储机制

nanobot 的记忆系统由 `MemoryStore` 类实现，位于 `nanobot/agent/memory.py` 文件中。记忆系统负责管理长期记忆和短期会话历史，为 Agent 提供上下文记忆能力。

```python
class MemoryStore:
    """Persistent memory for the agent."""

    def __init__(self, workspace: Path):
        self.workspace = workspace
        self.memory_dir = workspace / "memory"
        self.memory_dir.mkdir(exist_ok=True)

    def get_memory_context(self) -> str:
        """
        Get the memory context for the system prompt.
        Reads from MEMORY.md for persistent memories.
        """
        memory_file = self.memory_dir / "MEMORY.md"
        if memory_file.exists():
            return memory_file.read_text(encoding="utf-8")
        return ""

    def add_memory(self, content: str) -> None:
        """Add a new memory entry."""
        memory_file = self.memory_dir / "MEMORY.md"
        with open(memory_file, "a", encoding="utf-8") as f:
            f.write(f"\n{content}")

    def get_today_notes(self) -> str:
        """Get notes for the current day."""
        from datetime import datetime
        today_file = self.memory_dir / f"{datetime.now().strftime('%Y-%m-%d')}.md"
        if today_file.exists():
            return today_file.read_text(encoding="utf-8")
        return ""
```

### 3.2 记忆提示词整合

当系统提示词构建时，记忆内容以独立的 `# Memory` 部分插入。这种设计确保了重要信息能够被 Agent 在每次决策时参考。

```python
memory = self.memory.get_memory_context()
if memory:
    parts.append(f"# Memory\n\n{memory}")
```

**记忆文件格式建议：**

```markdown
# Memory

## 用户偏好
- 称呼用户为"您"
- 使用中文回复

## 重要日期
- 用户生日：2月15日
- 项目截止日期：3月1日

## 工作习惯
- 偏好简洁的代码风格
- 每天早上9点查看日程
```

## 四、技能系统提示词

### 4.1 技能加载机制

nanobot 的技能系统提供了更高层次的功能扩展能力。`SkillsLoader` 类负责加载和管理技能定义，支持 Always Skills（始终加载）和 Available Skills（按需加载）两种模式。

```python
class SkillsLoader:
    """Loads and manages skills for the agent."""

    def __init__(self, workspace: Path):
        self.workspace = workspace
        self.skills_dir = workspace / "skills"

    def get_always_skills(self) -> list[str]:
        """Get list of skills that are always loaded."""
        always_file = self.skills_dir / "always.txt"
        if always_file.exists():
            return always_file.read_text().strip().split("\n")
        return []

    def build_skills_summary(self) -> str:
        """Build a summary of available skills."""
        # 列出所有可用技能及其描述
        # Agent 可以通过 read_file 工具读取具体技能内容
        pass
```

### 4.2 技能提示词结构

技能提示词采用摘要式呈现，避免系统提示词过长。Agent 需要使用 `read_file` 工具读取具体技能内容后，才能使用该技能。

```python
skills_summary = self.skills.build_skills_summary()
if skills_summary:
    parts.append(f"""# Skills

The following skills extend your capabilities. To use a skill, read its SKILL.md file using the read_file tool.
Skills with available="false" need dependencies installed first - you can try installing them with apt/brew.

{skills_summary}""")
```

**技能文件结构：**

```
skills/
├── github/
│   └── SKILL.md          # GitHub 操作技能
├── weather/
│   └── SKILL.md           # 天气查询技能
└── always.txt             # 始终加载的技能列表
```

**SKILL.md 模板示例：**

```markdown
# GitHub 操作技能

## 描述
用于管理 GitHub 仓库、Issue 和 PR。

## 使用场景
- 创建 Issue
- 评论 Issue
- 查看 PR 状态

## 依赖
- GitHub Token（通过环境变量或配置文件）

## 示例使用
"帮我创建一个 Issue，标题是 Bug 报告"
"查看仓库的 PR 列表"
```

## 五、工具定义与提示词

### 5.1 工具定义格式

nanobot 的工具系统使用 OpenAI 的 Function Calling 格式定义工具。每个工具包含名称、描述和参数模式，这些信息会被传递给 LLM 以决定何时以及如何调用工具。

```python
class BaseTool:
    """Base class for all tools."""

    @property
    @abstractmethod
    def name(self) -> str:
        """Tool name for function calling."""
        pass

    @property
    @abstractmethod
    def description(self) -> str:
        """Tool description for the system prompt."""
        pass

    @property
    @abstractmethod
    def parameters(self) -> dict:
        """JSON Schema for tool parameters."""
        pass
```

### 5.2 内置工具定义

以下是 nanobot 内置工具的定义示例，展示了工具提示词的设计模式：

```python
class ReadFileTool(BaseTool):
    """Tool for reading files."""

    @property
    def name(self) -> str:
        return "read_file"

    @property
    def description(self) -> str:
        return """Read the contents of a file from the workspace.
Use this to read configuration files, source code, or any text content.
The path should be relative to the workspace root."""

    @property
    def parameters(self) -> dict:
        return {
            "type": "object",
            "properties": {
                "path": {
                    "type": "string",
                    "description": "Path to the file, relative to workspace root"
                }
            },
            "required": ["path"]
        }
```

**完整工具列表：**

| 工具名称 | 功能描述 | 参数 |
|----------|----------|------|
| `read_file` | 读取文件内容 | path: 文件路径 |
| `write_file` | 创建或覆盖文件 | path, content |
| `edit_file` | 编辑文件 | path, old_string, new_string |
| `list_dir` | 列出目录内容 | path |
| `exec` | 执行 Shell 命令 | command, timeout |
| `web_search` | 网页搜索 | query |
| `web_fetch` | 获取网页内容 | url |
| `message` | 发送消息到渠道 | channel, chat_id, content |
| `spawn` | 启动子代理 | name, goal, system_prompt |
| `cron_add` | 添加定时任务 | name, message, cron |
| `cron_list` | 列出定时任务 | (无) |
| `cron_remove` | 删除定时任务 | job_id |

### 5.3 工具参数验证

nanobot 使用 Pydantic 进行工具参数的验证和类型转换，确保传递给工具的参数符合预期格式。

```python
from pydantic import BaseModel, Field

class WriteFileToolConfig(BaseModel):
    """Configuration for write_file tool."""
    path: str = Field(..., description="Path to the file")
    content: str = Field(..., description="Content to write")
```

## 六、消息格式与对话上下文

### 6.1 消息结构定义

nanobot 使用标准化的消息格式与 LLM 进行通信。消息格式遵循 OpenAI 的 Chat Completion API 规范，支持多种角色类型。

```python
class ContextBuilder:
    """Builds messages for LLM calls."""

    def build_messages(
        self,
        history: list[dict[str, Any]],
        current_message: str,
        skill_names: list[str] | None = None,
        media: list[str] | None = None,
        channel: str | None = None,
        chat_id: str | None = None,
    ) -> list[dict[str, Any]]:
        """
        Build the complete message list for an LLM call.
        """
        messages = []

        # 系统提示词
        system_prompt = self.build_system_prompt(skill_names)
        if channel and chat_id:
            system_prompt += f"\n\n## Current Session\nChannel: {channel}\nChat ID: {chat_id}"
        messages.append({"role": "system", "content": system_prompt})

        # 对话历史
        messages.extend(history)

        # 当前消息
        user_content = self._build_user_content(current_message, media)
        messages.append({"role": "user", "content": user_content})

        return messages
```

### 6.2 工具调用消息格式

当 LLM 返回工具调用请求时，消息格式需要包含 tool_calls 和 tool_results 两种特殊消息类型。

```python
def add_tool_result(
    self,
    messages: list[dict[str, Any]],
    tool_call_id: str,
    tool_name: str,
    result: str
) -> list[dict[str, Any]]:
    """Add a tool result to the message list."""
    messages.append({
        "role": "tool",
        "tool_call_id": tool_call_id,
        "name": tool_name,
        "content": result
    })
    return messages

def add_assistant_message(
    self,
    messages: list[dict[str, Any]],
    content: str | None,
    tool_calls: list[dict[str, Any]] | None = None
) -> list[dict[str, Any]]:
    """Add an assistant message to the message list."""
    msg: dict[str, Any] = {"role": "assistant", "content": content or ""}
    if tool_calls:
        msg["tool_calls"] = tool_calls
    messages.append(msg)
    return messages
```

### 6.3 多模态内容处理

nanobot 支持在消息中附加图像内容，通过 base64 编码将本地图片转换为 Data URL 格式传递给 LLM。

```python
def _build_user_content(self, text: str, media: list[str] | None) -> str | list[dict[str, Any]]:
    """Build user message content with optional base64-encoded images."""
    if not media:
        return text

    images = []
    for path in media:
        p = Path(path)
        mime, _ = mimetypes.guess_type(path)
        if not p.is_file() or not mime or not mime.startswith("image/"):
            continue
        b64 = base64.b64encode(p.read_bytes()).decode()
        images.append({
            "type": "image_url",
            "image_url": {"url": f"data:{mime};base64,{b64}"}
        })

    if not images:
        return text
    return images + [{"type": "text", "text": text}]
```

## 七、设计模式与最佳实践

### 7.1 提示词设计模式

nanobot 的提示词系统体现了多种设计模式的综合运用：

**策略模式**体现在工具系统的设计上。每种工具都实现了统一的接口（BaseTool），使得添加新工具变得简单，同时保持系统的可扩展性。

**模板方法模式**体现在 ContextBuilder 的构建流程上。系统提示词的构建遵循固定的步骤流程，每一步的具体内容可以变化，但整体结构保持一致。

**责任链模式**体现在消息处理流程上。消息从 Channel 流入，经过 Bus、Agent Loop、Context Builder、Tool Registry 等多个处理环节，每个环节各司其职。

### 7.2 提示词优化建议

基于 nanobot 的架构设计，以下是针对提示词优化的建议：

**引导文件优化**：`bootstrap/` 目录下的文件是定制 Agent 行为的主要途径。建议为不同使用场景准备多套配置，通过文件切换来调整 Agent 行为。

**技能设计原则**：技能应该专注于特定领域，避免过度泛化。每个技能的 SKILL.md 应该包含清晰的示例，帮助 Agent 理解何时以及如何使用该技能。

**记忆管理策略**：定期整理 MEMORY.md 文件，删除过时信息，确保 Agent 能够获取准确的历史信息。同时，利用每日笔记功能记录临时信息。

**系统提示词精简**：避免在 bootstrap 文件中加入过多内容，过长的系统提示词可能导致 LLM 性能下降和成本增加。建议将详细信息放在可按需加载的文件中。

### 7.3 常见问题与解决方案

**问题一：Agent 频繁使用 message 工具**

表现：Agent 在普通对话中频繁调用 message 工具，而不是直接回复。

解决方案：在 IDENTITY.md 或 AGENTS.md 中强化指导，强调"直接回复用于对话，message 工具用于跨渠道通信"。

**问题二：Agent 不记得重要信息**

表现：每次对话 Agent 都像是第一次交流。

解决方案：检查 MEMORY.md 是否正确创建和更新。在核心身份提示词中强调记忆的重要性，并确保 Agent 有权限写入该文件。

**问题三：技能不可用**

表现：Agent 声称无法使用某个技能。

解决方案：检查 skills/ 目录下技能文件是否完整；确认 always.txt 中列出了需要始终加载的技能；检查技能的依赖是否已安装。

## 八、配置示例模板

### 8.1 最小配置

以下是一个最基本的配置示例，适用于仅使用 CLI 对话的场景：

```json
{
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-v1-xxx"
    }
  },
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-5"
    }
  }
}
```

### 8.2 完整配置

以下配置包含所有可用选项，适用于多渠道部署：

```json
{
  "agents": {
    "defaults": {
      "model": "anthropic/claude-opus-4-5",
      "max_iterations": 20
    }
  },
  "providers": {
    "openrouter": {
      "apiKey": "sk-or-v1-xxx"
    },
    "groq": {
      "apiKey": "gsk_xxx"
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "token": "123456:ABC...",
      "allowFrom": ["123456789"]
    },
    "whatsapp": {
      "enabled": false
    },
    "feishu": {
      "enabled": false,
      "appId": "cli_xxx",
      "appSecret": "xxx",
      "allowFrom": []
    }
  },
  "tools": {
    "web": {
      "search": {
        "apiKey": "BSA..."
      }
    },
    "exec": {
      "timeout": 60,
      "restrict_to_workspace": true
    }
  }
}
```

### 8.3 本地模型配置

以下配置用于 vLLM 本地部署场景：

```json
{
  "providers": {
    "vllm": {
      "apiKey": "dummy",
      "apiBase": "http://localhost:8000/v1"
    }
  },
  "agents": {
    "defaults": {
      "model": "meta-llama/Llama-3.1-8B-Instruct"
    }
  }
}
```

## 九、总结

nanobot 的提示词系统采用了**动态构建**与**分层配置**相结合的设计思路，通过核心身份定义、引导文件加载、记忆系统整合和技能摘要生成四个主要环节，构建出完整的系统提示词。这种设计既保证了系统的核心行为一致性，又提供了足够的灵活性以满足不同用户的需求。

与传统的固定提示词模板相比，nanobot 的方案具有以下优势：

1. **可维护性**：配置与代码分离，便于维护和升级。

2. **可扩展性**：通过技能系统和工具系统轻松添加新功能。

3. **可定制性**：用户可以通过编辑配置文件完全自定义 Agent 行为。

4. **模块化**：各组件职责清晰，易于理解和修改。

对于希望深入定制 nanobot 的用户，建议从 `bootstrap/` 目录下的引导文件开始，逐步了解系统的提示词构建机制，并根据实际需求进行针对性调整。
