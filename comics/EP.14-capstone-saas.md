# EP.14 综合实战：全栈 SaaS

> 对应正文：[第 14 章 · 全栈 SaaS 实战](../chapters/14-capstone/basics) · 配套实验：[labs/debug-cache-stale](../labs/debug-cache-stale) · 源码深潜：[ppr-postponed-state](../deep-dives/ppr-postponed-state)

## 剧情回顾

- **第 1 格**：标签君上线前夜发现租户 ID 来自 searchParams，营销页与后台还共用一套缓存。
- **第 2 格**：码叔给出蓝图：路径段租户、三道隔离、按数据特征分层缓存。
- **第 3 格**：像素酱实证租户上下文与查询强制前缀，再谈 i18n、PWA 与 AI 协作。
- **第 4 格**：码叔提醒别信前端租户、别把 Next 当完整后端、标签里别放敏感信息。

## 知识点拆解

### 第 1 格 · 多租户隔离

租户识别可用独立域名、子域名或路径段；中小团队从 `/[tenant]` 起步最省心。真正的安全来自会话、DAL 查询条件与数据库 RLS 三层强制。

- 会话载荷带 tenantId，登录时确定
- DAL 不导出可绕过租户条件的裸查询
- 数据库唯一约束或 RLS 兜底

```tsx
const tenant = searchParams.get('tenant')  // 危险
营销页与后台同一套缓存策略
i18n 文案散落，PWA 离线空白
AI 智能体没有项目上下文
```

### 第 2 格 · 路由与缓存分层

营销区、应用区用路由组分开；缓存按数据特征分层：公共内容可进静态外壳，租户共享数据带 tenantId 标签，用户私有数据用私有缓存或流式，实时数据只走 Stream。

- **/t/[tenant]**：证书与 DNS 复杂度最低
- **三道隔离**：会话 → DAL → 数据库 RLS
- **缓存分层**：公共 / 租户 / 私有 / 实时

### 第 3 格 · i18n、PWA 与 AI 协作

- 语言协商放在 proxy，文案与路由段统一管理
- PWA 离线壳与缓存策略要和框架缓存区分
- AI 智能体通过 AGENTS.md、Skills 与项目文档获得上下文

```tsx
export const getTenantContext = cache(async () => {
  const user = await getCurrentUser()
  return { tenantId: user.tenantId, role: user.role }
})
export async function listProjects() {
  const { tenantId } = await getTenantContext()
  return queryProjectsByTenant(tenantId)
}
```

### 第 4 格 · BFF 边界

Next.js 可以承担面向前端的编排层，但长任务、消息队列、独立扩缩容的服务仍应留在独立后端。把边界写进架构图，比事后拆服务便宜。

> ⚠️ 用客户端传入的 tenant 做归属判断
> ⚠️ 把 Next 当完整后端（长任务 / 队列应独立）
> ⚠️ 缓存标签里放租户名 / 用户邮箱

### 延伸 · 跑一遍真实实验

打开 [labs/debug-cache-stale](../labs/debug-cache-stale) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 用 `searchParams.tenant` 或表单字段决定数据归属。
2. 把 Next.js 当完整后端，塞进长任务与队列。
3. 缓存标签写入租户名、用户邮箱等敏感信息。

## 自测 3 题（含答案）

1. 租户身份应该来自哪里？——**服务端会话；前端参数只能用于非敏感界面偏好。**
2. 租户隔离的三道防线？——**会话载荷、DAL 查询条件、数据库约束或 RLS。**
3. 哪些能力应留在独立后端？——**长任务、消息队列、独立扩缩容与复杂域逻辑。**
