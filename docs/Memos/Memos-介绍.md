# Memos 项目介绍文档

## 一、项目概述（Why-What-How 框架）

### 1.1 Why - 为什么需要 Memos

在当今数字化时代，个人知识管理和笔记应用已成为人们日常工作和生活的重要组成部分。然而市面上的笔记应用大多存在以下问题：

**隐私与数据安全问题**

- 大多数云端笔记服务将用户数据存储在第三方服务器上，用户无法真正拥有自己的数据
- 存在数据泄露、商家跑路、服务关闭等风险
- 部分应用收集用户行为数据进行商业变现

**成本与锁定问题**

- 许多优质笔记应用采用订阅制收费模式，长期使用成本高昂
- 用户被锁定在特定生态系统中，数据迁移困难
- 导出功能受限或需要额外付费

**性能与体验问题**

- 云端服务依赖网络连接，离线时无法使用
- 页面加载速度受网络质量影响
- 功能臃肿复杂，学习成本高

Memos 的诞生正是为了解决这些痛点，为用户提供一个**隐私优先、完全自托管、免费开源**的笔记解决方案。

### 1.2 What - Memos 是什么

Memos 是一个开源的自托管笔记服务应用，具有以下核心定位：

- **轻量级**：设计简洁，启动快速，资源占用低
- **隐私优先**：所有数据存储在用户自己的服务器上，零遥测、零追踪
- **自托管**：用户完全掌控自己的数据和基础设施
- **Markdown 原生**：支持完整的 Markdown 语法，数据以纯文本形式存储
- **多数据库支持**：支持 SQLite、MySQL、PostgreSQL 三种数据库

**核心功能特性**

| 功能模块 | 描述 |
|---------|------|
| 备忘录管理 | 创建、编辑、删除、归档备忘录，支持置顶和标签管理 |
| Markdown 支持 | 完整的 Markdown 语法支持，包括代码块、任务列表、表格等 |
| 附件管理 | 支持图片、文件等附件的上传和管理 |
| 评论与互动 | 支持对备忘录添加评论和反应（点赞等） |
| 快捷方式 | 自定义快速筛选条件，提高检索效率 |
| 多用户支持 | 支持用户注册、认证和权限管理 |
| RSS 订阅 | 生成 RSS  feeds，方便内容聚合和订阅 |
| 开放 API | 提供完整的 REST 和 gRPC API，便于集成和扩展 |
| 主题支持 | 深色模式、浅色模式，自动切换 |

### 1.3 How - 如何使用 Memos

**快速启动（Docker 方式）**

```bash
# 使用 Docker 快速启动
docker run -d \
  --name memos \
  -p 5230:5230 \
  -v ~/.memos:/var/opt/memos \
  neosmemo/memos:stable
```

启动成功后，访问 `http://localhost:5230` 即可开始使用。

**其他安装方式**

- **Docker Compose**：适合生产环境部署
- **预编译二进制**：支持 Linux、macOS、Windows
- **Kubernetes**：提供 Helm Chart
- **源码编译**：适合开发和定制

**核心使用流程**

```
1. 首次访问 → 创建管理员账户
2. 设置实例名称和访问地址
3. 开始创建备忘录
4. 使用标签组织内容
5. 通过 API 集成外部工具
```

---

## 二、技术架构概览

### 2.1 技术栈

**后端技术**

| 技术 | 用途 |
|-----|------|
| Go 1.25 | 主要开发语言 |
| Echo v4 | HTTP 路由框架 |
| gRPC + ConnectRPC | 高性能 RPC 框架 |
| gRPC Gateway | RESTful API 网关 |
| SQLite / MySQL / PostgreSQL | 数据库支持 | 配置管理 |
 |
| Viper| Cobra | CLI 命令行支持 |
| Google CEL | 过滤表达式引擎 |

**前端技术**

| 技术 | 用途 |
|-----|------|
| React | UI 框架 |
| TypeScript | 类型安全 |
| i18next | 国际化 |
| CSS Modules | 样式管理 |

**插件与集成**

| 插件 | 用途 |
|-----|------|
| Markdown (Goldmark) | Markdown 解析渲染 |
| S3 Storage | AWS S3 兼容存储 |
| Email | 邮件通知 |
| Cron | 定时任务调度 |
| OAuth2 / OIDC | 第三方身份认证 |

### 2.2 目录结构

```
memos/
├── cmd/
│   └── memos/              # 程序入口
├── internal/
│   ├── base/               # 基础资源命名
│   ├── profile/            # 配置剖面
│   ├── util/               # 工具函数
│   └── version/            # 版本信息
├── plugin/
│   ├── cron/               # 定时任务插件
│   ├── email/              # 邮件插件
│   ├── filter/             # 过滤引擎
│   ├── httpgetter/         # HTTP 获取器
│   ├── idp/                # 身份提供商
│   ├── markdown/           # Markdown 处理
│   ├── scheduler/          # 调度器
│   ├── storage/            # 存储插件
│   └── webhook/            # Webhook 插件
├── proto/
│   ├── api/                # API 定义
│   │   └── v1/
│   └── store/              # 数据模型定义
├── server/
│   ├── auth/               # 认证模块
│   ├── router/             # 路由和服务
│   │   ├── api/v1/         # API 实现
│   │   ├── fileserver/     # 文件服务
│   │   ├── frontend/       # 前端静态资源
│   │   └── rss/            # RSS 订阅
│   └── runner/             # 后台任务
├── store/
│   ├── cache/              # 缓存层
│   ├── db/                 # 数据库实现
│   │   ├── mysql/
│   │   ├── postgres/
│   │   └── sqlite/
│   └── migration/          # 数据库迁移
└── web/                    # 前端源码
    ├── src/
    │   ├── components/     # React 组件
    │   ├── pages/          # 页面
    │   ├── hooks/          # 自定义 Hooks
    │   ├── locales/        # 国际化资源
    │   └── utils/          # 前端工具
    └── public/             # 静态资源
```

