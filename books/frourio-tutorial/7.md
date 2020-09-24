---
title: "コントローラーの作成"
---

# 下準備 ( frourio を dev モードで起動 )

ディレクトリを作ってコントローラーを書き始める前に、frourio を dev モードで起動しておきます。

```:Terminal
yarn dev:frourio
```

# ディレクトリの作成と自動生成

では、準備もできたところで `server/api/` に `memo/` を作ってみます。

すると、いま作ったディレクトリの下に `index.ts` `$relay.ts`、そして本題の `controller.ts` の 3 つのファイルが自動生成されました。

![](https://storage.googleapis.com/zenn-user-upload/n355k46qbpelv3zadw3o1il99r78)

先程 frourio を dev モードで起動させましたが、起動中は

- `server/api/` にディレクトリが追加されたらファイル群を自動生成
- `server/api/` に変更があったら `$server.ts` を自動生成

の 2 つの監視をしてくれます。

## `index.ts`

```ts
export type Methods = {
  get: {
    resBody: string
  }
}
```

先程触れたように、API リクエストの型定義を記述します。

## `$relay.ts`

Express とコントローラーとの架け橋です。`$` から始まるファイルは自動生成なので、触ることはありません。

## `controller.ts`

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
