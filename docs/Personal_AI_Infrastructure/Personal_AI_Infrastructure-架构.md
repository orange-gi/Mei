# Personal AI Infrastructure 架构文档

## 系统整体架构

PAI采用分层架构设计，从底层到顶层依次为：基础设施层提供核心运行环境（Claude Code/Bun/终端），其上构建持久记忆系统、模块化技能系统、事件驱动钩子系统，以及最顶层的目标理解和用户交互层。这种架构确保了系统的可扩展性和模块化特性。

```mermaid
graph TB
    subgraph 用户层["用户层"]
        User["用户"]
        CLI["命令行界面"]
        Terminal["终端 UI"]
    end

    subgraph 核心系统层["核心系统层"]
        Goals["目标系统<br/>TELOS"]
        Preferences["偏好系统"]
        Identity["身份系统"]
        Skills["技能路由器"]
    end

    subgraph 能力系统层["能力系统层"]
        Hooks["钩子系统"]
        Skills2["技能系统"]
        Memory["记忆系统"]
        Security["安全系统"]
    end

    subgraph 基础设施层["基础设施层"]
        Claude["Claude Code<br/>AI引擎"]
        Bun["Bun 运行时"]
        Platform["平台适配<br/>macOS/Linux"]
    end

    User --> CLI
    CLI --> Terminal
    Terminal --> 核心系统层
    核心系统层 --> 能力系统层
    能力系统层 --> 基础设施层

    Goals -.->|提供上下文| Skills
    Preferences -.->|提供上下文| Skills
    Identity -.->|提供上下文| Skills
    Hooks -->|事件触发| Skills2
    Memory -->|持续学习| Skills2
```

## 核心数据流

PAI的数据流遵循科学方法循环：观察→思考→计划→执行→验证→学习→改进。系统通过记忆系统捕获每次交互的信号（评分、情感、成功、失败），这些信号反馈到学习循环中，使系统随着时间推移不断优化。

```mermaid
flowchart TD
    subgraph 输入["输入阶段"]
        Request["用户请求"]
        Context["上下文加载<br/>目标/偏好/历史"]
    end

    subgraph 思考阶段["思考阶段"]
        Think1["观察 - 理解请求"]
        Think2["思考 - 分析需求"]
        Think3["计划 - 确定最佳方法"]
        Think4["选择 - 确定所需技能"]
    end

    subgraph 执行阶段["执行阶段"]
        Execute1["CODE - 确定性代码"]
        Execute2["CLI - 命令行工具"]
        Execute3["PROMPT - 提示词"]
        Execute4["SKILL - 技能执行"]
    end

    subgraph 验证阶段["验证阶段"]
        Verify1["验证输出质量"]
        Verify2["检查是否符合目标"]
    end

    subgraph 学习阶段["学习阶段"]
        Learn1["捕获信号"]
        Learn2["分析结果"]
        Learn3["更新记忆"]
        Learn4["改进系统"]
    end

    Request --> Context
    Context --> Think1
    Think1 --> Think2
    Think2 --> Think3
    Think3 --> Think4
    Think4 --> Execute1
    Execute1 -->|失败| Execute2
    Execute2 -->|失败| Execute3
    Execute3 -->|失败| Execute4
    Execute4 --> Verify1
    Verify1 -->|通过| Verify2
    Verify2 -->|通过| Learn1
    Verify2 -->|失败| Think1
    Learn1 --> Learn2
    Learn2 --> Learn3
    Learn3 --> Learn4
    Learn4 -->|"增强上下文"| Context
```

## 模块关系图

PAI的模块化设计通过包（Pack）系统实现，每个包都是独立的功能单元。包之间存在依赖关系：钩子系统是基础，其他所有包都依赖它；核心安装包提供身份、路由和记忆功能；技能包提供具体能力；其他系统包提供增强功能。