### 2.3 核心数据模型

**Memo（备忘录）**

```protobuf
message Memo {
  string name = 1;           // 资源名称
  State state = 2;           // 状态
  string creator = 3;        // 创建者
  google.protobuf.Timestamp create_time = 4;
  google.protobuf.Timestamp update_time = 5;
  google.protobuf.Timestamp display_time = 6;
  string content = 7;        // Markdown 内容
  Visibility visibility = 9; // 可见性
  repeated string tags = 10; // 标签
  bool pinned = 11;          // 是否置顶
  repeated Attachment attachments = 12;
  repeated MemoRelation relations = 13;
  repeated Reaction reactions = 14;
  Property property = 15;    // 计算属性
  optional string parent = 16;
  string snippet = 17;       // 内容片段
  optional Location location = 18;
}
```

**Visibility（可见性级别）**

- `PRIVATE`：仅自己可见
- `PROTECTED`：登录用户可见
- `PUBLIC`：公开访问

---

## 三、API 设计

### 3.1 API 架构

Memos 采用 **Protocol Buffers** 定义 API，同时支持 RESTful HTTP 和 gRPC 两种调用方式：

```
客户端 → HTTP/gRPC → API Gateway → Service → Store → Database
```

**主要服务**

| 服务 | 端口/路径 | 功能 |
|-----|----------|------|
| MemoService | /api/v1/memos | 备忘录 CRUD |
| UserService | /api/v1/users | 用户管理 |
| AuthService | /api/v1/auth | 认证授权 |
| AttachmentService | /api/v1/attachments | 附件管理 |
| ActivityService | /api/v1/activities | 活动记录 |
| IDPService | /api/v1/idps | 身份提供商 |
| InstanceService | /api/v1/instances | 实例配置 |
| ShortcutService | /api/v1/shortcuts | 快捷方式 |

### 3.2 认证机制

Memos 支持多种认证方式：

1. **本地账户**：用户名密码认证
2. **OAuth2**：支持 GitHub、Google 等第三方登录
3. **OIDC**：标准 OpenID Connect 协议
4. **JWT Token**：API 访问令牌

---

## 四、部署与配置

### 4.1 环境变量

| 变量 | 说明 | 默认值 |
|-----|------|-------|
| `MEMOS_MODE` | 运行模式 (prod/dev/demo) | dev |
| `MEMOS_ADDR` | 监听地址 | 空 |
| `MEMOS_PORT` | 监听端口 | 8081 |
| `MEMOS_DATA` | 数据目录 | 必须指定 |
| `MEMOS_DRIVER` | 数据库驱动 (sqlite/mysql/postgres) | sqlite |
| `MEMOS_DSN` | 数据库连接字符串 | 空 |
| `MEMOS_INSTANCE_URL` | 实例访问 URL | 空 |

### 4.2 Docker Compose 部署示例

```yaml
version: '3.8'
services:
  memos:
    image: neosmemo/memos:stable
    container_name: memos
    ports:
      - "5230:5230"
    volumes:
      - ./data:/var/opt/memos
    environment:
      - MEMOS_MODE=prod
```

### 4.3 数据库配置

**SQLite（默认）**

```
适合单机部署，零配置，数据存储在本地文件
```

**MySQL**

```
适合需要高可用或与其他应用共享数据库的场景
MEMOS_DRIVER=mysql
MEMOS_DSN=root:password@tcp(localhost:3306)/memos?parseTime=true
```

**PostgreSQL**

```
适合企业级部署，支持更多高级特性
MEMOS_DRIVER=postgres
MEMOS_DSN=postgres://user:password@localhost:5432/memos?sslmode=disable
```

---

## 五、数据导出与迁移

Memos 高度重视数据可移植性，提供以下数据导出方式：

1. **纯文本导出**：Markdown 格式
2. **JSON 导出**：结构化数据备份
3. **数据库直接访问**：支持 SQL 查询导出

用户可以随时导出所有数据，不存在厂商锁定问题。

---

## 六、项目统计

| 指标 | 数值 |
|-----|------|
| GitHub Stars | 54.9k |
| Forks | 3.9k |
| Contributors | 361+ |
| Releases | 83+ |
| 最新版本 | v0.25.3 (2025年11月) |
| 主要语言 | Go (53.7%), TypeScript (44.5%) |
| 许可证 | MIT |

---

## 七、社区与支持

- **官网**：https://usememos.com
- **文档**：https://usememos.com/docs
- **在线演示**：https://demo.usememos.com/
- **Discord 社区**：https://discord.gg/tfPJa4UmAv
- **Docker Hub**：https://hub.docker.com/r/neosmemo/memos

---

## 八、贡献指南

Memos 是一个社区驱动的开源项目，欢迎各种形式的贡献：

- 报告 Bug 和功能建议
- 提交 Pull Request
- 改进文档和翻译
- 赞助项目发展

---

*文档版本：1.0*
*最后更新：2026年1月*
