# Remotion 提示词分析

## 概述

Remotion 项目在 AI 集成方面投入了大量精力，设计了完整的提示词工程体系。这些提示词主要用于：
1. **教导 LLMs 遵循 Remotion 开发规则**
2. **为 AI 代理提供技能文件**
3. **支持自然语言生成视频**

本文档深入分析 Remotion 项目中的提示词设计模式、关键组件和最佳实践。

---

## 1. 系统提示词（System Prompt）

### 提示词位置

**文件路径**：`https://www.remotion.dev/docs/ai/system-prompt`

**获取方式**：
- 文档页面支持 `.md` 后缀访问原始 Markdown
- 支持 `Accept: text/markdown` 请求头
- 提供 `llms.txt` 专用文件供 AI 代理使用

### 完整提示词内容

```markdown
# Remotion System Prompt

Remotion is a framework for making videos programmatically using React.js.
All output should be valid React code written in TypeScript.

## Project Structure

### Entry file (`src/index.ts`)
```ts
import {registerRoot} from 'remotion';
import {Root} from './Root';
registerRoot(Root);
```

### Root file (`src/Root.tsx`)
```tsx
import {Composition} from 'remotion';
import {MyComp} from './MyComp';

export const Root: React.FC = () => {
	return (
		<>
			<Composition
				id="MyComp"
				component={MyComp}
				durationInFrames={120}
				width={1920}
				height={1080}
				fps={30}
				defaultProps={{}}
			/>
		</>
	);
};
```

### Composition Default Values
- fps: 30
- width: 1920
- height: 1080
- id: "MyComp"
```

### 提示词设计模式

#### 1.1 角色定义模式

```markdown
# Remotion System Prompt

Remotion is a framework for making videos programmatically using React.js.
All output should be valid React code written in TypeScript.
```

**设计特点**：
- **明确角色**：定义 Remotion 是视频创建的框架
- **输出规范**：强调必须输出有效的 React + TypeScript 代码
- **简洁定位**：一句话定义项目本质

#### 1.2 代码示例模式

```tsx
// 入口文件示例
import {registerRoot} from 'remotion';
import {Root} from './Root';
registerRoot(Root);

// Root 组件示例
import {Composition} from 'remotion';
import {MyComp} from './MyComp';

export const Root: React.FC = () => {
	return (
		<>
			<Composition
				id="MyComp"
				component={MyComp}
				durationInFrames={120}
				width={1920}
				height={1080}
				fps={30}
				defaultProps={{}}
			/>
		</>
	);
};
```

**设计特点**：
- **分层展示**：从入口文件到组件定义
- **完整代码**：提供可直接使用的代码片段
- **默认参数**：明确标注默认配置值

#### 1.3 规则约束模式

**核心组件规则**：
```tsx
// useCurrentFrame 钩子使用
export const MyComp: React.FC = () => {
	const frame = useCurrentFrame();
	return <div>Frame {frame}</div>;
};
```

**媒体标签规则**：
| 媒体类型 | 组件 | 导入来源 | 关键属性 |
|---------|------|---------|---------|
| 视频 | `<Video>` | @remotion/media | src, trimBefore, trimAfter, volume |
| 图片 | `<Img>` | remotion | src, style |
| 音频 | `<Audio>` | @remotion/media | src, trimBefore, trimAfter, volume |

**设计特点**：
- **表格化规则**：清晰展示组件与属性映射
- **按功能分类**：按媒体类型分组
- **示例驱动**：通过代码示例说明用法

#### 1.4 动画插值模式

```tsx
// interpolate() 值插值
import {interpolate} from 'remotion';

export const MyComp: React.FC = () => {
	const frame = useCurrentFrame();
	const value = interpolate(frame, [0, 100], [0, 1], {
		extrapolateLeft: 'clamp',
		extrapolateRight: 'clamp',
	});
	return <div>{value}</div>;
};

// spring() 弹簧动画
import {spring} from 'remotion';

export const MyComp: React.FC = () => {
	const frame = useCurrentFrame();
	const {fps} = useVideoConfig();
	const value = spring({
		fps,
		frame,
		config: {
			damping: 200,
		},
	});
	return <div>{value}</div>;
};
```

