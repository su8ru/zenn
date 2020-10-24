---
title: "aspida の API 定義をパッケージ化する"
emoji: "🚀"
type: "tech"
topics: ["aspida", "npm"]
published: false
---

自分用備忘録 兼 誰かのための記事

tsconfig をいい感じに設定して、npm に公開する記事です。

# なんでパッケージ化？

フロントエンドが 2 つに分かれたりすると、aspida の API 定義も 2 箇所に分かれてしまいます。なら共有できるようにパッケージ化しようということで

# サンプル

というよりこないだパッケージ化したリポジトリ（以前は swagger がここにありました）

[![afes-website/docs](https://github-readme-stats.vercel.app/api/pin/?username=afes-website&repo=docs&show_owner=true)](https://github.com/afes-website/docs)

# ディレクトリ構成

```
docs/
  ├ .github/
  │   └ workflows/
  │       └ publish.yml
  ├ tsconfig.json
  └ src/
      ├ api/    ← いつもの
      └ index.ts
```

# ファイルの中身

それぞれ最低限の要素 + α 程度のみ

## package.json

```json:package.json
{
  "name": "@afes-website/docs",
  "version": "2.2.11",
  "description": "73rd Afes website API",
  "repository": {
    "type": "git",
    "url": "https://github.com/afes-website/docs.git"
  },
  "main": "dist/index.js",
  "files": [
    "dist",
    "package.json"
  ],
  "scripts": {
    "api:build": "aspida",
    "build": "aspida --build && tsc --build tsconfig.json"
  },
  "license": "MIT",
  "peerDependencies": {
    "aspida": "^0.21.0",
    "typescript": "^4.0.2"
  },
  "devDependencies": {
    ...
  }
}
```

## tsconfig.json

```json:tsconfig.json
{
  "compilerOptions": {
    "rootDir": "./src",
    "baseUrl": "./src",
    "outDir": "./dist",
    "declaration": true,
    "target": "es6",
    "lib": [
      "esnext",
      "dom"
    ],
    "skipLibCheck": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "esModuleInterop": true,
    "module": "commonjs",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true
  },
  "include": [
    "**/*.ts"
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

## src/index.ts

`@types` 系を全部 export しておくと `import { hoge } from "@afes-website/docs"` ができて便利

```ts:src/index.ts
// apis
export { default as api } from "./apis/$api";
export { default } from "./apis/$api";

// types
export * from "./apis/@types";
export * from "./apis/auth/@types";
export * from "./apis/blog/articles/@types";
export * from "./apis/blog/revisions/@types";
export * from "./apis/blog/categories/@types";
export * from "./apis/blog/revisions/contrib/@types";
export * from "./apis/online/exhibition/@types";
export * from "./apis/online/drafts/@types";
export * from "./apis/onsite/exhibition/status/@types";
export * from "./apis/onsite/general/term/@types";
export * from "./apis/onsite/general/guest/@types";
export * from "./apis/onsite/general/guest/_id@string/log/@types";
export * from "./apis/onsite/reservation/@types";

// utils
export * from "./apis/images/@utils";
```

余談: 一番下の utils はエンドポイントからパスを返す関数が入っています。今の aspida では `$path` が実装されているので要らないですね………

# 公開する

```yaml:.github/workflows/publish.yml
name: publish as package

on:
  push:
    tags:
      - "v*"

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Cache yarn packages
        uses: actions/cache@v1
        with:
          path: ~/.cache/yarn
          key: ${{ runner.os }}-yarn-${{ hashFiles(format('{0}{1}', github.workspace, '/yarn.lock')) }}
          restore-keys:
            ${{ runner.os }}-yarn-
      - name: install packages
        run: yarn install
      - name: build
        run: yarn build
      - name: prepare npm config
        run: npm config set //registry.npmjs.org/:_authToken=$NPM_TOKEN
        env:
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
      - name: publish
        run: npm publish --access public
```

tag push をトリガーとして npm に公開しています。もう少しいい方法があると思いつつ、動くのでヨシ！しています………

# つかう

```sh:Terminal
$ yarn add @afes-website/docs@~2.2
```

```json:package.json
  "devDependencies": {
    "@afes-website/docs": "~2.2",
    ...
  }
```

```ts:hoge.ts
import api, { UserInfo } from "@afes-website/docs";
```

# おしまい

こここうしたほうがいいよ！とかあれば教えて下さい！

> 世界中のWebAPIの型定義をnpmに登録出来たらもっとフロントが楽しくなる
内製のAPIは上記のように一人が型定義をしておけば他のメンバーが安心してHTTPリクエストを行えるようになります
一方でTwitter, GoogleMap, YouTubeなど一般公開されててよく使うAPIは、型定義を誰かが書いてnpmに登録しておいてくれたら世界中のみんながラクにAPIを使えるようになりそうじゃないですか！？
ということで@types/nodeのノリで@aspida/twitterでTwitter APIの型定義をインストールできるようになるDefinitelyTypedみたいな仕組みも絶賛開発中です
[HTTPリクエストを型安全にする手法とNuxt TSでの実装例 \- Qiita](https://qiita.com/m_mitsuhide/items/c0cce7f3a79907c6e75c#%E4%B8%96%E7%95%8C%E4%B8%AD%E3%81%AEwebapi%E3%81%AE%E5%9E%8B%E5%AE%9A%E7%BE%A9%E3%82%92npm%E3%81%AB%E7%99%BB%E9%8C%B2%E5%87%BA%E6%9D%A5%E3%81%9F%E3%82%89%E3%82%82%E3%81%A3%E3%81%A8%E3%83%95%E3%83%AD%E3%83%B3%E3%83%88%E3%81%8C%E6%A5%BD%E3%81%97%E3%81%8F%E3%81%AA%E3%82%8B) 

これの実装を心待ちにしています

Special Thanks: Solufa, ibuki2003 
