# GitNexus 提示词分析

## 一、提示词系统概述

GitNexus 项目中的提示词系统是多层次的，旨在为 AI 智能体提供结构化的代码分析指导。整个提示词系统可以分为三个主要类别：MCP 工具定义中的描述性提示词、智能体技能文件中的工作流提示词，以及 MCP 服务器中的动态提示模板。这些提示词共同构成了 GitNexus 与外部 AI 编辑器（如 Claude Code、Cursor、Windsurf）交互的核心接口。

MCP 工具定义中的提示词主要出现在工具的描述字段（description）中，这些描述不仅说明了工具的功能，还包含了使用场景指导（WHEN TO USE）和后续步骤建议（AFTER THIS），形成了一种自引导的工作流机制。智能体技能文件则是独立的 Markdown 文档，为特定任务场景（如代码探索、影响分析、调试、重构）提供了标准化的操作流程和检查清单。MCP 服务器中的提示模板则用于生成复杂的引导式工作流，帮助 AI 智能体完成多步骤的代码分析任务。

## 二、MCP 工具定义提示词

### 2.1 工具描述结构分析

GitNexus 在 `gitnexus/src/mcp/tools.ts` 文件中定义了七个核心 MCP 工具。每个工具的描述字段都遵循统一的结构化设计模式，这种设计极大地提升了 AI 智能体使用工具的效率和准确性。

工具描述的标准结构包含三个核心部分：首先是功能说明（What It Does），简要描述工具的主要功能；其次是使用场景（WHEN TO USE），明确说明在什么情况下应该使用这个工具；第三是后续步骤（AFTER THIS），指导 AI 智能体在调用该工具后应该采取什么行动。这种「功能—场景—行动」的三段式结构，使每个工具都成为了一个完整的工作流节点，AI 智能体可以根据提示自动构建下一步行动计划。

以 `query` 工具为例，其描述内容如下：

```
Query the code knowledge graph for execution flows related to a concept.
Returns processes (call chains) ranked by relevance, each with its symbols and file locations.

WHEN TO USE: Understanding how code works together. Use this when you need execution flows and relationships, not just file matches. Complements grep/IDE search.
AFTER THIS: Use context() on a specific symbol for 360-degree view (callers, callees, categorized refs).

Returns results grouped by process (execution flow):
- processes: ranked execution flows with relevance priority
- process_symbols: all symbols in those flows with file locations
- definitions: standalone types/interfaces not in any process

Hybrid ranking: BM25 keyword + semantic vector search, ranked by Reciprocal Rank Fusion.
```

这种描述设计体现了几个关键的设计原则。首先是任务针对性，工具被设计来解决特定类型的问题，而不是提供泛化的功能；其次是上下文关联性，每个工具都知道自己在整个工作流中的位置，并指向下一个逻辑步骤；第三是结果可操作性，工具返回的结果格式是预定义的，AI 智能体知道如何解析和使用这些结果。

### 2.2 工具输入模式分析

每个 MCP 工具都定义了严格的输入模式（inputSchema），这些模式不仅指定了参数类型，还包含了详细的描述信息和默认值。输入模式的设计体现了以下特点：

**参数分组与层次化**：工具参数被组织成逻辑组，例如 `query` 工具的参数可以分为搜索参数（query、task_context、goal）、限制参数（limit、max_symbols）、输出参数（include_content）和上下文参数（repo）。这种分组方式帮助 AI 智能体理解每个参数的用途。

**智能默认值**：大多数可选参数都设置了合理的默认值，如 `limit` 默认为 5、`max_symbols` 默认为 10、`include_content` 默认为 false。这意味着 AI 智能体可以在只提供必需参数的情况下获得可用的结果，同时也保留了自定义的空间。

**条件参数**：某些参数只在特定条件下需要，如 `rename` 工具的 `symbol_uid` 参数用于零歧义查找，而 `file_path` 参数用于消歧 Common Name。这种设计支持了渐进式的复杂度提升，简单的查询可以用最少的参数完成，复杂的场景则可以提供更多上下文。

### 2.3 工具描述模板模式

通过对七个 MCP 工具描述的分析，可以总结出以下模板模式：

```typescript
// 工具描述模板
{
  name: "工具名称",
  description: `一句话功能描述。
详细的功能说明，包含返回值结构和使用细节。

WHEN TO USE: 使用场景说明，解释什么情况下应该调用此工具。
AFTER THIS: 后续步骤指导，说明调用后应该采取什么行动。

返回结果格式说明...`,
  inputSchema: {
    type: 'object',
    properties: {
      // 参数定义...
    },
    required: ['必需参数'],
  },
}
```

