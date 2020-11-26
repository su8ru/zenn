---
title: "frourio でフロントエンドとバックエンドを一緒に静的型検査する"
emoji: "🔍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["frourio", "aspida", "TypeScript"]
published: false
---

この記事は frourio チュートリアル連載 第 1 回です！以下の連載記事もぜひぜひ 👇️

**第 1 回** : frourio でフロントエンドとバックエンドを一緒に静的型検査する - [Qiita](https://qiita.com/su8ru/items/08d4222af6ddb8eb218b)
第 2 回 : frourio でサクッと API 型定義 & コントローラーを書く - [Qiita](https://qiita.com/su8ru/items/e4ba6fd311ee3905d174)
第 3 回 : frourio でログイン処理などを行える Hooks を定義する - [Qiita](https://qiita.com/su8ru/items/5f06dd45ed14117c291f)
第 4 回 : frourio × aspida で 4 種類のバリデーションを実装する - [Qiita](https://qiita.com/su8ru/items/c52b3e3b80edbc0363fa)

# その API 通信、本当に型安全ですか？

従来の HTTP 通信において、パスやリクエスト・レスポンスに対して型検査をすることはできませんでした。
そこで [aspida](https://github.com/aspida/aspida) を用いて予め API を型定義し、これを通して通信することでフロントエンドは静的型検査が可能になりました :tada:

詳細: [HTTPリクエストを型安全にする手法とOSS \- Qiita](https://qiita.com/m_mitsuhide/items/68406158d35a14fa0aa2)

しかしそのリクエストを受け取るサーバー側、すなわちバックエンドはどうでしょうか？
API 定義通りの実装がされているかどうか確かめるには、実際に動かしてみるしかありません。

「じゃあ aspida の型定義を使ってバックエンドを書けばいいのでは」

そのニーズに応えるのが 「 [frourio](https://github.com/frouriojs/frourio) 」です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225023/5dcb40a9-47af-ffa8-8bbe-37bdec3ff044.png)

この記事では、`create-frourio-app` を用いてプロジェクトを作成し、実際に静的型検査を試してみて、デプロイまでやってみます。

# プロジェクトの作成と起動

```sh:Terminal
$ npx create-frourio-app <project-name>
```
```sh:Terminal
$ npm init frourio-app <project-name>
```
```sh:Terminal
$ yarn create frourio-app <project-name>
```

お好みでどれかを実行します。ちなみに私は `yarn` 派です。

しばらく待つと、ブラウザで次のような画面が `localhost:3000` で開きます。

![Welcome to frourio!.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225023/89b840fb-7101-6d15-c95c-b9f9b1e5f5cc.png)

UI に沿って選択肢を選び、最後に「Create」を押すと、プロジェクトの作成が再開します。

今回は `Fastify`, `Next.js`, `Basic (next build)`, `axios`, `Yarn`, `None`, `None`, `None` で作成しました。
データベースを指定していませんが、この場合は `server/database.json` がデータベース代わりに使われます。

また、ここでの回答は `~/.frourio/create-frourio-app.json` に記録され、次回以降の初期値として使用されます。

```json
{"ver":1,"answers":{"dir":"demo-next","prismaDB":"mysql","orm":"none","server":"fastify","building":"basic"}}
```

作成後、おなじみの初期画面 + ToDo アプリが出れば完了です :tada:

| Next.js | Nuxt.js |
| --- | --- |
| ![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225023/89943c5d-1a97-fcbc-6979-1998b5c2b718.png) | ![](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225023/3ec36776-45c8-0735-c4c8-abcff4775c19.png) |

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225023/17ddaf61-703e-816d-1ff9-17f038227d9d.png)

ポチポチいじって動いていることが確認できればOK！

## `create-frourio-app v0.14.3` 以前の方法

いろいろ聞かれるので、好きなように回答していきます。今回は簡単なデモということでシンプルな以下の構成にしました。

```:Terminal
create-frourio-app v0.11.2
✨  Generating frourio project in demo-next
? Project name: demo-next
? Frontend framework: Next.js
? Building mode: Basic (next build)
? HTTP client of aspida: axios
? Package manager: Yarn
? Daemon process manager: None
? Database type of TypeORM: None
```

データベースを指定していません（`Database type of TypeORM: None`）が、その場合は `server/database.json` がデータベース代わりに使われます。

一通り終わるとこんな感じのが出るので、

```:Terminal
🎉  Successfully created project demo-next

  To get started:

        cd demo-next
        yarn dev

  To build & start for production:

        cd demo-next
        yarn build
        yarn start
```

ディレクトリに移動し、`yarn dev` でアプリを起動します。

`http://localhost:3000` でおなじみの初期画面 + ToDo アプリが出れば完了です 🎉

# ディレクトリ構成

```
<project-name>/ … プロジェクトルート
  ├ components/, pages/, public/ など … フロントエンドのあれこれなので省略
  └ server/         … サーバールート この中がバックエンドです
      ├ api/        … API 定義 兼 コントローラーなどの実装
      |               パスをディレクトリで表して API を定義します 記述方法は aspida と共通
      ├ public/     … 静的コンテンツ
      ├ types/      … 型定義
      ├ service/    … バリデーターとかいろいろ
      ├ validators/ … カスタムバリデータクラス
      ├ index.ts    … サーバーエントリーポイント 起動処理が書いてあります
      ├ index.js    … [自動生成] webpack によるバンドル結果
      ├ .upload/    … [自動生成] アップロードファイルの一時保管用
      └ $server.ts  … [自動生成] frourio によるビルド結果
```

ちなみにデフォルトでは

- `~` : `./*`
- `$` : `./server/*`

の 2 つのパスが通っています。

これにより、`server/types/` の中の型定義をフロントエンドからも簡単に参照できたりします。

# 静的型検査を試してみる

実際に型を変えてみて、エラーが出る様子を見てみます。

とりあえず `server/types/index.ts` にある `UserInfo` interface の `icon` を、`Blob` 型に書き換えてみます。

```diff:server/types/index.ts
 // ...

 export type UserInfo = {
   id: string
   name: string
-  icon: string
+  icon: Blob
 }
```

まず、`components/UserBanner.tsx` で型エラーが出ています。ここまでは aspida の機能です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225023/a8d624c5-a5b6-257f-a83b-e330d404b272.png)

さらに、`server/api/user/controller.ts` でも型エラーが出ています。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225023/2dd171aa-aa8a-94a9-5134-db8aa826e0a1.png)

これで、実際にフロントエンドとバックエンドを動かすこと無く、静的に型エラーを検出することができました :tada:

# デプロイと実行

基本的にフロントエンドとバックエンドは独立しているので、別々に実行することが可能です。
`yarn start:front` / `yarn start:server` でそれぞれ立ち上がりますし、`yarn start` で両方を同時に立ち上げることも可能です。

バックエンドのポートは `server/.env` で指定することができます。

```conf
SERVER_PORT=8080
```

フロントエンドはフレームワークのお作法通りにやっていけば問題ありません。
なかなかな量の npm-scripts が用意してあるので、一度 `package.json` を読んでおくことをおすすめします。

また、Nuxt.js / Next.js 共に SPA / [Static](https://nextjs.org/docs/advanced-features/static-html-export) に対応しています。
プロジェクト作成時に選択したモードにあわせて npm-scripts が生成されるので、あとは `yarn build:front` を叩くだけです。お手軽！

# おわりに

いま Vue.js + Lumen で個人開発しているウェブアプリがあるのですが、それを frourio で置き換えたくなるレベルで便利で驚いています。
型安全さいこー！！

次回は aspida と共通の型定義とコントローラー、つまり `server/api/` の中身について書く予定です。乞うご期待！

**第 1 回** : frourio でフロントエンドとバックエンドを一緒に静的型検査する - [Qiita](https://qiita.com/su8ru/items/08d4222af6ddb8eb218b)
第 2 回 : frourio でサクッと API 型定義 & コントローラーを書く - [Qiita](https://qiita.com/su8ru/items/e4ba6fd311ee3905d174)
第 3 回 : frourio でログイン処理などを行える Hooks を定義する - [Qiita](https://qiita.com/su8ru/items/5f06dd45ed14117c291f)
第 4 回 : frourio × aspida で 4 種類のバリデーションを実装する - [Qiita](https://qiita.com/su8ru/items/c52b3e3b80edbc0363fa)
