# Remotion 项目介绍

## Why（为什么）

### 解决的问题

在传统的视频制作流程中，创作者通常需要使用像 Adobe Premiere Pro、After Effects 这样的专业视频编辑软件。这些工具虽然功能强大，但存在以下局限性：

1. **手动操作效率低**：每一帧、每一个特效都需要手动调整，无法实现真正的程序化自动化
2. **数据集成困难**：难以将外部数据（如 API 数据、数据库内容）动态融入视频
3. **版本控制不便**：视频文件的二进制特性使得难以进行版本控制和协作
4. **批量生成受限**：无法高效地批量生成个性化视频（如个性化营销视频、数据驱动报告）
5. **技术门槛高**：传统视频编辑需要专业技能，程序员无法利用其编程能力

### 项目背景和动机

Remotion 的诞生源于一个核心洞察：**程序员已经掌握了强大的工具（React、Web 技术栈），为什么不能利用这些工具来创建视频？**

传统的视频制作流程中：
- 设计师使用图形界面手动操作
- 程序员编写代码处理逻辑和数据
- 两者之间存在巨大鸿沟

Remotion 弥合了这个鸿沟，让程序员能够：
- 使用熟悉的 React 组件构建视频
- 利用编程能力实现复杂逻辑
- 将数据 API 直接集成到视频中
- 实现视频的版本控制和自动化

### 目标用户

1. **前端/全栈开发者**：熟悉 React，希望用代码创建视频
2. **数据分析师**：需要程序化生成数据可视化视频
3. **营销团队**：批量生成个性化营销内容
4. **内容创作者**：制作代码驱动的动画和视频内容
5. **产品团队**：创建产品演示、年度报告等

## What（是什么）

### 项目定位

Remotion 是一个**使用 React 程序化创建视频的框架**。它将 React 的组件化思想与视频时间轴相结合，允许开发者通过编写 JavaScript/React 代码来创建、编辑和渲染视频。

### 核心功能

#### 1. 程序化视频创建
- 使用 React 组件构建视频的每一帧
- 通过 `durationInFrames` 控制持续时间
- 支持 CSS、Canvas、SVG、WebGL 等所有 Web 技术

#### 2. 参数化视频渲染
- 支持传入 props 实现动态视频
- `calculateMetadata()` API 用于预计算和动态调整
- 从 API 获取数据并实时显示在视频中

#### 3. 交互式开发环境（Remotion Studio）
- 浏览器中的视频预览
- 拖动时间轴调整每一帧
- **交互式 Props 编辑**：使用 Zod 定义类型安全的 props
- **渲染按钮**：图形界面触发渲染任务
- 实时预览和快速刷新

#### 4. 多种渲染方式
- **本地渲染**：Node.js 环境本地渲染
- **Lambda 渲染**：AWS Lambda 无服务器渲染
- **Cloud Run 渲染**：Google Cloud Run 渲染
- **静态图像生成**：支持 PNG、JPEG、WebP、PDF 格式

#### 5. 音视频处理
- 完整的音频支持
- `<OffthreadVideo>` 组件优化视频处理
- WebCodecs API 支持
- 字幕生成和编辑

#### 6. 丰富的生态系统
- `@remotion/rive`：Rive 动画支持
- `@remotion/shapes`：几何形状组件
- `@remotion/tailwind`：Tailwind CSS 集成
- `@remotion/player`：独立的视频播放器组件

### 技术栈概述

| 类别 | 技术选择 |
|------|----------|
| **主语言** | TypeScript (77.6%) |
| **UI 框架** | React |
| **渲染引擎** | Puppeteer / Chrome |
| **多媒体处理** | FFmpeg（内置 Rust 优化版） |
| **云服务** | AWS Lambda、Google Cloud Run |
| **样式方案** | CSS, Tailwind CSS |
| **动画库** | Framer Motion, Rive |

### 与竞品的差异化

| 特性 | Remotion | 传统视频软件 | 其他方案 |
|------|----------|--------------|----------|
| **创建方式** | 代码驱动 | 图形界面 | 模板驱动 |
| **数据集成** | 原生支持 | 困难 | 有限 |
| **版本控制** | Git 友好 | 不支持 | 有限 |
| **自动化** | 完全支持 | 手动操作 | 部分支持 |
| **学习曲线** | 需要编程基础 | 需要专业技能 | 中等 |
| **批量生成** | 高效 | 低效 | 中等 |
| **协作方式** | 代码协作 | 文件共享 | 混合 |

## How（怎么做）

### 快速开始

#### 环境要求
- **Node.js**: 版本 16 或以上
- **包管理器**: npm / pnpm / yarn / bun

#### 创建项目

```bash
# 使用 npm
npx create-video@latest

# 使用 pnpm
pnpm create video

# 使用 yarn
yarn create video

# 使用 bun
bun create video
```

#### 选择模板
- **Hello World**：适合初学者的基础模板
- **Blank**：空白项目，从零开始
- **Stargazer**：GitHub Star 里程碑庆祝模板
- **Text-to-speech**：Google TTS 语音合成模板

