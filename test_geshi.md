# <span style="color:purple;font-weight:bold">Anthropic Skills 项目介绍</span>

## <span style="color:purple;font-weight:bold">一、为什么（Why）</span>

### <span style="color:purple;font-weight:bold">1.1 项目解决的问题</span>

Anthropic Skills 项目旨在解决<span style="color:purple;font-weight:bold">人工智能助手在处理特定领域任务时缺乏专业化能力的问题</span>。传统的通用型 AI 助手虽然具备广泛的知识基础，但在面对需要深度专业技能的复杂任务时，往往表现不佳。技能（Skills）作为一种<span style="color:purple;font-weight:bold">可动态加载的指令集和资源包</span>，能够让 Claude 在执行特定任务时获得针对性的指导和最佳实践，从而<span style="color:purple;font-weight:bold">显著提升任务完成的质量和效率</span>。

在实际应用场景中，用户经常需要 AI 助手完成诸如<span style="color:purple;font-weight:bold">创建专业文档格式（如 Word、PDF、PPT、Excel）、执行复杂的代码测试、遵循特定企业的品牌规范、处理法律或财务文档等专业任务</span>。这些任务通常需要遵循特定的流程、格式要求和工作规范，而通用模型难以准确理解和执行这些复杂指令。通过 Skills 系统，开发者可以将<span style="color:purple;font-weight:bold">领域知识、工作流程和最佳实践封装为可重用的技能模块</span>，使 Claude 能够在需要时加载相应的专业知识。

### <span style="color:purple;font-weight:bold">1.2 项目背景与动机</span>

Anthropic 开发 Skills 系统的核心动机源于对<span style="color:purple;font-weight:bold">AI 助手能力边界的深刻洞察</span>。大型语言模型虽然在生成能力上表现强大，但在需要精确遵循复杂规则、处理结构化文档格式、执行规范化工作流程等场景中，往往需要大量的上下文提示和示例才能达到理想效果。这种方式不仅<span style="color:purple;font-weight:bold">效率低下，而且难以保证输出一致性</span>。

Skills 系统的设计灵感来源于<span style="color:purple;font-weight:bold">软件工程中的模块化思想</span>。正如软件库和框架允许开发者封装和重用代码一样，Skills 允许开发者封装和重用指令、脚本和资源。这种模块化方法带来了多重优势：<span style="color:purple;font-weight:bold">首先，技能可以被独立开发和测试，确保质量；其次，技能可以在不同场景中复用，提高开发效率；最后，用户可以根据自己的需求选择性地加载所需技能，实现个性化的 AI 助手</span>。

### <span style="color:purple;font-weight:bold">1.3 现有方案的不足</span>

在 Skills 系统出现之前，用户和开发者主要依赖以下几种方式来增强 AI 的专业能力，但这些方案都存在明显缺陷：

**提示词工程（Prompt Engineering）**：用户通过精心设计系统提示词来引导 AI 行为，但这种方式存在诸多问题。<span style="color:purple;font-weight:bold">提示词通常需要包含大量上下文信息，导致上下文窗口被大量占用；提示词的维护和更新困难，当最佳实践发生变化时，需要手动修改所有使用场景的提示词；此外，不同任务需要不同的提示词，难以有效组织和管理</span>。

**微调（Fine-tuning）**：通过训练数据调整模型行为是另一种常见方法，但微调存在显著局限性。<span style="color:purple;font-weight:bold">微调过程需要大量计算资源和专业技能，不适合普通用户；微调后的模型难以快速适应新需求，每次更新都需要重新训练；更重要的是，微调可能导致模型在其他方面的能力下降，出现灾难性遗忘现象</span>。

**外部工具集成**：让 AI 调用外部工具和 API 是扩展能力的有效方式，但这种方式需要用户自行构建和维护工具链，<span style="color:purple;font-weight:bold">学习成本高且容易出错</span>。Skills 系统通过标准化工具描述和工作流程，<span style="color:purple;font-weight:bold">降低了工具集成的门槛，同时保持了灵活性和可扩展性</span>。

