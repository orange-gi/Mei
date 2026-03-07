# Qwen-Agent 提示词设计分析

## 一、提示词系统概述

Qwen-Agent的提示词设计是整个框架的核心组成部分，它决定了Agent如何理解任务、调用工具、生成响应。项目中的提示词主要分布在以下几个模块：函数调用提示词（Function Calling）、ReAct推理提示词、知识库提示词和多智能体协作提示词。这些提示词经过精心设计，能够有效引导大语言模型完成各种复杂任务。

## 二、函数调用提示词设计

### 2.1 提示词位置与结构

函数调用提示词主要位于`qwen_agent/llm/fncall_prompts/`目录下，包含三个主要的提示词实现文件：

- `qwen_fncall_prompt.py`：针对Qwen模型的提示词模板
- `nous_fncall_prompt.py`：针对Nous模型的提示词模板
- `base_fncall_prompt.py`：基础提示词处理逻辑

### 2.2 Qwen函数调用提示词完整内容

以下是Qwen模型使用的函数调用提示词模板，包含了中文和英文两个版本：

```python
# 中文提示词
FN_CALL_TEMPLATE_INFO_ZH = """# 工具

## 你拥有如下工具：

{tool_descs}"""

FN_CALL_TEMPLATE_FMT_ZH = """## 你可以在回复中插入零次、一次或多次以下命令以调用工具：

✿FUNCTION✿: 工具名称，必须是[{tool_names}]之一。
✿ARGS✿: 工具输入
✿RESULT✿: 工具结果
✿RETURN✿: 根据工具结果进行回复，需将图片用![](url)渲染出来"""

# 英文提示词
FN_CALL_TEMPLATE_INFO_EN = """# Tools

## You have access to the following tools:

{tool_descs}"""

FN_CALL_TEMPLATE_FMT_EN = """## When you need to call a tool, please insert the following command in your reply, which can be called zero or multiple times according to your needs:

✿FUNCTION✿: The tool to use, should be one of [{tool_names}]
✿ARGS✿: The input of the tool
✿RESULT✿: Tool results
✿RETURN✿: Reply based on tool results. Images need to be rendered as ![](url)"""
```

### 2.3 并行函数调用提示词

对于支持并行函数调用的场景，系统还提供了专门的并行调用提示词模板：

```python
# 中文并行调用模板
FN_CALL_TEMPLATE_FMT_PARA_ZH = """## 你可以在回复中插入以下命令以并行调用N个工具：

✿FUNCTION✿: 工具1的名称，必须是[{tool_names}]之一
✿ARGS✿: 工具1的输入
✿FUNCTION✿: 工具2的名称
✿ARGS✿: 工具2的输入
...
✿FUNCTION✿: 工具N的名称
✿ARGS✿: 工具N的输入
✿RESULT✿: 工具1的结果
✿RESULT✿: 工具2的结果
...
✿RESULT✿: 工具N的结果
✿RETURN✿: 根据工具结果进行回复，需将图片用![](url)渲染出来"""

# 英文并行调用模板
FN_CALL_TEMPLATE_FMT_PARA_EN = """## Insert the following command in your reply when you need to call N tools in parallel:

✿FUNCTION✿: The name of tool 1, should be one of [{tool_names}]
✿ARGS✿: The input of tool 1
✿FUNCTION✿: The name of tool 2
✿ARGS✿: The input of tool 2
...
✿FUNCTION✿: The name of tool N
✿ARGS✿: The input of tool N
✿RESULT✿: The result of tool 1
✿RESULT✿: The result of tool 2
...
✿RESULT✿: The result of tool N
✿RETURN✿: Reply based on tool results. Images need to be rendered as ![](url)"""
```

### 2.4 特殊Token设计

Qwen-Agent函数调用系统使用了一组特殊的Token来标记函数调用的不同阶段：

| Token | 用途 | 说明 |
|-------|------|------|
| `✿FUNCTION✿` | 函数名称 | 标记要调用的工具名称 |
| `✿ARGS✿` | 函数参数 | 标记工具的输入参数 |
| `✿RESULT✿` | 执行结果 | 标记工具返回的结果 |
| `✿RETURN✿` | 最终响应 | 标记基于结果的后续响应 |

这些特殊Token的设计使得模型能够清晰地识别和生成结构化的函数调用请求，同时保持了提示词的可读性和可理解性。

## 三、ReAct推理提示词设计

### 3.1 提示词位置

ReAct（Reasoning + Acting）模式的提示词位于`qwen_agent/agents/react_chat.py`文件中。

### 3.2 完整提示词内容

```python
PROMPT_REACT = """Answer the following questions as best you can. You have access to the following tools:

{tool_descs}

Use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of [{tool_names}]
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Action Input/Observation can be repeated zero or more times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question

Begin!

Question: {query}
Thought: """
```

### 3.3 设计模式分析

ReAct提示词采用了few-shot的学习方式，通过示例明确展示了思考-行动-观察的循环过程：

**思考（Thought）**：引导模型分析当前情况，思考下一步应该做什么。

**行动（Action）**：明确列出可用工具，让模型选择合适的工具执行。

**行动输入（Action Input）**：说明工具需要什么参数。

**观察（Observation）**：展示工具执行后的结果，供模型参考进行下一步推理。

这种设计模式的优势在于：它将复杂的推理过程分解为清晰的步骤，使模型能够进行多步推理和迭代执行，有效提升了模型处理复杂问题的能力。

