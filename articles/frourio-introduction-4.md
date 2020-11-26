---
title: "frourio × aspida で 4 種類のバリデーションを実装する"
emoji: "✅"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frourio", "aspida", "TypeScript"]
published: false
---

この記事は frourio チュートリアル連載 第 4 回です。前の記事を読んでいない方は先に読むことをおすすめします！

第 1 回 : frourio でフロントエンドとバックエンドを一緒に静的型検査する - [Qiita](https://qiita.com/su8ru/items/08d4222af6ddb8eb218b)
第 2 回 : frourio でサクッと API 型定義 & コントローラーを書く - [Qiita](https://qiita.com/su8ru/items/e4ba6fd311ee3905d174)
第 3 回 : frourio でログイン処理などを行える Hooks を定義する - [Qiita](https://qiita.com/su8ru/items/5f06dd45ed14117c291f)
**第 4 回** : frourio × aspida で 4 種類のバリデーションを実装する - [Qiita](https://qiita.com/su8ru/items/c52b3e3b80edbc0363fa)

# 4 種類のバリデーション

frourio では、以下の 4 種類のバリデーションを定義することができます。

- Path parameter
- URL query
- JSON body
- Custom validation

# Path parameter

パス変数名の後ろに `@string` または `@number` を指定可能です。
`@number` の場合、自動的にバリデートされます。

指定しない場合、パス変数は **`string`** になります。
＊ aspida = フロントエンドのデフォルトは `number | string` ですが、サーバーには全て `string` で来るため、情報劣化を防ぐために `string` のみになっています。

> frourio から見ても `number | string` だと…？
>
> | aspida  | -> string ->  | frourio         |
> | :-:     | :-:           | :-:             |
> | `'a'`   | -> `'a'` ->   | `'a'`           |
> | `'1'`   | -> `'1'` ->   | `1` ? `'1'` ?   |
> | `1`     | -> `'1'` ->   | `1` ? `'1'` ?   |
> | `'1.0'` | -> `'1.0'` -> | `1` ? `'1.0'` ? |
>
> このように複数の選択肢が生じ、`number` を選ぶと情報劣化が起こりうるのです。

```ts:server/api/tasks/_taskId@number/index.ts
import { Task } from '$/types'

export type Methods = {
  get: {
    resBody: Task
  }
}
```

```ts:server/api/tasks/_taskId@number/controller.ts
import { defineController } from './$relay'
import { findTask } from '$/service/tasks'

export default defineController(() => ({
  get: async ({ params }) => {
    const task = await findTask(params.taskId)

    return task ? { status: 200, body: task } : { status: 404 }
  }
}))
```

```shell:Result
$ curl http://localhost:8080/api/tasks
[{"id":0,"label":"sample task","done":false}]

$ curl http://localhost:8080/api/tasks/0
{"id":0,"label":"sample task","done":false}

$ curl http://localhost:8080/api/tasks/1 -i
HTTP/1.1 404 Not Found

$ curl http://localhost:8080/api/tasks/abc -i
HTTP/1.1 400 Bad Request
```

サンプル出典: [Path parameter \| frourio](https://frourio.io/docs/validation/path-parameter/)

# URL query

`string`, `string[]`, `number`, `number[]` を指定可能です。

`number` または `number[]` を指定すると、自動的にバリデートされます。

```ts:server/api/tasks/index.ts
import { Task } from '$/types'

export type Methods = {
  get: {
    query?: {
      limit: number
    }
    resBody: Task[]
  }
}
```

```ts:server/api/tasks/controller.ts
import { defineController } from './$relay'
import { getTasks } from '$/service/tasks'

export default defineController(() => ({
  get: async ({ query }) => ({
    status: 200,
    body: (await getTasks()).slice(0, query?.limit)
  })
}))
```

```shell:Result
$ curl http://localhost:8080/api/tasks
[{"id":0,"label":"sample task 0","done":false},{"id":1,"label":"sample task 1","done":false},{"id":1,"label":"sample task 2","done":false}]

$ curl http://localhost:8080/api/tasks?limit=1
[{"id":0,"label":"sample task 0","done":false}]

$ curl http://localhost:8080/api/tasks?limit=abc -i
HTTP/1.1 400 Bad Request
```

サンプル出典: [URL query \| frourio](https://frourio.io/docs/validation/url-query)

# JSON body

`reqFormat` を指定しない場合、reqBody は `application/json` としてパースされます。
何らかの `reqFormat` が指定されていた場合、それに沿ってパースされます。

フォーマットが正しいか、パースが成功したか、この 2 点でバリデートされます。

```ts:server/api/tasks/index.ts
import { Task } from '$/types'

export type Methods = {
  post: {
    reqBody: Pick<Task, 'label'>
    resBody: Task
  }
}
```

```ts:server/api/tasks/controller.ts
import { defineController } from './$relay'
import { createTask } from '$/service/tasks'

export default defineController(() => ({
  post: async ({ body }) => {
    const task = await createTask(body.label)

    return { status: 201, body: task }
  }
}))
```

```shell:Result
$ curl -X POST -H "Content-Type: application/json" -d '{"label":"sample task3"}' http://localhost:8080/api/tasks
{"id":3,"label":"sample task 3","done":false}

$ curl -X POST -H "Content-Type: application/json" -d '{Invalid JSON}' http://localhost:8080/api/tasks -i
HTTP/1.1 400 Bad Request
```

サンプル出典: [JSON body \| frourio](https://frourio.io/docs/validation/json-body)

# Custom validation

[class-validator](https://github.com/typestack/class-validator) の作法通りに class を定義して、**`server/validators/index.ts` から export** します。

`server/validators/userAuth.ts` みたいなのを作って、`export * from './userAuth'` みたいにするのが現実的な運用でしょうか。

```ts:server/validators/index.ts
import { MinLength, IsString } from 'class-validator'

export class LoginBody { // <-- これが
  @MinLength(2)
  id: string

  @MinLength(4)
  pass: string
}

export class TokenHeader {
  @IsString()
  @MinLength(10)
  token: string
}
```

API 定義の `reqBody`, `reqHeaders`, `query` として使うと、自動的にバリデートされます。

```ts:server/api/token/index.ts
import { LoginBody, TokenHeader } from '$/validators'

export type Methods = {
  post: {
    reqBody: LoginBody // <-- ここ
    resBody: {
      token: string
    }
  }

  delete: {
    reqHeaders: TokenHeader
  }
}
```

```shell:Result
$ curl -X POST -H "Content-Type: application/json" -d '{"id":"correctId","pass":"correctPass"}' http://localhost:8080/api/token
{"token":"XXXXXXXXXX"}

$ curl -X POST -H "Content-Type: application/json" -d '{"id":"abc","pass":"12345"}' http://localhost:8080/api/token -i
HTTP/1.1 400 Bad Request

$ curl -X POST -H "Content-Type: application/json" -d '{"id":"incorrectId","pass":"incorrectPass"}' http://localhost:8080/api/token -i
HTTP/1.1 401 Unauthorized
```

サンプル出典: [Custom validation \| frourio](https://frourio.io/docs/validation/custom-validation)

Custom validation は、`preValidation Hooks` の最後に呼び出されます。
ユーザー定義 Hooks と被った場合でも大丈夫。

```ts:server/$server/ts
  fastify.post(
    `${basePath}/token`,
    {
      onRequest: hooks0.onRequest,
      preValidation: [
        hooks2.preValidation, // something user defined
        createValidateHandler(req => [
          validateOrReject(Object.assign(new Validators.LoginBody(), req.body as any))
        ])
      ]
    },
    methodToHandler(controller3.post)
  )
```

# おわりに

API 定義にバリデーションクラスを直接書けるのは便利ですね、API 仕様とコントローラーが分離しててわかりやすいです！

第 1 回 : frourio でフロントエンドとバックエンドを一緒に静的型検査する - [Qiita](https://qiita.com/su8ru/items/08d4222af6ddb8eb218b)
第 2 回 : frourio でサクッと API 型定義 & コントローラーを書く - [Qiita](https://qiita.com/su8ru/items/e4ba6fd311ee3905d174)
第 3 回 : frourio でログイン処理などを行える Hooks を定義する - [Qiita](https://qiita.com/su8ru/items/5f06dd45ed14117c291f)
**第 4 回** : frourio × aspida で 4 種類のバリデーションを実装する - [Qiita](https://qiita.com/su8ru/items/c52b3e3b80edbc0363fa)
