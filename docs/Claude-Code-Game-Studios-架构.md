# Claude Code Game Studios 架构图

本文档使用Mermaid图表展示Claude Code Game Studios项目的系统架构。

---

## 1. 系统整体架构

```mermaid
graph TB
    subgraph 用户层
        User["👤 用户"]
    end

    subgraph Claude Code层
        CC["🤖 Claude Code"]
        Settings["settings.json<br/>权限/钩子配置"]
    end

    subgraph 配置层 .claude/
        Agents["agents/<br/>49个代理定义"]
        Skills["skills/<br/>72个技能"]
        Hooks["hooks/<br/>12个钩子"]
        Rules["rules/<br/>11条规则"]
        Templates["docs/templates/<br/>39个模板"]
    end

    subgraph 输出层
        Src["src/<br/>游戏源代码"]
        Design["design/<br/>GDD/叙事"]
        Tests["tests/<br/>测试套件"]
    end

    User --> CC
    CC --> Settings
    CC --> Agents
    CC --> Skills
    CC --> Hooks
    CC --> Rules
    CC --> Templates
    CC --> Src
    CC --> Design
    CC --> Tests

    classDef user fill:#E3F2FD
    classDef claude fill:#F3E5F5
    classDef config fill:#E8F5E9
    classDef output fill:#FFF3E0

    class User user
    class CC claude
    class Settings,Agents,Skills,Hooks,Rules,Templates config
    class Src,Design,Tests output
```

---

## 2. 代理层级架构

```mermaid
graph TB
    subgraph "第一层 — 总监 (Opus)"
        CD["creative-director<br/>创意总监"]
        TD["technical-director<br/>技术总监"]
        P["producer<br/>制作人"]
    end

    subgraph "第二层 — 部门主管 (Sonnet)"
        GD["game-designer<br/>游戏设计师"]
        LP["lead-programmer<br/>首席程序员"]
        AD["art-director<br/>美术总监"]
        AUD["audio-director<br/>音频总监"]
        ND["narrative-director<br/>叙事总监"]
        QA["qa-lead<br/>质量主管"]
    end

    subgraph "第三层 — 专家专员 (Sonnet/Haiku)"
        GP["gameplay-programmer<br/>游戏程序员"]
        EP["engine-programmer<br/>引擎程序员"]
        AIP["ai-programmer<br/>AI程序员"]
        NP["network-programmer<br/>网络程序员"]
        UIP["ui-programmer<br/>UI程序员"]
        TP["tools-programmer<br/>工具程序员"]
        SD["systems-designer<br/>系统设计师"]
        LD["level-designer<br/>关卡设计师"]
    end

    CD --> GD
    CD --> AD
    TD --> LP
    TD --> EP
    P --> QA
    LP --> GP
    LP --> EP
    LP --> AIP
    LP --> NP
    LP --> UIP
    GD --> SD
    GD --> LD

    classDef tier1 fill:#FFCDD2
    classDef tier2 fill:#C8E6C9
    classDef tier3 fill:#BBDEFB

    class CD,TD,P tier1
    class GD,LP,AD,AUD,ND,QA tier2
    class GP,EP,AIP,NP,UIP,TP,SD,LD tier3
```

---

## 3. 7阶段开发流程

```mermaid
flowchart LR
    subgraph 阶段1_概念
        B["/brainstorm"]
        SM["/setup-engine"]
        MS["/map-systems"]
    end

    subgraph 阶段2_系统设计
        DS["/design-system"]
        DR["/design-review"]
    end

    subgraph 阶段3_技术设置
        CA["/create-architecture"]
        ADR["/architecture-decision"]
        CCM["/create-control-manifest"]
    end

    subgraph 阶段4_预生产
        UX["/ux-design"]
        PR["/prototype"]
        CE["/create-epics"]
        CS["/create-stories"]
    end

    subgraph 阶段5_生产
        DST["/dev-story"]
        SD["/story-done"]
        SP["/sprint-plan"]
    end

    subgraph 阶段6_打磨
        PP["/perf-profile"]
        BC["/balance-check"]
        TP["/team-polish"]
    end

    subgraph 阶段7_发布
        RC["/release-checklist"]
        LC["/launch-checklist"]
        TR["/team-release"]
    end

    阶段1_概念 --> 阶段2_系统设计
    阶段2_系统设计 --> 阶段3_技术设置
    阶段3_技术设置 --> 阶段4_预生产
    阶段4_预生产 --> 阶段5_生产
    阶段5_生产 --> 阶段6_打磨
    阶段6_打磨 --> 阶段7_发布

    classDef phase fill:#E1F5FE
    classDef skill fill:#F3E5F5

    class B,SM,MS,DS,DR,CA,ADR,CCM,UX,PR,CE,CS,DST,SD,SP,PP,BC,TP,RC,LC,TR skill
```

