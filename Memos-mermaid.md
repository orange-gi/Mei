# Memos 架构图文档

## 一、系统整体架构图

### 1.1 逻辑架构

```mermaid
graph TB
    subgraph Client["客户端层"]
        Web["Web 浏览器"]
        Mobile["移动端"]
        API["第三方 API 客户端"]
    end

    subgraph Gateway["网关层"]
        Echo["Echo HTTP Server"]
        FileServer["文件服务器"]
        RSS["RSS 服务"]
    end

    subgraph APILayer["API 层"]
        REST["REST Gateway"]
        gRPC["gRPC Server"]
        ConnectRPC["ConnectRPC"]
        Auth["认证模块"]
    end

    subgraph ServiceLayer["业务服务层"]
        MemoSVC["Memo Service"]
        UserSVC["User Service"]
        AuthSVC["Auth Service"]
        AttachmentSVC["Attachment Service"]
        ActivitySVC["Activity Service"]
        ShortcutSVC["Shortcut Service"]
        IDPSVC["IDP Service"]
    end

    subgraph Plugin["插件层"]
        Markdown["Markdown 解析器"]
        Storage["存储插件 S3/Local"]
        Email["邮件通知"]
        Cron["定时任务"]
        Filter["过滤引擎"]
    end

    subgraph Store["数据存储层"]
        Cache["缓存层"]
        SQLite["SQLite"]
        MySQL["MySQL"]
        PostgreSQL["PostgreSQL"]
    end

    Client --> Gateway
    Gateway --> APILayer
    APILayer --> ServiceLayer
    ServiceLayer --> Plugin
    ServiceLayer --> Store

    Echo --> REST
    Echo --> gRPC
    REST --> ConnectRPC
    Auth -.-> APILayer

    ServiceLayer --> Cache
    Cache --> SQLite
    Cache --> MySQL
    Cache --> PostgreSQL
```

### 1.2 技术栈架构

```mermaid
graph LR
    subgraph Frontend["前端技术栈"]
        React["React 18"]
        TS["TypeScript"]
        CSS["CSS Modules"]
        i18n["i18next"]
    end

    subgraph Backend["后端技术栈"]
        Go["Go 1.25"]
        Echo["Echo v4"]
        gRPC["gRPC"]
        Viper["Viper"]
    end

    subgraph Database["数据库层"]
        SQLite["SQLite"]
        MySQL["MySQL"]
        PostgreSQL["PostgreSQL"]
    end

    subgraph Infrastructure["基础设施"]
        Docker["Docker"]
        K8s["Kubernetes"]
        S3["AWS S3"]
    end

    Frontend -->|"HTTP/HTTPS"| Backend
    Backend -->|"Go Driver"| Database
    Backend -->|"SDK"| S3
    Docker --> Backend
    K8s --> Docker
```

---

## 二、核心数据流图

### 2.1 备忘录创建流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant W as Web 前端
    participant E as Echo Server
    participant A as Auth
    participant M as Memo Service
    participant C as Cache
    participant D as Database
    participant P as Plugin/Markdown

    U->>W: 输入 Markdown 内容
    W->>P: 解析 Markdown
    P-->>W: 返回解析结果

    W->>E: POST /api/v1/memos
    E->>A: 验证 JWT Token
    A-->>E: Token 有效

    E->>M: CreateMemo(memo)

    M->>P: 提取标签、链接、任务列表
    P-->>M: 返回解析属性

    M->>D: 插入备忘录记录
    D-->>M: 返回插入结果

    M->>C: 更新缓存
    C-->>M: 缓存更新成功

    M-->>E: 返回 Memo 对象
    E-->>W: HTTP 201 Created
    W-->>U: 显示新创建的备忘录
```

### 2.2 备忘录查询流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant W as Web 前端
    participant E as Echo Server
    participant A as Auth
    participant M as Memo Service
    participant C as Cache
    participant D as Database
    participant F as Filter

    U->>W: 发送查询请求 (筛选条件)
    W->>E: GET /api/v1/memos?filter=...

    E->>A: 验证 JWT Token
    A-->>E: Token 有效

    E->>M: ListMemos(request)

    M->>F: 解析 CEL 过滤表达式
    F-->>M: 返回过滤逻辑

    alt 缓存命中
        C-->>M: 返回缓存结果
    else 缓存未命中
        M->>D: 执行带筛选的查询
        D-->>M: 返回原始结果
        M->>M: 应用过滤条件
        M->>C: 更新缓存
    end

    M->>M: 提取标签、计算属性

    M-->>E: 返回分页结果
    E-->>W: HTTP 200 OK
    W-->>U: 显示备忘录列表
```

