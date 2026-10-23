# EP.02 路由与导航

> 对应正文：[第 2 章 · 路由与导航](../chapters/02-routing/basics) · 配套实验：[labs/routing-nested](../labs/routing-nested) · 源码深潜：[router-prefetch](../deep-dives/router-prefetch)

## 剧情回顾

- **第 1 格**：标签君受够了手写路由表——布局一切换，客户端状态就丢。
- **第 2 格**：码叔指出 `app/` 目录本身就是路由：`page` 产 URL，`layout` 保状态，`template` 每次重挂载。
- **第 3 格**：像素酱实证路由组、并行路由与拦截路由：URL 与布局可以解耦。
- **第 4 格**：码叔提醒三个高频坑：`page` 之外不产 URL、并行路由缺 `default.tsx`、软导航与刷新行为不同。

## 知识点拆解

### 第 1 格 · page、layout、template

只有 `page.tsx` 会成为可访问 URL 的叶子；`layout.tsx` 包裹子段且导航时不重渲染，`template.tsx` 结构相同但每次导航重新挂载。

- 需要保留搜索框、侧边栏状态 → layout
- 需要每次进入重置的表单或动画 → template

```tsx
// 传统：手写路由表
router.get('/blog/:slug', handler)
// 切换页面 → layout 卸载
// 404、嵌套布局都要自己兜底
```

### 第 2 格 · 动态段与路由组

- **page.tsx**：路由叶子，只有它产生 URL
- **layout.tsx**：共享外壳，导航时不重渲染
- **template.tsx**：每次进入都重新挂载

- `[slug]` 单段、`[...slug]` 捕获全部、`[[...slug]]` 可选捕获
- `(group)` 只影响布局分组，不出现在 URL 中
- `_folder` 是私有目录，不参与路由

### 第 3 格 · 并行与拦截路由

`@slot` 让同一布局并行渲染多个区域，每个 slot 需要 `default.tsx` 兜底；`(.)`、`(..)`、`(...)` 分别拦截同级、上一级与根级路由。

- 软导航时展示弹层或局部区域
- 直接刷新或分享链接时回退到完整页面

```tsx
app/(marketing)/page.tsx      // URL 仍是 /
app/@modal/(.)photo/[id]/page.tsx
// 软导航弹层，刷新回到整页
const { slug } = await params  // 16：Promise
```

### 第 4 格 · Link 与预取

`<Link>` 提供客户端跳转与视口内预取。16.3 开启 `partialPrefetching` 后，默认只预取共享 App Shell，再由 Suspense 流式补入动态区域。

> ⚠️ layout 不会产生 URL，别拿它当页面
> ⚠️ 并行路由缺少 default.tsx → 404
> ⚠️ 拦截路由刷新后行为与软导航不同

### 延伸 · 跑一遍真实实验

打开 [labs/routing-nested](../labs/routing-nested) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 把 `layout.tsx` 当成页面，期待它产生 URL。
2. 并行路由只写 `@slot/page.tsx`，漏掉 `default.tsx`。
3. 用 `<a>` 做站内跳转，丢掉客户端导航与预取。

## 自测 3 题（含答案）

1. `layout` 与 `template` 的关键区别？——**导航时 layout 保留实例与状态，template 每次都会重新挂载。**
2. 路由组 `(marketing)` 会出现在 URL 里吗？——**不会；它只用于组织布局与代码。**
3. 拦截路由刷新页面后会发生什么？——**直接访问时拦截不生效，回退到目标路由的完整页面。**
