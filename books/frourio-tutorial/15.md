---
title: "カスタムバリデータクラス"
---

バックエンドの作成に必須なバリデータ。もちろん frourio でも簡単に実装できます。

使うのは [class-validator](https://github.com/typestack/class-validator) 。JS バリデータの大御所感ありますね。

こんな感じで class-validator のお作法通りに定義して、

`server/validators/index.ts`

```ts
import { MinLength, IsString } from 'class-validator'

export class LoginBody { // <-- これが
  @MinLength(2)
  id: string

  @MinLength(4)
  pass: string
}

export class TokenHeader {
  @IsString()
  @MinLength(10)
  token: string
}
```

こんな感じで API 定義の `reqBody` / `reqHeaders` として使うだけ。簡単でわかりやすい！

`server/api/token/index.ts`

```ts
import { LoginBody, TokenHeader } from '$/validators'

export type Methods = {
  post: {
    reqBody: LoginBody // <-- ここ
    resBody: {
      token: string
    }
  }

  delete: {
    reqHeaders: TokenHeader
  }
}
```
