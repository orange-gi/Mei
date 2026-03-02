# OpenSandbox 协议与配置分析

## 概述

OpenSandbox 是一个基础设施平台项目，而非传统的人工智能大语言模型项目，因此不存在类似系统提示词（System Prompt）或用户提示词（User Prompt）的概念。该项目的核心价值在于提供隔离的沙箱环境供代码执行，而非生成或处理自然语言内容。

然而，从广义上讲，OpenSandbox 中确实存在一些类似「提示词」的设计理念——即通过结构化的配置和协议来「指导」系统行为。这些包括：OpenAPI 规范定义了客户端与服务器的交互方式，execd API 规范定义了沙箱内部执行引擎的行为，配置文件定义了沙箱实例的运行环境，以及各种 SDK 提供的编程接口设计模式。本文档将从协议设计、API 规范和配置模式三个角度，分析 OpenSandbox 中这些「类提示词」元素的设计理念。

## OpenAPI 规范分析

### Sandbox Lifecycle API 规范

Sandbox Lifecycle API（定义于 `specs/sandbox-lifecycle.yml`）是 OpenSandbox 平台的核心协议之一，负责管理沙箱实例的整个生命周期。该规范采用 OpenAPI 3.0 标准定义，提供了完整的沙箱创建、查询、更新和销毁接口。

**规范位置**：`specs/sandbox-lifecycle.yml`

**核心端点设计**：

```yaml
paths:
  /sandboxes:
    post:
      summary: Create a new sandbox
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                image:
                  type: string
                  description: 容器镜像地址
                entrypoint:
                  type: array
                  items:
                    type: string
                env:
                  type: object
                  additionalProperties:
                    type: string
                timeout:
                  type: string
                  description: 沙箱存活时间
                resources:
                  type: object
                  properties:
                    cpu:
                      type: number
                    memory:
                      type: string
                    gpu:
                      type: number
```

这种设计模式将沙箱的创建参数以结构化方式定义，开发者通过提供镜像地址、入口命令、环境变量和资源限制等配置来「指导」系统创建符合需求的沙箱实例。这与提示词工程中通过精确描述来引导模型行为的理念有异曲同工之妙——都是通过明确的结构化指令来获得预期的系统响应。

### Sandbox Execution API 规范

Sandbox Execution API（定义于 `specs/execd-api.yaml`）是另一个核心协议，它定义了与运行中沙箱实例交互的所有接口。该规范同样采用 OpenAPI 3.0 标准，包含代码解释、命令执行、文件操作和指标监控四大类接口。

**规范位置**：`specs/execd-api.yaml`

**API 分类结构**：

```yaml
paths:
  # 代码解释相关
  /code/context:
    post:
      summary: Create execution context
  /code:
    post:
      summary: Execute code
    delete:
      summary: Interrupt execution

  # 命令执行相关
  /command:
    post:
      summary: Execute shell command
    delete:
      summary: Interrupt command

  # 文件操作相关
  /files/upload:
    post:
      summary: Upload files
  /files/download:
    get:
      summary: Download files
  /files/search:
    get:
      summary: Search files by glob

  # 指标监控相关
  /metrics:
    get:
      summary: Get metrics snapshot
  /metrics/watch:
    get:
      summary: Stream metrics via SSE
```

这种统一的 API 设计模式使得不同编程语言的 SDK 能够以一致的方式与沙箱交互。每个 API 端点都有明确的输入输出定义，开发者可以精确地「指示」系统执行特定操作。

## 配置模式分析

### 沙箱配置文件结构

OpenSandbox 使用 TOML 格式的配置文件（默认路径 `~/.sandbox.toml`）来定义沙箱服务器的运行参数。这种配置方式允许用户以声明式的方法「提示」系统应该如何运行。

**示例配置结构**：

