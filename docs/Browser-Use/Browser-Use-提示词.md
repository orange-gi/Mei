# Browser-Use 提示词分析报告

> **作者**: Matrix Agent
> **项目地址**: https://github.com/browser-use/browser-use
> **分析时间**: 2026-01-24

---

## 一、提示词文件概览

### 1.1 提示词文件列表

Browser-Use 项目为不同的 LLM 模型和使用场景准备了多个提示词模板：

| 文件名 | 用途 | 大小 |
|--------|------|------|
| `system_prompt.md` | 默认通用提示词 | 15,732 bytes |
| `system_prompt_anthropic.md` | Anthropic 专用 (Claude) | 18,952 bytes |
| `system_prompt_anthropic_flash.md` | Anthropic Flash 模式 | 18,652 bytes |
| `system_prompt_anthropic_no_thinking.md` | Anthropic 无思维链 | 17,676 bytes |
| `system_prompt_browser_use.md` | Browser Use 云模型 | 816 bytes |
| `system_prompt_browser_use_flash.md` | Browser Use Flash 模式 | 574 bytes |
| `system_prompt_browser_use_no_thinking.md` | Browser Use 无思维链 | 633 bytes |
| `system_prompt_flash.md` | 通用 Flash 模式 | 1,894 bytes |
| `system_prompt_flash_anthropic.md` | Anthropic Flash 模式 (旧) | 2,001 bytes |
| `system_prompt_no_thinking.md` | 通用无思维链 | 15,632 bytes |

### 1.2 提示词加载逻辑

提示词模板的加载由 `SystemPrompt` 类控制，代码位于 `browser_use/agent/prompts.py`：

```python
def _load_prompt_template(self) -> None:
    """Load the prompt template from the markdown file."""
    # 选择逻辑：
    # 1. Browser-use 模型 (最简化提示词)
    if self.is_browser_use_model:
        if self.flash_mode:
            template_filename = 'system_prompt_browser_use_flash.md'
        elif self.use_thinking:
            template_filename = 'system_prompt_browser_use.md'
        else:
            template_filename = 'system_prompt_browser_use_no_thinking.md'

    # 2. Anthropic 4.5 模型 (优化缓存)
    elif self.is_anthropic_4_5:
        if self.flash_mode:
            template_filename = 'system_prompt_anthropic_flash.md'
        elif self.use_thinking:
            template_filename = 'system_prompt_anthropic.md'
        else:
            template_filename = 'system_prompt_anthropic_no_thinking.md'

    # 3. 通用模型
    elif self.flash_mode and self.is_anthropic:
        template_filename = 'system_prompt_flash_anthropic.md'
    elif self.flash_mode:
        template_filename = 'system_prompt_flash.md'
    elif self.use_thinking:
        template_filename = 'system_prompt.md'
    else:
        template_filename = 'system_prompt_no_thinking.md'
```

---

## 二、核心系统提示词分析

### 2.1 默认提示词结构 (system_prompt.md)

默认提示词采用模块化设计，包含以下核心部分：

```markdown
# 1. 角色定义 <intro>
You are an AI agent designed to operate in an iterative loop to automate browser tasks.

# 2. 语言设置 <language_settings>
- Default working language: **English**
- Always respond in the same language as the user request

# 3. 输入结构 <input>
描述每步输入的组成部分

# 4. 历史记录格式 <agent_history>
历史事件的序列化格式

# 5. 用户请求 <user_request>
核心任务目标

# 6. 浏览器状态 <browser_state>
页面元素描述格式

# 7. 视觉信息 <browser_vision>
截图说明

# 8. 浏览器规则 <browser_rules>
操作约束和最佳实践

# 9. 文件系统 <file_system>
文件操作指南

# 10. 任务完成规则 <task_completion_rules>
done 动作调用条件

# 11. 动作规则 <action_rules>
每步最大动作数

# 12. 效率指南 <efficiency_guidelines>
推荐的动作组合

# 13. 推理规则 <reasoning_rules>
thinking 块的内容要求

# 14. 示例 <examples>
好的输出模式参考

# 15. 输出格式 <output>
JSON 输出模板
```

