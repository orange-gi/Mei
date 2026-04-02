# JKVideo 提示词分析

## 概述

**项目类型说明**：JKVideo 是一个基于 React Native + Expo 的视频播放器客户端项目，主要功能包括 DASH 流播放、弹幕系统、直播功能等。该项目不属于 AI/LLM 相关项目，因此没有传统意义上的"提示词（Prompt）"资源。

本章节将从项目特性角度，分析其中涉及的用户交互文案和配置信息。

---

## 1. 用户界面文案

### 1.1 首页导航文案

```typescript
// 位置: app/index.tsx
const TABS: { key: TabKey; label: string }[] = [
  { key: "hot", label: "热门" },
  { key: "live", label: "直播" },
];
```

| 文案 | 含义 |
|-----|------|
| 热门 | 视频推荐/热门内容列表 |
| 直播 | 直播间列表 |

### 1.2 直播分区文案

```typescript
const LIVE_AREAS = [
  { id: 0, name: "推荐" },
  { id: 2, name: "网游" },
  { id: 3, name: "手游" },
  { id: 6, name: "单机游戏" },
  { id: 1, name: "娱乐" },
  { id: 9, name: "虚拟主播" },
  { id: 10, name: "生活" },
  { id: 11, name: "知识" },
];
```

### 1.3 弹幕相关文案

```typescript
// 位置: components/DanmakuList.tsx

// 舰长等级标签
const GUARD_LABELS: Record<number, { text: string; color: string }> = {
  1: { text: "总督", color: "#E13979" },
  2: { text: "提督", color: "#7B68EE" },
  3: { text: "舰长", color: "#00D1F1" },
};
```

| 标签 | 颜色 | 含义 |
|-----|------|------|
| 总督 | #E13979 | 直播间最高等级舰长 |
| 提督 | #7B68EE | 直播间中级舰长 |
| 舰长 | #00D1F1 | 直播间基础舰长 |

### 1.4 播放器控制文案

```typescript
// 位置: components/NativeVideoPlayer.tsx

// 时间格式化
function formatDuration(seconds: number): string {
  const h = Math.floor(seconds / 3600);
  const m = Math.floor((seconds % 3600) / 60);
  const s = Math.floor(seconds % 60);
  if (h > 0) return `${h}:${m.toString().padStart(2, "0")}:${s.toString().padStart(2, "0")}`;
  return `${m}:${s.toString().padStart(2, "0")}`;
}
```

---

## 2. API请求配置

### 2.1 WBI签名参数

```typescript
// 位置: utils/wbi.ts

export function signWbi(
  params: Record<string, string | number>,
  imgKey: string,
  subKey: string,
): Record<string, string | number> {
  const mixinKey = getMixinKey(imgKey, subKey);
  const wts = Math.floor(Date.now() / 1000);
  const all: Record<string, string | number> = { ...params, wts };
  const query = Object.keys(all)
    .sort()
    .map(k => {
      const v = String(all[k]).replace(/[!'()*]/g, '');
      return `${k}=${v}`;
    })
    .join('&');
  const w_rid = md5(query + mixinKey);
  return { ...all, w_rid };
}
```

**设计模式分析**：

| 特征 | 说明 |
|-----|------|
| 参数排序 | 按 key 字母顺序排列 |
| 特殊字符转义 | 移除 `!'()*` 字符 |
| 时间戳 | 添加 `wts` 参数防止重放攻击 |
| 签名算法 | MD5(query + mixinKey) |

### 2.2 清晰度配置

```typescript
// 位置: components/NativeVideoPlayer.tsx

const qualities: { qn: number; desc: string }[] = [
  { qn: 126, desc: "杜比 4K" },
  { qn: 120, desc: "4K" },
  { qn: 116, desc: "1080P60" },
  { qn: 112, desc: "HDR" },
  { qn: 80, desc: "1080P" },
  { qn: 64, desc: "720P" },
  { qn: 32, desc: "480P" },
  { qn: 16, desc: "360P" },
];
```

| qn值 | 描述 | 说明 |
|-----|------|------|
| 126 | 杜比 4K | 最高画质，需大会员 |
| 120 | 4K | 超高清，需大会员 |
| 116 | 1080P60 | 60帧高清，需大会员 |
| 112 | HDR | 高动态范围，需大会员 |
| 80 | 1080P | 全高清 |
| 64 | 720P | 高清 |
| 32 | 480P | 标清 |
| 16 | 360P | 流畅 |

---

## 3. 配置文件

### 3.1 环境变量

