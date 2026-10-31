# EP.13 测试、可观测与部署

> 对应正文：[第 13 章 · 测试、可观测与部署](../chapters/13-testing-deployment/basics) · 配套实验：[labs/observe-pipeline](../labs/observe-pipeline)

## 剧情回顾

- **第 1 格**：标签君的 async 服务端组件在 Vitest 里跑不起来，E2E 又跑在 dev server 上。
- **第 2 格**：码叔给出测试金字塔：Vitest 测纯逻辑与同步组件，Playwright 测生产构建的关键路径。
- **第 3 格**：像素酱实证 `instant()` 断言、OTel 与上线检查。
- **第 4 格**：码叔提醒别硬渲染 async RSC、E2E 要跑 build 产物、CI 要缓存 `.next/cache`。

## 知识点拆解

### 第 1 格 · 测试金字塔

- 单元：Vitest 测纯函数、校验、工具方法
- 组件：Vitest + Testing Library 测同步客户端组件
- E2E：Playwright 覆盖关键路径

```tsx
async RSC 在 Vitest 里渲染失败
E2E 跑 dev server，行为不接近线上
上线才发现即时导航退化
没有 OTel，排障只能靠猜
```

### 第 2 格 · async 服务端组件

Vitest 目前不支持直接渲染 async 服务端组件。把取数、鉴权、格式化抽成纯函数做单测，整页行为交给 E2E。

- **Vitest**：纯函数、schema、同步客户端组件
- **Playwright**：生产构建上的关键用户路径
- **instant()**：CI 中防止即时导航退化

### 第 3 格 · 即时导航断言与可观测

`@next/playwright` 的 `instant()` 断言把即时性约束搬进 CI。OTel 与结构化错误上报让线上排障从“猜”变成“查”。

```tsx
test('导航即时', async ({ page }) => {
  await page.goto('/')
  await page.getByRole('link', { name: 'About' }).click()
  await expect(page).toHaveURL('/about')
  await instant(page)  // @next/playwright
})
```

### 第 4 格 · 部署检查

- E2E 的 `webServer` 用 `build && start`
- CI 缓存 `.next/cache` 加速重复构建
- 自托管注意 Node 版本、环境变量与流式响应

> ⚠️ 用 Vitest 硬渲染 async 服务端组件
> ⚠️ E2E 跑 dev 而非 build 产物
> ⚠️ CI 不缓存 .next/cache，构建重复变慢

### 延伸 · 跑一遍真实实验

打开 [labs/observe-pipeline](../labs/observe-pipeline) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 在 Vitest 里硬渲染 async 服务端组件，得到无法解释的失败。
2. E2E 对着 dev server 跑，错过生产构建差异。
3. CI 每次冷构建，Turbopack 磁盘缓存收益归零。

## 自测 3 题（含答案）

1. async 服务端组件应该怎么测？——**用 Playwright E2E 覆盖；可抽出的纯逻辑单独单测。**
2. E2E 应跑在什么服务上？——**生产构建产物（`pnpm build && pnpm start`）。**
3. instant() 的作用？——**在 CI 中断言导航的即时性，防止性能退化。**