## 四、知识库提示词设计

### 4.1 提示词位置

知识库相关的提示词位于`qwen_agent/agents/assistant.py`文件中。

### 4.2 知识模板设计

```python
# 中文知识模板
KNOWLEDGE_TEMPLATE_ZH = """# 知识库

{knowledge}"""

# 英文知识模板
KNOWLEDGE_TEMPLATE_EN = """# Knowledge Base

{knowledge}"""

# 中文知识片段模板
KNOWLEDGE_SNIPPET_ZH = """## 来自 {source} 的内容：

```
{content}
```"""

# 英文知识片段模板
KNOWLEDGE_SNIPPET_EN = """## The content from {source}:

```
{content}
```"""
```

### 4.3 设计特点

知识库提示词的设计具有以下特点：

**来源标注**：每个知识片段都标注了来源（如文件名或URL），便于用户验证信息的可信度。

**格式统一**：使用Markdown代码块格式展示原始内容，保持格式一致性。

**双语支持**：同时提供中英文版本，支持国际化应用。

**动态填充**：通过占位符`{knowledge}`、`{source}`、`{content}`实现动态内容填充。

## 五、多智能体协作提示词

### 5.1 提示词位置

多智能体相关提示词主要位于`qwen_agent/agents/group_chat.py`文件中。

### 5.2 Agent角色构建提示词

```python
def _build_system_from_role_config(config: Dict):
    role_chat_prompt = """你是{name}。{description}\n\n{instructions}"""

    name = config.get('name', '').strip()
    description = config.get('description', '').lstrip('\n').rstrip()
    instructions = config.get('instructions', '').lstrip('\n').rstrip()

    # 如果instructions比description更详细，则忽略description
    if len(instructions) >= len(description):
        description = ''
    else:
        description = f'你的简介是：{description}'

    prompt = role_chat_prompt.format(
        name=name,
        description=description,
        instructions=instructions
    )

    return prompt, knowledge_files, selected_tools
```

### 5.2 群聊系统提示词

```python
groupchat_background = '你在一个群聊中，'
if cfgs.get('background', ''):
    groupchat_background += f'群聊背景为：{cfgs["background"]}'

# 构建每个Agent的系统提示
system_message = groupchat_background + system + \
    f'\n\n群里其他成员包括：{", ".join(other_agents)}，如果你想和别人对话，可以@成员名字。' + \
    '\n\n讲话时请直接输出内容，不要输出你的名字。\n\n其他群友的发言历史以如下格式展示：\n角色名: 说话内容'
```

### 5.3 设计模式分析

**角色定义**：通过name、description、instructions三个维度定义Agent角色，确保角色清晰明确。

**上下文感知**：每个Agent都能感知群聊背景和其他成员信息，便于进行有针对性的协作。

**交互规范**：明确说明发言格式（直接输出内容，不输出名字）、艾特成员的方式、历史消息的展示格式。

## 六、工具描述提示词

### 6.1 函数描述模板

每个工具在调用前都需要生成描述信息，提示词模板如下：

```python
# 中文工具描述模板
tool_desc_template_zh = '### {name_for_human}\n\n{name_for_model}: {description_for_model} 输入参数：{parameters} {args_format}'

# 英文工具描述模板
tool_desc_template_en = '### {name_for_human}\n\n{name_for_model}: {description_for_model} Parameters: {parameters} {args_format}'
```

### 6.2 参数格式说明

根据工具类型不同，参数格式说明也有所区别：

```python
# 代码解释器参数格式
args_format_code_interpreter = {
    'zh': '此工具的输入应为Markdown代码块。',
    'en': 'Enclose the code within triple backticks (`) at the beginning and end of the code.'
}

# 其他工具参数格式
args_format_other = {
    'zh': '此工具的输入应为JSON对象。',
    'en': 'Format the arguments as a JSON object.'
}
```

## 七、提示词优化建议

### 7.1 当前设计的优点

**结构化程度高**：使用特殊Token和结构化格式，使模型能够准确理解任务要求。

**双语支持完善**：所有提示词都提供了中英文版本，便于国际化部署。

**Few-shot示例丰富**：ReAct等提示词包含了完整的示例，降低模型理解难度。

**可扩展性强**：模板化设计使得添加新的工具或Agent类型非常方便。

### 7.2 潜在改进方向

**动态提示词优化**：可以考虑根据任务复杂度动态调整提示词内容，避免过度提示或提示不足。

**上下文压缩**：对于长对话场景，可以研究更高效的历史消息压缩技术，减少上下文长度。

**提示词版本管理**：可以引入提示词版本控制机制，便于追踪和回溯不同版本的提示词效果。

**自动化优化**：可以研究基于反馈的提示词自动优化方法，根据执行结果自动调整提示词内容。

**多模态扩展**：随着多模态能力的增强，可以设计更丰富的多模态提示词模板。

## 八、总结

Qwen-Agent的提示词设计体现了以下核心原则：

1. **明确的任务分解**：通过结构化的格式将复杂任务分解为清晰的步骤。

2. **充分的示例引导**：使用Few-shot技术帮助模型理解期望的行为模式。

3. **灵活的模板系统**：通过参数化的模板实现动态内容生成和扩展。

4. **多语言支持**：全面的中英文双语支持，满足国际化需求。

这些设计使得Qwen-Agent能够有效地引导大语言模型完成函数调用、多步推理，知识检索、多智能体协作等各种复杂任务，是构建高质量LLM应用的重要基础。
