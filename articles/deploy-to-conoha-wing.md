---
title: "ConoHa WING に GitHub Actions でデプロイしよう！"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["conoha", "conohawing", "githubactions", "アドベントカレンダー"]
published: false
---

# はじめに

この記事は、[ConoHa Advent Calender 2020](https://qiita.com/advent-calendar/2020/conoha) 19 日目の記事です 🎄✨
アドベントカレンダーの投稿は今年が初めてですが、どうか生暖かい目でご覧ください。

さて、この記事では、GitHub の指定ブランチに push されたものを、GitHub Actions を用いて ConoHa WING に自動デプロイする方法をご紹介します。

![](https://storage.googleapis.com/zenn-user-upload/26vkudnj72pz48fkuaklsnbvoob9)

まずは前提として、ConoHa WING および GitHub Actions について軽く確認しておきましょう。

:::message
この記事はデプロイ方法の紹介を目的としており、内容により生じた損害に対する責任は一切負いません。
~~ちなみに私は本番環境の `public_html/` をふっ飛ばしたことがあります。皆様もどうかお気をつけください。~~
:::

## ConoHa WING って？

> 月額 400 円から使える
> _国内最速_
> レンタルサーバー ConoHa WING
> [超高速レンタルサーバーならConoHa WING｜初期費用・最低利用期間なし](https://www.conoha.jp/wing/)

はい。ご存じの方も多いかもしれませんが、**ConoHa WING はレンタルサーバー**です。

…が、**SSH が使えます！**（やったー）
そしてなんと、**rsync が入ってました！！**（な、なんだってー）

というのも、1 年ちょっと前に、この記事と同じ内容を調べていたのですが、当時はまだ WING で rsync が使えないというのが**通説**でした。
（ 1 年前の私がちゃんとチェックしたか怪しいので濁します…）

> ~~…そういうところだぞ！このはちゃん！~~
> [ConoHa Wing で自動デプロイして快適に WordPress テーマを更新しよう！ \- Qiita](https://qiita.com/ezaki/items/9f22b48e3ad83d226160)

とか言われたりしていましたが、これでもう**最強のレンタルサーバー**ですね！

## GitHub Actions って？

> GitHub Actionsを使用すると、ワールドクラスのCI / CDですべてのソフトウェアワークフローを簡単に自動化できます。 GitHubから直接コードをビルド、テスト、デプロイでき、コードレビュー、ブランチ管理、問題のトリアージを希望どおりに機能させます。
> [Actions \| GitHub](https://github.co.jp/features/actions)

**GitHub Actions は GitHub 上で使える CI / CD** です。

比較的新しいサービスで、2019 年 8 月に β 版が一般公開、2019 年 11 月に正式公開されています。
とにかく安くて（ほとんど無料）、Actions による拡張性もあり、とても便利です。

# 下準備

この記事では、特筆しなければ、フロントエンドのなんやかんやをビルドしたあと、`dist` ディレクトリの中身をデプロイする想定になっています。
いい感じに読み替えてください（丸投げ）

## workflow

以下、この例での workflow です。
GitHub Actions の優れた解説記事はたくさんあるので、このあたりは特に解説しません。
（この workflow も完璧じゃないですし…）

```yml:.github/workflows/deploy.yml
name: Deploy into production server

on:
  push:
    branches:
    - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: prepare .ssh dir
        run: mkdir -p .ssh && chmod 700 .ssh
      - name: ssh key generate
        run: echo "$SSH_KEY" > .ssh/id_rsa && chmod 600 .ssh/id_rsa
        env:
          SSH_KEY: ${{ secrets.SSH_KEY }}
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
      ...
```

この workflow の一番下に、これから挙げる方法のうち、お好きなものを書き足して完成です。

## Secrets

リポジトリの Secrets に `SSH_HOST`, `SSH_PORT`, `SSH_KEY`, `SSH_USER` を登録するのも忘れずに！

![](https://storage.googleapis.com/zenn-user-upload/dzgnyhkohzzhfka8cs8famb8uywa)

`SSH_KEY` は秘密鍵を、他の情報は ConoHa コントロールパネルの サーバー管理 > SSH にある情報を入れておきましょう。

![](https://storage.googleapis.com/zenn-user-upload/4xrhyyncvps58jx7yqy2g6nir32b)

# デプロイ！

## rsync

> rsync は、UNIXシステムにおいて、差分符号化を使ってデータ転送量を最小化し、遠隔地間のファイルやディレクトリの同期を行うアプリケーションソフトウェアである。類似のプログラムやプロトコルにはない rsync 独自の特徴として、ミラーサイトとの転送が双方向に高々1回で済む点がある。 
> [rsync \- Wikipedia](https://ja.wikipedia.org/wiki/Rsync)

ど定番ですね。気づいたら使えるようになっててびっくりしました。
オプションは適宜調整してください、私は詳しくないので……

```yml
      - name: push with rsync
        run: rsync -rlptgoD -O --delete --exclude ".git/" -e "ssh -i .ssh/id_rsa -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -p ${SSH_PORT}" dist/ $SSH_USER@$SSH_HOST:$DIR
        env:
          DIR: ~/public_html/hoge
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_USER: ${{ secrets.SSH_USER }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
```

## scp

> Secure Copy（scp）は、sftp同様、Secure Shell（ssh）に含まれるsshの機能を使ってセキュリティの高い（セキュアな）ファイル転送を行うコマンドの一つである。scpで使用される通信プロトコルは、Secure Copy Protocol（SCP）と呼ばれる。
> [Secure copy \- Wikipedia](https://ja.wikipedia.org/wiki/Secure_copy)

[afes-website/front](https://github.com/afes-website/front) ではこの方法を採用していました。
シンプルに全部送りつけるという点では一番優れていると思います。

```yml
      - name: push with scp
        run: scp -r -o StrictHostKeyChecking=no -P $SSH_PORT -i .ssh/id_rsa ./dist/. $SSH_USER@$SSH_HOST:$DIR
        env:
          DIR: ~/public_html/hoge
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_USER: ${{ secrets.SSH_USER }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
```

## lftp

> LFTPは、UNIXおよびUnix系システム向けのコマンド行ファイル転送プログラム（FTPクライアント）の1つ。Alexander V. Lukyanov が開発し、GNU General Public License でリリースされている。
> FTPだけでなく、プロトコルを含んだURLの指定でFTPS、HTTP、HTTPS、HFTP、FISH、SFTPも使える。特に便利な機能は、FXP（クライアントを経由せずに、2つのFTPサーバ間でデータを転送させるプロトコル）をサポートしている点である。
> [Lftp \- Wikipedia](https://ja.wikipedia.org/wiki/Lftp)

ConoHa Advent Calender 2018 で紹介されていたものです。
引用元: [ConoHa Wing で自動デプロイして快適に WordPress テーマを更新しよう！ \- Qiita](https://qiita.com/ezaki/items/9f22b48e3ad83d226160)

```yml
      - name: push with lftp
        run: lftp -e "set net:max-retries 1; set sftp:connect-program \"ssh -a -x -p $SSH_PORT -i .ssh/id_rsa -o StrictHostKeyChecking=no\"; connect sftp://$SSH_USER:@$SSH_HOST; mirror -eR -x .git -x .ssh ./ $DIR; quit"
        env:
          DIR: ~/public_html/hoge
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_USER: ${{ secrets.SSH_USER }}
          SSH_PORT: ${{ secrets.SSH_PORT }}
```

## git

[afes-website/back](https://github.com/afes-website/back) で採用した方法です。
この例はバックエンドですので、workflow 全体を例示します。
また、migrate などの処理も含まれています。参考にどうぞ。

```yml:.github/workflows/deploy.yml
name: Deploy into production server

on:
  push:
    branches:
    - master

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: deploy at remote server
        uses: appleboy/ssh-action@master
        with:
          host:     ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key:      ${{ secrets.SSH_KEY }}
          port:     ${{ secrets.SSH_PORT }}
          envs: DIR
          script_stop: true
          script: |
            source .bashrc
            cd $DIR
            git pull origin master -f
            composer.phar install
            php artisan migrate
            php artisan db:seed --force
        env:
          DIR: public_html/hoge
```

# 完成 🎉

できあがった workflow を `.github/workflows/deploy.yml` とかに置いて（ファイル名はご自由に）、commti して push したら完成！
これで指定したブランチに push されると、自動デプロイが走ります。

そしてリポジトリの「Actions」タブからログが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/ha4b8ecl6fz9ai6zqzmkj2n032hh)

# おわりに

この記事は 11 月中に書いているのですが、19 日目と後半なこともあり、ネタ被りが発生しないかドキドキしています………

…なにはともあれ、さらに進化した ConoHa WING の発展を楽しみにしています！
あとこのはちゃんもね！
