# Open Agent SDK 项目介绍

## Why（为什么）

### 问题背景

在构建 AI 编程助手时，开发者面临一个核心挑战：如何将强大的 AI Agent 能力集成到自己的应用中，而不受限于官方工具的部署约束。Anthropic 官方提供的 `@anthropic-ai/claude-agent-sdk` 虽然功能强大，但存在一个根本性的架构限制——它需要通过启动本地 CLI 子进程来运行，这使得它在以下场景中难以使用：

- **云服务器部署**：需要在服务器环境中安装和配置 Claude Code CLI
- **Serverless 函数**：函数即服务（FaaS）环境无法运行持久化的 CLI 进程
- **Docker 容器**：需要将整个 CLI 打包进镜像，增加镜像体积和复杂度
- **CI/CD 流水线**：每次运行都需要初始化 CLI 环境，影响构建速度
- **嵌入式系统**：资源受限环境无法支持完整的 CLI 进程

### 项目动机

Open Agent SDK 由 [ShipAny](https://shipany.ai) 团队开发，旨在解决上述问题。项目作者意识到 Claude Code 本身是一个功能完善的 AI Agent 引擎，拥有完整的工具系统、权限管理、记忆管理和上下文压缩能力。如果能够将这些能力直接嵌入到应用程序中，而不需要依赖外部 CLI 进程，将大大简化 AI Agent 的集成难度。

### 目标用户

- 需要在服务器环境中运行 AI Agent 的后端开发者
- 希望将 AI 编程能力集成到现有应用中的软件团队
- 需要 Serverless AI Agent 能力的云原生开发者
- 希望在 CI/CD 流水线中自动化代码分析和修改的 DevOps 团队
- 构建 AI 原生应用的独立开发者

## What（是什么）

### 核心定义

Open Agent SDK 是一个开源的 Agent SDK，灵感来源于官方 `@anthropic-ai/claude-agent-sdk`，但采用了完全不同的架构设计。它将完整的 Claude Code Agent 引擎直接嵌入到应用程序进程中运行，无需任何外部依赖。

### 主要特性

#### 1. 进程内 Agent 循环

与官方 SDK 需要启动 CLI 子进程不同，Open Agent SDK 在进程内部直接运行完整的 Agent 循环：

```
// 官方 SDK 架构
Your code → SDK → spawn cli.js subprocess → stdin/stdout JSON → Anthropic API

// Open Agent SDK 架构
Your code → SDK → QueryEngine → Anthropic API (direct)
```

这种架构带来的优势：
- 零外部依赖安装
- 更低的延迟（无进程间通信开销）
- 更简单的部署流程
- 更小的资源占用

#### 2. 完整的工具集

SDK 内置了 26 个与 Claude Code 相同的工具。

#### 3. MCP 服务器集成

支持 Model Context Protocol（MCP），可以连接各种外部工具和服务。

#### 4. 多 Agent 系统

支持创建 Agent 团队，实现复杂的协作工作流。

#### 5. 权限控制系统

四层权限管道，确保 Agent 操作的安全性。

#### 6. 记忆系统

自动记忆系统，包含 4 种类型。

#### 7. 上下文压缩

当对话上下文超出限制时，自动进行压缩处理。

#### 8. 扩展思考能力

支持 Extended Thinking 配置。

## How（怎么做）

### 快速开始

#### 安装

```sh
npm install @shipany/open-agent-sdk
```

#### 配置环境变量

```sh
export ANTHROPIC_API_KEY=your-api-key
```

### 使用方式

#### 1. 单次查询

```typescript
import { query } from '@shipany/open-agent-sdk'

for await (const message of query({
  prompt: 'Find and fix the bug in auth.py',
  options: {
    allowedTools: ['Read', 'Edit', 'Bash'],
  },
})) {
  // 处理消息...
}
```

#### 2. 简单提示（阻塞式）

```typescript
import { createAgent } from '@shipany/open-agent-sdk'

const agent = createAgent({ model: 'claude-sonnet-4-6' })
const result = await agent.prompt('Read package.json and tell me the project name')
```

#### 3. 多轮对话

```typescript
const agent = createAgent({
  model: 'claude-sonnet-4-6',
  systemPrompt: 'You are a senior software engineer.',
})

const r1 = await agent.prompt('Read the main entry point')
const r2 = await agent.prompt('Now refactor the error handling')
```

#### 4. 自定义工具

```typescript
const weatherTool = {
  name: 'GetWeather',
  description: 'Get weather for a city',
  async call(input) {
    return { data: `Weather in ${input.city}: 22°C` }
  },
  // ... 其他必要属性
}

const agent = createAgent({
  tools: [...getAllBaseTools(), weatherTool],
})
```

#### 5. MCP 服务器集成

```typescript
const agent = createAgent({
  mcpServers: {
    filesystem: {
      command: 'npx',
      args: ['-y', '@modelcontextprotocol/server-filesystem', '/tmp'],
    },
  },
})
```

### 配置选项

| 选项 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `model` | `string` | `claude-sonnet-4-6` | Claude 模型 ID |
| `apiKey` | `string` | 环境变量 | API 密钥 |
| `baseURL` | `string` | Anthropic API | API 基础 URL |
| `cwd` | `string` | `process.cwd()` | 工作目录 |
| `systemPrompt` | `string` | - | 自定义系统提示 |
| `tools` | `Tool[]` | 所有内置工具 | 可用工具列表 |
| `allowedTools` | `string[]` | - | 工具白名单 |
| `permissionMode` | `string` | `bypassPermissions` | 权限模式 |
| `maxTurns` | `number` | `100` | 最大 Agent 轮次 |
| `maxBudgetUsd` | `number` | - | 最大 USD 消费 |
| `mcpServers` | `object` | - | MCP 服务器配置 |
| `agents` | `object` | - | 子 Agent 定义 |
| `hooks` | `object` | - | 生命周期钩子 |
| `thinking` | `object` | - | 扩展思考配置 |
| `env` | `object` | - | 环境变量 |

### 最佳实践

#### 1. 工具权限控制

对于需要严格控制的场景，使用只读权限。

#### 2. 成本控制

设置最大消费限制和轮次限制。

#### 3. 自定义系统提示

根据任务类型定制系统行为。

#### 4. 生命周期钩子

监控工具执行过程。

## 项目信息

- **npm 包**：[`@shipany/open-agent-sdk`](https://www.npmjs.com/package/@shipany/open-agent-sdk)
- **当前版本**：0.1.7
- **许可证**：MIT
- **Node.js 要求**：>=18.0.0

## 总结

Open Agent SDK 提供了一个强大的、零外部依赖的 AI Agent 解决方案。通过将完整的 Claude Code 引擎直接嵌入到应用程序中，它使开发者能够轻松地在任何环境中部署智能 Agent，无论是云服务器、Serverless 函数还是 CI/CD 流水线。