### 2.3 文件上传流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant W as Web 前端
    participant E as Echo Server
    participant A as Auth
    participant F as FileServer
    participant S as Storage Plugin
    participant D as Database

    U->>W: 上传文件
    W->>E: POST /api/v1/attachments

    E->>A: 验证 JWT Token
    A-->>E: Token 有效

    E->>F: 处理文件上传

    alt 本地存储
        F->>F: 保存到本地文件系统
    else S3 存储
        F->>S: 上传到 S3
        S-->>F: 返回文件 URL
    end

    F->>D: 保存附件元数据
    D-->>F: 返回附件记录

    F-->>E: 返回 Attachment 对象
    E-->>W: HTTP 200 OK
    W-->>U: 显示上传结果
```

---

## 三、模块关系图

### 3.1 核心服务模块

```mermaid
classDiagram
    class Server {
        -Secret: string
        -Profile: *Profile
        -Store: *Store
        +Start(ctx context.Context) error
        +Shutdown(ctx context.Context)
        +StartBackgroundRunners(ctx context.Context)
    }

    class APIV1Service {
        -secret: string
        -profile: *Profile
        -store: *Store
        -MarkdownService
        +RegisterGateway(ctx, echoServer)
        +ConnectHandler() http.Handler
    }

    class Store {
        -profile: *Profile
        -driver: Driver
        -cacheConfig: Config
        -instanceSettingCache: *Cache
        -userCache: *Cache
        -userSettingCache: *Cache
        +GetDriver() Driver
        +Close() error
    }

    class DBDriver {
        <<interface>>
        +Close() error
    }

    class SQLite {
        +NewDB(profile) *DB
    }

    class MySQL {
        +NewDB(profile) *DB
    }

    class PostgreSQL {
        +NewDB(profile) *DB
    }

    Server --> Store : creates
    Store --> DBDriver : uses
    DBDriver <|-- SQLite
    DBDriver <|-- MySQL
    DBDriver <|-- PostgreSQL
    APIV1Service --> Store : uses
```

### 3.2 Memo 服务详细结构

```mermaid
classDiagram
    class MemoService {
        +CreateMemo(ctx, request) (*Memo, error)
        +ListMemos(ctx, request) (*ListMemosResponse, error)
        +GetMemo(ctx, request) (*Memo, error)
        +UpdateMemo(ctx, request) (*Memo, error)
        +DeleteMemo(ctx, request) error
    }

    class MemoConverter {
        +StoreToAPIMemo(storepb.Memo) *apiv1.Memo
        +APIToStoreMemo(apiv1.Memo) *storepb.Memo
    }

    class MemoFilter {
        +ParseFilterCEL(expression string) *cel.Env
        +ApplyFilter(memos []storepb.Memo, filter string) []storepb.Memo
    }

    class MemoRelationService {
        +SetMemoRelations(ctx, request) error
        +ListMemoRelations(ctx, request) (*ListMemoRelationsResponse, error)
    }

    class MemoAttachmentService {
        +SetMemoAttachments(ctx, request) error
        +ListMemoAttachments(ctx, request) (*ListMemoAttachmentsResponse, error)
    }

    class ReactionService {
        +UpsertMemoReaction(ctx, request) (*Reaction, error)
        +DeleteMemoReaction(ctx, request) error
        +ListMemoReactions(ctx, request) (*ListMemoReactionsResponse, error)
    }

    MemoService --> MemoConverter : uses
    MemoService --> MemoFilter : uses
    MemoService --> MemoRelationService : uses
    MemoService --> MemoAttachmentService : uses
    MemoService --> ReactionService : uses
```

---

## 四、用户交互流程图

### 4.1 典型用户操作流程

```mermaid
flowchart TD
    A[打开 Memos] --> B{已登录?}

    B -->|否| C[登录/注册页面]
    C --> D[本地账户登录]
    C --> E[OAuth 第三方登录]
    D --> F[验证凭据]
    E --> G[OAuth 回调]
    F --> H[生成 JWT Token]
    G --> H
    H --> I[进入主界面]

    B -->|是| I

    I --> J[备忘录列表]

    J --> K[新建备忘录]
    J --> L[搜索/筛选]
    J --> M[查看详情]

    K --> N[输入 Markdown 内容]
    N --> O{包含标签?}
    O -->|是| P[提取标签]
    O -->|否| Q[继续]
    P --> Q

    Q --> R{包含附件?}
    R -->|是| S[上传附件]
    R -->|否| T[保存]
    S --> T

    T --> U[渲染显示]
    U --> J

    L --> V[应用筛选条件]
    V --> W[显示筛选结果]
    W --> J

    M --> X[显示备忘录详情]
    X --> Y{需要编辑?}
    Y -->|是| Z[编辑模式]
    Y -->|否| J
    Z --> AA[更新内容]
    AA --> U
```

### 4.2 响应式设计流程

```mermaid
flowchart LR
    subgraph Desktop["桌面端"]
        D1[浏览器窗口]
        D2[宽屏布局]
    end

    subgraph Tablet["平板端"]
        T1[浏览器窗口]
        T2[中等布局]
    end

    subgraph Mobile["移动端"]
        M1[手机浏览器]
        M2[单列布局]
    end

    D1 -->|响应式 CSS| D2
    T1 -->|响应式 CSS| T2
    M1 -->|响应式 CSS| M2