**设计特点**：
- **函数签名完整**：展示所有必要参数
- **默认参数建议**：明确标注推荐配置（如 clamp）
- **参数说明**：解释每个参数的作用

#### 1.5 确定性约束模式

```tsx
// ✅ 必须使用
import {random} from 'remotion';
export const MyComp: React.FC = () => {
	return <div>Random number: {random('my-seed')}</div>;
};

// ❌ 禁止使用
// Math.random() API
```

**设计特点**：
- **正反示例**：同时展示正确和错误做法
- **强调原因**：解释为什么需要确定性（渲染可重复）
- **明确禁令**：使用 "禁止" 词汇强调规则

---

## 2. 代理技能文件（Agent Skills）

### 技能文件位置

**GitHub 路径**：`https://github.com/remotion-dev/remotion/tree/main/packages/skills`

**安装命令**：
```bash
npx skills add remotion-dev/skills
```

### 技能文件内容概览

Remotion 提供的代理技能包括：

1. **Remotion Core Skills** - 核心开发技能
2. **Remotion Animation Skills** - 动画制作技能
3. **Remotion Media Skills** - 媒体处理技能
4. **Remotion Rendering Skills** - 渲染技能

### 技能文件设计模式

#### 2.1 技能描述模式

```markdown
# Skill: Remotion Core Development

## Description
当用户需要创建 Remotion 视频项目时使用此技能。

## When to Use
- 初始化新的 Remotion 项目
- 创建 Composition 组件
- 设置项目结构

## Best Practices
1. 使用 TypeScript 编写所有代码
2. 遵循默认配置（fps=30, 1920x1080）
3. 正确注册 Root 组件
```

**设计特点**：
- **场景化描述**：说明何时使用该技能
- **最佳实践**：提供具体操作建议
- **结构化**：清晰的技能描述模板

#### 2.2 代码模板模式

```tsx
// 基础视频模板
import { Composition, useCurrentFrame, interpolate, AbsoluteFill } from 'remotion';

export const MyVideo = () => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ background: 'black' }}>
      {/* 视频内容 */}
    </AbsoluteFill>
  );
};
```

**设计特点**：
- **可复用的模板**：提供可直接使用的代码骨架
- **导入完整**：包含所有必要的导入语句
- **最佳实践集成**：代码本身就体现最佳实践

---

## 3. Bolt.new 集成提示词

### 集成位置

**GitHub 仓库**：`https://github.com/stackblitz/starters/tree/main/bolt-remotion`

### 提示词设计模式

Bolt.new 的 Remotion 模板提示词主要包含：

#### 3.1 模板初始化提示词

```markdown
Create a new Remotion video project with the following requirements:
1. Use the Hello World template as base
2. Include the following components:
   - Title animation
   - Background video
   - Audio track
3. Configure for local development
```

**设计特点**：
- **模板化**：基于现有模板定制
- **组件化**：明确指定需要的组件
- **开发配置**：指定开发环境要求

---

## 4. 提示词变量和模板系统

### 4.1 Composition 变量模板

```tsx
// 动态 Composition 定义
<Composition
  id={id}                    // 变量：组合 ID
  component={Component}      // 变量：组件引用
  durationInFrames={frames}  // 变量：帧数
  fps={fps}                  // 变量：帧率
  width={width}              // 变量：宽度
  height={height}            // 变量：高度
  defaultProps={props}       // 变量：默认属性
/>
```

### 4.2 动画参数模板

```tsx
// 动画插值模板
interpolate(
  input: frame,                    // 当前帧
  outputRange: [start, end],       // 输入范围
  inputRange: [min, max],          // 输出范围
  options: {
    extrapolateLeft: 'clamp',      // 左外推策略
    extrapolateRight: 'clamp'      // 右外推策略
  }
)

// 弹簧动画模板
spring({
  fps: videoConfig.fps,            // 帧率
  frame: currentFrame,             // 当前帧
  config: {
    damping: 200,                  // 阻尼系数
    stiffness: 100,                // 刚度
    mass: 1                        // 质量
  }
})
```

---

## 5. MCP 提示词集成

### MCP 集成位置

**文档路径**：`https://www.remotion.dev/docs/ai/mcp`

### MCP 提示词设计