```mermaid
graph BT
    subgraph Infrastructure["基础设施包"]
        Hook["pai-hook-system<br/>事件驱动自动化"]
        Core["pai-core-install<br/>核心技能/身份/记忆"]
        Status["pai-statusline<br/>状态显示"]
    end

    subgraph Capability["能力包"]
        Voice["pai-voice-system<br/>语音通知"]
        Observe["pai-observability-server<br/>监控服务"]
        Browser["pai-browser-skill<br/>浏览器自动化"]
    end

    subgraph Skills["技能包"]
        Agents["pai-agents-skill<br/>动态代理"]
        Research["pai-research-skill<br/>研究能力"]
        Telos["pai-telos-skill<br/>目标管理"]
        System["pai-system-skill<br/>系统维护"]
        Other["其他18个技能包..."]
    end

    Hook --> Core
    Hook --> Status
    Core --> Voice
    Core --> Browser
    Core --> Observe
    Core --> Agents
    Core -->|依赖| Telos
    Core --> System
    Core --> Research
    Core --> Other

    Telos -.->|提供目标| Agents
    Telos -.->|提供目标| Research
    System -.->|维护| Core
```

## 用户交互流程

用户与PAI的交互遵循一个循环流程：启动会话时加载用户上下文和目标，用户提出请求后系统通过技能路由器选择合适的技能，执行过程中钩子监控系统状态，任务完成后学习系统捕获结果信号，最终系统自我改进以更好地服务用户。

```mermaid
sequenceDiagram
    participant User as 用户
    participant Terminal as 终端
    participant Hooks as 钩子系统
    participant Context as 上下文加载器
    participant Router as 技能路由器
    participant Skills as 技能执行器
    participant Memory as 记忆系统
    participant Voice as 语音系统

    Note over User,Voice: 会话启动阶段
    User->>Terminal: 启动Claude Code
    Terminal->>Hooks: 触发SessionStart
    Hooks->>Context: 加载用户目标/偏好/历史
    Context-->>Terminal: 返回完整上下文

    Note over User,Voice: 任务执行阶段
    User->>Terminal: 提出请求
    Terminal->>Router: 路由请求
    Router->>Skills: 选择并执行技能
    Skills->>Hooks: 触发ToolUse/TaskComplete
    Hooks-->>Terminal: 验证/通知
    Skills-->>Terminal: 返回结果

    Note over User,Voice: 学习阶段
    Terminal->>Memory: 捕获信号
    Memory->>Memory: 分析结果
    Memory->>Memory: 更新学习
    Memory->>Terminal: 改进建议

    Note over User,Voice: 通知阶段
    Terminal->>Voice: 任务完成
    Voice-->>User: 语音通知
```

## 部署架构

PAI采用本地部署模式，运行在用户的终端环境中。系统支持macOS和Linux平台，通过Bun运行时执行。部署结构分为用户空间和系统空间，用户自定义存储在USER目录，系统升级时用户数据不受影响，确保了数据的安全性和可移植性。

```mermaid
graph TB
    subgraph Deployment["PAI 部署架构"]
        subgraph UserSpace["用户空间 ~/.claude/"]
            USER["USER/<br/>用户自定义<br/>身份/目标/偏好"]
            MEMORY["MEMORY/<br/>记忆系统<br/>历史/学习/信号"]
            Custom["自定义技能<br/>和工作流"]
        end

        subgraph SystemSpace["系统空间"]
            Skills["skills/<br/>技能目录"]
            Hooks["hooks/<br/>钩子目录"]
            Settings["settings.json<br/>配置"]
            Install["INSTALL.ts<br/>安装向导"]
        end

        subgraph Runtime["运行时环境"]
            Bun["Bun 运行时"]
            Claude["Claude Code"]
            Terminal["终端"]
        end

        subgraph External["外部服务"]
            ElevenLabs["ElevenLabs<br/>语音服务"]
            Discord["Discord<br/>通知"]
            Ntfy["ntfy<br/>推送通知"]
        end

        USER -->|持久化| MEMORY
        Custom -->|扩展| Skills
        Hooks -->|触发| Skills
        Settings -->|配置| Install

        Skills -->|执行| Bun
        Hooks -->|执行| Bun
        Bun -->|运行| Claude
        Claude -->|交互| Terminal

        Voice -.->|语音通知| ElevenLabs
        Hooks -.->|集成| Discord
        Hooks -.->|推送| Ntfy
    end

    style UserSpace fill:#e1f5fe
    style SystemSpace fill:#f3e5f5
    style Runtime fill:#e8f5e9
    style External fill:#fff3e0
```

## 包依赖关系

PAI的包系统具有明确的依赖层次。基础设施包构成系统的基石，技能包基于基础设施构建。安装顺序至关重要：钩子系统必须首先安装，因为它是所有其他功能的基础；核心安装包提供身份、路由和记忆功能；之后才能安装技能包。

