# EP.07 Server Actions 与表单变更

> 对应正文：[第 7 章 · Server Actions](../chapters/07-server-actions/basics) · 配套实验：[labs/server-action-todo](../labs/server-action-todo) · 源码深潜：[action-handler](../deep-dives/action-handler)

## 剧情回顾

- **第 1 格**：标签君一个表单写了接口、校验、loading、错误、刷新四份代码。
- **第 2 格**：码叔用 `'use server'` 把链路收成一个异步函数，表单可直接绑定。
- **第 3 格**：像素酱实证 `useActionState` 的三态与 `useOptimistic` 的乐观更新。
- **第 4 格**：码叔强调 Action 是公开 POST 端点，鉴权与回滚都不能省。

## 知识点拆解

### 第 1 格 · 定义与调用

`'use server'` 可标在文件顶部（所有导出都是 Server Function），也可标在单个异步函数内。底层使用 HTTP POST，客户端组件只能导入模块级 Action。

- 服务端组件表单：`<form action={fn}>`
- 客户端表单：导入 Action，配合 `useActionState`
- 事件触发：在 `startTransition` 中调用

```tsx
onSubmit → fetch('/api/todo')
手写校验 / loading / 错误 / 刷新
接口 + 客户端状态 + 重新取数
一个表单四份代码
```

### 第 2 格 · 表单三态

`useActionState` 返回 `[state, formAction, pending]`。服务端函数签名变为 `(prevState, formData)`，便于回显校验错误与禁用提交按钮。

- **'use server'**：函数只在服务端执行
- **<form action>**：无 JS 也能提交，渐进增强
- **useActionState**：状态 / 包装后的 action / pending

### 第 3 格 · 乐观更新

`useOptimistic` 在请求发出前先更新 UI，失败时由 React 回滚到真实状态。适合点赞、待办勾选等高频轻量操作。

```tsx
'use server'
export async function createTodo(prev, formData) {
  const title = String(formData.get('title'))
  if (!title) return { error: '标题必填' }
  await db.todo.create({ data: { title } })
  revalidatePath('/todos')
}
```

### 第 4 格 · 安全边界

- 每个 Action 内部重新校验会话与权限
- 用 schema 校验输入，不信任 FormData
- 写后按标签或路径失效，避免陈旧数据

> ⚠️ Action 是公开 POST 端点，必须自行鉴权
> ⚠️ 参数必须可序列化，不能传函数或类实例
> ⚠️ 乐观更新失败要回滚，不能假装成功

### 延伸 · 跑一遍真实实验

打开 [labs/server-action-todo](../labs/server-action-todo) 跟随可运行步骤练一遍；遇到诡异现象，回到[排障图谱](../troubleshooting)按症状查根因。

## 坑点清单

1. 把 Action 当成“内部函数”，不在内部做鉴权。
2. 向 Action 传递不可序列化参数或超大对象。
3. 乐观更新没有失败回滚，界面与数据库不一致。

## 自测 3 题（含答案）

1. Server Action 底层使用什么 HTTP 方法？——**POST；也只有 POST 能调用。**
2. useActionState 返回哪三项？——**状态、包装后的 action、pending 布尔值。**
3. 为什么 Action 内部必须鉴权？——**它是公开可调用的 POST 端点，前端隐藏按钮不等于服务端安全。**
