# OpenSandbox 项目介绍

## 为什么需要 OpenSandbox（Why）

在人工智能快速发展的今天，AI 应用程序面临着越来越多的安全隔离和代码执行需求。传统的开发模式中，开发者可以直接在本地环境运行代码，但对于 AI 应用场景，特别是涉及 AI Agent、代码生成、自动化测试等用例时，面临着严峻的挑战。

首先，AI 生成代码的安全性是一个核心问题。AI 模型生成的代码可能包含恶意指令或存在安全漏洞，如果在宿主环境中直接执行，可能会对系统造成不可逆的损害。传统的虚拟机方案虽然提供了隔离，但启动速度慢、资源占用高，难以满足 AI 应用对实时性的要求。其次，不同的 AI 框架和工具对运行环境有着各自独特的需求。Claude Code 需要完整的开发环境，Playwright 需要浏览器自动化能力，而机器学习训练则需要 GPU 资源支持。缺乏一个统一、可扩展的解决方案会导致运维复杂度急剧上升。再者，企业级应用需要完善的生命周期管理，包括沙箱的创建、监控、暂停、恢复和销毁等操作，同时还需要支持大规模分布式调度和多租户隔离。这些功能如果从零开始开发，将耗费大量的时间和资源。

OpenSandbox 正是为了解决这些痛点而诞生的。它提供了一个通用的高性能沙箱平台，让 AI 应用能够安全、可控地执行代码和管理文件。

## OpenSandbox 是什么（What）

OpenSandbox 是阿里巴巴开源的通用人工智能沙箱平台，专为 AI 应用场景设计。该项目提供了多语言 SDK、统一的沙箱 API 规范以及灵活的运行时支持，覆盖了编程 Agent、GUI Agent、Agent 评估、AI 代码执行和强化学习训练等多种典型应用场景。

从技术架构上看，OpenSandbox 采用四层架构设计。最顶层是 SDK 层，提供 Python、Java、Kotlin、JavaScript、TypeScript、C# 等多种语言的客户端库，开发者可以通过这些 SDK 方便地与沙箱进行交互。第二层是规范层，定义了 Sandbox Lifecycle API 和 Sandbox Execution API 两套 OpenAPI 规范，明确了沙箱生命周期管理和沙箱内部执行的协议接口。第三层是运行时层，基于 Python FastAPI 实现的服务器负责沙箱实例的 orchestration，支持 Docker 和 Kubernetes 两种运行方式。最后是沙箱实例层，每个运行中的容器都注入了 execd 守护进程，实现了代码解释器、命令执行和文件操作等核心功能。

在核心特性方面，OpenSandbox 具备多项显著优势。多语言 SDK 支持使得开发者可以使用自己熟悉的编程语言与沙箱交互。内置的沙箱环境覆盖了命令行、文件系统和代码解释器等基础能力，示例代码涵盖了 Claude Code、Chrome 浏览器自动化、Playwright 网页抓取以及 VNC 远程桌面等多种场景。网络策略方面，平台提供了统一的 Ingress Gateway 支持多种路由策略，同时每个沙箱都可以配置独立的出口网络控制。在协议设计上，OpenSandbox 采用了协议优先的理念，所有交互都通过 OpenAPI 规范定义，支持多语言实现和自定义运行时扩展。安全性方面，平台提供 API 密钥认证、Token 认证、沙箱隔离环境和资源配额限制等多重保护机制。

## 如何使用 OpenSandbox（How）

### 快速开始

使用 OpenSandbox 的第一步是安装和配置沙箱服务器。确保本地环境已安装 Docker（用于本地执行）和 Python 3.10 或更高版本（推荐）。通过 pip 安装 opensandbox-server 包后，使用 init-config 命令生成配置文件：

```bash
uv pip install opensandbox-server
opensandbox-server init-config ~/.sandbox.toml --example docker
```

启动沙箱服务器非常简单，直接运行 opensandbox-server 命令即可。如果需要查看帮助信息，可以加上 -h 参数。服务器启动后，会监听相应的端口并等待客户端请求。

接下来是安装 Code Interpreter SDK 并创建沙箱执行代码。通过 pip 安装 opensandbox-code-interpreter 包后，开发者可以使用 Python API 创建沙箱实例并执行代码。以下是一个完整的示例流程：首先创建沙箱，然后执行 Shell 命令，接着写入和读取文件，最后创建代码解释器并执行 Python 代码：

