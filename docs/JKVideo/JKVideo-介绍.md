# JKVideo 项目介绍

## Why（为什么）—— 项目背景与动机

### 解决的问题

在移动端观看Bilibili等视频平台内容时，用户面临以下痛点：

1. **官方App过于臃肿**：官方客户端功能繁多，占用大量存储空间和系统资源
2. **网页版体验受限**：浏览器端无法原生支持DASH流播放，且受限于防盗链机制
3. **弹幕同步困难**：官方App的弹幕系统虽有特色，但交互体验仍有优化空间
4. **离线下载不便**：官方下载格式存在限制，跨设备分享困难

### 背景与动机

JKVideo的开发者希望打造一款**轻量、高颜值的第三方视频播放器**，专注于核心的视频播放体验，同时保留用户喜爱的弹幕功能。该项目采用React Native + Expo技术栈，实现了真正的跨平台开发（Android、iOS、Web），降低了维护成本。

### 目标用户

- 视频内容爱好者，特别是弹幕文化爱好者
- 需要轻量级播放器替代官方App的用户
- 跨平台用户（同时使用多设备）
- 技术爱好者（可基于此项目进行二次开发）

---

## What（是什么）—— 核心功能与特性

### 核心功能概览

| 功能模块 | 描述 |
|---------|------|
| DASH流播放 | 支持1080P+、4K HDR、杜比视界等高清格式 |
| 视频弹幕 | XML时间轴同步、5车道飘屏覆盖 |
| 直播弹幕 | WebSocket实时接收、舰长标记、礼物计数 |
| WBI签名 | 纯TypeScript实现，无外部加密依赖 |
| 扫码登录 | 二维码生成 + 轮询 + Cookie自动提取 |
| 下载管理 | 多清晰度下载、内置HTTP服务器局域网分享 |
| 迷你播放器 | 全局浮层续播，跨组件状态同步 |

### 技术栈

```
┌─────────────────────────────────────────────┐
│                   框架层                     │
│  React Native 0.83 + Expo SDK 55 + TypeScript │
├─────────────────────────────────────────────┤
│                   路由层                     │
│        expo-router v4（文件系统路由）          │
├─────────────────────────────────────────────┤
│                  状态管理层                   │
│               Zustand v5                     │
├──────────────────┬──────────────────────────┤
│    视频播放层      │        网络层            │
│  react-native-video │       Axios          │
│  (DASH MPD/HLS)  │                        │
├──────────────────┴──────────────────────────┤
│                  存储层                      │
│  @react-native-async-storage/async-storage  │
│            expo-secure-store                │
├─────────────────────────────────────────────┤
│                  UI层                        │
│   @expo/vector-icons / expo-linear-gradient │
│          react-native-pager-view            │
└─────────────────────────────────────────────┘
```

### 差异化特点

1. **纯TypeScript实现**：WBI签名等加密逻辑完全手写，不依赖任何加密库
2. **DASH MPD动态构建**：将B站API响应转换为本地MPD文件供ExoPlayer播放
3. **性能优化**：使用Animated.Value对象池减少GC压力，虚拟列表优化内存占用
4. **跨平台一致性**：同一套代码运行在Android、iOS、Web三个平台

---

## How（怎么做）—— 快速开始与最佳实践

### 快速开始

#### 方式一：Expo Go（5分钟，无需编译）

```bash
git clone https://github.com/orange-gi/JKVideo.git
cd JKVideo
npm install
npx expo start
```

> 部分清晰度受限，视频播放降级为WebView方案

#### 方式二：Dev Build（完整功能，推荐）

```bash
npm install
npx expo run:android   # Android
npx expo run:ios       # iOS（需 macOS + Xcode）
```

#### 方式三：Web端

```bash
npm install
npx expo start --web
```

> Web端图片需本地代理服务器：`node scripts/proxy.js`

### 核心使用方式

#### 1. 视频播放

```typescript
import { NativeVideoPlayer } from './components/NativeVideoPlayer';

// 播放器组件支持DASH MPD播放
<NativeVideoPlayer
  playData={playUrlResponse}
  qualities={[{ qn: 80, desc: '1080P' }, { qn: 64, desc: '720P' }]}
  currentQn={80}
  onQualityChange={(qn) => setQn(qn)}
  danmakus={danmakuItems}
/>
```

#### 2. WBI签名

```typescript
import { signWbi } from './utils/wbi';

// 对API参数进行签名
const signedParams = signWbi(
  { mid: 123456, pn: 1, ps: 20 },
  imgKey,
  subKey
);
```

#### 3. 构建DASH MPD

```typescript
import { buildDashMpdUri } from './utils/dash';

// 将B站API响应转换为本地MPD URI
const mpdUri = await buildDashMpdUri(playData, qn);
```

### 关键配置说明

| 配置项 | 位置 | 说明 |
|-------|------|------|
| 登录状态 | `store/authStore.ts` | 使用SecureStore存储SESSDATA |
| 下载设置 | `store/downloadStore.ts` | 管理下载队列和存储路径 |
| 播放设置 | `store/settingsStore.ts` | 默认清晰度、弹幕开关等 |
| WBI密钥 | `utils/wbi.ts` | nav接口12小时自动缓存 |

### 项目结构

```
JKVideo/
├── app/                    # expo-router 页面路由
│   ├── index.tsx           # 首页（热门/直播 Tab）
│   ├── video/[bvid].tsx    # 视频详情页
│   ├── live/[roomId].tsx   # 直播详情页
│   ├── search.tsx          # 搜索页
│   ├── downloads.tsx       # 下载管理页
│   └── settings.tsx        # 设置页
├── components/             # UI 组件
│   ├── NativeVideoPlayer.tsx    # 原生视频播放器
│   ├── DanmakuList.tsx          # 弹幕列表
│   ├── DanmakuOverlay.tsx       # 弹幕飘屏
│   └── BigVideoCard.tsx         # 大视频卡片
├── hooks/                  # 数据 Hooks
│   ├── useVideoList.ts     # 视频列表数据
│   ├── useVideoDetail.ts   # 视频详情数据
│   └── useLiveDanmaku.ts   # 直播弹幕数据
├── services/               # API 封装
│   └── api.ts              # Bilibili API 调用
├── store/                  # Zustand 状态
│   ├── authStore.ts        # 认证状态
│   ├── videoStore.ts       # 播放状态
│   └── downloadStore.ts    # 下载状态
└── utils/                  # 工具函数
    ├── wbi.ts              # WBI签名
    ├── dash.ts             # DASH MPD构建
    └── danmaku.ts          # 弹幕解析
```

### 最佳实践建议

1. **开发环境**：优先使用Expo Go进行快速迭代，复杂功能使用Dev Build
2. **状态管理**：Zustand store按功能模块划分，避免状态耦合
3. **性能优化**：
   - 视频卡片使用`removeClippedSubviews`减少内存占用
   - 弹幕组件使用Animated.Value对象池避免频繁GC
4. **安全存储**：敏感信息（SESSDATA）使用expo-secure-store而非AsyncStorage

---

## 已知限制

| 限制 | 原因 | 解决方案 |
|------|------|---------|
| 4K/1080P+需大会员 | B站API策略限制 | 登录大会员账号 |
| FLV直播流不支持 | HTML5/ExoPlayer限制 | 自动降级为HLS |
| Web端需本地代理 | 图片防盗链限制 | 运行`node scripts/proxy.js` |
| 二维码10分钟过期 | 安全策略 | 重新打开登录弹窗 |

---

## License

MIT License © 2026 JKVideo Contributors
