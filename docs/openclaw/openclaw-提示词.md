# OpenClaw 提示词分析

## 概述

OpenClaw 采用模块化的系统提示词设计，为每个 Agent 运行动态构建定制的系统提示词。这种设计使得 OpenClaw 能够在不同场景下提供一致但又可定制的 AI 助手体验。OpenClaw 不使用 pi-coding-agent 的默认提示词，而是完全自主构建和注入提示词内容。

## 提示词位置

### 1. 核心系统提示词构建

**代码位置**：`src/agents/` 或相关 Agent 模块

OpenClaw 在每次 Agent 运行时会动态组装系统提示词，包含以下固定模块：

```
工具列表（Tooling）
安全准则（Safety）
技能列表（Skills）
自我更新指南（OpenClaw Self-Update）
工作区路径（Workspace）
本地文档（Documentation）
工作区启动文件注入（Workspace Files）
沙箱配置（Sandbox）
当前日期时间（Current Date & Time）
回复标签（Reply Tags）
心跳提示（Heartbeats）
运行时信息（Runtime）
推理配置（Reasoning）
```

### 2. 工作区启动文件（Workspace Bootstrap）

**代码位置**：`~/.openclaw/workspace/` 或自定义工作区目录

OpenClaw 支持在工作区中注入多个 Markdown 文件作为提示词的一部分：

| 文件名 | 用途 |
|-------|------|
| `AGENTS.md` | Agent 配置和角色定义 |
| `SOUL.md` | AI 助手的人格/灵魂定义 |
| `TOOLS.md` | 自定义工具说明 |
| `IDENTITY.md` | 身份标识 |
| `USER.md` | 用户相关信息 |
| `HEARTBEAT.md` | 心跳配置 |
| `BOOTSTRAP.md` | 启动引导（仅新工作区） |

### 3. 技能提示词

**代码位置**：`~/.openclaw/workspace/skills/<skill>/SKILL.md`

技能文件在需要时由 Agent 主动读取，而非预先注入。

### 4. 文档提示词引用

**代码位置**：`docs/` 目录或 npm 包中的文档

系统提示词包含指向本地和在线文档的引用。

## 提示词内容详解

### 1. 核心系统提示词结构

```markdown
# 工具列表（Tooling）
[当前可用工具列表 + 简短描述]
[包含所有注册的工具及其功能说明]

# 安全准则（Safety）
[简短的安全提醒]
[防止权力寻求行为和规避监督的指导原则]

# 技能列表（Skills）
[当有可用技能时注入]
<available_skills>
  <skill>
    <name>技能名称</name>
    <description>简短描述</description>
    <location>文件路径</location>
  </skill>
</available_skills>

# OpenClaw 自我更新
[如何运行 config.apply 和 update.run]
[Agent 如何自我更新配置的指南]

# 工作区（Workspace）
[当前工作目录路径]
[agents.defaults.workspace 配置值]

# 文档（Documentation）
[本地 OpenClaw 文档目录路径]
[在线文档地址：docs.openclaw.ai]
[ClawHub 技能发现地址]
[提示模型优先查阅本地文档]

# 工作区文件注入（Workspace Files）
[以下bootstrap文件已注入]
[Project Context 部分]

# 沙箱配置（Sandbox）
[当启用沙箱时]
[沙箱运行时信息]
[沙箱路径]
[是否可用提升权限执行]

# 当前日期和时间（Current Date & Time）
[用户本地时间]
[时区信息]
[时间格式]

# 回复标签（Reply Tags）
[可选的回复标签语法]
[支持的 provider 的标签格式]

# 心跳（Heartbeats）
[心跳提示]
[确认行为]

# 运行时信息（Runtime）
[主机信息]
[操作系统]
[Node 版本]
[模型名称]
[代码仓库根目录（如果检测到）]
[思考级别（单行）]

# 推理（Reasoning）
[当前可见性级别]
[/reasoning 切换提示]
```

### 2. 提示词模式（Prompt Modes）

OpenClaw 支持三种提示词渲染模式：

**完整模式（full）** - 默认
- 包含上述所有部分
- 用于主要的用户交互会话

**最小模式（minimal）**
- 用于子 Agent（sub-agents）
- 省略部分：**Skills**、**Memory Recall**、**OpenClaw Self-Update**、**Model Aliases**、**User Identity**、**Reply Tags**、**Messaging**、**Silent Replies**、**Heartbeats**
- 保留：工具列表、**Safety**、工作区、沙箱、当前日期时间（如果已知）、运行时信息、注入的上下文
- 子 Agent 上下文标记为 **Subagent Context** 而非 **Group Chat Context**

**无模式（none）**
- 仅返回基础身份行
- 用于极度受限的场景

### 3. 工作区文件注入规则

Bootstrap 文件会被修剪后附加在 **Project Context** 部分下，使模型能够看到身份和配置上下文，而无需显式读取：