```toml
[server]
host = "0.0.0.0"
port = 8080
api_key = "your-api-key-here"

[runtime]
type = "docker"  # 或 "kubernetes"

[runtime.docker]
network_mode = "bridge"
private_registry_auth = true

[defaults]
default_timeout = "10m"
default_image = "opensandbox/code-interpreter:v1.0.1"
```

配置文件的每个部分都对应着系统的一个方面，开发者通过设置这些参数来「指导」平台的默认行为。这种声明式配置的方式与提示词工程中设置系统上下文的理念相似——都是通过明确的声明来建立执行环境。

### 环境变量注入模式

在创建沙箱实例时，可以通过 `env` 参数注入环境变量，这为沙箱运行时提供了灵活的「提示」机制：

```python
sandbox = await Sandbox.create(
    "opensandbox/code-interpreter:v1.0.1",
    entrypoint=["/opt/opensandbox/code-interpreter.sh"],
    env={
        "PYTHON_VERSION": "3.11",
        "INSTALL_DEPS": "pip install numpy pandas",
        "WORKSPACE": "/tmp/workspace"
    },
    timeout=timedelta(minutes=10),
)
```

这种环境变量注入机制允许开发者在沙箱创建时「预设」特定的运行环境上下文，类似于提示词中的前置条件设置。

## SDK 接口设计模式

### Python SDK 的链式调用设计

OpenSandbox 的 Python SDK 采用了链式调用的设计模式，使得代码表达流畅自然。这种设计可以被视为一种「编程式提示词」——通过方法调用链来明确表达操作意图。

**典型使用模式**：

```python
# 链式调用示例
result = await (
    await CodeInterpreter.create(sandbox)
).codes.run(
    "import sys; print(sys.version)",
    language=SupportedLanguage.PYTHON
)

# 另一种写法
interpreter = await CodeInterpreter.create(sandbox)
result = await interpreter.codes.run(code, language=SupportedLanguage.PYTHON)
```

这种设计模式将操作意图清晰地表达为一系列方法调用，每个方法都对应一个具体的动作，通过这种结构化的方式「指导」系统逐步完成复杂任务。

### 异步编程模型

OpenSandbox 的 SDK 全面采用异步编程模型（async/await），这不仅提高了性能，也使得复杂的异步操作序列能够以同步代码的简洁形式表达：

```python
async def main():
    async with sandbox:
        # 并行执行多个操作
        results = await asyncio.gather(
            sandbox.commands.run("echo 'task1'"),
            sandbox.commands.run("echo 'task2'"),
            sandbox.files.read_file("/tmp/data.txt")
        )
```

异步模型使得开发者可以「提示」系统并行执行多个独立任务，系统则负责优化执行顺序和时间安排。

## 代码解释器协议设计

### Jupyter Kernel 集成模式

OpenSandbox 的代码解释器功能建立在 Jupyter 协议之上，这是一种结构化的代码执行「协议」。与自然语言提示词不同，Jupyter 协议定义了严格的消息格式来「指导」内核执行代码：

**代码执行请求结构**：

```json
{
    "header": {
        "msg_id": "unique-message-id",
        "msg_type": "execute_request"
    },
    "content": {
        "code": "print('Hello World')",
        "silent": false,
        "store_history": true,
        "user_variables": [],
        "allow_stdin": true
    }
}
```

**执行响应结构**：

```json
{
    "header": {
        "msg_type": "execute_reply"
    },
    "content": {
        "status": "ok",
        "execution_count": 1,
        "user_expressions": {},
        "payload": []
    }
}
```

这种结构化的协议设计确保了代码执行的确定性，每一次「提示」（代码提交）都会得到可预测的响应。这与自然语言提示词工程追求的目标一致——通过明确的输入获得期望的输出。

### 多语言支持设计

OpenSandbox 通过统一的接口支持多种编程语言，这种设计模式可以被理解为「语言无关的提示词抽象」：

