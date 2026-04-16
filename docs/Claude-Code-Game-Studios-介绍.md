# Claude Code Game Studios 项目介绍

## Why（为什么）—— 项目背景与动机

### 解决什么问题？

单独使用AI进行游戏开发虽然强大，但传统单会话模式存在严重缺陷：
- **缺乏结构化流程**——没有设计审查、代码质量把关、团队协作机制
- **容易硬编码魔数**——随意使用magic numbers，缺乏数据驱动设计
- **设计文档缺失**——跳过GDD直接写代码，技术债累积
- **质量保证缺失**——没有QA环节，没有设计评审

### 现有方案为何不够好？

传统的Claude Code使用方式是一个通用的AI助手，没有任何项目特定的结构和约束：
- 任何文件都可以被随意修改
- 没有设计师问"这是否符合游戏愿景"
- 没有代码审查把关架构决策
- 没有任何质量门控机制

### Claude Code Game Studios的诞生

该项目将Claude Code从单一助手转变为完整的**游戏开发工作室架构**：
- 不是让AI替代开发者，而是为AI提供**专业团队的框架和流程**
- 用户仍然是最终决策者，AI提供结构和专业知识的辅助
- 通过**协作协议**确保用户始终掌控方向

---

## What（是什么）—— 核心功能与特性

### 项目概述

Claude Code Game Studios是一个基于Claude Code的**游戏开发工作室工作流系统**，通过49个专业代理、72个斜杠命令、12个自动化钩子和11条路径作用域规则，为独立开发者提供专业游戏工作室的开发流程。

### 核心组件

| 类别 | 数量 | 描述 |
|------|------|------|
| **代理 (Agents)** | 49 | 涵盖设计、编程、美术、音频、叙事、QA和制作的专业子代理 |
| **技能 (Skills)** | 72 | 覆盖所有工作流阶段的斜杠命令（`/start`、`/design-system`、`/dev-story`等） |
| **钩子 (Hooks)** | 12 | 自动化验证：提交验证、推送验证、资源变更、会话生命周期等 |
| **规则 (Rules)** | 11 | 路径作用域的编码标准，编辑代码时自动应用 |
| **模板 (Templates)** | 39 | 文档模板：GDD、UX规范、ADR、冲刺计划等 |

### 代理层级架构

```
第一层 — 总监（使用Opus模型）
├── creative-director（创意总监）
├── technical-director（技术总监）
└── producer（制作人）

第二层 — 部门主管（使用Sonnet模型）
├── game-designer（游戏设计师）
├── lead-programmer（首席程序员）
├── art-director（美术总监）
├── audio-director（音频总监）
├── narrative-director（叙事总监）
├── qa-lead（质量主管）
├── release-manager（发布经理）
└── localization-lead（本地化主管）

第三层 — 专家专员（使用Sonnet/Haiku模型）
├── gameplay-programmer（游戏程序员）
├── engine-programmer（引擎程序员）
├── ai-programmer（AI程序员）
├── network-programmer（网络程序员）
├── ui-programmer（UI程序员）
├── tools-programmer（工具程序员）
├── systems-designer（系统设计师）
├── level-designer（关卡设计师）
├── economy-designer（经济设计师）
├── technical-artist（技术美术）
├── sound-designer（音效设计师）
├── writer（编剧）
├── world-builder（世界观构建师）
├── ux-designer（UX设计师）
├── prototyper（原型设计师）
├── performance-analyst（性能分析师）
├── devops-engineer（DevOps工程师）
├── analytics-engineer（数据分析工程师）
├── security-engineer（安全工程师）
├── qa-tester（QA测试员）
├── accessibility-specialist（无障碍专家）
├── live-ops-designer（运营设计师）
└── community-manager（社区经理）
```

### 引擎专家代理集

项目包含三个主流引擎的完整专家集：

