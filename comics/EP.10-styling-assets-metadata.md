# EP.10 样式、静态资源与 Metadata

> 对应正文：[第 10 章 · 样式、资源与 Metadata](../chapters/10-styling-assets/basics) · 配套实验：[labs/styling-assets](../labs/styling-assets)

## 剧情回顾

- **第 1 格**：标签君的样式、字体、图片、meta 各写各的，LCP 与 CLS 一起崩。
- **第 2 格**：码叔梳理四件事：Tailwind v4、next/font、next/image、Metadata API。
- **第 3 格**：像素酱实证 CSS 令牌、构建期字体与 metadata / sitemap / OG。
- **第 4 格**：码叔提醒全局 CSS 边界、图片尺寸与 metadata 覆盖规则。

## 知识点拆解

### 第 1 格 · Tailwind v4

v4 不再需要 `tailwind.config.js` 与三条 `@tailwind` 指令，改用 `@import 'tailwindcss'`，设计令牌写在 `@theme` 中，自动生成 `bg-*`、`text-*` 等实用类。

- PostCSS 只需 `@tailwindcss/postcss`
- 非模块全局样式只能在 root layout 引入

```tsx
@tailwind base/components/utilities  // v3
字体手动 preload，图片直接放大图
<img> 不写尺寸 → CLS 抖动
metadata 散落，OG 图缺失
```

### 第 2 格 · CSS Modules 与 CSS-in-JS

CSS Modules 内置支持，文件名以 `.module.css` 结尾，类名构建期哈希。CSS-in-JS 需要处理 RSC 样式注册，动态主题项目再考虑。

- **Tailwind v4**：CSS 里 @theme 定义设计令牌
- **next/font**：构建期自托管，零布局偏移
- **next/image**：尺寸、懒加载、格式优化

### 第 3 格 · 字体与图片

- `next/font` 构建期下载并自托管，自动生成字体子集与尺寸指标
- `next/image` 需要尺寸或 `fill`，自动懒加载与格式协商
- 首屏主图可加 `priority`，避免 LCP 被懒加载拖慢

```tsx
@import 'tailwindcss'
@theme { --color-brand: #1d6ff2 }
const geist = Geist({ subsets: ['latin'] })
<body className={geist.className}>
export const metadata: Metadata = { title, openGraph }
// sitemap.ts · robots.ts · opengraph-image.tsx
```

### 第 4 格 · Metadata 与 SEO

静态页面导出 `metadata`，动态页面用 `generateMetadata`。`sitemap.ts`、`robots.ts`、`opengraph-image.tsx` 用文件约定生成对应资产。

> ⚠️ 全局 CSS 只能在 root layout 引入
> ⚠️ next/image 必须给 width/height 或 fill
> ⚠️ metadata 与 generateMetadata 不要重复覆盖

### 延伸 · 跑一遍真实实验

打开 [labs/styling-assets](../labs/styling-assets) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 在子布局或页面引入非模块全局 CSS，导航后样式残留。
2. 使用 `next/image` 却不给尺寸，CLS 反而更差。
3. 同时写 `metadata` 与 `generateMetadata`，后者覆盖前者却不自知。

## 自测 3 题（含答案）

1. Tailwind v4 的设计令牌写在哪？——**CSS 文件的 `@theme` 块中，例如 `--color-brand`。**
2. next/font 的主要收益？——**构建期自托管字体并生成尺寸指标，减少请求与布局偏移。**
3. 动态路由怎样生成页面级 metadata？——**导出 `generateMetadata`，基于 params 异步返回。**