```python
# Python 代码执行
result = await interpreter.codes.run(
    "result = 2 + 2",
    language=SupportedLanguage.PYTHON
)

# JavaScript 代码执行
result = await interpreter.codes.run(
    "const result = 2 + 2; result",
    language=SupportedLanguage.JAVASCRIPT
)

# Java 代码执行
result = await interpreter.codes.run(
    "public class Main { public static void main(String[] args) { System.out.println(2+2); } }",
    language=SupportedLanguage.JAVA
)
```

相同的接口设计，不同的语言支持，这种抽象模式使得开发者可以用统一的「提示词风格」来操作不同语言的代码执行环境。

## 网络策略配置模式

### Ingress 路由策略

OpenSandbox 的 Ingress Gateway 提供了多种路由策略，这些策略可以视为「网络请求的提示词」——告诉系统如何路由和转发请求：

```yaml
# 路由策略配置示例
routing:
  strategies:
    - name: "path-based"
      rules:
        - path: "/sandboxes/{id}/port/{port}"
          target: "sandbox-port-forwarding"
    - name: "header-based"
      rules:
        - header: "X-Sandbox-Id"
          target: "specific-sandbox"
    - name: "weight-based"
      rules:
        - backend: "sandbox-v1"
          weight: 90
        - backend: "sandbox-v2"
          weight: 10
```

### Egress 出口控制策略

出口控制配置定义了沙箱可以访问的外部网络资源，这是一种「安全边界的提示词」：

```yaml
# 出口控制配置示例
egress:
  policy:
    - name: "allow-internal"
      destination: "10.0.0.0/8"
      action: "allow"
    - name: "allow-https"
      destination: "0.0.0.0/0"
      ports: [443]
      action: "allow"
    - name: "deny-all"
      destination: "0.0.0.0/0"
      action: "deny"
```

这种声明式的网络策略配置让运维人员可以精确地「指示」沙箱的网络访问行为，实现安全隔离的同时保持必要的连通性。

## 设计模式总结

通过以上分析，我们可以看到 OpenSandbox 中虽然没有传统意义上的「提示词」，但其设计理念与提示词工程有着共通之处：

**声明式 vs 命令式**：OpenSandbox 大量采用声明式配置（配置文件、环境变量、策略规则）来「提示」系统行为，这比直接编写命令式代码更加声明式和可预测。

**结构化输入**：所有的交互都通过结构化的数据格式（JSON、YAML、TOML）进行，这与提示词工程中使用结构化提示词来获得更准确响应的做法一致。

**抽象层次**：SDK 提供了高级抽象，使得开发者无需关心底层实现细节，只需表达「做什么」而非「怎么做」。

**可预测性**：通过严格的协议定义和类型约束，确保每一次「提示」都会得到可预测的响应，这与提示词工程追求的可控性目标相同。

## 优化建议

尽管 OpenSandbox 的协议设计已经相当完善，以下几点可能有助于进一步提升开发体验：

**配置验证增强**：当前配置文件主要依赖运行时验证，可以增加更完善的 schema 验证和 IDE 支持，提供配置项的自动补全和类型检查提示。

**SDK 方法链优化**：考虑为常用操作序列提供更简洁的封装，例如提供 `sandbox.run_code(language, code)` 这样的快捷方法。

**错误消息改进**：当前某些错误消息较为技术化，可以增加面向开发者的友好提示，帮助快速定位和解决问题。

**文档示例丰富**：增加更多场景的完整示例，包括错误处理、最佳实践和性能优化建议。

## 结论

OpenSandbox 作为一个专注于代码执行沙箱的平台，虽然不涉及自然语言处理领域的提示词，但其设计理念与提示词工程有着深层的共鸣。通过 OpenAPI 规范、配置文件、SDK 接口和网络策略等多种机制，开发者可以精确地「指导」系统行为，获得可预测的结果。这种声明式、结构化的设计哲学值得其他系统借鉴和学习。
