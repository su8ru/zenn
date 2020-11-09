---
title: "frourio でログイン処理などを行える Hooks を定義する"
emoji: "🎣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frourio", "TypeScript"]
published: false
---

この記事は frourio チュートリアル連載 第 3 回です。前の記事を読んでいない方は先に以下の記事を読むことをおすすめします！

第 1 回 : [frourio でフロントエンドとバックエンドを一緒に静的型検査する \- Qiita](https://qiita.com/su8ru/items/08d4222af6ddb8eb218b)
第 2 回 : [frourio でサクッと API 型定義 & コントローラーを書く \- Qiita](https://qiita.com/su8ru/items/e4ba6fd311ee3905d174)

# 環境

- OS
  - Ubuntu 20.04 on Windows 10 Pro (WSL2)
  - Ubuntu 20.04 (本物)
- node `v14.14.0`

# 構成

- 基本 Fastify です
  Express で大きく違うところは両方書いてあります
- Next.js です（今回はあまり関係ありませんが）
- DB などは省略し、最小構成です

---

# 概要

frourio には、Fastify 風の Hooks が Fastify / Express ともに再定義されています。

# Hooks の種類とライフサイクル

- `onRequest`
- `preParsing`
- `preValidation`
- `preHandler`

これらは、以下のライフサイクルに沿って hook されます。

第 3 引数として受け取れる `done`(Fastify) / `next`(Express) を呼ぶことで次に進みます。
＊ async / await 使用時の Fastify を除く（[後述](#async--await-%E3%82%92%E4%BD%BF%E3%81%86)）

```
Incoming Request
  │
  └─▶ Routing
         │
  404 ◀─┴─▶ onRequest Hook
               │
    4**/5** ◀─┴─▶ preParsing Hook
                     │
          4**/5** ◀─┴─▶ Parsing
                           │
                4**/5** ◀─┴─▶ preValidation Hook
                                 │
                      4**/5** ◀─┴─▶ Validation
                                       │
                                400 ◀─┴─▶ preHandler Hook
                                             │
                                  4**/5** ◀─┴─▶ User Handler
                                                   │
                                        4**/5** ◀─┴─▶ Outgoing Response
```

出典: [Lifecycle of hooks \| frourio](https://frourio.io/docs/hooks/lifecycle)

# 最小の Hooks

前回と同様、まずは frourio を dev モードで起動しておきましょう。
`hooks.ts` などのファイル作成を検知して、空ファイルの場合にテンプレートを入れてくれます。

```shell:Terminal
yarn dev:frourio
# or
yarn dev # all
```

ためしに、`/api` 直下に `hooks.ts` を作成してみます。

```shell:Terminal
touch server/api/hooks.ts
```

すると、以下のような、`console.log` するだけの `directory level hook`(＊[後述](#directory-level-hooks)) が出来上がります。 

## Fastify の場合

```ts:hooks.ts
import { defineHooks } from './$relay'

export default defineHooks(() => ({
  onRequest: (req, reply, done) => {
    console.log('Directory level onRequest hook:', req.url)
    done()
  }
}))
```

## Express の場合

```ts:hooks.ts
import { defineHooks } from './$relay'

export default defineHooks(() => ({
  onRequest: (req, res, next) => {
    console.log('Directory level onRequest hook:', req.path)
    next()
  }
}))
```

# Hooks の有効範囲

Hooks は、各エンドポイントの `controller.ts` または `hooks.ts` で export することで有効になります。どちらで export するかによって、Hooks の有効範囲が変わります。

## Controller Level Hooks

`controller.ts` で export した場合、その階層でのみ Hooks が有効に
(`controller level hooks`)

```ts:/server/api/tasks/controller.ts
export const hooks = defineHooks(() => ({
  preHandler: (req, reply, done) => {
    console.log("Hi!")
    done()
  }
}))
```

**`export default` ではないので注意**

## Directory Level Hooks

`hooks.ts` で export した場合、配下のディレクトリ全てで Hooks が有効に
(`directory level hooks`)

```ts:/server/api/user/hooks.ts
export default defineHooks((fastify) => ({
  preHandler: fastify.auth([
    (req, reply, done) => {
      ...
      done()
    }
  ])
}))
```

# async / await を使う

## Fastify の場合

```ts:hooks.ts
export default defineHooks(() => ({
  preHandler: async (req) => {
    console.log('[/] Directory Level Hook onRequest hook:', req.url)
    await new Promise((r) => setTimeout(r, 3000))
    console.log('timeout done!')
  }
}))
```

Fastify の場合は本家そのままです。`done` を呼ぶ必要はありません。

## Express の場合

```ts:hooks.ts
export default defineHooks(() => ({
  preHandler: async (req, _, next) => {
    console.log('[/] Directory Level Hook onRequest hook:', req.url)
    await new Promise((r) => setTimeout(r, 3000))
    console.log('timeout done!')
    next()
  }
}))
```

**Express では、async / await を使う場合でも、`next` を呼ぶ必要があります。**

# HTTP メソッドによる場合分け

Hooks は、HTTP メソッドを区別せず、コントローラーを定義した全てのメソッドで有効になります。

一部のメソッドでのみ有効にしたい / 一部のメソッドは弾きたい 場合は、第 1 引数 (`FastifyRequest`) 内に `method` があるのでそれを使います。

```ts:/server/api/tasks/controller.ts
export const hooks = defineHooks(() => ({
  preHandler: (req, reply, done) => {
    if (req.method === 'GET') done() // 早期 done も可能
    if (req.method === 'POST') console.log('POST!')
    done()
  }
}))
```

# 同タイミングの Hooks の呼び出し順

複数箇所で同じタイミングでの Hooks が定義されていた場合、以下の順で呼ばれます。

1. 先祖から引き継いだ `directory level hooks`
2. 自身の階層の `directory level hooks`
3. 自分の `controller level hooks`

```
[/] Directory Level Hook onRequest hook: /api/user
[/user] Directory Level Hook onRequest hook: /api/user
[/user] Controller Level Hook onRequest hook: /api/user
```

Hooks の実際の呼び出し順は `$server.ts` のこのあたりで確認できます。

```ts:/server/$server.ts
// (省略)
import hooksFn0 from './api/hooks'
import hooksFn1 from './api/tasks/_taskId@number/hooks'
import hooksFn2 from './api/user/hooks'
// (省略)
export default (fastify: FastifyInstance, options: FrourioOptions = {}) => {
// (省略)
  const hooks0 = hooksFn0(fastify)
  const hooks1 = hooksFn1(fastify)
  const hooks2 = hooksFn2(fastify)
// (省略)
  fastify.get(
    `${basePath}/user`,
    {
      onRequest: [hooks0.onRequest, hooks2.onRequest],
      preValidation: createValidateHandler(req => [
          validateOrReject(Object.assign(new Validators.TokenHeader(), req.headers as any))
        ]),
      preHandler: ctrlHooks0.preHandler
    },
    methodToHandler(controller4.get)
  )
// (省略)
```

# コントローラーの引数を追加する

`AdditionalRequest` として request に追加したいものを含めて export し、Hooks 内で `req` (第 1 引数) に追加すると、コントローラー内の各メソッドで引数として受け取ることができます。

ちなみに、Hooks の時点では存在が確定していないため、`Partial<AdditionalRequest>` が `req` に含まれます。
コントローラーの引数には `AdditionalRequest` がそのまま含まれます。

＊ サンプルの `/server/api/user/hooks.ts` で型が含まれて無さそうに見えますが、これは `fastify.auth([...])` で囲うことによって `defineHooks` の型推論から外れてしまうためです。

```ts:hooks.ts
// request に追加したいものを含めて export
export type AdditionalRequest = {
  hoge: string
}

export default defineHooks(() => ({
  onRequest: (req, _, done) => {
    req.hoge = 'fuga'
    done()
  }
}))
```
```ts:controller.ts
export default defineController(() => ({
  // 各メソッドで hoge を受け取れる！
  get: ({ hoge }) => ({ status: 200, hoge })
}))
```

# おわりに

ライフサイクルを理解してつよつよになりたい！

あとこないだ Solufa さんの紹介記事がバズってましたね、この調子で frourio が広まって欲しいです～
[憧れのTypeScriptフルスタック環境がコマンド1発で作れる超軽量フレームワーク「frourio」 \- Qiita](https://qiita.com/m_mitsuhide/items/00b139bb565dddf8006a)

(私もバズりてぇ～～～)