### 2.2 提示词模板详细分析

#### 2.2.1 角色定义模块

```markdown
<intro>
You excel at following tasks:
1. Navigating complex websites and extracting precise information
2. Automating form submissions and interactive web actions
3. Gathering and saving information
4. Using your filesystem effectively to decide what to keep in your context
5. Operate effectively in an agent loop
6. Efficiently performing diverse web tasks
</intro>
```

**设计模式**：
- ✅ **任务列表式**：明确列出代理的六大核心能力
- ✅ **行为边界**：定义了"能做什么"和"擅长什么"
- ⚠️ **可优化**：可添加"不能做什么"的明确限制

#### 2.2.2 语言设置模块

```markdown
<language_settings>
- Default working language: **English**
- Always respond in the same language as the user request
</language_settings>
```

**设计模式**：
- ✅ **默认 + 动态**：默认英文，根据用户请求切换语言
- ✅ **简单明确**：规则清晰易执行

#### 2.2.3 输入结构模块

```markdown
<input>
At every step, your input will consist of:
1. <agent_history>: A chronological event stream...
2. <agent_state>: Current <user_request>, summary of <file_system>...
3. <browser_state>: Current URL, open tabs...
4. <browser_vision>: Screenshot...
5. <read_state>...
</input>
```

**设计模式**：
- ✅ **结构化输入**：每步输入都有明确的 XML 标签包装
- ✅ **自文档化**：输入格式本身就是自描述的

#### 2.2.4 浏览器规则模块

```markdown
<browser_rules>
- Only interact with elements that have a numeric [index] assigned.
- Only use indexes that are explicitly provided.
- If research is needed, open a **new tab** instead of reusing the current one.
- If the page changes after an input text action, analyse if you need to interact with new elements...
- By default, only elements in the visible viewport are listed.
- If a captcha appears, attempt solving it if possible...
- ...
</browser_rules>
```

**设计模式**：
- ✅ **约束式规则**：通过"不能做什么"来约束行为
- ✅ **场景覆盖**：覆盖了常见异常情况（CAPTCHA、页面变化等）
- ✅ **实用性**：规则都是可操作的具体建议

**关键规则统计**：
- 元素交互规则：3 条
- 页面导航规则：4 条
- 异常处理规则：5 条
- 任务类型规则：2 条

#### 2.2.5 推理规则模块

```markdown
<reasoning_rules>
You must reason explicitly and systematically at every step in your `thinking` block.
Exhibit the following reasoning patterns:
- Reason about <agent_history> to track progress...
- Analyze the most recent "Next Goal" and "Action Result"...
- Explicitly judge success/failure/uncertainty of the last action...
- If todo.md is empty and the task is multi-step, generate a stepwise plan...
- Analyze whether you are stuck, e.g. when you repeat the same actions...
- ...
</reasoning_rules>
```

**设计模式**：
- ✅ **CoT (Chain of Thought)**：强制要求思维链输出
- ✅ **具体模式**：给出了具体的推理模式示例
- ✅ **容错机制**：强调验证动作结果而非假设成功

#### 2.2.6 输出格式模块

```markdown
<output>
You must ALWAYS respond with a valid JSON in this exact format:
{{
  "thinking": "A structured <think>-style reasoning block...",
  "evaluation_previous_goal": "Concise one-sentence analysis...",
  "memory": "1-3 sentences of specific memory...",
  "next_goal": "State the next immediate goal...",
  "action":[{{"navigate": {{ "url": "url_value"}}}}]
}}
Action list should NEVER be empty.
</output>
```

**设计模式**：
- ✅ **严格 JSON 格式**：确保输出可解析
- ✅ **必填字段**：thinking、evaluation_previous_goal、memory、next_goal、action
- ✅ **字数约束**：memory 限制 1-3 句，简洁明确

