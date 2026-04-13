# AI 智能卡路里计算器 - 项目介绍

## 为什么需要这个项目 (Why)

### 问题背景

在现代社会，越来越多的人开始关注饮食健康和体重管理。然而，传统的手动记录卡路里方式存在诸多痛点：

1. **操作繁琐**：需要手动搜索每种食物的卡路里数据，逐一记录，工作量巨大
2. **估算不准确**：用户往往难以准确估计食物的分量和热量，导致数据偏差
3. **缺乏便捷性**：市面上的卡路里记录应用要求用户输入文字或选择食物，难以坚持使用
4. **特殊场景识别困难**：对于复杂菜品（如沙拉、火锅、混合饮品等），手动记录更加困难

### 项目动机

`Calorie-Calculator` 项目的诞生正是为了解决上述问题。通过利用人工智能图像识别技术，用户只需上传食物照片，系统即可自动识别食物种类并计算总热量，让卡路里追踪变得简单、直观、高效。

### 目标用户

- 健身爱好者和减重人群：需要精确控制每日热量摄入
- 饮食管理需求者：关注健康饮食的普通用户
- 营养师和健康顾问：辅助为客户制定饮食计划
- 对新技术感兴趣的用户：体验 AI 在生活中的实际应用

## 这个项目是什么 (What)

### 核心功能

**AI 智能食物识别与卡路里计算**

用户上传食物图片，系统利用 Google Gemini AI 进行图像分析，自动识别图片中的食物项目，并返回每种食物的名称以及总热量数值。

### 技术特性

| 特性 | 说明 |
|------|------|
| **AI 图像识别** | 使用 Google Gemini 2.5 Pro 模型进行食物识别 |
| **实时处理** | 图片上传后即时返回识别结果 |
| **JSON 数据格式** | 返回结构化的 JSON 数据，便于后续处理 |
| **免费使用** | 提供在线免费工具，无使用门槛 |
| **响应式设计** | 支持桌面端和移动端访问 |

### 技术栈概览

```
前端框架：Next.js (React)
UI 样式：Tailwind CSS + DaisyUI
AI 服务：Google Gemini API
部署平台：支持 Vercel、Cloudflare 等
```

### 与竞品的差异化

1. **无应用安装**：基于 Web，无需下载安装任何应用
2. **图片直传识别**：直接上传食物照片，无需手动选择食物类型
3. **智能 JSON 输出**：返回结构化数据，便于开发者二次集成

## 如何使用这个项目 (How)

### 快速开始

#### 1. 环境配置

在项目根目录创建 `.env` 文件，配置必要的环境变量：

```bash
# Google 分析追踪 ID（可选）
NEXT_PUBLIC_GOOGLE_ANALYTICS_ID="your_google_analytics_id_here"

# Google Gemini API 密钥（必须）
NEXT_PUBLIC_GOOGLE_AI_API_KEY="your_google_ai_api_key_here"
```

**获取 Google Gemini API Key 的步骤：**

1. 访问 [Google AI Studio](https://makersuite.google.com/app/apikey)
2. 登录 Google 账号
3. 点击 "Create API Key" 生成新的 API 密钥
4. 将生成的密钥填入上述配置文件

#### 2. 本地运行

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

应用将在 `http://localhost:3000` 运行。

#### 3. 生产部署

```bash
# 构建生产版本
npm run build

# 启动生产服务器
npm start
```

### 核心使用流程

```
┌─────────────────────────────────────────────────────────┐
│                     使用流程图                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   [上传食物图片] → [AI 图像识别] → [返回卡路里数据]     │
│         ↓                ↓                ↓             │
│   支持 JPG/PNG    Gemini 2.5 Pro   JSON 格式结果        │
│   最大 100MB      多食物识别      总热量 + 食物列表     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### API 调用说明

项目提供了 `/api/detect_food` 接口，可供其他应用调用：

**请求示例：**

```javascript
fetch('/api/detect_food', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    image: 'data:image/jpeg;base64,/9j/4AAQSkZJRg...' // Base64 编码的图片
  }),
})
```

**响应示例：**

```json
{
  "success": true,
  "items": ["苹果", "香蕉", "面包"],
  "count": 285
}
```

### 最佳实践建议

1. **图片质量**：上传清晰、光线充足的食物照片可提高识别准确率
2. **单一菜品**：尽量上传单一菜品的照片，多个复杂菜品可能影响识别精度
3. **API 限流**：Google Gemini API 有每分钟 2 次请求的限制，请合理使用
4. **错误处理**：建议在生产环境中实现重试机制，以防网络波动导致请求失败

### 项目结构

```
Calorie-Calculator/
├── src/
│   ├── components/           # React 组件
│   │   ├── calorie-calculator.js  # 主计算器组件
│   │   ├── analyse.js        # Google Analytics
│   │   ├── haeder.js         # 页头导航
│   │   ├── footer.js          # 页脚
│   │   ├── spinner.js         # 加载动画
│   │   ├── layout.js          # 布局组件
│   │   └── model/
│   │       └── alert.js       # 警告弹窗
│   ├── pages/                 # Next.js 页面
│   │   ├── api/
│   │   │   └── detect_food.js # 食物识别 API
│   │   ├── _app.js
│   │   ├── _document.js
│   │   └── index.js
│   └── styles/               # 全局样式
├── public/                   # 静态资源
├── package.json
├── tailwind.config.js        # Tailwind 配置
└── next.config.mjs           # Next.js 配置
```

### 注意事项

1. **API 密钥安全**：不要将包含 API 密钥的 `.env` 文件提交到版本控制系统
2. **隐私保护**：用户上传的图片仅用于当次识别，不会被存储或用于其他目的
3. **识别局限性**：AI 识别结果为估算值，实际热量可能因食材、品牌、制作方式有所差异