这种模板模式的优势在于它的可扩展性和一致性。开发者可以轻松添加新工具，同时保持与现有工具相同的信息结构和用户体验。AI 智能体也可以轻松理解任何工具的用途和使用方式，因为它们遵循相同的模式。

## 三、下一步提示系统

### 3.1 提示生成机制

GitNexus MCP 服务器的一个独特设计是「下一步提示」（Next-step Hints）系统。在 `gitnexus/src/mcp/server.ts` 文件中，`getNextStepHint` 函数根据刚执行的工具名称和参数，生成针对性的后续行动建议。这一设计解决了 AI 智能体常见的问题：执行完一个工具后不知道下一步该做什么。

```typescript
function getNextStepHint(toolName: string, args: Record<string, any> | undefined): string {
  const repo = args?.repo;
  const repoParam = repo ? `, repo: "${repo}"` : '';
  const repoPath = repo || '{name}';

  switch (toolName) {
    case 'list_repos':
      return `\n\n---\n**Next:** READ gitnexus://repo/{name}/context for any repo above to get its overview and check staleness.`;

    case 'query':
      return `\n\n---\n**Next:** To understand a specific symbol in depth, use context({name: "<symbol_name>"${repoParam}}) to see categorized refs and process participation.`;

    case 'context':
      return `\n\n---\n**Next:** If planning changes, use impact({target: "${args?.name || '<name>'}", direction: "upstream"${repoParam}}) to check blast radius. To see execution flows, READ gitnexus://repo/${repoPath}/processes.`;

    // ... 其他工具的提示
  }
}
```

### 3.2 提示设计原则

下一步提示的设计遵循了几个核心原则。首先是行动导向性，每个提示都包含具体的行动建议，而不是模糊的建议。提示中直接包含了要调用的工具名称和参数格式，降低了 AI 智能体执行行动的门槛。

其次是上下文感知，提示会根据已执行的工具和传入的参数动态生成。例如，如果用户指定了特定的仓库（repo 参数），提示中会包含该仓库名称，使后续步骤更加精准。

第三是资源导航结合，提示不仅指向其他工具，还经常指向知识图谱资源（如 `gitnexus://repo/{name}/processes`）。这种设计鼓励 AI 智能体利用知识图谱的轻量级资源进行导航，而不是总是使用重量级的工具调用。

### 3.3 提示覆盖范围

当前的提示系统覆盖了所有七个 MCP 工具，以及一些遗留工具名称。以下是完整的提示映射表：

| 工具名称 | 下一步提示指向 |
|----------|----------------|
| list_repos | 读取仓库上下文资源 |
| query | 调用 context 工具深入符号 |
| context | 调用 impact 工具或读取进程资源 |
| impact | 读取受影响的进程列表 |
| detect_changes | 审查变更和受影响进程 |
| rename | 运行 detect_changes 验证 |
| cypher | 使用 context 工具探索结果 |
| search（遗留） | 调用 context 工具 |
| explore（遗留） | 调用 impact 工具 |
| overview（遗留） | 读取集群或进程资源 |

## 四、智能体技能提示词

### 4.1 技能文件结构

GitNexus 在 `gitnexus/skills/` 目录下提供了四个智能体技能文件，这些文件是完整的 Markdown 文档，为特定任务场景提供了标准化的指导。每个技能文件都包含以下标准部分：

- **元数据头部**：包含技能名称和描述
- **使用场景**：说明何时应该使用该技能
- **工作流程**：推荐的操作步骤序列
- **检查清单**：可勾选的任务列表
- **资源参考**：该技能可能用到的知识图谱资源
- **工具参考**：该技能依赖的 MCP 工具及其使用示例

这种结构化设计使技能文件既可以被 AI 智能体解析和执行，也可以作为人类开发者的参考文档。

### 4.2 探索技能提示词分析

`exploring.md` 技能文件用于指导 AI 智能体探索不熟悉的代码库。其核心工作流程设计如下：

```
1. READ gitnexus://repos                          → 发现已索引的仓库
2. READ gitnexus://repo/{name}/context             → 代码库概览，检查新鲜度
3. gitnexus_query({query: "<what you want to understand>"})  → 找到相关执行流程
4. gitnexus_context({name: "<symbol>"})            → 深入了解特定符号
5. READ gitnexus://repo/{name}/process/{name}      → 追踪完整执行流程
```

这个工作流的设计体现了「先广后深」的原则：首先通过资源读取获取整体概览，然后通过查询找到相关执行流程，再通过上下文工具深入特定符号，最后通过进程资源追踪完整流程。每一步都为下一步提供必要的上下文。

### 4.3 影响分析技能提示词分析

`impact-analysis.md` 技能文件用于指导 AI 智能体在修改代码前分析潜在影响。其核心工作流程设计如下：

