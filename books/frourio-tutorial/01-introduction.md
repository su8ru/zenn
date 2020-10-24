---
title: "その API 通信、本当に型安全ですか？"
---

従来の HTTP 通信において、パスやリクエスト・レスポンスに対して型検査をすることはできませんでした。
そこで [aspida](https://github.com/aspida/aspida) を用いて予め API を型定義し、これを通して通信することでフロントエンドは静的型検査が可能になりました 🎉

詳細: [HTTPリクエストを型安全にする手法とOSS \- Qiita](https://qiita.com/m_mitsuhide/items/68406158d35a14fa0aa2)

しかしそのリクエストを受け取るサーバー側、すなわちバックエンドはどうでしょうか？
API 定義通りの実装がされているかどうか確かめるには、実際に動かしてみるしかありません。

「じゃあ aspida の型定義を使ってバックエンドを書けばいいのでは」

そのニーズに応えるのが 「 [frourio](https://github.com/frouriojs/frourio) 」です。

![](https://storage.googleapis.com/zenn-user-upload/qkfys42u0v8f9gbkn7r4snps954d)

まずは、`create-frourio-app` を用いてプロジェクトを作成してみましょう。
