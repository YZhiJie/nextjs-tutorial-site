# EP.08 渲染模式与静态生成

> 对应正文：[第 8 章 · 渲染模式与静态生成](../chapters/08-rendering/basics) · 配套实验：[labs/mini-isr](../labs/mini-isr) · 源码深潜：[ppr-postponed-state](../deep-dives/ppr-postponed-state)

## 剧情回顾

- **第 1 格**：标签君用 `force-dynamic` 一刀切，结果要么内容旧，要么每次都慢。
- **第 2 格**：码叔分清 SSG、SSR、ISR：动态输入决定路由是否在请求期执行。
- **第 3 格**：像素酱实证 `generateStaticParams`、`revalidate` 与按需再生。
- **第 4 格**：码叔提醒 16 的 params 是 Promise，async 页面不等于动态。

## 知识点拆解

### 第 1 格 · 动态输入清单

未启用 Cache Components 时，组件链上访问 `cookies()`、`headers()`、`searchParams`、未预渲染的 `params`，或显式 `force-dynamic`，都会让路由在请求期渲染。

```tsx
博客想静态：CDN 快、成本低
库存想动态：每次都要新鲜
export const dynamic = 'force-dynamic'
// 结果：要么旧，要么慢
```

### 第 2 格 · generateStaticParams

返回数组，每项是一组 params。返回的路径在构建期生成静态 HTML；没返回的路径默认首次请求按需生成，之后同样缓存。

- **SSG**：构建期生成，内容稳定
- **SSR**：每请求执行，依赖 cookies / headers
- **ISR**：静态服务 + 后台再生

- 嵌套动态段的内层会收到外层已解析的 params
- 数据不存在时在组件内调用 `notFound()`

### 第 3 格 · ISR

`export const revalidate = 60` 让静态页面在后台按秒再生：先返回旧内容，再异步刷新。经典模型下与 Cache Components 的 `cacheLife` 并存，迁移时逐段替换。

```tsx
export async function generateStaticParams() {
  return posts.map(p => ({ slug: p.slug }))
}
export const revalidate = 60
// 未返回的路径：首次请求按需生成
const { slug } = await params
```

### 第 4 格 · PPR 与软动态

16.4 用 Cache Components 表达部分预渲染：静态外壳先出，动态区域在 Suspense 边界内流式补入。无法预渲染的访问必须显式缓存或流式。

> ⚠️ params / searchParams / cookies 在 16 都是 Promise
> ⚠️ generateStaticParams 返回的是 params 数组
> ⚠️ 以为 async 页面一定动态——await 纯函数不会

### 延伸 · 跑一遍真实实验

打开 [labs/mini-isr](../labs/mini-isr) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 同步读取 `params` / `searchParams`，升级 16 后直接报错。
2. 以为页面写了 `async` 就一定动态渲染。
3. 对每个页面都 `force-dynamic`，放弃 CDN 与静态优化。

## 自测 3 题（含答案）

1. 哪些输入会让路由变动态？——**cookies、headers、searchParams、未预渲染的 params，或显式 force-dynamic。**
2. generateStaticParams 返回什么？——**一个 params 对象数组，每个对象对应一条要预渲染的路径。**
3. ISR 的 revalidate = 60 是什么行为？——**静态服务，超过 60 秒后的首个请求触发后台再生。**