```
1. gitnexus_impact({target: "X", direction: "upstream"})  → 找出依赖项
2. READ gitnexus://repo/{name}/processes                   → 检查受影响的执行流程
3. gitnexus_detect_changes()                               → 映射当前 Git 变更到受影响流程
4. 评估风险并向用户报告
```

这个工作流的设计重点是风险评估和变更验证。第一步找出所有依赖项，第二步理解这些依赖项参与的执行流程，第三步将分析扩展到当前待提交的变更，最后综合评估风险等级。

技能文件还提供了详细的风险评估标准，包括按受影响符号数量和进程数量划分风险等级（LOW/MEDIUM/HIGH/CRITICAL），以及按依赖深度划分的影响程度（d=1 为必然破坏，d=2 为可能受影响，d=3 为可能需要测试）。

### 4.4 调试和重构技能

`debugging.md` 和 `refactoring.md` 技能文件提供了针对特定任务场景的指导。调试技能侧重于通过调用链追踪问题根源，强调使用 `context` 工具追踪函数的调用者和被调用者，以及使用 `process` 资源理解执行流程。重构技能则侧重于使用 `impact` 工具评估修改范围，使用 `rename` 工具执行安全的多文件重命名，以及使用 `detect_changes` 工具验证重构结果。

## 五、MCP 提示模板

### 5.1 detect_impact 提示模板

在 `gitnexus/src/mcp/server.ts` 中，GitNexus 定义了两个 MCP 提示模板。第一个是 `detect_impact`，用于在提交前分析当前代码变更的影响。该提示包含了一个完整的多步骤工作流：

```typescript
{
  name: 'detect_impact',
  description: 'Analyze the impact of your current changes before committing. Guides through scope selection, change detection, process analysis, and risk assessment.',
  arguments: [
    { name: 'scope', description: 'What to analyze: unstaged, staged, all, or compare', required: false },
    { name: 'base_ref', description: 'Branch/commit for compare scope', required: false },
  ],
}
```

当 AI 智能体调用此提示时，服务器会生成以下消息：

```
Analyze the impact of my current code changes before committing.

Follow these steps:
1. Run `detect_changes({scope: "all", base_ref: ""})` to find what changed and affected processes
2. For each changed symbol in critical processes, run `context({name: "<symbol>"})` to see its full reference graph
3. For any high-risk items (many callers or cross-process), run `impact({target: "<symbol>", direction: "upstream"})` for blast radius
4. Summarize: changes, affected processes, risk level, and recommended actions

Present the analysis as a clear risk report.
```

### 5.2 generate_map 提示模板

第二个提示模板 `generate_map` 用于从知识图谱生成架构文档。该提示同样包含了一个系统化的工作流：

```typescript
{
  name: 'generate_map',
  description: 'Generate architecture documentation from the knowledge graph. Creates a codebase overview with execution flows and mermaid diagrams.',
  arguments: [
    { name: 'repo', description: 'Repository name (omit if only one indexed)', required: false },
  ],
}
```

生成的提示消息指导 AI 智能体：

```
Generate architecture documentation for this codebase using the knowledge graph.

Follow these steps:
1. READ `gitnexus://repo/{name}/context` for codebase stats
2. READ `gitnexus://repo/{name}/clusters` to see all functional areas
3. READ `gitnexus://repo/{name}/processes` to see all execution flows
4. For the top 5 most important processes, READ `gitnexus://repo/{name}/process/{name}` for step-by-step traces
5. Generate a mermaid architecture diagram showing the major areas and their connections
6. Write an ARCHITECTURE.md file with: overview, functional areas, key execution flows, and the mermaid diagram
```

### 5.3 提示模板设计模式

这两个提示模板的设计体现了几个共同特点。首先是步骤明确性，每个提示都包含编号的步骤序列，AI 智能体可以按顺序执行。其次是工具组合，步骤中混合使用了工具调用（`detect_changes`、`context`、`impact`）和资源读取（`gitnexus://repo/...`），展示了两者的协同使用方式。第三是产出导向，每个提示都指定了期望的产出格式（风险报告、架构文档），使 AI 智能体有明确的交付目标。

## 六、CLAUDE.md 上下文文件

### 6.1 文件用途与结构

`CLAUDE.md` 是 GitNexus 项目为自己准备的 Claude Code 上下文文件，它使用了 GitNexus 的特殊注释语法来标注索引信息。该文件的主要功能是告诉 Claude Code 如何使用 GitNexus 工具来分析这个项目本身。

