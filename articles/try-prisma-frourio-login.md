---
title: "最近話題の TS 製 ORM「Prisma 2」でログイン処理を試してみたら超快適だった"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["typescript", "prisma", "frourio", "アドベントカレンダー"]
published: true
---

# はじめに

この記事は、[Qiita: TypeScript Advent Calendar 2020](https://qiita.com/advent-calendar/2020/typescript) 24 日目の記事です 🎄✨

今回は、[Front\-End Study \#1「Cloud Native時代のフロントエンド」 \- connpass](https://forkwell.connpass.com/event/190313/) で紹介されていた [Prisma 2](//github.com/prisma/prisma) を、同じく紹介されていた [frourio](//github.com/frouriojs/frourio) を使って試してみます。

@[youtube](JUz_QUu8Mus)

https://zenn.dev/mizchi/articles/c638f1b3b0cd239d3eea

# 環境構築

## frourio で一気に

まずは [create-frourio-app](//github.com/frouriojs/create-frourio-app) でサクッと環境構築します。

```text:Terminal
yarn create frourio-app try-prisma
```

今回は以下の構成で試してみます。
MySQL を建てるのが面倒な場合は、SQLite でもいいと思います。

- **Core framework of frourio : `Fastify`**
- **Frontend framework : `Next.js`**
- Building mode : `Basic`
- HTTP client of aspida : `axios`
- Package manager : `Yarn`
- Deamon process manager : `None`
- **O/R mapping tool : `Prisma`**
- Database type of Prisma : `MySQL`
- Testing framework : `None`

少し待つだけで **Next.js × Fastify × frourio × Prisma** の環境の出来上がりです！

:::details WSL 2 で create に失敗することがある
一部の Windows Subsystem for Linux 2 環境において、GUI が自動的に開かないというバグが存在します。
エラーが出ても、`localhost:3000` を手動で開くことでセットアップを継続できます。
[WSL 2 で GUI が自動的に開かない · Issue \#72 · frouriojs/create\-frourio\-app](https://github.com/frouriojs/create-frourio-app/issues/72)

また、CUI でのセットアップも可能です。
[Setup with CUI \| frourio](https://frourio.io/docs/installation/cui)
:::

## 依存関係を入れる

続いて `bcryptjs` を依存関係に追加します。
追加先は `server/package.json` であることに注意してください。

```text:Terminal
cd server
yarn add bcryptjs @types/bcryptjs
cd ../
```

create-frourio-app v0.18.0 以降では、`fastify-jwt` が元から入っていますので、ここで入れる必要はありません。

ついでに front, back, aspida, frourio, prisma を立ち上げておきましょう！
（create 後に勝手に立ち上がるので、それがまだ生きてたら飛ばして OK です）

```text:Terminal
yarn dev
```

# User model を追加

今回は、frourio サンプルのログイン処理を、DB の User テーブルを使ってログインできるようにします。

まず、Prisma の schema に User model を追加して、migrate します。

```prisma:server/prisma/schema.prisma
model User {
  id   String @id
  name String
  pass String
  icon String
}
```

```text:Terminal
$ yarn migrate:dev

[...]

✔ Name of migration … add User model

The following migration(s) have been created and applied from new schema changes:

migrations/
  └─ 20201211070757_add_user_model/
    └─ migration.sql

✔ Generated Prisma Client (2.13.0) to ./node_modules/@prisma/client in 181ms
```

すると、`server/prisma/migrations/` にディレクトリが増えているのが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/700rad3qfks2ci7z1gqmf8pey1l4)

`20201211070757_add_user_model/migration.sql` を覗いてみるとこんな感じ。

```sql:server/prisma/migrations/20201211070757_add_user_model/migration.sql
-- CreateTable
CREATE TABLE `User` (
    `id` VARCHAR(191) NOT NULL,
    `name` VARCHAR(191) NOT NULL,
    `pass` VARCHAR(191) NOT NULL,
    `icon` VARCHAR(191) NOT NULL,

    PRIMARY KEY (`id`)
) DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

先ほど `server/prisma/schema.prisma` に書き込んだ内容が SQL に変換されています。

# service を書き換える

続いて、`server/service/user.ts` を書き換えます。

PrismaClient は自動生成されるクエリビルダーです。各 CRUD 操作では Promise が返されるので、以下では async / await を用いて処理しています。
`prisma generate` は先ほど実行した `yarn dev` を通して `prisma generate --watch` が監視・生成しているため、ここで改めて実行する必要はありません。

![](https://i.imgur.com/aRJmVFY.png)
[Generating Prisma Client \(Reference\) \| Prisma Docs](https://www.prisma.io/docs/concepts/components/prisma-client/generating-prisma-client) より

```ts:server/service/user.ts
import { PrismaClient } from '@prisma/client'
import bcrypt from 'bcryptjs'
import fs from 'fs'
import path from 'path'
import { Multipart } from 'fastify-multipart'
import { API_ORIGIN, BASE_PATH } from './envValues'

const prisma = new PrismaClient()

const iconsDir = 'public/icons'
const createIconURL = (name: string) =>
  `${API_ORIGIN}${BASE_PATH}/icons/${name}`

export const validateUser = async (id: string, pass: string) => {
  const _user = await prisma.user.findUnique({ where: { id } })
  return await bcrypt.compare(pass, _user?.pass || '')
}

export const getUserInfoById = async (id: string) => {
  const _user = await prisma.user.findUnique({ where: { id } })
  if (_user) {
    // eslint-disable-next-line @typescript-eslint/no-unused-vars
    const { pass, ..._userInfo } = _user
    return _userInfo
  }
}

export const changeIcon = async (id: string, iconFile: Multipart) => {
  const iconName = `${Date.now()}${path.extname(iconFile.filename)}`

  await fs.promises.writeFile(
    path.resolve(iconsDir, iconName),
    await iconFile.toBuffer()
  )

  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  const { pass, ..._userInfo } = await prisma.user.update({
    where: { id },
    data: { icon: createIconURL(iconName) }
  })
  return _userInfo
}
```

# controller, hooks を書き換える

frourio の controller および hooks を書き換えていきます。

hooks では `req.jwtVerify()` で token の確認をしています。
この関数はおそらく `Authorization` ヘッダーに対してしか使えません（引数に token を取れない）。
任意の token を渡したい場合は、`defineController((fastify) => ({` のように fastify インスタンスを受け取り、`fastify.jwt.verify(token)` とするのが良さそうです。

```ts:server/api/user/hooks.ts
import { defineHooks } from './$relay'

export type AdditionalRequest = {
  user: {
    id: string
  }
}

export default defineHooks(() => ({
  preHandler: async (req, reply, done) => {
    try {
      await req.jwtVerify()
      done()
    } catch (err) {
      done(err)
    }
  }
}))

```

```ts:server/api/user/controller.ts
import { defineController } from './$relay'
import { getUserInfoById, changeIcon } from '$/service/user'

export default defineController(() => ({
  get: async ({ user }) => {
    const _userInfo = await getUserInfoById(user.id)
    if (_userInfo) return { status: 200, body: _userInfo }
    else return { status: 404 }
  },
  post: async ({ user, body }) => ({
    status: 201,
    body: await changeIcon(user.id, body.icon)
  })
}))
```

また、今回はログイン・ユーザー周りしかつくっていませんが、`/api/tasks/hooks.ts` などに同様の hooks を置くことで、task の操作に認証を要求することも可能かと思われます。

## `request.user` について

fastify-jwt README には

> Aftewards [sic], just use request.user in order to retrieve the user information:
> ```ts
> module.exports = async function(fastify, opts) {
>   fastify.get("/", async function(request, reply) {
>     return request.user
>   })
> }
> ```
> [fastify/fastify\-jwt: JWT utils for Fastify #Usage](https://github.com/fastify/fastify-jwt#usage)

という記述があり、`request.user` にユーザー情報が格納されているように思えます。

しかしながら、fastify-jwt が拡張してくれるのは、あくまで `FastifyRequest` のみであり、frourio はこれに気づくことができません。
そのため、今回は `AdditionalRequest` を用いて frourio に対しても `request.user` を拡張しました。

fastify-jwt による `FastifyRequest` の拡張を見に行くとこんな感じ。`request.user` の型が `JWTTypes.SignPayloadType` であることが確認できます。

```ts:server/node_modules/fastify-jwt/jwt.d.ts
declare module 'fastify' {
  // (省略)
  interface FastifyRequest {
    jwtVerify<Decoded extends JWTTypes.VerifyPayloadType>(options?: jwt.VerifyOptions): Promise<Decoded>;
    jwtVerify<Decoded extends JWTTypes.VerifyPayloadType>(callback: JWTTypes.VerifyCallback<Decoded>): void;
    jwtVerify<Decoded extends JWTTypes.VerifyPayloadType>(
      options: jwt.VerifyOptions,
      callback: JWTTypes.VerifyCallback<Decoded>,
    ): void;
    user: JWTTypes.SignPayloadType;
  }
}
```

どちらにせよ `type SignPayloadType = object | string | Buffer` なので、`fastify.jwt.sign({ id: body.id })` と併せて Payload の型をどこかに持ち、自分で `AdditionalRequest` で拡張する必要がありそうだなと思いました。

# おわりに

軽く触った感想としては、シンプルでおもしろい ORM だと思います。
多様な機能があるとその分学習コストも大きくなるので、object と Promise で SQL first なほうが使い勝手はいいかも。
また、frourio はまだ `v0.x.x` のため、今後の発展にも期待です！

最終日🎄である明日は、この記事にも登場した frourio の作者である [m_mitsuhide](https://qiita.com/m_mitsuhide) さんによる「frourioで2020年を締めます」です！わくわく！

---

# 参考文献一覧

- [**prisma/prisma**: Modern database access \(ORM alternative\) for Node\.js & TypeScript](https://github.com/prisma/prisma)
  - [Prisma \- Next\-generation ORM for Node\.js and TypeScript](https://www.prisma.io/)
- [**frouriojs/frourio**: Fast and type\-safe full stack framework, for TypeScript](https://github.com/frouriojs/frourio)
  - [Fast and type\-safe full stack framework, for TypeScript \| frourio](https://frourio.io/)
- [**fastify/fastify\-jwt**: JWT utils for Fastify](https://github.com/fastify/fastify-jwt#readme)
- [ORM成分高めで帰ってきたPrisma 2 \- Qiita](https://qiita.com/Quramy/items/5c5fc3bc69f2f9a91732#fn4)
- frourio チュートリアル連載シリーズ （~~ダイレクトマーケティング~~）
  1. [frourio でフロントエンドとバックエンドを一緒に静的型検査する - Qiita](https://qiita.com/su8ru/items/08d4222af6ddb8eb218b)
  2. [frourio でサクッと API 型定義 & コントローラーを書く - Qiita](https://qiita.com/su8ru/items/e4ba6fd311ee3905d174)
  3. [frourio でログイン処理などを行える Hooks を定義する - Qiita](https://qiita.com/su8ru/items/5f06dd45ed14117c291f)
  4. [frourio × aspida で 4 種類のバリデーションを実装する - Qiita](https://qiita.com/su8ru/items/c52b3e3b80edbc0363fa)
