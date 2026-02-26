# 每天了解一个AI项目

#### 1. 用why-what-how的方式介绍XX项目
#### 2. 用mermaid图了解XX项目架构
#### 3. 聚焦XX项目是如何使用prompt的

---

## 项目目录

每个项目包含三篇分析文档：**介绍**（Why-What-How）、**架构**（Mermaid 图）、**提示词**（Prompt 分析）。

| # | 项目 | 介绍 | 架构 | 提示词 |
|---|------|------|------|--------|
| 1 | Browser-Use | [介绍](docs/Browser-Use/Browser-Use-介绍.md) | [架构](docs/Browser-Use/Browser-Use-架构.md) | [提示词](docs/Browser-Use/Browser-Use-提示词.md) |
| 2 | Clawdbot | [介绍](docs/Clawdbot/Clawdbot-介绍.md) | [架构](docs/Clawdbot/Clawdbot-架构.md) | [提示词](docs/Clawdbot/Clawdbot-提示词.md) |
| 3 | DataScienceForBeginners | [介绍](docs/DataScienceForBeginners/DataScienceForBeginners-介绍.md) | [架构](docs/DataScienceForBeginners/DataScienceForBeginners-架构.md) | — |
| 4 | DeepDiagram | [介绍](docs/DeepDiagram/DeepDiagram-介绍.md) | [架构](docs/DeepDiagram/DeepDiagram-架构.md) | [提示词](docs/DeepDiagram/DeepDiagram-提示词.md) |
| 5 | dexter | [介绍](docs/dexter/dexter-介绍.md) | [架构](docs/dexter/dexter-架构.md) | [提示词](docs/dexter/dexter-提示词.md) |
| 6 | FlashMLA | [介绍](docs/FlashMLA/FlashMLA-介绍.md) | [架构](docs/FlashMLA/FlashMLA-架构.md) | [提示词](docs/FlashMLA/FlashMLA-提示词.md) |
| 7 | FossFLOW | [介绍](docs/FossFLOW/FossFLOW-介绍.md) | [架构](docs/FossFLOW/FossFLOW-架构.md) | [提示词](docs/FossFLOW/FossFLOW-提示词.md) |
| 8 | GitNexus | [介绍](docs/GitNexus/GitNexus-介绍.md) | [架构](docs/GitNexus/GitNexus-架构.md) | [提示词](docs/GitNexus/GitNexus-提示词.md) |
| 9 | graphiti | [介绍](docs/graphiti/graphiti-介绍.md) | [架构](docs/graphiti/graphiti-架构.md) | [提示词](docs/graphiti/graphiti-提示词.md) |
| 10 | langextract | [介绍](docs/langextract/langextract-介绍.md) | [架构](docs/langextract/langextract-架构.md) | [提示词](docs/langextract/langextract-提示词.md) |
| 11 | Memos | [介绍](docs/Memos/Memos-介绍.md) | [架构](docs/Memos/Memos-架构.md) | [提示词](docs/Memos/Memos-提示词.md) |
| 12 | nanobot | [介绍](docs/nanobot/nanobot-介绍.md) | [架构](docs/nanobot/nanobot-架构.md) | [提示词](docs/nanobot/nanobot-提示词.md) |
| 13 | openclaw | [介绍](docs/openclaw/openclaw-介绍.md) | [架构](docs/openclaw/openclaw-架构.md) | [提示词](docs/openclaw/openclaw-提示词.md) |
| 14 | pathway | [介绍](docs/pathway/pathway-介绍.md) | — | — |
| 15 | pentagi | [介绍](docs/pentagi/pentagi-介绍.md) | [架构](docs/pentagi/pentagi-架构.md) | [提示词](docs/pentagi/pentagi-提示词.md) |
| 16 | Personal AI Infrastructure | [介绍](docs/Personal_AI_Infrastructure/Personal_AI_Infrastructure-介绍.md) | [架构](docs/Personal_AI_Infrastructure/Personal_AI_Infrastructure-架构.md) | [提示词](docs/Personal_AI_Infrastructure/Personal_AI_Infrastructure-提示词.md) |
| 17 | skills | [介绍](docs/skills/skills-介绍.md) | [架构](docs/skills/skills-架构.md) | [提示词](docs/skills/skills-提示词.md) |
| 18 | SuperMemory | [介绍](docs/SuperMemory/SuperMemory-介绍.md) | [架构](docs/SuperMemory/SuperMemory-架构.md) | [提示词](docs/SuperMemory/SuperMemory-提示词.md) |
| 19 | Superpowers | [介绍](docs/Superpowers/Superpowers-介绍.md) | [架构](docs/Superpowers/Superpowers-架构.md) | [提示词](docs/Superpowers/Superpowers-提示词.md) |
| 20 | system_prompts_leaks | [介绍](docs/system_prompts_leaks/system_prompts_leaks-介绍.md) | [架构](docs/system_prompts_leaks/system_prompts_leaks-架构.md) | [提示词](docs/system_prompts_leaks/system_prompts_leaks-提示词.md) |
| 21 | vibetunnel | [介绍](docs/vibetunnel/vibetunnel-介绍.md) | [架构](docs/vibetunnel/vibetunnel-架构.md) | [提示词](docs/vibetunnel/vibetunnel-提示词.md) |
| 22 | whatsapp-web.js | [介绍](docs/whatsapp-web.js/whatsapp-web.js-介绍.md) | [架构](docs/whatsapp-web.js/whatsapp-web.js-架构.md) | [提示词](docs/whatsapp-web.js/whatsapp-web.js-提示词.md) |

## 对比分析

| 主题 | 文档 |
|------|------|
| Pathway vs Kafka | [对比](docs/compare/pathway-vs-kafka.md) |
| PostgreSQL vs Redis | [对比](docs/compare/postgre-vs-redis.md) |

## 目录结构

```
├── README.md
├── mkdocs.yml
├── docs/
│   ├── index.md               # MkDocs 首页
│   ├── 项目名/
│   │   ├── 项目名-介绍.md    # Why-What-How 介绍
│   │   ├── 项目名-架构.md    # Mermaid 架构图分析
│   │   └── 项目名-提示词.md  # Prompt 提示词分析
│   ├── ...
│   └── compare/               # 项目对比分析
│       ├── pathway-vs-kafka.md
│       └── postgre-vs-redis.md
```
