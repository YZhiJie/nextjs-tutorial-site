# EP.05 缓存与重新验证

> 对应正文：[第 5 章 · 缓存与重新验证](../chapters/05-caching/basics) · 配套实验：[labs/mini-cache](../labs/mini-cache) · 源码深潜：[cache-layers](../deep-dives/cache-layers)

## 剧情回顾

- **第 1 格**：标签君发现 16.x 默认动态，缓存必须明写，可寿命与失效又该怎么管？
- **第 2 格**：码叔拆出三件套：`'use cache'`、`cacheLife`、`cacheTag`。
- **第 3 格**：像素酱实证数据函数缓存与写后 `updateTag` 失效。
- **第 4 格**：码叔提醒：忘记开 `cacheComponents`、不写寿命、拿用户输入当键都会出事。

## 知识点拆解

### 第 1 格 · 动态默认与 cacheComponents

16.x 翻转了缓存模型：数据读取默认在请求时执行，缓存是显式动作。开启 `cacheComponents: true` 后，`'use cache'`、`cacheLife`、`cacheTag` 才生效。

- 构建期会校验无法预渲染的访问
- 未包 Suspense 的动态读取会得到可操作的修复提示

```tsx
fetch(url)                       // 不缓存
fetch(url, { cache: 'force-cache' })
// 16：动态是默认，缓存要显式
// 寿命、失效、标签靠什么管？
```

### 第 2 格 · 两种缓存粒度

- **'use cache'**：缓存函数返回值或整段组件输出
- **cacheLife**：stale / revalidate / expire 三档时间
- **cacheTag**：挂标签，写后按标签精准失效

- 异步函数顶部：缓存返回值，适合数据库查询与聚合
- 异步组件顶部：缓存整段 RSC 输出，适合稳定 UI 块
- 文件顶部：该文件所有导出都被缓存，只适合纯数据模块

### 第 3 格 · cacheLife 与 cacheTag

`cacheLife` 有 `seconds`、`minutes`、`hours`、`days`、`weeks`、`max`、`default` 七档。`cacheTag` 挂逻辑标签，写操作后按标签失效。

- 选档依据是内容变化频率，不是页面重要性
- 不写 cacheLife 会套用隐式 default，读代码时看不出寿命

```tsx
export async function getProducts() {
  'use cache'
  cacheLife('hours')
  cacheTag('products')
  return db.product.findMany()
}
// 写后：updateTag('products')
```

### 第 4 格 · 失效与私有缓存

写后可用 `updateTag` / `revalidateTag` 精准失效；用户私有数据应使用私有缓存或流式，而不是进入跨用户共享缓存。

> ⚠️ 忘记开 cacheComponents → 'use cache' 不生效
> ⚠️ 每个 use cache 都应显式 cacheLife
> ⚠️ 用用户输入做公共缓存键，导致跨用户泄漏

### 延伸 · 跑一遍真实实验

打开 [labs/mini-cache](../labs/mini-cache) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 没开 `cacheComponents` 就写 `'use cache'`，以为已缓存。
2. 所有缓存都不写 `cacheLife`，嵌套时寿命互相影响。
3. 把搜索词、用户 ID 之类输入塞进公共缓存键。

## 自测 3 题（含答案）

1. 16.x 默认缓存数据吗？——**不缓存；动态是默认，缓存需要显式 `'use cache'` 或 fetch 缓存配置。**
2. cacheLife 的档位按什么选？——**按内容变化频率选，例如实时股价 seconds、博客 days。**
3. 写操作后如何让商品列表立即更新？——**给缓存挂 `cacheTag('products')`，写后调用 `updateTag('products')`。**