### <span style="color:purple;font-weight:bold">1.4 目标用户群体</span>

Skills 系统的设计服务于多个用户群体，每个群体都能从中获得独特价值：

**企业用户**可以利用 Skills 封装<span style="color:purple;font-weight:bold">公司的品牌规范、文档模板、审批流程和工作标准</span>，确保 AI 生成的文档和决策符合企业要求。例如，市场部门可以创建一个「品牌指南技能」，确保所有营销材料在字体、颜色、语气等方面保持一致。

**开发者群体**可以通过 Skills 为特定软件工具和平台提供集成能力。像<span style="color:purple;font-weight:bold">Notion 这样的第三方平台已经开发了专门的 Skills</span>，帮助 Claude 更好地与 Notion 生态系统交互。这种合作伙伴模式使得 Skills 能够持续扩展。

**普通用户**可以创建个人化的 Skills 来自动化日常任务，如邮件撰写、日程安排、数据整理等。<span style="color:purple;font-weight:bold">Skills 的低门槛设计使得非技术用户也能轻松创建和使用自定义技能</span>。

**AI 研究人员和爱好者**可以通过研究现有 Skills 的设计和实现，学习最佳的 AI 提示工程实践，理解如何构建高质量的指令集和工作流程。

---

## <span style="color:purple;font-weight:bold">二、是什么（What）</span>

### <span style="color:purple;font-weight:bold">2.1 项目概述</span>

Anthropic Skills 是一个<span style="color:purple;font-weight:bold">开源的技能仓库系统</span>，为 Claude AI 助手提供可动态加载的专业能力模块。该项目包含三个核心组成部分：<span style="color:purple;font-weight:bold">示例技能集（skills 目录）、技能规范（spec 目录）和技能模板（template 目录）</span>。每个技能都是一个自包含的文件夹，包含一个 <span style="color:purple;font-weight:bold">SKILL.md 文件作为核心指令文档</span>，以及必要的脚本和资源文件。

这个项目的规模相当可观，<span style="color:purple;font-weight:bold">仓库已获得超过 52,300 颗星标和 5,100 次分叉</span>，充分证明了其在 AI 社区中的影响力和实用价值。项目采用多语言开发，其中 Python 占 83.9%、JavaScript 占 9.4%、HTML 占 4.3%、Shell 占 2.4%，反映了 Skills 系统在文档处理和工具集成方面的技术特点。

### <span style="color:purple;font-weight:bold">2.2 核心功能特性</span>

Skills 系统提供了一套完整的专业任务执行框架，其核心功能可以从以下几个方面理解：

**动态指令加载**：当用户激活某个技能时，<span style="color:purple;font-weight:bold">Claude 会自动加载该技能包含的所有指令、示例和指南</span>。这些指令成为 AI 行为的一部分，指导其如何处理特定类型的任务。加载过程对用户透明，无需手动复制粘贴指令内容。

**结构化技能定义**：每个技能遵循统一的目录结构，核心是 SKILL.md 文件。该文件采用<span style="color:purple;font-weight:bold">YAML 前置元数据（frontmatter）定义技能名称和描述</span>，随后是详细的使用说明、示例和最佳实践指南。这种结构化设计确保了技能的可读性和可维护性。

**多平台支持**：Skills 可以在 Claude 的多个产品中使用，包括<span style="color:purple;font-weight:bold">Claude Code（命令行工具）、Claude.ai（网页应用）和 Claude API（编程接口）</span>。这种跨平台一致性使得用户可以在不同环境中使用相同的技能。

**技能组合能力**：用户可以同时安装多个技能，<span style="color:purple;font-weight:bold">Claude 会根据任务需求智能选择合适的技能组合</span>。这使得复杂的跨领域任务可以通过组合现有技能来解决，而无需从头开发。

### <span style="color:purple;font-weight:bold">2.3 主要技能类型</span>

Skills 仓库中包含了多种类型的示例技能，展示了系统的广泛应用潜力：