```markdown
# Remotion MCP Server

You are a Remotion expert assistant that can:
1. Help users create and edit Remotion projects
2. Provide code suggestions for video components
3. Debug rendering issues
4. Explain Remotion concepts and APIs

## Available Tools
- create_composition: Create a new video composition
- render_video: Render a video from composition
- add_animation: Add animation to a component
- configure_props: Configure component props
```

**设计特点**：
- **工具定义**：明确列出可用工具
- **能力边界**：定义 MCP 服务器能做什么
- **上下文提供**：提供项目状态信息

---

## 6. 优化建议

### 6.1 提示词改进建议

#### 建议 1：增加错误处理模式

```tsx
// 当前缺少的错误处理示例
export const MyVideo: React.FC<Props> = ({ data }) => {
  // 建议添加
  if (!data) {
    return <ErrorFallback message="No data provided" />;
  }

  // 正常渲染逻辑
  return <MainContent data={data} />;
};
```

#### 建议 2：增加性能优化提示

```tsx
// 性能优化最佳实践
export const OptimizedVideo: React.FC = () => {
  // ✅ 推荐：使用 OffthreadVideo 处理大视频
  return (
    <OffthreadVideo
      src="large-video.mp4"
      showInTimeline={false}
    />
  );

  // ❌ 不推荐：使用普通 Video 组件
  return <Video src="large-video.mp4" />;
};
```

#### 建议 3：增加 TypeScript 类型定义示例

```tsx
// Props 类型定义
interface MyVideoProps {
  title: string;
  subtitle?: string;
  duration: number;
  theme: 'light' | 'dark';
}

// 使用类型安全的方式定义组件
export const MyVideo: React.FC<MyVideoProps> = ({
  title,
  subtitle,
  duration,
  theme
}) => {
  return <div className={`video-${theme}`}>{title}</div>;
};
```

### 6.2 文档可访问性改进

Remotion 已实现的优化：
- ✅ `.md` 后缀访问原始 Markdown
- ✅ `llms.txt` 专用文件
- ✅ `Accept: text/markdown` 请求头支持
- ✅ AI 代理友好的内容协商

建议进一步改进：
- 🔄 提供结构化的 JSON 格式提示词
- 🔄 增加多语言版本提示词
- 🔄 提供版本化的提示词历史

---

## 7. 总结

### 7.1 提示词设计亮点

| 特性 | 实现方式 | 效果 |
|------|----------|------|
| **角色定义** | 开头明确声明框架定位 | AI 快速理解上下文 |
| **代码示例** | 完整的可运行代码片段 | 减少理解误差 |
| **规则约束** | 正反示例对比 | 明确正确做法 |
| **表格化信息** | 组件属性映射表 | 快速查阅 |
| **默认值约定** | 明确标注默认参数 | 简化配置 |
| **确定性要求** | 强调随机数使用规范 | 保证渲染一致性 |

### 7.2 最佳实践总结

1. **分层组织**：从概述到细节，逐步深入
2. **示例驱动**：通过代码示例说明概念
3. **规则明确**：使用 "必须/禁止" 等强约束词汇
4. **上下文完整**：提供完整的导入和依赖信息
5. **多格式支持**：支持 Markdown、llms.txt 等多种格式

### 7.3 应用场景

Remotion 的提示词设计适用于：

- 🤖 **AI 代码助手**：Claude Code、Codex、Cursor 等
- 📝 **文档生成**：自动生成项目文档
- 🎓 **新手教程**：引导新用户正确使用框架
- 🔧 **重构辅助**：帮助 AI 理解和修改现有代码

---

## 参考资源

- **系统提示词**：[remotion.dev/docs/ai/system-prompt](https://www.remotion.dev/docs/ai/system-prompt)
- **代理技能**：[remotion.dev/docs/ai/skills](https://www.remotion.dev/docs/ai/skills)
- **Bolt.new 集成**：[remotion.dev/docs/ai/bolt](https://www.remotion.dev/docs/ai/bolt)
- **MCP 集成**：[remotion.dev/docs/ai/mcp](https://www.remotion.dev/docs/ai/mcp)
- **技能 GitHub**：[github.com/remotion-dev/remotion/tree/main/packages/skills](https://github.com/remotion-dev/remotion/tree/main/packages/skills)

---

**作者**: Matrix Agent
**最后更新**: 2026-01-23