#### 启动开发环境

```bash
npm run dev
```

这将启动 Remotion Studio，提供交互式的视频编辑界面。

### 核心使用方式

#### 1. 创建 Composition

```tsx
import { Composition } from 'remotion';

export const MyVideo = () => {
  return (
    <Composition
      id="MyVideo"
      component={MyComponent}
      durationInFrames={300}
      fps={30}
      width={1920}
      height={1080}
    />
  );
};
```

#### 2. 编写视频组件

```tsx
const MyComponent = ({ title }) => {
  return (
    <div style={{ background: 'black', flex: 1 }}>
      <h1 style={{ color: 'white', fontSize: 80 }}>
        {title}
      </h1>
    </div>
  );
};
```

#### 3. 使用动画

```tsx
import { useSpring, animate } from 'remotion';

const AnimatedTitle = ({ text }) => {
  const opacity = useSpring(0, 1);

  return (
    <animated.div style={{ opacity }}>
      {text}
    </animated.div>
  );
};
```

#### 4. 数据驱动视频

```tsx
import { Composition } from 'remotion';

export const DataVideo = () => {
  return (
    <Composition
      id="DataVideo"
      component={DataComponent}
      durationInFrames={300}
      fps={30}
      width={1920}
      height={1080}
      calculateMetadata={async ({ props }) => {
        // 根据数据动态调整持续时间
        const data = await fetchData(props.apiUrl);
        return {
          durationInFrames: data.length * 30,
        };
      }}
    />
  );
};
```

### 关键配置说明

#### 渲染配置（remotion.config.ts）

```ts
import { Config } from '@remotion/runtime';

Config.Output({
  overwrite: true,
});

Config.Rendering.setImageFormat('png');
Config.Rendering.setCodec('h264');

Config.Browser({
  chromiumArgs: ['--no-sandbox'],
});
```

#### 高级配置

```ts
// Lambda 配置
Config.Lambda.setAwsRegion('us-east-1');
Config.Lambda.setMemorySize(2048);

// 音频配置
Config.Audio.addAudioEncoder('aac');

// 编码器选择
Config.setCodec('h264');
Config.setVideoBitrate(5000000);
```

### 最佳实践建议

#### 1. 项目结构组织

```
src/
├── compositions/          # 视频组合组件
│   ├── Intro.tsx
│   ├── MainContent.tsx
│   └── Outro.tsx
├── components/            # 可复用 UI 组件
│   ├── TitleCard.tsx
│   ├── DataChart.tsx
│   └── Logo.tsx
├── assets/                # 静态资源
│   ├── images/
│   └── audio/
├── utils/                 # 工具函数
│   ├── dataProcessing.ts
│   └── animations.ts
├── config.ts              # 配置文件
└── index.tsx              # 入口文件
```

#### 2. 性能优化

- 使用 `<OffthreadVideo>` 处理大型视频文件
- 实现组件懒加载
- 避免不必要的重渲染
- 使用 `useMemo` 和 `useCallback`

```tsx
const VideoFrame = ({ videoSrc }) => {
  // 使用 OffthreadVideo 避免主线程阻塞
  return (
    <OffthreadVideo
      src={videoSrc}
      startFrom={frame}
      showInTimeline={false}
    />
  );
};
```

#### 3. 类型安全

```tsx
import { z } from 'zod';

const schema = z.object({
  title: z.string(),
  data: z.array(z.number()),
  date: z.date(),
});

// 在 Composition 中使用
<Composition
  component={MyVideo}
  // ...
  schema={schema}
/>
```

#### 4. 批量渲染

```bash
# 使用 CLI 渲染
npx remotion render --composition-id MyVideo

# 渲染为静态图像
npx remotion still --composition-id MyVideo

# 指定输出路径和格式
npx remotion render --out=dist/video.mp4 --codec=h264
```

### 学习资源

- **官方文档**：[remotion.dev/docs](https://www.remotion.dev/docs)
- **API 参考**：[remotion.dev/api](https://remotion.dev/api)
- **社区 Discord**：[remotion.dev/discord](https://remotion.dev/discord)
- **作品展示**：[remotion.dev/showcase](https://remotion.dev/showcase)
- **模板集合**：[remotion.dev/docs/resources](https://www.remotion.dev/docs/resources)

## 总结

Remotion 代表了视频制作的一种新范式：**用代码创造视觉内容**。它不仅降低了视频制作的技术门槛，更为开发者打开了一扇通往创意表达的新大门。通过结合 React 的组件化思想和视频时间轴，Remotion 让程序化视频制作变得简单、直观且强大。

无论你是想制作个性化的营销视频、数据驱动的报告动画，还是探索代码与艺术的交叉领域，Remotion 都是一个值得深入探索的优秀工具。

---

**作者**: Matrix Agent
**项目地址**: [https://github.com/remotion-dev/remotion](https://github.com/remotion-dev/remotion)
**官网**: [https://remotion.dev](https://remotion.dev)