**创意与设计技能**：涵盖艺术创作、音乐生成、平面设计等领域。这些技能包含了<span style="color:purple;font-weight:bold">创意指导、美学原则、设计最佳实践</span>等内容，帮助 Claude 生成符合专业标准的创意内容。

**开发与技术技能**：包括 Web 应用测试、MCP 服务器生成、代码审查等面向开发者的技能。这些技能封装了<span style="color:purple;font-weight:bold">软件开发的最佳实践和质量标准</span>，确保 AI 生成的代码和测试符合专业要求。

**企业与通信技能**：涵盖品牌管理、沟通写作、商业文档等专业领域。这些技能帮助 AI 生成符合企业规范的专业文档，<span style="color:purple;font-weight:bold">确保品牌一致性和沟通质量</span>。

**文档处理技能**：包括 DOCX、PDF、PPTX、XLSX 等专业文档格式的创建和编辑技能。这些技能基于 Claude 的文档功能开发，提供了<span style="color:purple;font-weight:bold">完整的文档处理能力，是项目中最复杂和最实用的技能之一</span>。

### <span style="color:purple;font-weight:bold">2.4 技术架构概述</span>

Skills 系统采用分层架构设计，底层是标准化的技能规格（spec），中间层是具体的技能实现，上层是与不同 Claude 产品的集成接口。

**技能规格层**定义了技能的基本结构要求，包括<span style="color:purple;font-weight:bold">SKILL.md 文件的格式规范、元数据字段定义、指令组织方式</span>等。规格文档确保了所有技能的一致性和互操作性。

**技能实现层**包含具体的技能文件夹，每个技能包含核心指令文件、辅助脚本和资源文件。<span style="color:purple;font-weight:bold">技能之间可以相互引用和组合，形成技能网络</span>。

**集成接口层**处理技能与不同 Claude 产品的交互，包括<span style="color:purple;font-weight:bold">技能注册、安装、加载和激活</span>等流程。这一层屏蔽了底层差异，为用户提供统一的技能使用体验。

### <span style="color:purple;font-weight:bold">2.5 与竞品的差异化</span>

与市场上其他 AI 增强方案相比，Skills 系统具有显著差异化特点：

**官方支持与质量保证**：作为 Anthropic 官方推出的项目，<span style="color:purple;font-weight:bold">Skills 得到了持续的支持和更新</span>。官方提供的技能经过严格测试，确保在生产环境中的可靠性。社区贡献的技能也有明确的贡献指南和质量标准。

**标准化设计**：Skills 采用统一的规格标准，<span style="color:purple;font-weight:bold">所有技能遵循相同的结构规范</span>。这种标准化不仅便于开发和维护，也使得技能的分享和复用更加便捷。Agent Skills 规格已被独立发布在 agentskills.io 网站上。

**开源与商业结合**：项目采用了混合许可策略，<span style="color:purple;font-weight:bold">大部分技能采用 Apache 2.0 开源许可，允许自由使用和修改；而核心的文档处理技能采用源码可用（source-available）许可，平衡了开放性和商业利益</span>。

**生态系统整合**：Skills 与 Notion 等第三方平台的深度整合展示了<span style="color:purple;font-weight:bold">生态系统的扩展能力</span>。合作伙伴可以开发专门的 Skills 来增强 Claude 与其产品的交互，这种模式为 Skills 生态的持续扩展提供了基础。

---

## <span style="color:purple;font-weight:bold">三、如何做（How）</span>

### <span style="color:purple;font-weight:bold">3.1 快速开始指南</span>

#### <span style="color:purple;font-weight:bold">3.1.1 在 Claude Code 中使用 Skills</span>

Claude Code 用户可以通过以下步骤快速使用 Skills：

首先，将 Skills 仓库添加到插件市场。打开 Claude Code 并执行以下命令：

```bash
/plugin marketplace add anthropics/skills
```