---

## 4. 核心数据流

```mermaid
flowchart TB
    subgraph 输入
        GDD["design/gdd/*.md<br/>游戏设计文档"]
        ADR["docs/adr/*.md<br/>架构决策记录"]
        CC["Control Manifest<br/>技术偏好"]
    end

    subgraph 处理
        EPIC["创建史诗<br/>/create-epics"]
        STORY["拆分故事<br/>/create-stories"]
        DEV["实施开发<br/>/dev-story"]
    end

    subgraph 输出
        CODE["src/<br/>源代码"]
        TEST["tests/<br/>测试"]
        DOC["docs/<br/>文档"]
    end

    subgraph 代理路由
        GP["gameplay-programmer"]
        EP["engine-programmer"]
        AIP["ai-programmer"]
        NP["network-programmer"]
    end

    GDD --> EPIC
    ADR --> EPIC
    CC --> EPIC
    EPIC --> STORY
    STORY --> DEV
    DEV --> CODE
    CODE --> TEST

    DEV --> GP
    DEV --> EP
    DEV --> AIP
    DEV --> NP

    classDef input fill:#E3F2FD
    classDef process fill:#F3E5F5
    classDef output fill:#E8F5E9
    classDef agent fill:#FFF3E0

    class GDD,ADR,CC input
    class EPIC,STORY,DEV process
    class CODE,TEST,DOC output
    class GP,EP,AIP,NP agent
```

---

## 5. 协作协议流程

```mermaid
sequenceDiagram
    participant U as 👤 用户
    participant A as 🤖 代理

    Note over U: 用户提出请求
    U->>A: 请求任务

    A->>U: 提问澄清
    Note over U: 用户回答问题

    U->>A: 提供需求信息
    A->>U: 呈现选项 (2-4个)

    Note over U: 用户分析选项
    U->>A: 选择方案

    A->>U: 展示草案
    Note over U: 用户审查

    U->>A: 批准方案
    A->>A: 写入文件

    A->>U: 完成确认
    Note over U: 用户验证
```

---

## 6. 钩子执行时序

```mermaid
sequenceDiagram
    participant G as Git
    participant H as 钩子系统
    participant CC as Claude Code
    participant FS as 文件系统

    Note over H,CC: 会话开始
    CC->>H: session-start.sh
    H->>FS: 检查active.md
    H->>CC: 显示分支/提交信息

    CC->>H: detect-gaps.sh
    H->>FS: 审计项目结构
    H->>CC: 建议/start

    Note over G: 提交时
    G->>H: validate-commit.sh
    H->>FS: 检查设计文档引用
    H->>FS: 验证JSON
    H->>FS: 检测魔数
    H->>G: 通过/警告

    Note over G: 推送时
    G->>H: validate-push.sh
    H->>G: 检查分支保护
    H->>G: 阻止/警告

    Note over H,CC: 会话结束
    CC->>H: session-stop.sh
    H->>FS: 保存日志
```

---

## 7. 故事生命周期

```mermaid
stateDiagram-v2
    [*] --> Epic创建: /create-epics

    Epic创建 --> Story拆分: /create-stories

    Story拆分 --> 开发中: /dev-story

    开发中 --> 待审查: 提交PR

    待审查 --> 代码审查: /review-code

    代码审查 --> 测试中: CI运行

    测试中 --> 待合并: 所有测试通过

    待合并 --> 已完成: PR合并

    已完成 --> [*]: /story-done

    开发中 --> 待审查: 审查失败
    代码审查 --> 开发中: 需要修改
    测试中 --> 开发中: 测试失败

    state 开发中 {
        [*] --> 实现
        实现 --> 自测
        自测 --> 提交
        提交 --> [*]
    }
```

---

## 8. 权限控制流

```mermaid
flowchart TB
    subgraph settings_json
        AL["allow:<br/>git status<br/>git diff<br/>测试运行"]
        AD["deny:<br/>force push<br/>rm -rf<br/>.env访问"]
    end

    subgraph 执行检查
        CMD["用户命令"]
        CHK["权限检查器"]
        DEC["决策"]
    end

    subgraph 结果
        OK["✅ 执行"]
        BLOCK["❌ 阻止"]
        ASK["❓ 询问用户"]
    end

    CMD --> CHK
    CHK --> DEC

    DEC --> |allow列表| OK
    DEC --> |deny列表| BLOCK
    DEC --> |不在列表| ASK

    classDef settings fill:#E3F2FD
    classDef check fill:#F3E5F5
    classDef result fill:#E8F5E9

    class AL,AD settings
    class CMD,CHK,DEC check
    class OK,BLOCK,ASK result
```

