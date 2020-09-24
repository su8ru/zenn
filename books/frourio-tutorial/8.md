---
title: "コントローラーの実装"
---

# DB に書き込んでそのままレスポンス

`server/api/task/controller.ts`

```ts
import { defineController } from './$relay'
import { findAllTask, createTask } from '$/service/tasks'

export default defineController(() => ({
  get: async () => ({ status: 200, body: await findAllTask() }),
  post: async ({ body }) => ({
    status: 201,
    body: await createTask(body.label)
  })
}))
```

`createTask()` の返り値は `Task` なので、async / await を用いてそのまま返す実装ができます。

# なんかしてからレスポンス

`server/api/task/_taskId@number/controller.ts`

```ts
import { defineController } from './$relay'
import { updateTask, removeTask } from '$/service/tasks'

export default defineController(() => ({
  patch: async ({ body, params }) => {
    await updateTask(params.taskId, body)
    return { status: 204 }
  },
  delete: async ({ params }) => {
    await removeTask(params.taskId)
    return { status: 204 }
  }
}))
```

すぐに return しないといけないわけではありません。関数内でなにか処理をしてからレスポンスすることも可能です。

# 関数の引数

さて、上の例を見ると、関数が引数を受け取っていることがわかります。

受け取れるのは `path, method, query, body, headers` の 5 つで、名前の通りの中身が入っています。
