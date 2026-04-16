# Claude Code Game Studios 提示词分析

本文档深入分析Claude Code Game Studios项目中使用的各类提示词（Prompts），包括代理定义、技能配置、钩子脚本等。

---

## 1. 提示词位置总览

| 类型 | 位置 | 数量 | 用途 |
|------|------|------|------|
| **代理定义** | `.claude/agents/*.md` | 49 | 定义专业代理的角色、职责和行为规范 |
| **技能定义** | `.claude/skills/*/skill.md` | 72 | 斜杠命令的具体实现和工作流 |
| **主配置** | `CLAUDE.md` | 1 | 项目整体配置和协作协议 |
| **规则文件** | `.claude/rules/*.md` | 11 | 路径作用域的编码标准 |
| **文档模板** | `.claude/docs/templates/*` | 39 | 各种文档的标准模板 |
| **钩子脚本** | `.claude/hooks/*.sh` | 12 | 自动化验证脚本 |

---

## 2. 代理提示词分析

### 2.1 代理定义结构

每个代理文件（如`.claude/agents/creative-director.md`）遵循标准结构：

```yaml
---
name: creative-director
description: 创意总监代理描述
model: opus  # 模型选择
tools: [Read, Write, Edit, Glob, Grep, Bash]  # 可用工具
---

# 创意总监角色定义

[详细的角色说明、职责、协作协议等]
```

### 2.2 创意总监提示词示例

以下是`creative-director.md`的提示词结构分析：

```markdown
# Creative Director Agent

## 角色定义
你是一位经验丰富的游戏创意总监，拥有10年以上的AAA游戏和独立游戏开发经验。

## 核心职责
1. **守护游戏愿景** - 确保所有决策与游戏核心pillars保持一致
2. **设计决策把关** - 审查所有游戏设计文档（GDD）
3. **冲突解决** - 解决跨部门的设计冲突
4. **质量门控** - 执行设计评审gate

## 专业知识
- MDA框架（力学、动力学、美学）
- 自我决定理论（自主性、能力、关联性）
- 心流设计原则
- Bartle玩家类型分析
- 游戏平衡理论

## 协作协议
**严格遵循用户驱动协作**：
1. 提问 → 2. 呈现选项 → 3. 用户决策 → 4. 草案 → 5. 批准 → 6. 写入

## 决策边界
- ✅ 可以：建议设计方向、提出权衡方案、审查设计文档
- ❌ 不可：直接修改代码、跳过用户批准、单方面做跨域决策
```

### 2.3 代理提示词设计模式

#### 模式1：角色+职责+边界

```markdown
# [Agent Name]

## 角色
[一句话描述角色定位]

## 职责
1. [主要职责]
2. [次要职责]
3. [质量把关职责]

## 专业知识
- [专业领域1]
- [专业领域2]

## 协作协议
[遵循的标准流程]

## 决策边界
- ✅ 可以：[允许的行为]
- ❌ 不可：[禁止的行为]
```

#### 模式2：分层架构提示

```markdown
## Tier 1 - 总监（战略层）
[使用Opus模型，进行重大决策]

## Tier 2 - 部门主管（战术层）
[使用Sonnet模型，进行领域内决策]

## Tier 3 - 专家专员（执行层）
[使用Sonnet/Haiku模型，执行具体任务]
```

#### 模式3：引擎专用提示

```markdown
# Godot Specialist

## 引擎信息
- **引擎**: Godot 4.x
- **主要语言**: GDScript / C#
- **版本要求**: 4.6+

## 专业知识
- Godot 4 新特性（分离3D、物理改进等）
- GDScript最佳实践
- GDExtension开发
- Godot着色器系统

## 项目集成
- 遵循项目的naming conventions
- 使用项目的performance budgets
- 验证knowledge gaps（版本检查）
```

---

## 3. 技能提示词分析

### 3.1 技能定义结构

每个技能目录包含：
- `skill.md` - 主要提示词文件
- `description.md` - 技能描述
- 辅助模板文件

### 3.2 核心技能提示词示例

#### `/brainstorm` 技能

```markdown
# Brainstorm Skill

## 目的
协作式游戏构思，使用MDA框架生成和评估游戏概念。

## 前置条件
- 项目处于概念阶段（阶段1）
- 无需已有游戏概念（可从零开始）

## 工作流程

### 阶段1：探索
1. 询问用户：
   - 游戏类型偏好
   - 主题/氛围偏好
   - 平台目标
   - 时间/团队规模约束

### 阶段2：概念生成
生成10个游戏概念，每个包含：
- **Elevator Pitch**: 一句话描述
- **Core Fantasy**: 核心幻想
- **MDA Breakdown**:
  - Mechanics（力学）
  - Dynamics（动力学）
  - Aesthetics（美学）
- **Target Audience**: Bartle类型

### 阶段3：深度分析
用户选择2-3个概念进行深度分析：
- 核心循环图
- 独特卖点（USP）
- 可比较游戏及差异化
- 游戏Pillars（3-5个非协商设计价值）
- Anti-Pillars（刻意避免的元素）

### 阶段4：选择与文档化
用户选择最终概念 → 创建design/gdd/game-concept.md

## 输出
- design/gdd/game-concept.md
```

