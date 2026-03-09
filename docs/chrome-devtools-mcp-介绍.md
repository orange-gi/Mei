# Chrome DevTools MCP 项目介绍

## 一、为什么做这个项目（Why）

### 1.1 项目背景与解决的问题

在现代软件开发中，AI编码助手（如Claude、Cursor、Copilot、Gemini等）已经成为开发者日常工作中不可或缺的工具。然而，这些AI助手在处理浏览器相关的任务时存在显著的局限性。传统的AI编码助手只能处理静态代码，无法直接与运行中的浏览器进行交互，这严重限制了它们在以下场景中的能力：

- **端到端测试自动化**：AI助手无法直接控制浏览器执行点击、表单填写等用户交互操作
- **性能问题诊断**：无法获取网站的真实运行时性能数据，如页面加载时间、渲染性能等
- **动态内容调试**：对于依赖JavaScript渲染的现代Web应用，传统方式难以获取准确的调试信息
- **网络请求分析**：无法检查和分析网站发出的网络请求，无法诊断API相关问题

**Chrome DevTools MCP** 项目的核心目标就是解决这些问题。该项目由Google Chrome DevTools团队开发，是一个Model Context Protocol（MCP）服务器，它将Chrome浏览器的全部功能以标准化的方式暴露给AI编码助手，使AI能够像人类开发者一样使用Chrome DevTools的强大功能。

### 1.2 现有方案的不足

在Chrome DevTools MCP出现之前，开发者尝试过多种方案来赋予AI浏览器控制能力，但都存在明显的局限性：

**方案一：使用Selenium或Playwright直接控制浏览器**

这种方案虽然能够实现浏览器自动化，但存在以下问题：

- 需要为每种编程语言编写特定的集成代码
- 缺乏标准化的接口，不同项目之间的代码难以复用
- 无法直接访问Chrome DevTools的高级功能（如性能分析、Lighthouse审计等）
- AI需要编写大量底层代码才能完成简单任务

**方案二：使用简单的HTTP请求模拟**

这种方法只能处理静态页面，无法应对现代Web应用的复杂交互：

- 无法处理JavaScript渲染的内容
- 无法获取运行时性能指标
- 无法检查控制台日志和错误信息
- 无法模拟真实的用户交互行为

**方案三：使用浏览器扩展**

虽然浏览器扩展可以访问Chrome DevTools协议，但存在部署困难，安全限制等问题，不适合与AI助手集成。

### 1.3 目标用户群体

Chrome DevTools MCP主要面向以下用户群体：

- **AI编码助手用户**：使用Claude、Cursor、Copilot、Codex等AI工具的开发者，希望AI能够执行浏览器自动化任务
- **自动化测试工程师**：需要AI辅助编写和维护端到端测试的工程师
- **性能优化专家**：希望利用AI分析网站性能问题的开发者
- **前端开发者**：需要AI辅助调试和检查Web应用的开发者

## 二、项目是什么（What）

### 2.1 项目概述

**Chrome DevTools MCP** 是Google Chrome DevTools团队官方开发的Model Context Protocol（MCP）服务器。它的核心功能是将Chrome浏览器的完整功能通过MCP协议暴露给AI编码助手，使AI能够直接控制和检查运行中的Chrome浏览器实例。

该项目采用TypeScript开发，基于Puppeteer实现浏览器控制，并利用Chrome DevTools协议提供丰富的调试和自动化能力。作为一个开源项目（Apache-2.0许可证），它得到了广泛的社区支持，并且已经与主流的AI编码助手实现了集成。

### 2.2 核心功能特性

#### 2.2.1 性能分析与洞察

Chrome DevTools MCP提供了强大的性能分析功能，能够帮助AI获取网站的真实性能数据：

- **性能跟踪记录**：使用Chrome DevTools记录页面的详细性能跟踪数据
- **性能洞察分析**：利用Chrome的性能洞察API提取可操作的性能改进建议
- **内存快照**：获取页面的内存堆快照，用于诊断内存泄漏问题
- **CrUX数据集成**：自动从Google Chrome用户体验报告（CrUX）获取真实用户性能数据

#### 2.2.2 高级浏览器调试

项目提供了全面的浏览器调试能力：

- **网络请求检查**：列出和分析所有网络请求，获取请求详情
- **控制台消息**：读取浏览器控制台日志，支持源映射的堆栈跟踪
- **页面快照**：获取页面的DOM快照，了解页面结构
- **脚本执行**：在页面上下文中执行JavaScript代码
- **Lighthouse审计**：运行Lighthouse审计，获取SEO、可访问性、性能等方面的评分

#### 2.2.3 可靠的自动化能力

基于Puppeteer实现的自动化功能确保了操作的可靠性：

