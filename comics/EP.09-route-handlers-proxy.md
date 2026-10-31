# EP.09 Route Handlers 与 proxy.ts

> 对应正文：[第 9 章 · Route Handlers 与 proxy.ts](../chapters/09-route-handlers/basics) · 配套实验：[labs/route-handlers-proxy](../labs/route-handlers-proxy)

## 剧情回顾

- **第 1 格**：标签君把旧 API 迁进 `app/`，又发现 middleware 在 Edge 里连不上 Node 库。
- **第 2 格**：码叔区分 `route.ts` 与 `proxy.ts`：前者是端点，后者是 Node 运行时粗粒度拦截。
- **第 3 格**：像素酱实证 Webhook 验签必须读原始 body，以及流式响应。
- **第 4 格**：码叔提醒 body 只能读一次、Route Handler 默认不缓存、proxy 不该查库。

## 知识点拆解

### 第 1 格 · 最小端点

在 `app/` 任意层级放 `route.ts`，导出以 HTTP 方法命名的异步函数。使用标准 `Request` / `Response`，也可用 `NextRequest` / `NextResponse`。未导出的方法自动返回 405。

```tsx
// pages/api 迁到 app/route.ts
export async function GET(req) { ... }
// middleware.ts 在 Edge，连不上 Node 库
// Webhook 需要原始 body 验签
```

### 第 2 格 · 动态参数与查询

第二个参数 `ctx` 提供 `params`，16 中同样是 Promise。查询字符串用 `new URL(request.url).searchParams` 读取，无需额外 API。

- **route.ts**：GET/POST/PUT... 标准 Web API
- **405**：未导出的方法自动拒绝
- **proxy.ts**：Node 运行时，可做重写与跳转

### 第 3 格 · Webhook 与流式

- 验签必须使用原始 body 文本，先 `text()` 再 `JSON.parse`
- 返回 `ReadableStream` 即可实现 SSE / LLM 流式输出
- CORS 需要显式处理 `OPTIONS` 与响应头

```tsx
export async function POST(req: NextRequest) {
  const raw = await req.text()
  verifySignature(raw, req.headers)
  const body = JSON.parse(raw)
  return Response.json({ ok: true }, { status: 201 })
}
// proxy.ts 只做语言协商 / 租户路径粗筛
```

### 第 4 格 · proxy.ts 迁移

16.4 中 `middleware.ts` 迁移为 `proxy.ts`，运行在 Node 运行时。适合语言协商、租户路径粗筛、重写与跳转；不应在其中做数据库查询或重鉴权。

> ⚠️ 读两次 body（json 后再 text）会报错
> ⚠️ Route Handler 默认不缓存
> ⚠️ 在 proxy 里做数据库鉴权，慢且不该

### 延伸 · 跑一遍真实实验

打开 [labs/route-handlers-proxy](../labs/route-handlers-proxy) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 先 `await request.json()` 再 `await request.text()`，请求体已被消费。
2. 期待 Route Handler 像静态页面一样被缓存。
3. 把完整鉴权与数据库查询塞进 `proxy.ts`。

## 自测 3 题（含答案）

1. Route Handler 未导出 DELETE 会怎样？——**框架自动返回 405 Method Not Allowed。**
2. Webhook 为什么要先 text() 再 JSON.parse？——**验签需要原始请求体，json() 会消费并改变格式。**
3. proxy.ts 适合做什么？——**语言协商、租户路径粗筛、重写与跳转等轻量路由决策。**
