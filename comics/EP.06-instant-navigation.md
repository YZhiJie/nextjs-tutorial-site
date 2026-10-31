# EP.06 Instant Navigations 即时导航

> 对应正文：[第 6 章 · Instant Navigations](../chapters/06-instant-navigation/basics) · 配套实验：[labs/debug-instant-degradation](../labs/debug-instant-degradation) · 源码深潜：[router-prefetch](../deep-dives/router-prefetch)

## 剧情回顾

- **第 1 格**：标签君点击链接后只能等 RSC 请求，服务端慢查询决定了白屏时长。
- **第 2 格**：码叔亮出两个开关与每个 await 的三选一：Stream、Cache、Block。
- **第 3 格**：像素酱实证配置与 Suspense 边界，并说明整段阻塞的 `instant = false`。
- **第 4 格**：码叔提醒冷缓存、依赖关系与 Instant Insights 的硬约束。

## 知识点拆解

### 第 1 格 · 即时导航的定义

用户点击的那一刻，浏览器就开始渲染新页面：静态内容、缓存内容与回退 UI 立刻出现，剩余内容由服务器流式补入。前提是缓存已预热。

```tsx
click → RSC 请求 → 服务端渲染
→ 慢查询 → 响应返回 → 切页
// 用户盯着旧页面或白屏
// 等待时长 = 最慢的那个请求
```

### 第 2 格 · 两个配置开关

- **cacheComponents**：页面拆成静态外壳 + 动态区域
- **partialPrefetching**：预取共享 App Shell，而非整页
- **Stream / Cache / Block**：流式、缓存，或显式放弃即时

- `cacheComponents: true`：启用 Cache Components 模型
- `partialPrefetching: true`：`<Link>` 只预取共享 App Shell
- 两者有依赖关系，后者必须在前者开启后工作

### 第 3 格 · Stream / Cache / Block

预渲染沿组件树遇到需要等待的 await 时，只能三选一：放进 Suspense 流式、用 `'use cache'` 缓存，或导出 `instant = false` 显式阻塞。

- Stream：每次都要新鲜的数据
- Cache：变化不频繁、可接受陈旧窗口的数据
- Block：暂时无法改造、必须同步就绪的段

```tsx
// next.config.ts
cacheComponents: true
partialPrefetching: true
// 库存：每次请求新鲜
<Suspense fallback={<Skeleton />}>
  <Inventory params={params} />
</Suspense>
```

### 第 4 格 · Instant Insights 与 instant()

开发期，不够即时的导航会被上报为带修复菜单的错误；`@next/playwright` 的 `instant()` 把同一约束搬进 CI，防止即时性退化。

> ⚠️ 冷缓存首次导航不可能即时
> ⚠️ 只开 partialPrefetching 不生效
> ⚠️ 未缓存 await 未包 Suspense → Instant Insights 报错

### 延伸 · 跑一遍真实实验

打开 [labs/debug-instant-degradation](../labs/debug-instant-degradation) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 以为冷缓存首次访问也能做到零等待。
2. 只开 `partialPrefetching`，忘记 `cacheComponents`。
3. 把慢查询留在外壳路径上，且不包 Suspense。

## 自测 3 题（含答案）

1. 即时导航的两个开关是什么？——**`cacheComponents` 与 `partialPrefetching`，后者依赖前者。**
2. 一个 await 有哪三种合法处理方式？——**Stream（Suspense 流式）、Cache（'use cache'）、Block（instant = false）。**
3. 冷缓存第一次导航为什么可能仍要等？——**缓存尚未预热，服务器仍需计算一次。**