```markdown
[Project Context]

## Bootstrap Files

### AGENTS.md
[文件内容或截断标记]

### SOUL.md
[文件内容或截断标记]

### TOOLS.md
[文件内容或截断标记]

[其他 bootstrap 文件...]
```

**截断规则**：
- 大文件会使用标记进行截断
- 单个文件最大字符数由 `agents.defaults.bootstrapMaxChars` 控制（默认：20000 字符）
- 缺失的文件会注入简短的缺失文件标记

**拦截机制**：
- 内部钩子可以通过 `agent:bootstrap` 拦截此步骤
- 可以突变或替换注入的 bootstrap 文件
- 例如：可以将 `SOUL.md` 替换为备用 persona

### 4. 技能提示词格式

当存在符合条件的技能时，OpenClaw 会注入紧凑的可用技能列表：

```xml
<available_skills>
  <skill>
    <name>skill_name</name>
    <description>This skill does XYZ</description>
    <location>/path/to/workspace/skills/skill_name/SKILL.md</location>
  </skill>
  <skill>
    <name>another_skill</name>
    <description>Another skill description</description>
    <location>/path/to/workspace/skills/another_skill/SKILL.md</location>
  </skill>
</available_skills>
```

**技能加载机制**：
- 提示词指导模型使用 `read` 工具加载指定路径的 `SKILL.md`
- 支持三种技能位置：workspace、managed、bundled
- 如果没有符合条件的技能，则省略 Skills 部分
- 这种设计保持基础提示词小巧，同时支持有针对性的技能使用

## 设计模式

### 1. 模块化提示词组装

OpenClaw 采用**模块化组装**的设计模式：

```typescript
// 伪代码示例
function buildSystemPrompt(config: AgentConfig): string {
  const sections = []

  // 1. 工具列表
  sections.push(buildToolingSection())

  // 2. 安全准则
  sections.push(buildSafetySection())

  // 3. 技能（如果可用）
  if (config.hasSkills) {
    sections.push(buildSkillsSection())
  }

  // 4. 文档引用
  sections.push(buildDocumentationSection())

  // 5. 工作区注入
  sections.push(buildWorkspaceInjection())

  // 6. 沙箱配置（如果启用）
  if (config.sandboxEnabled) {
    sections.push(buildSandboxSection())
  }

  // 7. 运行时信息
  sections.push(buildRuntimeSection())

  return sections.join('\n\n')
}
```

**优势**：
- 每个部分独立可测试
- 便于添加新功能
- 支持条件化渲染

### 2. 按需技能加载

技能不预先注入，而是按需加载：

```
基础提示词（包含技能列表引用）
    ↓
模型决定使用某个技能
    ↓
使用 read 工具加载 SKILL.md
    ↓
技能指令被添加到上下文
```

**优势**：
- 保持基础提示词小巧（~20000 字符限制）
- 避免提示词过长导致的性能问题
- 支持动态技能发现

### 3. 提示词缓存优化

时间信息处理采用缓存友好设计：

```markdown
# 当前日期和时间
时区：Asia/Shanghai
[不再包含动态时钟]
```

**优化策略**：
- 仅包含时区（静态信息），不包含动态时钟
- 使用 `session_status` 获取当前时间戳
- 避免因时间变化导致提示词缓存失效

### 4. 截断与标记系统

大文件处理采用智能截断：

```
[文件内容前 N 个字符...]
---TRUNCATED---
[文件路径：/path/to/file.md]
[使用 read 工具读取完整内容]
```

**配置项**：
- `agents.defaults.bootstrapMaxChars`：单文件最大字符数（默认：20000）
- `agents.defaults.userTimezone`：用户时区配置
- `agents.defaults.timeFormat`：时间格式（auto | 12 | 24）

### 5. 钩子系统

通过 `agent:bootstrap` 钩子支持运行时修改：

```typescript
// 内部钩子示例
interface AgentBootstrapHook {
  name: 'agent:bootstrap'
  handler: (context: BootstrapContext) => ModifiedContext
}

// 使用场景
- 替换 SOUL.md 为备用 persona
- 添加动态生成的内容
- 根据用户配置调整注入内容
```

## 变量和模板

### 1. 动态变量

系统提示词中包含多个动态变量：

| 变量 | 来源 | 示例 |
|-----|------|-----|
| `{tool_list}` | 工具注册表 | `bash`, `read`, `write` 等 |
| `{workspace_path}` | 配置 `agents.defaults.workspace` | `~/.openclaw/workspace` |
| `{timezone}` | 系统或配置 | `Asia/Shanghai` |
| `{runtime_info}` | 运行时检测 | `macOS 14.0, Node 22.12.0` |
| `{model_name}` | 模型配置 | `anthropic/claude-opus-4-5` |
| `{skills_list}` | 技能注册表 | 技能 XML 列表 |
| `{docs_path}` | 安装路径 | `./docs` 或 npm 包 |