---

## 三、模型特定优化

### 3.1 Browser Use 云模型提示词

**文件**: `system_prompt_browser_use.md`

这是最简化的提示词模板，仅包含核心结构：

```markdown
You are a browser-use agent operating in thinking mode.
You automate browser tasks by outputting structured JSON actions.

<output>
You must ALWAYS respond with a valid JSON in this exact format:
{{
  "thinking": "A structured reasoning block...",
  "evaluation_previous_goal": "...",
  "memory": "1-3 sentences...",
  "next_goal": "...",
  "action": [{{"action_name": {{...params...}}}}]
}}
Action list should NEVER be empty.
</output>
```

**优化策略**：
- 🔸 **极简主义**：移除了所有冗余的规则说明
- 🔸 **信任模型**：假设云端模型已经过微调，理解任务本质
- 🔸 **减少 token**：显著降低提示词开销

### 3.2 Anthropic 模型优化

**文件**: `system_prompt_anthropic.md`

针对 Anthropic Claude 模型的额外优化：

```markdown
# 新增的浏览器规则
- If you encounter access denied (403), bot detection, or rate limiting,
  do NOT repeatedly retry the same URL. Try alternative approaches...
- Detect and break out of unproductive loops: if you are on the same URL
  for 3+ steps without meaningful progress...

# 新增动作规则
Check the browser state each step to verify your previous action achieved
its goal. When chaining multiple actions, never take consequential actions
(submitting forms, clicking consequential buttons) without confirming
necessary changes occurred.
```

**优化策略**：
- 🔸 **增强异常处理**：Claude 更擅长处理复杂边界情况
- 🔸 **安全约束**：防止 Claude 在不确定的情况下执行危险操作
- 🔸 **循环检测**：明确要求 Claude 识别并跳出无效循环

### 3.3 Flash 模式提示词

**文件**: `system_prompt_flash.md`

为快速、低 token 场景设计的精简版：

```markdown
You are an AI agent designed to operate in an iterative loop to automate browser tasks.
Your ultimate goal is accomplishing the task provided in <user_request>.

# 移除了大量详细规则说明...
```

**优化策略**：
- 🔸 **移除次要规则**：只保留核心约束
- 🔸 **减少示例**：大幅压缩提示词体积
- 🔸 **适合高频调用**：适合需要快速响应的场景

---

## 四、提示词设计模式总结

### 4.1 采用的设计模式

| 模式名称 | 应用场景 | 效果 |
|----------|----------|------|
| **角色定义 (Persona)** | 代理身份设定 | 明确代理定位和行为边界 |
| **结构化输入 (Structured Input)** | XML 标签包装 | 提高信息解析准确性 |
| **约束规则 (Constraints)** | 浏览器操作规则 | 防止错误操作 |
| **思维链 (Chain of Thought)** | reasoning_rules | 提升复杂任务处理能力 |
| **Few-shot 示例** | examples 模块 | 提供输出参考 |
| **严格输出格式** | output 模块 | 确保 JSON 可解析 |
| **动态模板变量** | {max_actions} | 运行时注入配置 |
| **模型特定优化** | 多提示词模板 | 针对不同模型优化 |

### 4.2 变量和模板机制

Browser-Use 使用 `format()` 方法注入动态变量：

```python
# SystemPrompt 类中
prompt = self.prompt_template.format(max_actions=self.max_actions_per_step)
```

**可注入的变量**：
- `{max_actions}`：每步最大动作数（默认 3）
- `{step_number}`：步骤编号（在历史记录中）

**设计优势**：
- ✅ **单一模板**：避免为每个配置创建单独文件
- ✅ **运行时定制**：根据 Agent 配置动态调整
- ✅ **易于维护**：修改规则只需改一处

---

## 五、用户消息提示词

### 5.1 消息构造流程