- **智能等待机制**：自动等待操作结果，无需手动添加延迟
- **输入自动化**：支持点击、拖拽、悬停、键盘输入、表单填写等操作
- **文件上传**：支持文件上传到浏览器
- **对话框处理**：自动处理浏览器的各种对话框
- **多标签页管理**：支持创建、关闭、切换浏览器的多个标签页

#### 2.2.4 浏览器仿真功能

- **设备仿真**：模拟各种设备的视口和用户代理
- **页面调整**：调整页面大小以适应不同屏幕尺寸
- **地理位置仿真**：模拟不同的地理位置

### 2.3 工具分类概览

Chrome DevTools MCP提供了29个MCP工具，涵盖以下类别：

**输入自动化工具（9个）**

| 工具名称 | 功能描述 |
|---------|---------|
| click | 点击页面元素 |
| drag | 拖拽页面元素 |
| fill | 填写输入框 |
| fill_form | 填写表单 |
| handle_dialog | 处理对话框 |
| hover | 悬停在元素上 |
| press_key | 按下键盘按键 |
| type_text | 输入文本 |
| upload_file | 上传文件 |

**导航自动化工具（6个）**

| 工具名称 | 功能描述 |
|---------|---------|
| close_page | 关闭页面 |
| list_pages | 列出所有页面 |
| navigate_page | 导航到指定页面 |
| new_page | 创建新页面 |
| select_page | 选择指定页面 |
| wait_for | 等待条件满足 |

**仿真工具（2个）**

| 工具名称 | 功能描述 |
|---------|---------|
| emulate | 仿真设备配置 |
| resize_page | 调整页面大小 |

**性能工具（4个）**

| 工具名称 | 功能描述 |
|---------|---------|
| performance_analyze_insight | 分析性能洞察 |
| performance_start_trace | 开始性能跟踪 |
| performance_stop_trace | 停止性能跟踪 |
| take_memory_snapshot | 获取内存快照 |

**网络工具（2个）**

| 工具名称 | 功能描述 |
|---------|---------|
| get_network_request | 获取网络请求详情 |
| list_network_requests | 列出所有网络请求 |

**调试工具（6个）**

| 工具名称 | 功能描述 |
|---------|---------|
| evaluate_script | 执行JavaScript脚本 |
| get_console_message | 获取控制台消息 |
| lighthouse_audit | 运行Lighthouse审计 |
| list_console_messages | 列出控制台消息 |
| take_screenshot | 截图 |
| take_snapshot | 获取页面快照 |

### 2.4 技术栈概述

Chrome DevTools MCP采用了现代化的技术栈：

- **语言**：TypeScript（强类型支持，便于维护和调试）
- **运行时**：Node.js（支持v20.19+和v22.12+）
- **浏览器控制**：Puppeteer（Google官方的浏览器自动化库）
- **协议**：Model Context Protocol（MCP）标准协议
- **性能分析**：Chrome DevTools协议 + Lighthouse
- **构建工具**：Rollup（打包）+ TypeScript（编译）

### 2.5 与竞品的差异化

相比其他浏览器自动化方案，Chrome DevTools MCP具有以下独特优势：

- **官方支持**：由Google Chrome DevTools团队开发，确保与Chrome最新功能的兼容性
- **标准化协议**：基于MCP协议，与所有支持MCP的AI助手无缝集成
- **完整功能**：不仅支持基础自动化，还提供完整的DevTools功能
- **智能等待**：内置自动等待机制，无需手动处理异步问题
- **多客户端支持**：已与20+种主流AI编码助手完成集成

## 三、如何使用这个项目（How）

### 3.1 快速开始

#### 3.1.1 环境要求

在使用Chrome DevTools MCP之前，需要确保满足以下环境要求：

- **Node.js**：v20.19或更新的长期支持版本
- **Chrome浏览器**：当前稳定版或更新版本
- **npm**：用于安装和管理依赖

#### 3.1.2 基本配置

在MCP客户端配置文件中添加以下内容即可开始使用：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

#### 3.1.3 首次使用测试

配置完成后，可以向AI助手发送以下测试提示：

```
检查 https://developers.chrome.com 的性能表现
```

AI助手将自动启动浏览器，访问该页面，并执行性能跟踪分析。

### 3.2 高级配置选项

Chrome DevTools MCP提供了丰富的配置选项，可以根据具体需求进行定制：

#### 3.2.1 连接模式配置

**模式一：自动启动浏览器（默认）**

MCP服务器将自动启动一个新的Chrome浏览器实例：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest"]
    }
  }
}
```

**模式二：连接到已运行的Chrome实例**

通过远程调试端口连接到正在运行的Chrome浏览器：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": [
        "chrome-devtools-mcp@latest",
        "--browser-url=http://127.0.0.1:9222"
      ]
    }
  }
}
```