### 2. 条件注入模板

```markdown
{{#if sandboxEnabled}}
# 沙箱配置
沙箱路径：{sandbox_path}
执行权限：{elevated_exec}
{{/if}}

{{#if hasSkills}}
# 可用技能
{skills_list}
{{/if}}

{{#if timeKnown}}
# 当前日期和时间
时区：{timezone}
{{/if}}
```

### 3. 技能位置模板

```xml
<available_skills>
{{#each skills}}
  <skill>
    <name>{{this.name}}</name>
    <description>{{this.description}}</description>
    <location>{{this.location}}</location>
  </skill>
{{/each}}
</available_skills>
```

### 4. 截断标记模板

```
[文件内容前 {maxChars} 个字符...]
---TRUNCATED---
文件路径：{filePath}
[使用 read 工具读取完整内容]
```

## 优化建议

### 1. 提示词大小优化

**当前实现**：
- 单个 bootstrap 文件最大 20000 字符
- 总提示词大小受模型上下文窗口限制

**优化建议**：
- 实施更激进的截断策略
- 使用摘要而非完整内容注入
- 考虑压缩不常用部分

```typescript
// 示例：智能摘要
function summarizeBootstrapFile(content: string): string {
  // 提取关键配置和人格定义
  const summary = extractKeySections(content)

  // 添加"完整内容请使用 read"提示
  return `${summary}

[此文件已摘要，完整内容请使用 read 工具读取]
`
}
```

### 2. 动态部分分离

**当前问题**：
- 运行时信息可能随每次调用变化
- 导致提示词缓存失效

**建议方案**：
- 将动态部分移至独立区块
- 使用 `session_status` 传递动态信息
- 保持系统提示词核心部分稳定

```typescript
// 建议：分离动态信息
const systemPrompt = buildStablePrompt()  // 缓存这部分
const dynamicInfo = buildDynamicInfo()    // 每次会话更新
```

### 3. 技能加载优化

**当前流程**：
1. 技能列表已注入提示词
2. Agent 决定使用某技能
3. 使用 `read` 加载完整技能文件

**优化方向**：
- 预加载高频使用的技能摘要
- 实施技能缓存机制
- 技能使用统计自动优化

### 4. 多语言支持增强

**当前状态**：
- 系统提示词为英文
- 支持本地化文档

**建议**：
- 支持工作区文件的本地化版本
- 根据用户偏好注入不同语言的人格定义

### 5. 版本化提示词

**建议实施**：
- 提示词模板版本控制
- 迁移脚本支持提示词升级
- 回退机制处理版本不兼容

```yaml
# 提示词版本配置
system_prompt:
  version: "2024.1"
  template: "full"
  fallback_version: "2023.4"
```

### 6. 性能监控集成

**建议添加**：
- 提示词构建时间监控
- Token 使用统计
- 性能回归检测

```typescript
// 建议：性能监控
function buildSystemPrompt(config: Config): PromptMetrics {
  const start = performance.now()
  const prompt = doBuild(config)
  const duration = performance.now() - start

  return {
    prompt,
    metrics: {
      buildTime: duration,
      tokenCount: countTokens(prompt),
      sections: config.sections.length
    }
  }
}
```

### 7. 调试和可观测性

**当前支持**：
- `/context list` 查看上下文
- `/context detail` 查看详细信息

**增强建议**：
- 提示词构建日志
- 各部分贡献度可视化
- Token 预算分配建议

### 8. 安全增强

**当前设计**：
- Safety 部分为建议性质
- 硬执行依赖工具策略、沙箱等

**优化方向**：
- 提示词注入完整性校验
- 敏感信息自动脱敏
- 注入内容的签名验证

## 总结

OpenClaw 的提示词系统是一个精心设计的模块化架构：

**核心优势**：
1. **灵活性**：支持多种提示词模式（full/minimal/none）
2. **可扩展性**：通过钩子系统支持运行时修改
3. **性能优化**：截断机制和按需技能加载
4. **一致性**：动态组装但保持核心结构稳定

**设计亮点**：
- 工作区 bootstrap 文件注入机制
- 条件化渲染模板
- 缓存友好的时间处理
- 完整的生命周期钩子

**改进空间**：
- 动态内容进一步分离
- 多语言本地化支持
- 版本化提示词管理
- 性能监控和可观测性增强

这种设计使得 OpenClaw 能够在保持系统提示词稳定性的同时，提供高度可定制的 AI 助手体验，非常适合需要个性化配置的个人 AI 助手场景。

## 参考资料

- **系统提示词文档**：`docs/concepts/system-prompt.md`
- **上下文管理**：`docs/concepts/context.md`
- **Agent 配置**：`docs/concepts/agent.md`
- **配置参考**：`docs/gateway/configuration.md`