| 引擎 | 主导代理 | 专家专员 |
|------|---------|---------|
| **Godot 4** | godot-specialist | GDScript、Shaders、GDExtension |
| **Unity** | unity-specialist | DOTS/ECS、Shaders/VFX、Addressables、UI Toolkit |
| **Unreal Engine 5** | unreal-specialist | GAS、Blueprints、Replication、UMG/CommonUI |

### 主要斜杠命令

**入门与导航**
- `/start` — 引导式入职，路由到正确的工作流
- `/help` — 上下文感知的"下一步做什么"
- `/project-stage-detect` — 全项目审计确定当前阶段

**游戏设计**
- `/brainstorm` — 使用MDA分析的协作构思
- `/map-systems` — 将概念分解为系统索引
- `/design-system` — 分节引导式GDD创作
- `/review-all-gdds` — 跨GDD一致性和设计理论审查

**故事与冲刺**
- `/create-epics` — 从GDD和ADR创建史诗
- `/create-stories` — 将史诗拆分为故事文件
- `/dev-story` — 实施故事，自动路由到正确的程序员代理
- `/sprint-plan` — 创建和管理冲刺计划

**团队编排**
- `/team-combat` — 战斗系统：设计到实施
- `/team-narrative` — 叙事内容：结构到对话
- `/team-ui` — UI功能：UX规范到实施
- `/team-release` — 发布协调：构建+QA+部署

---

## How（怎么做）—— 快速开始与最佳实践

### 快速开始

#### 前置要求