**模式三：自动连接（Chrome 144+）**

使用自动连接功能，无需手动启动Chrome：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["chrome-devtools-mcp@latest", "--autoConnect"]
    }
  }
}
```

#### 3.2.2 浏览器配置选项

**无头模式**

```json
{
  "args": ["-y", "chrome-devtools-mcp@latest", "--headless=true"]
}
```

**指定Chrome渠道**

```json
{
  "args": ["-y", "chrome-devtools-mcp@latest", "--channel=canary"]
}
```

**隔离模式（每次使用临时配置文件）**

```json
{
  "args": ["-y", "chrome-devtools-mcp@latest", "--isolated=true"]
}
```

**自定义Chrome可执行文件路径**

```json
{
  "args": ["-y", "chrome-devtools-mcp@latest", "--executable-path=/path/to/chrome"]
}
```

#### 3.2.3 功能模块配置

可以根据需要启用或禁用特定的功能类别：

- **仿真工具**：`--categoryEmulation=false`
- **性能工具**：`--categoryPerformance=false`
- **网络工具**：`--categoryNetwork=false`
- **扩展工具**：`--categoryExtensions=false`

#### 3.2.4 精简模式配置

如果只需要基本的浏览器功能，可以使用精简模式：

```json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["-y", "chrome-devtools-mcp@latest", "--slim", "--headless"]
    }
  }
}
```

精简模式只暴露3个核心工具：导航、脚本执行和截图。

### 3.3 主流AI编码助手集成指南

Chrome DevTools MCP已经与20多种主流AI编码助手完成了集成，以下是部分常用助手的配置指南：

#### 3.3.1 Claude Code配置

```bash
claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest
```

或者安装为插件（包含技能）：

```bash
/plugin marketplace add ChromeDevTools/chrome-devtools-mcp
/plugin install chrome-devtools-mcp
```

#### 3.3.2 Cursor配置

在Cursor设置中添加MCP服务器：

```
MCP Servers -> New MCP Server
```

然后填入标准配置即可。

#### 3.3.3 VS Code / Copilot配置

```bash
code --add-mcp '{"name":"io.github.ChromeDevTools/chrome-devtools-mcp","command":"npx","args":["-y","chrome-devtools-mcp"],"env":{}}'
```

#### 3.3.4 Gemini CLI配置

```bash
gemini mcp add chrome-devtools npx chrome-devtools-mcp@latest
```

### 3.4 常见使用场景示例

#### 3.4.1 性能测试场景

让AI分析网站性能：

```
分析 https://example.com 的页面加载性能，找出主要性能瓶颈
```

AI将执行以下操作：

1. 启动浏览器并导航到目标页面
2. 开始性能跟踪
3. 等待页面完全加载
4. 停止性能跟踪
5. 分析跟踪数据
6. 生成性能改进建议

#### 3.4.2 自动化测试场景

让AI执行端到端测试：

```
在 https://demo.example.com 上执行以下操作：
1. 点击"登录"按钮
2. 输入用户名 "testuser" 和密码 "password123"
3. 点击"提交"按钮
4. 验证登录成功
5. 截图保存登录后的页面状态
```

#### 3.4.3 调试场景

让AI帮助调试问题：

```
检查 https://example.com 的控制台错误，并分析可能导致的问题原因
```

### 3.5 最佳实践建议

#### 3.5.1 安全注意事项

- 避免在使用Chrome DevTools MCP时浏览敏感网站
- 可以在配置中使用`--isolated=true`创建临时浏览器配置文件
- 使用`--accept-insecure-certs`选项时要谨慎

#### 3.5.2 性能优化建议

- 不需要性能分析时，使用精简模式（`--slim`）减少资源消耗
- 合理使用`wait_for`工具，避免不必要的等待时间
- 完成任务后及时关闭浏览器会话

#### 3.5.3 故障排查

如果遇到问题，可以启用详细日志进行诊断：

```bash
DEBUG=* npx chrome-devtools-mcp@latest
```

或者使用日志文件：

```json
{
  "args": ["-y", "chrome-devtools-mcp@latest", "--log-file=/path/to/log.txt"]
}
```

详细的故障排查指南请参阅官方文档。

## 四、总结

Chrome DevTools MCP是一个开创性的项目，它将Chrome浏览器的全部功能以标准化的方式带给AI编码助手。通过这个项目，AI不再局限于处理静态代码，而是能够像人类开发者一样与真实的浏览器环境进行交互，执行复杂的自动化任务，进行深入的性能分析，以及进行全面的调试工作。

无论您是使用哪种AI编码助手，Chrome DevTools MCP都提供了一个统一、标准化的解决方案，使AI能够充分利用Chrome DevTools的强大功能。随着Web应用变得越来越复杂，这种能力对于AI辅助开发将变得越来越重要。
