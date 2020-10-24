---
title: "技術選定とその理由"
---

第 73 回文化祭ウェブサイトでは、以下の構成になっています。

- front-end: Vue.js + TypeScript
- back-end: Lumen
- manage-app: React + TypeScript

# front-end

## 😕 Vue.js

**React にすべきだった**

Vue.js と React を比べた場合、日本人ユーザーは Vue.js のほうが圧倒的に多い気がしています。
その影響もあってか、React を知る前に Vue.js を採用してしまいました。

Vue.js は HTML 内で静的型チェックをすることができません。とにかくこのデメリットが大きく、ずっと後悔しています。
ただ、私はまだ Vue 3 の Composition API に触れていないので、その点では Vue.js が不利かもしれません。

1 つ言えるのは、私は後述する manage-app で React を触り、感動しました。

## 😕 Bootstrap Vue

**せめて Vuetify にして、部分的採用に留めておくべきだった（React なら Material UI）**

CSS ライブラリは実装がお手軽で楽ですが、「よく見るやつ」になってしまうというデメリットも抱えています。今回はそのデメリットが直撃しました。

## 😆 aspida

**とにかくすごい。みんな使え**

> [aspida/aspida: TypeScript friendly HTTP client wrapper for the browser and node\.js\.](https://github.com/aspida/aspida)

aspida とは、HTTP 通信を型安全にしてくれる、HTTP クライアントのラッパーです。

とにかくすごくて、API を叩く全ての TypeScript プロジェクトはこれを採用すべきです。

# back-end

## 😕 Lumen (PHP)

**back-end も TypeScript で統一すべきだった**

ただひたすらこれに尽きる。

# manage-app

## 😆 React

**front もこれにしていれば……**

JSX 良すぎる。アンチもいるみたいですが、私は大好きです。

## 😆 Material UI

**front もこれにしていれば……**

割と最近な感じでモダンなので、部分部分で使えば「よく見るやつ」になることはあまりなさそうです。

シンプルなので、ウェブサイト全体のデザインを邪魔することもなく、最高です。

# その他

## 😕 Swagger

**最初から aspida でよかったのでは（YAML 書きづらいし………）**

最初の半年間は、Swagger で API を設計 → front の aspida に書き起こして型定義 という手順を踏んでいました。

しかし、manage-app を作ろうとして aspida の型定義を共有したくなり、最終的に API 定義を Swagger から aspida 用型定義に変更しました。
これをパッケージ化して管理できるようになったので、とてもいい構成だと考えています。

> [aspida の API 定義をパッケージ化する - Zenn](https://zenn.dev/su8ru/articles/aspida-api-definition-to-package)（ダイマ）

Swagger UI のように見ることはできなくなりましたが、コードを書きながら TSDoc として API の説明を見れるのでとても楽です。

# 所感

一般的に使われているものは、自分の使いたい用途に合致するとは限らないというのがミソですね。

なにかプロジェクトを立ち上げる際には、自分の手で実際に使ってみて、試しながら技術選定していくのが理想なのだと思います。