- [Git](https://git-scm.com/)
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (`npm install -g @anthropic-ai/claude-code`)
- 推荐安装：[jq](https://jqlang.github.io/jq/)（用于钩子验证）和Python 3（JSON验证）

#### 设置步骤

1. **克隆或使用为模板**
   ```bash
   git clone https://github.com/Donchitos/Claude-Code-Game-Studios.git my-game
   cd my-game
   ```

2. **打开Claude Code并启动会话**
   ```bash
   claude
   ```

3. **运行 `/start`** — 系统会询问你目前的状态并引导到正确的工作流

4. **验证钩子是否正常工作** — 新会话启动时应看到session-start.sh的输出

### 7阶段开发流程

```
阶段1: 概念 → 阶段2: 系统设计 → 阶段3: 技术设置
    ↓              ↓                ↓
/brainstorm    /design-system   /create-architecture
/setup-engine  /design-review   /architecture-decision
/map-systems                     /create-control-manifest

阶段4: 预生产 → 阶段5: 生产 → 阶段6: 打磨 → 阶段7: 发布
    ↓              ↓              ↓            ↓
/ux-design    /dev-story     /perf-profile  /release-checklist
/prototype     /story-done    /balance-check /launch-checklist
/create-epics  /sprint-plan   /team-polish   /team-release
/create-stories
```

### 协作协议

所有代理遵循**用户驱动的协作协议**，而非自主执行：

1. **提问** — 代理在提出解决方案前先问问题
2. **呈现选项** — 代理展示2-4个选项及其优缺点
3. **用户决策** — 用户始终做决定
4. **草案** — 代理先展示工作再定稿
5. **批准** — 未经用户批准不写入任何内容

### 自动化安全机制

#### 钩子系统（12个自动化钩子）

| 钩子 | 触发时机 | 功能 |
|------|---------|------|
| `session-start.sh` | 会话开始 | 显示分支、最近提交、检测active.md恢复 |
| `detect-gaps.sh` | 会话开始 | 检测新项目并建议/start |
| `validate-commit.sh` | 提交前 | 检查设计文档引用、JSON有效性、魔数 |
| `validate-push.sh` | 推送前 | 警告推送到受保护分支 |
| `validate-assets.sh` | 资源文件写入后 | 验证命名规范和大小 |
| `pre-compact.sh` | 压缩前 | 保存会话状态 |
| `post-compact.sh` | 压缩后 | 提醒从active.md恢复状态 |
| `log-agent.sh` | 代理启动 | 审计追踪 |
| `log-agent-stop.sh` | 代理停止 | 完成审计记录 |
| `session-stop.sh` | 会话结束 | 最终会话日志 |
| `notify.sh` | 通知事件 | Windows系统通知 |
| `validate-skill-change.sh` | 技能文件修改后 | 建议运行/skill-test |

#### 权限规则

`settings.json`中配置：
- **自动允许**：git status、git diff、测试运行等安全操作
- **自动阻止**：force push、rm -rf、.env文件访问等危险操作

### 路径作用域编码标准

| 路径 | 强制标准 |
|------|---------|
| `src/gameplay/**` | 数据驱动值、delta time使用、无UI引用 |
| `src/core/**` | 热路径零分配、线程安全、API稳定性 |
| `src/ai/**` | 性能预算、可调试性、数据驱动参数 |
| `src/networking/**` | 服务器权威、版本化消息、安全性 |
| `src/ui/**` | 无游戏状态所有权、本地化就绪、无障碍 |
| `design/gdd/**` | 必需8节、公式格式、边缘情况 |
| `tests/**` | 测试命名、覆盖率要求、fixture模式 |
| `prototypes/**` | 宽松标准、必需README、记录假设 |

### 审核模式

设置审核强度（通过`/start`或编辑`production/review-mode.txt`）：

| 模式 | 执行内容 | 适用场景 |
|------|---------|----------|
| `full` | 每一步都运行所有总监门控 | 新项目、学习系统 |
| `lean` | 仅在阶段转换时运行总监 | 有经验的开发者 |
| `solo` | 无总监审核 | 游戏jam、原型、最大速度 |

### 项目结构

```
my-game/
├── CLAUDE.md                     # 主配置
├── .claude/
│   ├── settings.json             # 钩子、权限、安全规则
│   ├── agents/                   # 49个代理定义
│   ├── skills/                   # 72个斜杠命令
│   ├── hooks/                    # 12个钩子脚本
│   ├── rules/                    # 11条路径作用域规则
│   └── statusline.sh             # 状态行脚本
├── src/                          # 游戏源代码
├── assets/                       # 美术、音频、VFX、着色器、数据文件
├── design/                       # GDD、叙事文档、关卡设计
├── docs/                         # 技术文档和ADR
├── tests/                        # 测试套件
├── tools/                        # 构建和流水线工具
├── prototypes/                   # 临时原型（与src隔离）
└── production/                   # 冲刺计划、里程碑、发布跟踪
```

### 关键配置说明

#### 技术偏好（`.claude/docs/technical-preferences.md`）

运行`/setup-engine`后创建，包含：
- 命名规范
- 性能预算
- 引擎特定默认值
- 知识差距检测

#### 阶段文件（`production/stage.txt`）

记录当前开发阶段，由`/gate-check`通过时更新：
- 控制状态行显示
- 影响`/help`的行为

#### 冲刺状态（`production/sprint-status.yaml`）

机器可读的故事追踪器：
- 由`/sprint-plan`初始化
- 由`/story-done`更新
- 被`/sprint-status`读取

---

## 设计理念

项目基于专业游戏开发实践：

- **MDA框架** — 力学、动力学、美学分析
- **自我决定理论** — 自主性、能力、关联性
- **心流设计** — 挑战-技能平衡
- **Bartle玩家类型** — 受众定位和验证
- **验证驱动开发** — 测试优先，然后实现

---

## 技术栈支持

| 组件 | 支持情况 |
|------|----------|
| **操作系统** | Windows 10 (Git Bash)、macOS、Linux |
| **游戏引擎** | Godot 4、Unity、Unreal Engine 5 |
| **编程语言** | GDScript、C#、C++、Blueprint |
| **版本控制** | Git（基于主干的开发） |

---

## 许可

MIT License — 详见 [LICENSE](LICENSE)