#### `/design-system` 技能

```markdown
# Design System Skill

## 目的
引导式游戏设计文档（GDD）创作，分节逐步完成。

## GDD必需8节

| # | 章节 | 内容要求 |
|---|------|---------|
| 1 | Overview | 一段式系统概述 |
| 2 | Player Fantasy | 玩家使用系统时的想象/感受 |
| 3 | Detailed Rules | 明确的机械规则 |
| 4 | Formulas | 所有计算，包含变量定义和范围 |
| 5 | Edge Cases | 异常情况处理 |
| 6 | Dependencies | 双向依赖关系 |
| 7 | Tuning Knobs | 可安全调整的值及范围 |
| 8 | Acceptance Criteria | 可测试的具体标准 |

## 附加章节：Game Feel
- Feel参考
- 输入响应（ms/帧）
- 动画目标
- 冲击时刻
- 重量配置

## 工作流程

### 每节遵循：Context → Questions → Options → Decision → Draft → Approval → Write

1. **Context**: 介绍章节背景和重要性
2. **Questions**: 提出澄清问题
3. **Options**: 展示2-4个设计方案
4. **Decision**: 用户选择
5. **Draft**: 基于选择起草内容
6. **Approval**: 用户批准
7. **Write**: 立即写入文件（增量保存）

### 路由到专家
- 数值/公式 → systems-designer
- 经济系统 → economy-designer
- 叙事系统 → narrative-director
- UI需求 → ux-designer

## 冲突检测
检查与现有已批准GDD的一致性：
- 规则冲突
- 依赖矛盾
- 命名不一致
```

#### `/dev-story` 技能

```markdown
# Dev Story Skill

## 目的
将故事文件实现为代码，自动路由到正确的程序员代理。

## 输入
production/epics/[epic]/story-[slug].md

## 工作流程

### 1. 读取故事文件
解析故事中嵌入的：
- GDD引用（TR-ID）
- ADR引用
- Control manifest版本
- 验收标准

### 2. 验证就绪性
检查：
- [x] 设计完整性
- [x] 架构覆盖
- [x] ADR状态（Proposed → Blocked）
- [x] Control manifest新鲜度
- [x] 范围清晰度

### 3. 路由到代理
根据故事类型自动选择：

| 故事类型 | 代理 | 路径 |
|---------|------|------|
| 游戏机制 | gameplay-programmer | src/gameplay/ |
| 引擎系统 | engine-programmer | src/core/ |
| AI行为 | ai-programmer | src/ai/ |
| 多人功能 | network-programmer | src/networking/ |
| UI组件 | ui-programmer | src/ui/ |
| 工具 | tools-programmer | src/tools/ |

### 4. 协作实施
遵循标准协议：
1. 代理读取设计文档
2. 提出澄清问题
3. 呈现架构选项
4. 用户决策
5. 实施代码
6. 编写测试
7. 请求用户批准写入

### 5. 遵循规则
- 路径作用域规则自动应用
- 使用项目的naming conventions
- 遵守performance budgets
```

---

## 4. 协作协议提示词

### 4.1 标准协作协议

```markdown
## 协作协议（所有代理必须遵守）

### 核心原则：用户驱动，非自主执行

所有任务遵循以下流程：

1. **提问（Ask）**
   - 在提出解决方案前先问问题
   - 澄清需求、约束、偏好
   - 示例："这个功能的目标用户是谁？"

2. **呈现选项（Present Options）**
   - 提供2-4个方案选项
   - 每个选项包含：优点、缺点、风险
   - 示例："我们可以A、B、C三种方式实现"

3. **用户决策（You Decide）**
   - 用户做出最终决定
   - 不主动选择，除非用户授权

4. **草案（Draft）**
   - 基于用户决策起草内容
   - 显示工作但不提交

5. **批准（Approve）**
   - 用户审查和修改
   - 用户明确批准后执行

6. **写入（Write）**
   - 获得批准后写入文件
   - 写入前询问："我可以写入[filepath]吗？"
```

### 4.2 多代理协调协议