用户消息由 `AgentMessagePrompt` 类构造，包含多个信息来源：

```python
class AgentMessagePrompt:
    def __init__(
        self,
        browser_state_summary: 'BrowserStateSummary',
        file_system: 'FileSystem',
        agent_history_description: str | None = None,
        read_state_description: str | None = None,
        task: str | None = None,
        screenshots: list[str] | None = None,
        # ... 其他参数
    ):
```

### 5.2 消息结构

```xml
<agent_history>
    # 历史步骤记录
</agent_history>

<agent_state>
    <user_request>
        # 用户任务描述
    </user_request>
    <file_system>
        # 文件系统摘要
    </file_system>
    <todo_contents>
        # Todo 列表内容
    </todo_contents>
    <step_info>
        # 当前步骤信息
    </step_info>
</agent_state>

<browser_state>
    <page_stats>
        # 页面统计信息
    </page_stats>
    <page_info>
        # 页面位置信息
    </page_info>
    <interactive_elements>
        # 可交互元素列表
    </interactive_elements>
</browser_state>

<read_state>
    # extract/read_file 动作的输出
</read_state>
```

### 5.3 页面统计信息提取

```python
def _extract_page_statistics(self) -> dict[str, int]:
    """Extract high-level page statistics from DOM tree for LLM context"""
    stats = {
        'links': 0,
        'iframes': 0,
        'shadow_open': 0,
        'shadow_closed': 0,
        'scroll_containers': 0,
        'images': 0,
        'interactive_elements': 0,
        'total_elements': 0,
    }
```

**设计亮点**：
- ✅ **上下文摘要**：在详细 DOM 之前提供页面概览
- ✅ **异常检测**：识别 SPA 未加载、shadow DOM 等特殊情况
- ✅ **信息压缩**：帮助 LLM 快速理解页面复杂度

---

## 六、其他提示词分析

### 6.1 特殊动作提示词

#### 6.1.1 AI Step 动作提示词

用于 AI 从网页中提取数据的特殊提示词：

```python
def get_ai_step_system_prompt() -> str:
    return """
You are an expert at extracting data from webpages.

<input>
You will be given:
1. A query describing what to extract
2. The markdown of the webpage
3. Optionally, a screenshot
</input>

<instructions>
- Extract information from the webpage that is relevant to the query
- ONLY use the information available in the webpage
- If the information is not available, mention that clearly
</instructions>

<output>
- Present ALL relevant information in a concise way
- Do not use conversational format
</output>
"""
```

**设计模式**：
- 🔸 **专注单一任务**：专门用于数据提取
- 🔸 **明确约束**：强调"只能使用网页上的信息"
- 🔸 **简洁输出**：避免冗长的格式说明

#### 6.1.2 重运行摘要提示词

用于分析任务重运行结果的提示词：

```python
def get_rerun_summary_prompt(original_task: str, total_steps: int,
                            success_count: int, error_count: int) -> str:
    return f'''You are analyzing the completion of a rerun task...
Original task: {original_task}
Execution statistics:
- Total steps: {total_steps}
- Successful steps: {success_count}
- Failed steps: {error_count}

Analyze the screenshot to determine:
1. Whether the task completed successfully
2. What the final state shows
3. Overall completion status (complete/partial/failed)
'''
```

---

## 七、优化建议

### 7.1 现有提示词的优势

1. **模块化设计**：清晰的功能分区，便于维护和扩展
2. **多模型适配**：为不同 LLM 提供定制化提示词
3. **容错机制**：完善的错误处理和恢复指导
4. **示例丰富**：通过示例降低输出格式错误率
5. **渐进式复杂度**：从极简到完整的多级提示词

### 7.2 可改进的方向

#### 7.2.1 提示词压缩

**问题**：默认提示词较大（约 15KB），增加 token 成本

**建议**：
```markdown
# 优化方案：使用引用代替重复
# 当前：每个规则都完整列出
# 建议：建立规则索引，只在需要时展开
```

