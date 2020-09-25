---
title: "$api.ts と $server.ts の中身"
---

`$` が付いているファイルは自動生成なので、自分の手でいじることはありません。
しかし、中身がどうなっているかを確認しておくのは悪いことではないはずです。さて、見てみましょう！

# `$api.ts`

このファイルは **aspida** による自動生成です。`dev:aspida` で変更検知→生成してくれます。

中身は単純で、各パス→メソッドへと入れ子になったオブジェクト `api` があるだけです。

ちなみに `server/api/` 以下の各定義部分に JSDoc を記述すると、ちゃんとこのファイルに出力されます。
[sample: afes-website/docs](https://github.com/afes-website/docs)

# `$server.ts`

このファイルは **frourio** による自動生成です。`dev:frourio` で変更検知→生成してくれます。

こっちはちょっとボリュームが多いですが、メインは後半の Express ルーティング部分です。

```ts
...

export default (app: Express, options: FrourioOptions = {}) => {
  const basePath = options.basePath ?? ''
  const uploader = multer(
    options.multer ?? {
      dest: path.join(__dirname, '.upload'),
      limits: { fileSize: 1024 ** 3 }
    } // 邪魔なので整形しました
  ).any()

  app.get(`${basePath}/`, methodsToHandler(controller0.get))

  app.get(`${basePath}/memo`, methodsToHandler(controller1.get))

  ...
```

Express のルーティングは以下の構造で定義されます。

```ts
app.METHOD(PATH, HANDLER)
```
出典･詳細: [Express の基本的なルーティング - Express.js](https://expressjs.com/ja/starter/basic-routing.html)

つまり、`methodsToHandler` で各コントローラー内の各メソッドを取り出し、Express のルーティングとして定義しているわけです。

バリデータが絡むと少しややこしくなりますが、関数名の通りの挙動を追っていくと意外と読みやすいです。

```ts
  app.post(`${basePath}/token`, [
    parseJSONBoby,
    createValidateHandler(req => [
      validateOrReject(Object.assign(new Validators.LoginBody(), req.body))
    ]),
    methodsToHandler(controller4.post)
  ])

  app.delete(`${basePath}/token`, [
    createValidateHandler(req => [
      validateOrReject(Object.assign(new Validators.TokenHeader(), req.headers))
    ]),
    methodsToHandler(controller4.delete)
  ])
```