```mermaid
graph LR
    subgraph Layer1["第一层 - 必需"]
        HookSystem["pai-hook-system<br/>事件驱动自动化"]
    end

    subgraph Layer2["第二层 - 核心"]
        CoreInstall["pai-core-install<br/>核心系统"]
        StatusLine["pai-statusline<br/>状态显示"]
    end

    subgraph Layer3["第三层 - 能力"]
        Voice["pai-voice-system"]
        Browser["pai-browser-skill"]
        Agents["pai-agents-skill"]
    end

    subgraph Layer4["第四层 - 技能"]
        Research["pai-research-skill"]
        Telos["pai-telos-skill"]
        Recon["pai-recon-skill"]
        OSINT["pai-osint-skill"]
        Other["其他14个技能"]
    end

    HookSystem --> CoreInstall
    HookSystem --> StatusLine
    CoreInstall --> Voice
    CoreInstall --> Browser
    CoreInstall --> Agents
    Agents --> Research
    Agents --> Telos
    Agents --> Recon
    Agents --> OSINT
    Research --> Other

    style Layer1 fill:#ffcdd2
    style Layer2 fill:#fff9c4
    style Layer3 fill:#c8e6c9
    style Layer4 fill:#bbdefb
```

## 记忆系统架构

PAI的记忆系统采用三层架构设计：热层存储当前会话的即时信息，温层保存最近的历史记录和学习成果，冷层归档长期数据。系统通过信号捕获机制持续收集用户反馈（评分、情感、验证结果），这些信号经过分析后用于更新学习，最终实现系统的自我改进。

```mermaid
flowchart TB
    subgraph Input["信号输入"]
        Ratings["评分信号"]
        Sentiment["情感信号"]
        Verification["验证结果"]
        Success["成功/失败"]
    end

    subgraph Hot["热层 - 即时会话"]
        Current["当前会话<br/>实时访问"]
    end

    subgraph Warm["温层 - 最近历史"]
        Recent["最近历史<br/>快速查询"]
    end

    subgraph Cold["冷层 - 长期存储"]
        Archive["归档数据<br/>深度分析"]
    end

    subgraph Learning["学习系统"]
        Analyze["分析信号"]
        Extract["提取模式"]
        Update["更新知识"]
        Improve["改进系统"]
    end

    Input -->|"捕获"| Hot
    Hot -->|"定期刷新"| Warm
    Warm -->|"归档"| Cold

    Hot -->|"学习"| Analyze
    Warm -->|"学习"| Analyze
    Cold -->|"深度分析"| Analyze

    Analyze --> Extract
    Extract --> Update
    Update --> Improve

    Improve -.->|"增强"| Current
    Improve -.->|"优化工作流"| Recent
    Improve -.->|"调整策略"| Archive
```

## 安全架构

PAI采用多层安全策略，在系统级别和用户级别都定义了安全策略。安全钩子在命令执行前进行验证，阻止危险操作的同时允许正常工作流通过。这种设计确保用户无需使用--dangerously-skip-permissions标志也能获得不间断的体验。

```mermaid
flowchart LR
    subgraph User["用户请求"]
        Command["用户命令"]
    end

    subgraph Validation["安全验证"]
        Check1["语法检查"]
        Check2["权限检查"]
        Check3["路径检查"]
        Check4["内容检查"]
    end

    subgraph Policy["策略引擎"]
        Allow["允许列表"]
        Block["阻止列表"]
        Warn["警告列表"]
    end

    subgraph Execution["执行结果"]
        Execute["执行命令"]
        Blocked["阻止执行"]
        Warned["警告后执行"]
    end

    Command --> Check1
    Check1 -->|通过| Check2
    Check1 -->|失败| Blocked
    Check2 -->|通过| Check3
    Check2 -->|失败| Blocked
    Check3 -->|通过| Check4
    Check3 -->|失败| Blocked
    Check4 --> Policy

    Policy -->|允许| Execute
    Policy -->|阻止| Blocked
    Policy -->|警告| Warned

    Allow -.->|更新| Policy
    Block -.->|更新| Policy

    style Validation fill:#fff3e0
    style Policy fill:#e8f5e9
    style Execution fill:#e3f2fd
```