文件结构包含以下几个关键部分：首先是用 `<!-- gitnexus:start -->` 和 `<!-- gitnexus:end -->` 标记的代码块，描述了项目的索引统计信息（1309 个符号、3350 个关系、101 个执行流程）；其次是起始指导，强调任何代码理解任务都必须从读取上下文资源开始；第三是技能参考表，列出了不同任务对应的技能文件；第四是工具参考，列出了所有可用的 MCP 工具及其用途；最后是资源参考，列出了可用的知识图谱资源。

### 6.2 强制工作流设计

CLAUDE.md 文件中的一个重要设计是强制工作流要求：「For any task involving code understanding, debugging, impact analysis, or refactoring, you must:」这意味着当 Claude Code 处理涉及代码理解的任务时，它被强制要求首先读取项目的 GitNexus 上下文资源，然后根据任务类型选择相应的技能文件。

这种强制设计解决了 AI 智能体的「冷启动」问题：AI 智能体往往不知道或不习惯使用特定项目的工具，通过在上下文文件中强制要求初始步骤，确保工具总是被正确使用。

## 七、变量与模板处理

### 7.1 动态参数替换

GitNexus 的提示词系统大量使用动态参数替换来实现上下文感知。以下是几种常见的模式：

**仓库参数替换**：

```typescript
const repo = args?.repo;
const repoParam = repo ? `, repo: "${repo}"` : '';
const repoPath = repo || '{name}';
```

这种模式允许提示根据用户是否指定了特定仓库而动态调整。如果用户指定了仓库，提示会包含具体的仓库名称；如果没有指定，提示会使用占位符 `{name}`。

**工具参数回显**：

```typescript
case 'context':
  return `\n\n---\n**Next:** If planning changes, use impact({target: "${args?.name || '<name>'}", direction: "upstream"${repoParam}})...`;
```

这种模式将用户传入的参数回显在提示中，使后续步骤可以直接使用这些参数，减少了参数传递的麻烦。

### 7.2 资源 URI 模板

GitNexus 使用 URI 模板来表示知识图谱资源，这些模板支持动态参数：

| 资源类型 | URI 模板 | 用途 |
|----------|----------|------|
| 仓库列表 | `gitnexus://repos` | 发现已索引仓库 |
| 仓库上下文 | `gitnexus://repo/{name}/context` | 代码库概览和统计 |
| 集群列表 | `gitnexus://repo/{name}/clusters` | 所有功能区域 |
| 特定集群 | `gitnexus://repo/{name}/cluster/{clusterName}` | 集群成员详情 |
| 进程列表 | `gitnexus://repo/{name}/processes` | 所有执行流程 |
| 特定进程 | `gitnexus://repo/{name}/process/{processName}` | 执行流程详情 |
| 图模式 | `gitnexus://repo/{name}/schema` | Cypher 查询模式 |

URI 模板的设计允许 AI 智能体动态构造资源访问路径，只需替换大括号中的参数即可。

## 八、提示词优化建议

### 8.1 当前设计优势

GitNexus 的提示词系统在以下几个方面的设计值得肯定。首先是自引导工作流设计，下一步提示机制有效解决了 AI 智能体「下一步做什么」的问题，使工具使用形成了自然的流程。其次是结构化信息组织，工具描述、技能文件、提示模板都遵循统一的结构，降低了理解和使用的门槛。第三是渐进式复杂度支持，通过智能默认值和可选参数设计，简单的任务可以用最少的输入完成，复杂的场景可以提供更多上下文。

### 8.2 潜在改进方向

基于对现有实现的分析，可以考虑以下几个优化方向：

**提示词国际化支持**：当前所有提示词都是英文的，对于非英语用户可能存在理解障碍。可以考虑添加多语言支持或至少提供英文术语的解释。

**更丰富的示例库**：虽然工具描述中包含了一些示例，但可以扩展为更完整的示例库，展示各种典型使用场景。

**条件分支提示**：当前的提示主要是线性的，可以考虑添加条件分支逻辑，根据上一步的结果动态调整下一步的建议。

**反馈循环机制**：可以添加让 AI 智能体报告工具使用效果的机制，用于持续优化提示词设计。

## 九、总结

GitNexus 的提示词系统是一个精心设计的多层次系统，涵盖了从单个工具描述到完整工作流指导的各个层面。通过 MCP 工具定义中的描述性提示词、智能体技能文件中的工作流指导、下一步提示系统的动态建议，以及提示模板中的多步骤流程，GitNexus 为 AI 智能体提供了全面的代码分析指导。

这种设计体现了「预计算关系智能」的理念，不仅在知识图谱数据的结构化中得以体现，也在提示词系统的设计中得到了贯彻。AI 智能体不需要自行探索如何使用工具，而是可以跟随结构化的提示完成复杂的代码分析任务。这大大降低了 AI 智能体有效使用 GitNexus 的门槛，使项目能够更好地服务于各种规模的代码库分析和智能体辅助开发场景。
