# EP.04 数据获取与流式渲染

> 对应正文：[第 4 章 · 数据获取与流式渲染](../chapters/04-data-fetching/basics) · 配套实验：[labs/data-streaming](../labs/data-streaming) · 源码深潜：[flight-stream](../deep-dives/flight-stream)

## 剧情回顾

- **第 1 格**：标签君的两个请求明明互不依赖，却因为顺序 await 串成 2000ms 瀑布。
- **第 2 格**：码叔给出两条路：并行发请求，以及用 Suspense 先把壳流出去。
- **第 3 格**：像素酱实证 `Promise.all` 与 Suspense 边界，骨架屏先到、数据后补。
- **第 4 格**：码叔强调 16.x 的 fetch 默认不缓存，未包边界的动态读取会阻塞整段。

## 知识点拆解

### 第 1 格 · 16.x 的 fetch 语义

默认 `fetch(url)` 不持久化缓存；需要时显式写 `cache: 'force-cache'` 或 `next: { revalidate: 60 }`。ORM 与数据库驱动没有自动缓存，需要 `'use cache'`。

- 开发环境 HMR 会临时记忆 fetch 响应，排查“数据没变”先整页刷新
- 未完成的请求默认阻塞渲染，不是 bug

```tsx
const artist = await getArtist()  // 800ms
const albums = await getAlbums()  // 1200ms
// 总耗时 2000ms，本可并行
// 页面整块空白等最慢请求
```

### 第 2 格 · 并行化

先创建 Promise，再在需要结果时 await。`Promise.all` 适合全部依赖；`Promise.allSettled` 适合允许部分失败的聚合。

- **Promise.all**：无依赖请求同时发出
- **Suspense**：先给回退 UI，数据到了再替换
- **loading.tsx**：路由级骨架，自动包住子段

### 第 3 格 · Suspense 与流式

把读取动态输入的组件放进 `<Suspense>`，外层壳会先发送，慢区域随后流式补入。`loading.tsx` 是路由级的自动 Suspense 边界。

```tsx
const artistP = getArtist(u)
const albumsP = getAlbums(u)
const [artist, albums] = await Promise.all([
  artistP, albumsP
])
<Suspense fallback={<Skeleton />}>
  <Reviews />
</Suspense>
```

### 第 4 格 · 客户端取数边界

客户端组件可以用 SWR / React Query 等库做交互后取数，但首屏数据优先在服务端取；否则会把请求瀑布搬到浏览器。

> ⚠️ fetch 默认不持久化缓存（16.x）
> ⚠️ 未包 Suspense 的动态读取会阻塞整段
> ⚠️ 无依赖请求却顺序 await，制造瀑布

### 延伸 · 跑一遍真实实验

打开 [labs/data-streaming](../labs/data-streaming) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 把互不依赖的请求写成连续 `await`，延迟直接相加。
2. 把动态读取放在没有 Suspense 的外壳路径上。
3. 沿用 15.x 的“fetch 默认缓存”心智，误判数据新鲜度。

## 自测 3 题（含答案）

1. 两个无依赖请求怎样并行？——**先创建 Promise，再用 `Promise.all` 统一 await。**
2. Suspense 在流式渲染里做什么？——**先发送回退 UI，慢数据完成后流式替换该区域。**
3. 16.x 中 `fetch(url)` 默认缓存吗？——**不缓存；需要显式配置 `force-cache` 或 `revalidate`。**