#### 7.2.2 动态规则选择

**问题**：部分规则可能不适用于所有任务

**建议**：
```python
# 根据任务类型动态加载规则子集
if task_type == "RESEARCH":
    prompt = base_prompt + research_rules
elif task_type == "FORM_FILLING":
    prompt = base_prompt + form_filling_rules
```

#### 7.2.3 多语言增强

**问题**：当前语言设置较为简单

**建议**：
```markdown
<language_settings>
- Default working language: **English**
- Always respond in the same language as the user request
- For technical terms, prefer using English terminology
- For user-facing content, use the user's language
</language_settings>
```

#### 7.2.4 工具使用指南

**问题**：自定义工具使用说明分散

**建议**：
```markdown
<tool_usage>
- You have access to tools defined in <tools_registry>
- Each tool has a description and required parameters
- Prefer native browser actions over custom tools when possible
- Tool results are returned in <read_state>
</tool_usage>
```

### 7.3 具体优化示例

#### 示例 1：减少重复规则

```markdown
# 优化前 (~50 行)
<browser_rules>
- Only interact with elements that have a numeric [index] assigned.
- Only use indexes that are explicitly provided.
- If research is needed, open a **new tab**...
- If the page changes after an input text action...
- By default, only elements in the visible viewport are listed.
- ...
</browser_rules>

# 优化后 (~20 行)
<browser_rules>
Core constraints:
- Use only indexed elements: [N]
- New tabs for research
- Verify state after stateful actions

Extended guidelines (expand if needed):
{{browser_rules_extended}}
</browser_rules>
```

#### 示例 2：任务类型前缀

```markdown
# 动态注入任务类型
<task_type>{task_type}</task_type>

# 不同任务类型的特殊规则
<task_specific_rules>
# FORM_FILLING: Focus on input fields and submission
# RESEARCH: Open new tabs, extract comprehensively
# NAVIGATION: Prioritize links and clear goals
# TESTING: Be thorough, verify each action
</task_specific_rules>
```

---

## 八、总结

### 8.1 Browser-Use 提示词设计亮点

1. **层次化模板系统**：基础模板 + 模型特定优化
2. **强大的 XML 结构**：清晰的输入输出格式定义
3. **完善的错误处理**：覆盖常见异常场景
4. **灵活的变量注入**：运行时定制提示词
5. **丰富的示例**：降低使用门槛

### 8.2 关键设计原则

| 原则 | 实践 |
|------|------|
| **明确性** | 使用 XML 标签明确标记各部分 |
| **可操作性** | 规则都是具体可执行的动作 |
| **容错性** | 强调验证而非假设 |
| **可扩展性** | 通过 Registry 支持自定义工具 |
| **高效性** | Flash 模式降低 token 消耗 |

### 8.3 应用建议

对于希望在项目中集成类似提示词设计的开发者：

1. **从模板开始**：复用 Browser-Use 的模板结构
2. **按需裁剪**：根据实际需求移除不必要的规则
3. **模型适配**：为不同 LLM 创建定制版本
4. **持续优化**：通过实际运行数据改进提示词
5. **版本管理**：跟踪提示词变更对效果的影响

---

## 附录：提示词文件位置

| 文件路径 | 说明 |
|----------|------|
| `browser_use/agent/system_prompts/system_prompt.md` | 默认提示词 |
| `browser_use/agent/system_prompts/system_prompt_anthropic.md` | Anthropic 专用 |
| `browser_use/agent/system_prompts/system_prompt_browser_use.md` | Browser Use 云模型 |
| `browser_use/agent/system_prompts/system_prompt_flash.md` | Flash 模式 |
| `browser_use/agent/system_prompts/system_prompt_no_thinking.md` | 无思维链模式 |

**提示词加载代码**：`browser_use/agent/prompts.py` 中的 `SystemPrompt` 类