```

---

## 五、部署架构图

### 5.1 单机部署架构

```mermaid
graph TB
    subgraph Server["服务器"]
        subgraph Docker["Docker 容器"]
            MemosApp["Memos 应用"]
            DataVolume["数据卷 ~/.memos"]
        end

        OS["Linux OS"]
        Network["网络栈"]
    end

    subgraph External["外部服务"]
        DNS["DNS 服务器"]
        Browser1["Chrome 浏览器"]
        Browser2["Safari 浏览器"]
    end

    Browser1 -->|HTTP/HTTPS| DNS
    Browser2 -->|HTTP/HTTPS| DNS
    DNS -->|路由| Network
    Network -->|端口 5230| Docker
    Docker --> MemosApp
    MemosApp --> DataVolume
    OS --> Docker
```

### 5.2 生产环境部署架构

```mermaid
graph TB
    subgraph Internet["互联网"]
        User1["用户"]
        User2["用户"]
        User3["用户"]
    end

    subgraph DMZ["DMZ 区域"]
        Nginx["Nginx 反向代理"]
        SSL["SSL/TLS 证书"]
    end

    subgraph App["应用网络"]
        Memos["Memos Docker"]
        Cron["定时任务容器"]
    end

    subgraph Data["数据网络"]
        PostgreSQL["PostgreSQL 主库"]
        Redis["Redis 缓存"]
        S3["S3 兼容存储"]
    end

    subgraph Backup["备份网络"]
        Backup["备份存储"]
    end

    User1 -->|HTTPS| Nginx
    User2 -->|HTTPS| Nginx
    User3 -->|HTTPS| Nginx

    Nginx -->|代理| SSL
    SSL -->|HTTP| Memos

    Memos -->|连接| PostgreSQL
    Memos -->|连接| Redis
    Memos -->|上传/下载| S3

    Memos -->|定时备份| Backup
    PostgreSQL -->|同步| Backup
```

### 5.3 Kubernetes 部署架构

```mermaid
graph TB
    subgraph K8s["Kubernetes 集群"]
        subgraph Ingress["Ingress"]
            IngressCtl["Ingress Controller"]
            TLS["TLS Termination"]
        end

        subgraph Deploy["Deployment"]
            MemosPod["Memos Pod"]
            Replica["副本集"]
        end

        subgraph Svc["Service"]
            LoadBalancer["LoadBalancer"]
            ClusterIP["ClusterIP"]
        end

        subgraph Data["持久化"]
            PV["PersistentVolume"]
            PVC["PersistentVolumeClaim"]
        end

        subgraph Config["配置"]
            ConfigMap["配置映射"]
            Secret["密钥管理"]
        end
    end

    External["外部用户"] --> Ingress
    Ingress --> LoadBalancer
    LoadBalancer --> MemosPod
    MemosPod --> ClusterIP
    MemosPod --> PV
    MemosPod --> ConfigMap
    MemosPod --> Secret
```

---

## 六、数据库模式图

### 6.1 核心表关系

```mermaid
erDiagram
    User ||--o{ Memo : creates
    User ||--o{ UserSetting : has
    User ||--o{ Reaction : makes
    User ||--o{ Activity : performs

    Memo ||--o{ Attachment : contains
    Memo ||--o{ MemoRelation : relates
    Memo ||--o{ Reaction : has
    Memo ||--o{ Tag : has

    UserSetting ||--o{ UserSettingValue : stores

    InstanceSetting ||--o{ InstanceSettingValue : stores

    IDP ||--o{ IDPUser : manages

    User {
        uuid id PK
        string username
        string email
        string avatar_url
        timestamp created_at
        timestamp updated_at
    }

    Memo {
        uuid id PK
        uuid creator_id FK
        string content
        string visibility
        bool pinned
        timestamp created_at
        timestamp updated_at
        timestamp display_time
    }

    Attachment {
        uuid id PK
        uuid memo_id FK
        string filename
        string uri
        string type
        int size
    }

    Tag {
        uuid id PK
        string name
        uuid creator_id FK
    }

    Reaction {
        uuid id PK
        uuid creator_id FK
        uuid memo_id FK
        string type
    }
```

---

## 七、安全架构图

### 7.1 安全防护层次

```mermaid
graph TB
    subgraph L7["应用层安全"]
        Auth["JWT 认证"]
        RBAC["RBAC 权限控制"]
        Input["输入验证"]
        CSRF["CSRF 保护"]
    end

    subgraph L4["传输层安全"]
        TLS["TLS 1.3"]
        HTTPS["HTTPS 强制"]
    end

    subgraph L3["基础设施安全"]
        RateLimit["速率限制"]
        Firewall["防火墙"]
        IDS["入侵检测"]
    end

    subgraph Data["数据安全"]
        Encryption["加密存储"]
        Backup["备份加密"]
        Audit["审计日志"]
    end

    L7 --> TLS
    TLS --> RateLimit
    RateLimit --> Encryption
```

---

*文档版本：1.0*
*最后更新：2026年1月*