```markdown
## 多代理协调协议

### 垂直委托
- 总监 → 部门主管 → 专家专员
- 从不跳过层级进行复杂决策

### 水平咨询
- 同级代理可以相互咨询
- 不能做跨域绑定决策

### 冲突解决
- 设计冲突 → creative-director
- 技术冲突 → technical-director
- 范围冲突 → producer

### 跨域变更
- 不能单方面修改其他域的文件
- 需要相关域主管批准
```

---

## 5. 路径作用域规则提示词

### 5.1 规则结构

```markdown
# gameplay-code.md

## 范围
src/gameplay/**

## 强制标准

### 数据驱动
- ❌ 禁止：硬编码值
- ✅ 必须：使用配置数据
- 示例：
  ```gdscript
  # 错误
  var damage = 50

  # 正确
  var damage = config.get("combat.damage.base")
  ```

### Delta Time
- ❌ 禁止：不使用delta_time的动画/移动
- ✅ 必须：所有时间相关计算使用delta_time
- 示例：
  ```gdscript
  # 错误
  position += Vector2.RIGHT * speed

  # 正确
  position += Vector2.RIGHT * speed * delta
  ```

### 无状态UI引用
- ❌ 禁止：从gameplay代码直接引用UI节点
- ✅ 必须：通过信号/接口通信
```

### 5.2 规则设计模式

```markdown
## [路径] 编码标准

### 规则1：[规则名称]
- **要求**：[正面要求]
- **禁止**：[禁止行为]
- **示例**：
  ```代码
  # 正确示例
  [正确代码]

  # 错误示例
  [错误代码]
  ```

### 规则2：[规则名称]
...
```

---

## 6. 钩子脚本提示词

### 6.1 validate-commit.sh 分析

```bash
#!/bin/bash
# validate-commit.sh - 提交验证钩子

# 检查是否为git commit命令
if [[ "$1" != "commit" ]]; then
    exit 0  # 非commit命令，直接通过
fi

# 检查：TODO/FIXME格式
echo "检查TODO/FIXME格式..."
git diff --cached --name-only | xargs grep -E "TODO|FIXME" && {
    echo "警告：发现TODO/FIXME，请使用标准格式"
    echo "格式：TODO(author): description"
}

# 检查：硬编码值
echo "检查硬编码值..."
git diff --cached --name-only | xargs grep -E "[0-9]{3,}" | grep -vE "version|port|speed" && {
    echo "警告：发现可能的硬编码值"
}

# 检查：JSON有效性
echo "检查JSON文件..."
git diff --cached --name-only | grep "\.json$" | while read file; do
    python -m json.tool "$file" > /dev/null || {
        echo "错误：$file 不是有效的JSON"
        exit 1
    }
done

echo "提交验证通过"
exit 0
```

### 6.2 detect-gaps.sh 分析

```bash
#!/bin/bash
# detect-gaps.sh - 项目缺口检测

# 检查：新项目检测
if [[ ! -f ".claude/docs/technical-preferences.md" ]]; then
    echo "📌 检测到新项目"
    echo "建议运行 /start 进行引导式入职"
fi

# 检查：设计文档缺失
if [[ -d "src/" ]] && [[ ! -d "design/gdd/" ]]; then
    echo "📌 检测到代码存在但无设计文档"
    echo "建议：/design-system 或 /reverse-document"
fi

# 检查：原型缺少README
if [[ -d "prototypes/" ]]; then
    find prototypes/ -name "README.md" -empty | while read empty_readme; do
        echo "📌 原型缺少README: $empty_readme"
    done
fi
```

---

## 7. 主配置提示词 (CLAUDE.md)

```markdown
# Claude Code Game Studios -- Game Studio Agent Architecture

## 技术栈
- **Engine**: [CHOOSE: Godot 4 / Unity / Unreal Engine 5]
- **Language**: [CHOOSE: GDScript / C# / C++ / Blueprint]
- **Version Control**: Git with trunk-based development
- **Build System**: [SPECIFY after choosing engine]
- **Asset Pipeline**: [SPECIFY after choosing engine]

## 协作协议
**用户驱动协作，非自主执行**
每个任务遵循：Question → Options → Decision → Draft → Approval

- 代理在写入前必须询问"可以写入[filepath]吗？"
- 多文件变更需要用户明确批准
- 禁止未经用户指令提交

## 编码标准
@.claude/docs/coding-standards.md

## 协调规则
@.claude/docs/coordination-rules.md

## 上下文管理
@.claude/docs/context-management.md

## 首次会话
如果项目未配置引擎且无游戏概念，运行 /start
```

---

## 8. 提示词优化建议

### 8.1 代理提示词优化

| 当前问题 | 优化建议 |
|---------|---------|
| 部分代理描述过长 | 拆分为：summary + detailed两部分 |
| 模型选择固定 | 考虑任务复杂度动态选择 |
| 缺少错误处理 | 添加"如果X则Y"的错误处理模式 |

