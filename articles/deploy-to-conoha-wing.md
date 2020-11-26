---
title: "ConoHa WING に GitHub Actions でデプロイしよう！"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["conoha", "conohawing", "githubactions"]
published: false
---

![](https://img.shields.io/badge/-ConoHa_Advent_Calendar_2020_%7C%2019%E6%97%A5%E7%9B%AE-EF5350?style=flat-square)

# はじめに

この記事は、[ConoHa Advent Calender 2020](https://qiita.com/advent-calendar/2020/conoha) 19 日目の記事です。
アドベントカレンダーの投稿は今年が初めてですが、どうか生暖かい目でご覧ください。

:::message alert
この記事はデプロイ方法の紹介を目的としており、生じた損害に対する責任は一切負いません。
~~ちなみに私は本番環境の `public_html` をふっ飛ばしたことがあります。~~
:::

さて、**ConoHa WING に GitHub Actions を用いてデプロイする** というのがこの記事の主題ですが、まず前提として ConoHa WING について軽く確認しておきましょう。

## ConoHa WING って？

ConoHa WING トップページを見てみましょう。

> 月額 400 円から使える
> _国内最速_
> レンタルサーバー ConoHa WING

はい。ご存じの方も多いかもしれませんが、**ConoHa WING はレンタルサーバー**です。

…が、**SSH が使えます！**（やったー）
そしてなんと、**rsync が入ってました！！**（な、なんだってー）

というのも、1 年ちょっと前に、この記事と同じ内容を調べていたのですが、当時はまだ WING で rsync が使えないというのが**通説**でした。
（ 1 年前の私がちゃんとチェックしたか怪しいので濁します…）

> ~~…そういうところだぞ！このはちゃん！~~
> [ConoHa Wing で自動デプロイして快適に WordPress テーマを更新しよう！ \- Qiita](https://qiita.com/ezaki/items/9f22b48e3ad83d226160)

とか言われたりしていましたが、これでもう**最強のレンタルサーバー**ですね！

# 下準備

この記事では、特記事項がない限り、フロントエンドのなんやかんやをビルドしたあと、`dist` ディレクトリの中身をデプロイする想定になっています。
いい感じに読み替えてください（丸投げ）

以下、この例での workflow です。
GitHub Actions の優れた解説記事がたくさんあるので、このあたりは特に解説しません。

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

# いろいろな方法

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
シンプルに全部送りつけるというう点では一番優れていると思います。

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
          key:      ${{ secrets.SSH_SSHKEY }}
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