```python
import asyncio
from datetime import timedelta

from code_interpreter import CodeInterpreter, SupportedLanguage
from opensandbox import Sandbox
from opensandbox.models import WriteEntry

async def main() -> None:
    # 创建沙箱实例
    sandbox = await Sandbox.create(
        "opensandbox/code-interpreter:v1.0.1",
        entrypoint=["/opt/opensandbox/code-interpreter.sh"],
        env={"PYTHON_VERSION": "3.11"},
        timeout=timedelta(minutes=10),
    )

    async with sandbox:
        # 执行 Shell 命令
        execution = await sandbox.commands.run("echo 'Hello OpenSandbox!'")
        print(execution.logs.stdout[0].text)

        # 写入文件
        await sandbox.files.write_files([
            WriteEntry(path="/tmp/hello.txt", data="Hello World", mode=644)
        ])

        # 读取文件
        content = await sandbox.files.read_file("/tmp/hello.txt")
        print(f"Content: {content}")

        # 创建代码解释器
        interpreter = await CodeInterpreter.create(sandbox)

        # 执行 Python 代码
        result = await interpreter.codes.run(
            "import sys; print(sys.version); result = 2 + 2; result",
            language=SupportedLanguage.PYTHON,
        )
        print(result.result[0].text)

    # 清理沙箱
    await sandbox.kill()

if __name__ == "__main__":
    asyncio.run(main())
```

### 核心概念理解

理解 OpenSandbox 的几个核心概念有助于更好地使用该平台。Sandbox 类是管理沙箱生命周期的主要入口，提供了创建（create）、监控状态（manage）、销毁（kill）等方法，支持异步操作、资源配额管理、TTL 自动过期等高级功能。Filesystem 组件提供了完整的文件系统操作能力，包括文件的增删改查、批量上传下载、Glob 模式搜索、权限管理等。Commands 组件支持在沙箱内执行 Shell 命令，支持前台同步执行和后台分离执行两种模式，可以通过 SSE 实时获取输出流。CodeInterpreter 组件则提供了有状态的代码解释能力，基于 Jupyter 协议实现，支持 Python、Java、JavaScript、TypeScript、Go、Bash 等多种编程语言。

### 部署架构选择

OpenSandbox 支持两种主要的部署模式。Docker 运行时适合本地开发和测试环境，直接与 Docker API 集成，支持主机模式和桥接模式两种网络配置，可以快速启动和销毁沙箱实例。Kubernetes 运行时则适合大规模生产环境，支持 Pod 级别的沙箱实例管理，原生支持 Kubernetes 资源管理、多租户隔离和水平扩展，项目中包含了完整的 Kubernetes 部署配置和示例。

### 典型应用场景

OpenSandbox 的典型应用场景非常广泛。在 AI 代码生成与执行方面，平台可以作为 AI 模型的代码执行后端，让 Claude、GPT-4、 Gemini 等模型生成的代码在隔离环境中安全执行，同时支持多种编程语言和跨次执行的状态维护。交互式编码环境方面，可以基于 OpenSandbox 构建 Web 版编程平台和 Jupyter Notebook，提供代码执行、文件管理、终端访问等完整功能。浏览器自动化与测试方面，平台支持 Chrome 无头浏览器和 Playwright，可用于网页抓取、自动化测试和远程调试。远程开发环境方面，可以提供基于 VS Code Server 的云端开发工作空间，或使用 VNC 构建完整桌面环境。持续集成与测试方面，沙箱可用于运行构建和测试流水线，每个构建都在隔离的容器环境中进行，确保可重复性和并行执行能力。

## 总结

OpenSandbox 为构建 AI 驱动的应用程序提供了一个完整、生产级的基础设施平台。其核心优势体现在以下几个方面：通用性方面，平台可以与任何容器镜像配合使用，不依赖特定的基础镜像；可扩展性方面，插件式运行时架构支持自定义实现，多语言 SDK 满足不同开发团队的需求；开发者友好方面，一致的 API 设计和丰富的示例代码降低了学习成本；生产级特性方面，健壮的生命周期管理、可观测性和安全隔离机制确保了系统的可靠性。无论是构建 AI 编码助手、交互式笔记本还是远程开发环境，OpenSandbox 都能提供坚实的技术基础。