---

## 9. 状态行架构

```mermaid
flowchart LR
    subgraph 输入
        ST["production/stage.txt<br/>当前阶段"]
        SS["production/sprint-status.yaml<br/>冲刺状态"]
    end

    subgraph 脚本
        STATUS["statusline.sh<br/>状态行脚本"]
    end

    subgraph 输出
        DIS["显示格式<br/>[阶段] 🔥5 💤2 ⚠1"]
    end

    ST --> STATUS
    SS --> STATUS
    STATUS --> DIS
```

状态行显示示例：
```
[Phase 3] 🔥5 stories | 💤2 blocked | ⚠1 gap
```

---

## 10. 文件结构映射

```mermaid
graph TB
    subgraph 源代码 src/
        Src["src/"]
        SrcCore["src/core/<br/>引擎/核心系统"]
        SrcGameplay["src/gameplay/<br/>游戏逻辑"]
        SrcAI["src/ai/<br/>AI行为树"]
        SrcNet["src/networking/<br/>多人/网络"]
        SrcUI["src/ui/<br/>界面组件"]
        SrcTools["src/tools/<br/>编辑器工具"]
    end

    subgraph 设计 design/
        Design["design/"]
        DesignGDD["design/gdd/<br/>游戏设计文档"]
        DesignNarrative["design/narrative/<br/>叙事内容"]
        DesignUX["design/ux/<br/>UX规范"]
    end

    subgraph 资源 assets/
        Assets["assets/"]
        AssetsArt["assets/art/<br/>精灵/模型/纹理"]
        AssetsAudio["assets/audio/<br/>音乐/SFX"]
        AssetsData["assets/data/<br/>配置/平衡数据"]
    end

    subgraph 测试 tests/
        Tests["tests/<br/>测试套件"]
    end

    subgraph 原型 prototypes/
        Prototypes["prototypes/<br/>临时原型"]
    end

    Root["项目根目录"]

    Root --> Src
    Root --> Design
    Root --> Assets
    Root --> Tests
    Root --> Prototypes

    Src --> SrcCore
    Src --> SrcGameplay
    Src --> SrcAI
    Src --> SrcNet
    Src --> SrcUI
    Src --> SrcTools

    Design --> DesignGDD
    Design --> DesignNarrative
    Design --> DesignUX

    Assets --> AssetsArt
    Assets --> AssetsAudio
    Assets --> AssetsData

    classDef src fill:#E3F2FD
    classDef design fill:#F3E5F5
    classDef assets fill:#E8F5E9
    classDef tests fill:#FFF3E0
    classDef proto fill:#FFEBEE

    class Src,SrcCore,SrcGameplay,SrcAI,SrcNet,SrcUI,SrcTools src
    class Design,DesignGDD,DesignNarrative,DesignUX design
    class Assets,AssetsArt,AssetsAudio,AssetsData assets
    class Tests tests
    class Prototypes proto
```

---

## 11. 引擎特定架构

```mermaid
graph TB
    subgraph Godot_4
        GAgent["godot-specialist"]
        GGDE["GDScript专家"]
        GShader["着色器专家"]
        GExt["GDExtension专家"]
    end

    subgraph Unity
        UAgent["unity-specialist"]
        UDOTS["DOTS/ECS专家"]
        UShader["Shader/VFX专家"]
        UAddr["Addressables专家"]
    end

    subgraph Unreal_5
        UEAgent["unreal-specialist"]
        UEGAS["GAS专家"]
        UEBP["Blueprints专家"]
        UERep["Replication专家"]
    end

    GAgent --> GGDE
    GAgent --> GShader
    GAgent --> GExt

    UAgent --> UDOTS
    UAgent --> UShader
    UAgent --> UAddr

    UEAgent --> UEGAS
    UEAgent --> UEBP
    UEAgent --> UERep

    classDef godot fill:#B3E5FC
    classDef unity fill:#C8E6C9
    classDef unreal fill:#E1BEE7

    class GAgent,GGDE,GShader,GExt godot
    class UAgent,UDOTS,UShader,UAddr unity
    class UEAgent,UEGAS,UEBP,UERep unreal
```
