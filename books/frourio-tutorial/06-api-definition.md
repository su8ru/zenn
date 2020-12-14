---
title: "API 定義の基本的な書き方"
---

frourio の API 定義部分は、[aspida](https://github.com/aspida/aspida) と共通仕様になっています。

# パスの指定

まず、パスをディレクトリ階層で表します。

例えばエンドポイントが `/auth/user/` なら、対応する定義は `server/api/auth/user/index.ts` に書くこととなります。

:::message
frourio ではパスのディレクトリ内にコントローラーを置くため、旧来の `/auth/user.ts` のような記法はサポートされていません。
:::

また、パス変数を含むパス `/tasks/{taskId}/` は、`server/api/tasks/_taskId/index.ts` のようになります。

`_` から始まるパス変数のデフォルトの型は `number | string` です。明示的に指定する場合は変数名の後に `@number` などをつけ、`_taskId@number` と指定します。

# メソッドの型定義

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

# 型定義を別ファイルに置いておく

aspida 単体での使用時は `@types.ts` などを作ることが多かったのですが、frourio では `server/types/` が用意されているのでここを使いましょう。
`$/types/` で参照できます。