### 8.2 技能提示词优化

| 当前问题 | 优化建议 |
|---------|---------|
| 技能间有重复工作流 | 提取公共工作流为可复用模块 |
| 缺少条件分支 | 添加"如果用户已有X，则跳过Y"的逻辑 |
| 输出格式不够结构化 | 使用JSON Schema定义输出格式 |

### 8.3 规则提示词优化

| 当前问题 | 优化建议 |
|---------|---------|
| 示例代码有限 | 为每种支持的语言添加示例 |
| 规则覆盖不完整 | 添加更多边界情况示例 |
| 缺少自动修复建议 | 添加"如果违反规则，如何修复"的指引 |

---

## 9. 提示词设计最佳实践

### 实践1：清晰的层级结构

```markdown
# 代理名称

## 一句话描述（<20词）
[核心定位]

## 职责（3-5项）
1. [主要]
2. [次要]
3. [质量把关]

## 专业知识（5-8项）
- [领域1]
- [领域2]

## 工作方式
[协作模式]

## 边界
- ✅ 可以
- ❌ 不可
```

### 实践2：示例驱动的规则

```markdown
## 规则：数据驱动值

### 要求
所有游戏数值必须来自配置，不能硬编码

### 正确示例
```gdscript
var damage = config.get("combat.player.attack_damage")
```

### 错误示例
```gdscript
var damage = 50
```

### 验证方法
运行 /validate-data-driven
```

### 实践3：条件化的工作流

```markdown
## 工作流：根据项目状态分支

### 如果：项目无GDD
1. 运行 /design-system
2. 创建design/gdd/[system].md
3. 路由到 /design-review

### 如果：项目有GDD但需更新
1. 运行 /design-system retrofit [path]
2. 检测缺失章节
3. 填充缺失内容
4. 不覆盖现有内容

### 如果：快速变更
1. 运行 /quick-design "[变更描述]"
2. 创建design/quick-specs/[slug].md
3. 路由到相关代理
```

---

## 10. 提示词索引

### 代理索引

| 代理名称 | 文件 | 层级 | 模型 |
|---------|------|------|------|
| creative-director | creative-director.md | 1 | opus |
| technical-director | technical-director.md | 1 | opus |
| producer | producer.md | 1 | opus |
| game-designer | game-designer.md | 2 | sonnet |
| lead-programmer | lead-programmer.md | 2 | sonnet |
| art-director | art-director.md | 2 | sonnet |
| gameplay-programmer | gameplay-programmer.md | 3 | sonnet |
| godot-specialist | godot-specialist.md | 3 | sonnet |
| ... | ... | ... | ... |

### 核心技能索引

| 技能 | 路径 | 阶段 | 用途 |
|------|------|------|------|
| /start | skills/start/ | 入门 | 引导式入职 |
| /brainstorm | skills/brainstorm/ | 1 | 协作构思 |
| /design-system | skills/design-system/ | 2 | GDD创作 |
| /dev-story | skills/dev-story/ | 5 | 故事实施 |
| /gate-check | skills/gate-check/ | 所有 | 阶段门控 |

### 规则索引

| 规则 | 路径 | 强制内容 |
|------|------|---------|
| gameplay-code.md | src/gameplay/** | 数据驱动、delta time |
| engine-code.md | src/core/** | 零分配、线程安全 |
| ai-code.md | src/ai/** | 性能预算、可调试性 |
| network-code.md | src/networking/** | 服务器权威、安全 |
| ui-code.md | src/ui/** | 无状态、无障碍 |

---

## 附录：关键提示词文件位置

```
.claude/
├── agents/
│   ├── creative-director.md      # 创意总监
│   ├── technical-director.md     # 技术总监
│   ├── producer.md               # 制作人
│   ├── game-designer.md          # 游戏设计师
│   ├── lead-programmer.md        # 首席程序员
│   ├── gameplay-programmer.md   # 游戏程序员
│   ├── godot-specialist.md       # Godot专家
│   └── ...                       # 其他47个代理
├── skills/
│   ├── start/
│   │   └── skill.md              # /start 技能
│   ├── brainstorm/
│   │   └── skill.md              # /brainstorm 技能
│   ├── design-system/
│   │   └── skill.md              # /design-system 技能
│   ├── dev-story/
│   │   └── skill.md              # /dev-story 技能
│   └── ...                       # 其他68个技能
├── rules/
│   ├── gameplay-code.md         # 游戏代码规则
│   ├── engine-code.md           # 引擎代码规则
│   └── ...                       # 其他9条规则
├── hooks/
│   ├── validate-commit.sh        # 提交验证
│   ├── validate-push.sh          # 推送验证
│   └── ...                       # 其他10个钩子
└── docs/
    └── templates/                # 39个文档模板
```
