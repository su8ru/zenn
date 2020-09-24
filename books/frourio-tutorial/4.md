---
title: "静的型検査を試してみる"
---

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
![](https://storage.googleapis.com/zenn-user-upload/zgcrabic3bip8edtgxm6m9gxj9h5)

さらに、`server/api/user/controller.ts` でも型エラーが出ています。
![](https://storage.googleapis.com/zenn-user-upload/5jehp07kbwi490jd1naopt7ci09v)

これで、実際にフロントエンドとバックエンドを動かすこと無く、静的に型エラーを検出することができました 🎉
