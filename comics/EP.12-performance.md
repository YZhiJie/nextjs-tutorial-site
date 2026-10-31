# EP.12 性能优化体系

> 对应正文：[第 12 章 · 性能优化体系](../chapters/12-performance/basics) · 配套实验：[labs/observe-pipeline](../labs/observe-pipeline)

## 剧情回顾

- **第 1 格**：标签君凭感觉上 memo，bundle 没降、LCP 没变，也没有基线。
- **第 2 格**：码叔坚持度量先行：LCP / INP / CLS，实验室回归 + 字段决策。
- **第 3 格**：像素酱实证 Web Vitals 上报与分层定位：TTFB 找服务端，INP 找客户端。
- **第 4 格**：码叔提醒 Compiler 不减少 bundle，别乱拆包，也别只看 Lighthouse。

## 知识点拆解

### 第 1 格 · 三个核心指标

LCP 衡量首屏主内容出现时间，INP 衡量交互响应，CLS 衡量布局偏移。TTFB 与 FCP 是辅助指标，用来判断瓶颈在服务端还是客户端。

- 实验室数据用于本地回归
- 字段数据用于真实决策

```tsx
// 先上 useMemo / React.memo
bundle 没降，LCP 没变
// 不知道慢在服务端还是客户端
// 没有基线，无法判断是否回归
```

### 第 2 格 · React Compiler 与 Turbopack

React Compiler 自动插入记忆化，减少引用变化导致的重渲染；它不修复 O(n²) 计算，也不减少 bundle。Turbopack 在 16.4 默认开启内存回收与 build 磁盘缓存。

- **LCP**：首屏主内容：服务端、图片、字体
- **INP**：交互跟手：JS 体积、长任务
- **CLS**：布局稳定：尺寸预留、字体切换

- 历史项目可用注解模式逐组件灰度
- CI 缓存 `.next/cache` 才能吃到重复构建收益

### 第 3 格 · Bundle 与预取

- 大依赖用 `next/dynamic` 延迟加载，但避免拆得过碎
- 图片、字体、首屏脚本优先于“全面懒加载”
- `partialPrefetching` 减少无效整页预取

```tsx
'use client'
import { useReportWebVitals } from 'next/web-vitals'
useReportWebVitals(m => sendBeacon('/vitals', m))
// 先定位层：TTFB → 服务端；INP → 客户端
// 再动 bundle / 图片 / 字体 / 预取
```

### 第 4 格 · 优化闭环

建立基线 → 定位层（服务端/客户端/网络）→ 单点改动 → 回归同一指标。没有基线的“优化”无法证明有效，也无法防止退化。

> ⚠️ 开 Compiler 后立刻删掉所有手写 memo
> ⚠️ 只看 Lighthouse，不看真实用户字段数据
> ⚠️ dynamic import 拆得太碎，请求数反而增加

### 延伸 · 跑一遍真实实验

打开 [labs/observe-pipeline](../labs/observe-pipeline) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 把 React Compiler 当成 bundle 优化工具。
2. 只用实验室分数决策，忽略中端机与弱网字段数据。
3. 为了“按需加载”把页面拆成大量小 chunk，请求数反增。

## 自测 3 题（含答案）

1. LCP、INP、CLS 分别衡量什么？——**首屏主内容出现、交互响应、布局偏移。**
2. TTFB 差应该先查哪一层？——**服务端：取数、渲染或基础设施，而不是客户端 bundle。**
3. React Compiler 能减少 bundle 吗？——**不能；它优化重渲染，不改变包体积。**
