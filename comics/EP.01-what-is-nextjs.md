# EP.01 认识 Next.js 16.3

> 对应正文：[第 1 章 · 认识 Next.js 16.3](../chapters/01-introduction/basics) · 配套实验：[labs/hello-next](../labs/hello-next)

## 剧情回顾

- **第 1 格**：标签君发现页面在浏览器、数据在另一台服务器，首屏还要自己拼 SSR——两套仓库、两条发布线。
- **第 2 格**：码叔画出 Next.js 的答案——`app/` 文件即路由，RSC 直出 HTML，写操作才需要端点。
- **第 3 格**：像素酱翻开 16.3 配置：`cacheComponents`、`partialPrefetching`、Turbopack 默认与 `proxy.ts`。
- **第 4 格**：码叔结案——先画清服务端/客户端边界，再谈缓存与性能。

## 知识点拆解

### 第 1 格 · 三种渲染先分清

SSR 是请求时服务端生成 HTML，SSG 是构建时生成，CSR 是浏览器拿到 JS 后才渲染。Next.js 允许按路由选择，而不是全站一刀切。

- App Router 的服务端组件默认直出 HTML，零客户端 JS
- 客户端组件同样会被 SSR 成 HTML，再水合事件处理器

```tsx
// 传统：SPA + 独立后端
fetch('/api/posts')   // 至少一跳
首屏空白，SEO 靠补渲染
两个仓库 · 两条发布线
```

### 第 2 格 · 一个仓库、一次部署

传统 SPA 要先设计并长期维护 API 契约；Next.js 中服务端组件可以直接查库，只有对外能力才写 Route Handler。

- **app/ 文件**：文件即路由，page 才成为 URL
- **RSC 直出**：服务端组件查库，零客户端 JS
- **Action / API**：写操作与对外能力才写端点

- 少掉前端 fetch 自家接口的一跳
- 但鉴权不能只靠 proxy，必须落到每个 Action / Handler / DAL

### 第 3 格 · 16.3 的四个关键词

- Instant Navigations：`cacheComponents` + `partialPrefetching`
- Turbopack：dev 内存回收，build 磁盘缓存默认开启
- SSR 改用 Node 原生流，Edge Runtime 废弃
- `middleware.ts` 迁移为 `proxy.ts`（Node 运行时）

```tsx
// next.config.ts
cacheComponents: true
partialPrefetching: true
// Turbopack 默认 · proxy.ts 取代 middleware
// params / cookies() 全部 await
```

### 第 4 格 · 升级检查表

- `params`、`searchParams`、`cookies()`、`headers()` 全部 await
- `experimental.ppr` 改为 `cacheComponents`
- 缓存默认动态，需要时显式写 `'use cache'`

> ⚠️ 把密钥 / 数据库代码写进客户端组件
> ⚠️ 升级 16 后忘记 await params、cookies
> ⚠️ 以为全站必须 SSR 或必须静态

### 延伸 · 跑一遍真实实验

打开 [labs/hello-next](../labs/hello-next) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 把 Next.js 当成“带路由的 React”，忽略服务端边界。
2. 在客户端组件里读 `process.env` 密钥或直接访问数据库。
3. 升级 16.3 后仍按旧写法同步读取 `params` 与 `cookies()`。

## 自测 3 题（含答案）

1. 服务端组件和客户端组件，谁会被 SSR 成 HTML？——**两者都会；区别是服务端组件不向浏览器发送组件 JS，客户端组件会水合交互。**
2. 16.3 开启即时导航需要哪两个开关？——**`cacheComponents: true` 与 `partialPrefetching: true`，后者依赖前者。**
3. 为什么说 Next.js 不是“完整后端替代品”？——**长任务、队列、独立扩缩容等仍应放在独立服务；Next 更适合 BFF 与 Web 产品主体。**
