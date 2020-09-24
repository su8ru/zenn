---
title: "デプロイと実行"
---

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