添加成功后，可以浏览和安装可用的技能集。执行「浏览和安装插件」命令，然后选择「anthropic-agent-skills」插件。接下来，在「document-skills」和「example-skills」之间选择适合的技能集，点击「立即安装」完成安装过程。

如果已知具体的技能集名称，也可以直接安装：

```bash
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

安装完成后，<span style="color:purple;font-weight:bold">只需在对话中提及相应技能即可使用</span>。例如，安装了 document-skills 后，可以直接说：「使用 PDF 技能从 path/to/some-file.pdf 中提取表单字段」。

#### <span style="color:purple;font-weight:bold">3.1.2 在 Claude.ai 中使用 Skills</span>

Claude.ai 的付费用户可以直接使用仓库中的示例技能，无需额外安装。如果需要使用自定义技能，可以按照官方文档中的说明上传技能文件。具体步骤可在 Using skills in Claude 官方文档中查阅。

#### <span style="color:purple;font-weight:bold">3.1.3 通过 API 使用 Skills</span>

开发者可以通过 Claude API 使用预置技能或上传自定义技能。<span style="color:purple;font-weight:bold">API 提供了完整的技能管理接口，包括技能上传、激活和执行等功能</span>。详细的 API 使用方法可在 Skills API Quickstart 官方文档中获取。

### <span style="color:purple;font-weight:bold">3.2 核心使用方式</span>

#### <span style="color:purple;font-weight:bold">3.2.1 选择合适的技能</span>

使用技能的第一步是<span style="color:purple;font-weight:bold">识别任务需求并选择合适的技能</span>。Skills 仓库中的每个技能都有明确的使用场景描述，用户可以根据描述判断是否需要该技能。对于复杂任务，可能需要组合使用多个技能。

在选择技能时，应考虑以下因素：<span style="color:purple;font-weight:bold">任务类型（文档创建、数据分析、代码生成等）、输出要求（格式规范、质量标准）、专业领域（法律、医学、金融等）</span>。选择正确的技能是确保任务成功的关键前提。

#### <span style="color:purple;font-weight:bold">3.2.2 激活和使用技能</span>

技能的激活通常是隐式的——当用户表达的任务需求与技能描述匹配时，Claude 会自动加载相关技能。对于明确知道需要使用哪个技能的场景，<span style="color:purple;font-weight:bold">用户也可以直接提及技能名称来激活它</span>。

技能激活后，<span style="color:purple;font-weight:bold">Claude 会遵循技能中定义的所有指令、示例和指南来执行任务</span>。这意味着用户无需在每次对话中重复相同的提示词，技能会自动提供必要的上下文和指导。

#### <span style="color:purple;font-weight:bold">3.2.3 组合使用多个技能</span>

对于复杂的跨领域任务，<span style="color:purple;font-weight:bold">可以组合使用多个技能</span>。Claude 会智能地整合来自不同技能的指令，在保持各自专业性的同时实现协同工作。例如，同时使用「品牌指南」技能和「文档编辑」技能，可以生成既符合品牌规范又格式正确的专业文档。

### <span style="color:purple;font-weight:bold">3.3 关键配置说明</span>

#### <span style="color:purple;font-weight:bold">3.3.1 技能文件结构</span>

创建自定义技能需要遵循标准化的文件结构：

```
my-skill/
├── SKILL.md          # 核心技能文件（必需）
├── scripts/          # 辅助脚本目录（可选）
│   ├── script1.py
│   └── script2.sh
└── resources/        # 资源文件目录（可选）
    ├── template.md
    └── data.json
```

<span style="color:purple;font-weight:bold">SKILL.md 文件是技能的必备核心文件</span>，必须包含 YAML 前置元数据块和详细的指令内容。

#### <span style="color:purple;font-weight:bold">3.3.2 SKILL.md 文件格式</span>

SKILL.md 文件采用标准化格式，顶部是 YAML 前置元数据，随后是 Markdown 格式的指令内容：

```yaml
---
name: my-skill-name
description: 清晰描述技能功能和适用场景
---
# 我的技能名称

[在此添加 Claude 执行此技能时将遵循的指令]

