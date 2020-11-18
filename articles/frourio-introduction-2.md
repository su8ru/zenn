---
title: "frourio でサクッと API 型定義 & コントローラーを書く"
emoji: "🎮"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frourio", "aspida", "TypeScript"]
published: false
---

この記事は frourio チュートリアル連載 第 2 回です。前の記事を読んでいない方は先に読むことをおすすめします！

第 1 回 : frourio でフロントエンドとバックエンドを一緒に静的型検査する - [Qiita](https://qiita.com/su8ru/items/08d4222af6ddb8eb218b)
**第 2 回** : frourio でサクッと API 型定義 & コントローラーを書く - [Qiita](https://qiita.com/su8ru/items/e4ba6fd311ee3905d174)
第 3 回 : frourio でログイン処理などを行える Hooks を定義する - [Qiita](https://qiita.com/su8ru/items/5f06dd45ed14117c291f)

# API 定義の基本的な書き方

frourio の API 定義部分は、[aspida](https://github.com/aspida/aspida) と共通仕様になっています。

## パスの指定

まず、パスをディレクトリ階層で表します。

例えばエンドポイントが `/auth/user/` なら、対応する定義は `server/api/auth/user/index.ts` に書くこととなります。

frourio ではパスのディレクトリ内にコントローラーを置くため、**旧来の `/auth/user.ts` のような記法はサポートされていません。**

また、パス変数を含むパス `/tasks/{taskId}/` は、`server/api/tasks/_taskId/index.ts` のようになります。

`_` から始まるパス変数のデフォルトの型は `number | string` です。明示的に指定する場合は変数名の後に `@number` などをつけ、`_taskId@number` と指定します。

## メソッドの型定義

各パスの `index.ts` では、`Methods` type を export します。

実際に `server/api/user/index.ts` を見てみます。

```ts
import { TokenHeader } from '$/validators'
import { UserInfo } from '$/types'

export type Methods = {
  get: {
    reqHeaders: TokenHeader
    resBody: UserInfo
  }

  post: {
    reqHeaders: TokenHeader
    reqFormat: FormData
    reqBody: { icon: Blob }
    resBody: UserInfo
  }
}
```

まず、HTTP メソッド ( GET とか POST とか ) を key として書き、その中にリクエストやレスポンスの型を羅列していきます。

| key | 説明 |
| --- | --- |
| `reqHeaders` | ヘッダー。`{ Authorization: string }` とかそういうのを指定することが多いです。 |
| `reqFormat` | リクエストのフォーマット。`FormData` とかを送信したいときには明示する必要があります。 |
| `query` | リクエストクエリ。別名 URL パラメーター。クエリも型指定できます。 |
| `reqBody` | リクエストデータの中身。 |
| `resBody` | レスポンスデータの中身。aspida のレスポンスの型はこれになります。 |

詳しくは [aspida の README](https://github.com/aspida/aspida/tree/master/packages/aspida/docs/ja#tips) を見てください。

## 型定義を別ファイルに置いておく

aspida 単体での使用時は `@types.ts` などを作ることが多かったのですが、frourio では `server/types/` が用意されているのでここを使いましょう。
`$/types/` で参照できます。

#  コントローラーの作成

## 下準備 ( frourio を dev モードで起動 )

ディレクトリを作ってコントローラーを書き始める前に、frourio を dev モードで起動しておきます。

```:Terminal
yarn dev:frourio
```

## ディレクトリの作成と自動生成

では、準備もできたところで `server/api/` に `memo/` を作ってみます。

すると、いま作ったディレクトリの下に `index.ts` `$relay.ts`、そして本題の `controller.ts` の 3 つのファイルが自動生成されました。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225023/baf034df-cc8d-ae30-8585-ecb7502bc961.png)

先程 frourio を dev モードで起動させましたが、起動中は

- `server/api/` にディレクトリが追加されたらファイル群を自動生成
- `server/api/` に変更があったら `$server.ts` を自動生成

の 2 つの監視をしてくれます。

### `index.ts`

```ts
export type Methods = {
  get: {
    resBody: string
  }
}
```

先程触れたように、API リクエストの型定義を記述します。

### `$relay.ts`

Express とコントローラーとの架け橋です。`$` から始まるファイルは自動生成なので、触ることはありません。

### `controller.ts`

```ts
import { defineController } from './$relay'

export default defineController(() => ({
  get: () => ({ status: 200, body: 'Hello' })
}))
```

ここには、リクエストを受け取った際のコントローラーとしての処理を記述します。

各 HTTP メソッドに対してレスポンスデータを返す関数を指定し、それを `defineController()` の引数に渡すことで、コントローラーが `$server.ts` に定義されます。

`$server.ts` 抜粋:

```ts
app.get(`${basePath}/memo`, methodsToHandler(controller1.get))
```

Express のルート定義が自動生成されていることがわかります。

# コントローラーの実装

## DB に書き込んでそのままレスポンス

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

## なんかしてからレスポンス

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

## 関数の引数

さて、上の例を見ると、関数が引数を受け取っていることがわかります。

受け取れるのは `path, method, query, body, headers` の 5 つで、名前の通りの中身が入っています。

# $api.ts と $server.ts の中身

`$` が付いているファイルは自動生成なので、自分の手でいじることはありません。
しかし、中身がどうなっているかを確認しておくのは悪いことではないはずです。さて、見てみましょう！

## `$api.ts`

このファイルは **aspida** による自動生成です。`dev:aspida` で変更検知→生成してくれます。

中身は単純で、各パス→メソッドへと入れ子になったオブジェクト `api` があるだけです。

ちなみに `server/api/` 以下の各定義部分に JSDoc を記述すると、ちゃんとこのファイルに出力されます。
[sample: afes-website/docs](https://github.com/afes-website/docs)

## `$server.ts`

このファイルは **frourio** による自動生成です。`dev:frourio` で変更検知→生成してくれます。

こっちはちょっとボリュームが多いですが、メインは後半の Express ルーティング部分です。

```ts
...

export default (app: Express, options: FrourioOptions = {}) => {
  const basePath = options.basePath ?? ''
  const uploader = multer(
    options.multer ?? {
      dest: path.join(__dirname, '.upload'),
      limits: { fileSize: 1024 ** 3 }
    } // 邪魔なので整形しました
  ).any()

  app.get(`${basePath}/`, methodsToHandler(controller0.get))

  app.get(`${basePath}/memo`, methodsToHandler(controller1.get))

  ...
```

Express のルーティングは以下の構造で定義されます。

```ts
app.METHOD(PATH, HANDLER)
```
出典･詳細: [Express の基本的なルーティング - Express.js](https://expressjs.com/ja/starter/basic-routing.html)

つまり、`methodsToHandler` で各コントローラー内の各メソッドを取り出し、Express のルーティングとして定義しているわけです。

バリデータが絡むと少しややこしくなりますが、関数名の通りの挙動を追っていくと意外と読みやすいです。

```ts
  app.post(`${basePath}/token`, [
    parseJSONBoby,
    createValidateHandler(req => [
      validateOrReject(Object.assign(new Validators.LoginBody(), req.body))
    ]),
    methodsToHandler(controller4.post)
  ])

  app.delete(`${basePath}/token`, [
    createValidateHandler(req => [
      validateOrReject(Object.assign(new Validators.TokenHeader(), req.headers))
    ]),
    methodsToHandler(controller4.delete)
  ])
```

# おわりに

aspida の型定義の中にコントローラーも書けるのは理にかなっていて、管理とかも楽そうです。

次回は Hooks についての予定です！乞うご期待！

第 1 回 : frourio でフロントエンドとバックエンドを一緒に静的型検査する - [Qiita](https://qiita.com/su8ru/items/08d4222af6ddb8eb218b)
**第 2 回** : frourio でサクッと API 型定義 & コントローラーを書く - [Qiita](https://qiita.com/su8ru/items/e4ba6fd311ee3905d174)
第 3 回 : frourio でログイン処理などを行える Hooks を定義する - [Qiita](https://qiita.com/su8ru/items/5f06dd45ed14117c291f)