```bash
# 位置: .env
# B站API相关配置（示例）
API_BASE_URL=https://api.bilibili.com
WBI_KEY_URL=https://api.bilibili.com/x/web-interface/nav
```

### 3.2 EAS构建配置

```json
// 位置: eas.json
{
  "build": {
    "preview": {
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "android": {
        "buildType": "apk"
      }
    }
  }
}
```

---

## 4. 弹幕协议分析

### 4.1 视频弹幕格式

```typescript
// 位置: services/types.ts

interface DanmakuItem {
  id: number;          // 弹幕ID
  time: number;        // 出现时间（秒）
  type: number;        // 弹幕类型 1-滚动 2-底端 3-顶端
  size: number;        // 字体大小
  color: string;       // 颜色（十六进制）
  text: string;        // 弹幕文本
  uid: number;         // 发送者UID
  uname?: string;      // 发送者名称
  timeline?: string;  // 时间线（直播）
  guardLevel?: number; // 舰长等级
  medalLevel?: number; // 粉丝牌等级
  medalName?: string;  // 粉丝牌名称
}
```

### 4.2 弹幕颜色转换

```typescript
// 位置: utils/danmaku.ts

export function danmakuColorToCss(color: string): string {
  // 将数字颜色值转换为CSS格式
  // 例如: 16777215 -> "#ffffff"
  return '#' + Number(color).toString(16).padStart(6, '0');
}
```

---

## 5. 用户交互提示

### 5.1 加载状态文案

```typescript
// 位置: app/index.tsx

<ListFooterComponent>
  {loading && <ActivityIndicator color="#00AEEC" />}
</ListFooterComponent>

// 位置: components/DanmakuList.tsx

<ListEmptyComponent>
  <Text style={styles.empty}>
    {danmakus.length === 0 ? "暂无弹幕" : "弹幕将随视频播放显示"}
  </Text>
</ListEmptyComponent>
```

| 场景 | 文案 |
|-----|------|
| 列表加载中 | 加载中... |
| 无弹幕 | 暂无弹幕 |
| 弹幕加载中 | 弹幕将随视频播放显示 |
| 新弹幕提示 | {n} 条新弹幕 |

### 5.2 登录状态文案

```typescript
// 位置: components/LoginModal.tsx

const statusMessages = {
  waiting: "请使用B站App扫码登录",
  scanning: "扫码成功，请在App中确认",
  expired: "二维码已过期，点击刷新",
  success: "登录成功",
  error: "登录失败，请重试",
};
```

---

## 6. 错误处理文案

### 6.1 播放器错误

```typescript
// 位置: components/NativeVideoPlayer.tsx

onError={(e) => {
  // 杜比视界播放失败时自动降级到 1080P
  if (currentQn === 126) {
    onQualityChange(80);
    return;
  }
  console.warn("Video playback error:", e);
}}
```

### 6.2 API错误处理

```typescript
// 位置: services/api.ts

export async function getUserInfo(): Promise<UserInfo> {
  try {
    // API调用
  } catch (error) {
    // 返回mock数据作为降级方案
    return MOCK_USER_INFO;
  }
}
```

---

## 7. 性能优化配置

### 7.1 列表优化

```typescript
// 位置: app/index.tsx

<FlatList
  windowSize={7}                    // 窗口内渲染数量
  maxToRenderPerBatch={6}          // 每批最大渲染数
  removeClippedSubviews={true}      // 移除屏幕外视图
  viewabilityConfig={{
    itemVisiblePercentThreshold: 50 // 可见性阈值
  }}
/>
```

### 7.2 弹幕动画池

```typescript
// 位置: components/DanmakuList.tsx

// Animated.Value对象池，减少频繁创建/GC
const animPool: Animated.Value[] = [];
const POOL_MAX = 64;

function acquireAnim(): Animated.Value {
  const v = animPool.pop();
  if (v) { v.setValue(0); return v; }
  return new Animated.Value(0);
}

function releaseAnims(items: DisplayedDanmaku[]) {
  for (const item of items) {
    if (animPool.length < POOL_MAX) {
      animPool.push(item._fadeAnim);
    }
  }
}
```

---

## 总结

JKVideo 项目虽然不是 AI/LLM 相关项目，但其在用户界面文案、配置管理和错误处理方面有着良好的设计：

| 方面 | 特点 |
|-----|------|
| 文案设计 | 简洁直观，符合用户习惯 |
| 配置管理 | 清晰的结构化配置，便于维护 |
| 错误处理 | 多级降级策略，保证可用性 |
| 性能优化 | 虚拟列表、对象池等最佳实践 |

这些设计为构建高质量的移动应用提供了参考范例。