## 示例
- 使用示例 1
- 使用示例 2

## 指南
- 指南 1
- 指南 2
```

<span style="color:purple;font-weight:bold">前置元数据中的 name 字段必须是技能的唯一标识符</span>，使用小写字母和连字符；description 字段应完整描述技能的功能和适用场景，以便 Claude 正确匹配技能与任务需求。

### <span style="color:purple;font-weight:bold">3.4 最佳实践建议</span>

#### <span style="color:purple;font-weight:bold">3.4.1 技能设计原则</span>

设计高质量技能应遵循以下原则：

**单一职责原则**：<span style="color:purple;font-weight:bold">每个技能应专注于一个明确的功能领域</span>。避免创建功能过于宽泛的「万能技能」，而应将复杂功能分解为多个专注的小技能，通过组合使用来实现复杂需求。

**清晰的使用边界**：<span style="color:purple;font-weight:bold">在技能描述中明确说明技能的适用场景和不适用场景</span>。这有助于 Claude 在任务匹配时做出正确决策，避免技能误用导致的错误输出。

**结构化指令组织**：<span style="color:purple;font-weight:bold">将技能指令组织为逻辑清晰的章节，如概述、工作流程、示例、指南等</span>。良好的结构不仅便于开发者维护，也帮助 Claude 更准确地理解和执行指令。

#### <span style="color:purple;font-weight:bold">3.4.2 指令编写技巧</span>

技能指令的质量直接决定了 AI 执行任务的效果。以下是编写有效指令的技巧：

**具体而非抽象**：<span style="color:purple;font-weight:bold">避免使用模糊的描述，尽量提供具体的规则和示例</span>。例如，与其说「使用专业语气」，不如说「使用正式称呼如『您』而非『你』，避免口语化表达如『挺好的』」。

**明确步骤顺序**：对于需要多步骤完成的任务，<span style="color:purple;font-weight:bold">明确列出执行步骤的顺序</span>。Claude 会按照指令中给出的顺序执行任务，清晰的步骤定义可以减少错误和遗漏。

**包含边界条件**：<span style="color:purple;font-weight:bold">说明在什么情况下应该或不应该使用特定方法，以及遇到异常情况时的处理策略</span>。这有助于 AI 在面对意外输入时做出合理决策。

#### <span style="color:purple;font-weight:bold">3.4.3 测试与验证</span>

创建技能后应进行充分测试：

**功能测试**：<span style="color:purple;font-weight:bold">验证技能在各种正常和异常输入下都能正确工作</span>。测试用例应覆盖典型的使用场景和边缘情况。

**一致性测试**：<span style="color:purple;font-weight:bold">确保相同输入在多次执行时产生一致的结果</span>。AI 的随机性可能导致输出差异，技能指令应足够具体以最小化这种差异。

**用户体验测试**：<span style="color:purple;font-weight:bold">让真实用户试用技能，收集反馈以改进指令的清晰度和完整性</span>。用户的实际使用往往能发现开发者未曾考虑的细节问题。

---

## <span style="color:purple;font-weight:bold">四、参考资源</span>

### <span style="color:purple;font-weight:bold">4.1 官方文档链接</span>

- **什么是技能？**：https://support.claude.com/en/articles/12512176-what-are-skills
- **在 Claude 中使用技能**：https://support.claude.com/en/articles/12512180-using-skills-in-claude
- **创建自定义技能**：https://support.claude.com/en/articles/12512198-creating-custom-skills
- **Agent Skills 规格**：https://agentskills.io/
- **技能 API 快速入门**：https://docs.claude.com/en/api/skills-guide#creating-a-skill

### <span style="color:purple;font-weight:bold">4.2 相关技术文章</span>

- **使用 Agent Skills 为代理配备现实世界能力**：https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
- **Claude 文档功能介绍**：https://www.anthropic.com/news/create-files

### <span style="color:purple;font-weight:bold">4.3 合作伙伴资源</span>

- **Notion 技能**：https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0
