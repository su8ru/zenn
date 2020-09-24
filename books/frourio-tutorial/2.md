---
title: "プロジェクトの作成と起動"
---

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

そうするといろいろ聞かれるので、好きなように回答していきます。今回は簡単なデモということでシンプルな以下の構成にしました。

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

| Next.js | Nuxt.js |
| --- | --- |
| ![](https://storage.googleapis.com/zenn-user-upload/4z6lkxsy1b34tg1rhf79kggx1bqz) | ![](https://storage.googleapis.com/zenn-user-upload/mj32nxecsnd4kgpzipc6vtp7kbfr) |

![](https://storage.googleapis.com/zenn-user-upload/rlmidjycxewjcfz4wkom03tt4ol7)

ポチポチいじって動いていることが確認できればOK！
