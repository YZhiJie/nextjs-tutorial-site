# EP.11 认证、授权与安全

> 对应正文：[第 11 章 · 认证、授权与安全](../chapters/11-auth-security/basics) · 配套实验：[labs/debug-auth-bypass](../labs/debug-auth-bypass)

## 剧情回顾

- **第 1 格**：标签君把 JWT 放 localStorage，XSS 一行就能偷走，Action 与 API 还在裸奔。
- **第 2 格**：码叔拆开认证、会话、授权三层，并给出 Cookie 的四个安全属性。
- **第 3 格**：像素酱实证 httpOnly 会话与 DAL 查询级授权。
- **第 4 格**：码叔强调最小载荷、服务端校验与私有缓存。

## 知识点拆解

### 第 1 格 · 三个概念

认证确认身份，会话在请求间记住登录状态，授权决定能做什么。Next.js 不是认证框架，生产建议使用成熟库，但落点仍是 Cookie + DAL + proxy。

```tsx
localStorage.setItem('token', jwt)
// XSS 一行就能偷走
if (!user) redirect('/login')  // 只挡页面
// Action / route.ts 仍可被直接调用
```

### 第 2 格 · 会话 Cookie 属性

- **认证**：校验凭证，签发最小会话
- **会话**：httpOnly / secure / sameSite
- **授权**：DAL 每个查询都带角色与归属

- `httpOnly`：客户端 JS 读不到，降低 XSS 窃取风险
- `secure`：仅 HTTPS 发送
- `sameSite: 'lax'`：挡住大部分 CSRF
- `expires/maxAge` 与 `path`：限定寿命与作用路径

### 第 3 格 · DAL 授权

数据访问层是最终判定点：每个查询都从会话取 userId / role / tenantId，并写入 where 条件。不要导出可绕过授权的裸查询函数。

```tsx
import 'server-only'
const cookieStore = await cookies()
cookieStore.set('session', jwt, {
  httpOnly: true, secure: true, sameSite: 'lax'
})
// DAL：getCurrentUser() → 角色 → where 条件
```

### 第 4 格 · 缓存时代的鉴权

公共缓存只能放与用户无关的数据；用户私有内容用私有缓存或 Suspense 流式。CSP、输入校验与最小权限同样是上线清单的一部分。

> ⚠️ Cookie 里放手机号 / 邮箱 / 密码
> ⚠️ 只在前端隐藏按钮，服务端不校验
> ⚠️ 用户私有数据未用 private 缓存或流式

### 延伸 · 跑一遍真实实验

打开 [labs/debug-auth-bypass](../labs/debug-auth-bypass) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 把会话 JWT 放进 localStorage，暴露给任意脚本。
2. 只在页面做 `redirect`，忘记在每个 Action 与 Handler 内校验。
3. 把用户私有数据写进跨用户共享缓存。

## 自测 3 题（含答案）

1. 会话 Cookie 最重要的三个属性？——**httpOnly、secure、sameSite。**
2. 为什么授权要放在 DAL？——**所有数据访问都经过 DAL，能保证每个查询都带用户与角色条件。**
3. 用户私有数据能用公共 'use cache' 吗？——**不能；应使用私有缓存或流式，避免跨用户泄漏。**
