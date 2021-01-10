---
title: "月 5.5ドルで使える Vultr で nginx-proxy を構築して frourio を Docker で動かす！"
emoji: "🐳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [vps, docker, nginxproxy, frourio, githubactions]
published: false
---

# はじめに

この記事は、いままで Docker を開発環境としてお気持ちで使っていた初心者が書いています。
玄人の方々におかれましては、暖かい目で見守って頂けますと幸いです。
（誤っている情報などありましたらディスカッションにてお願いします…！）

最初は下の記事で紹介されている Vercel + Heroku での運用を考えていたのですが、Free Dyno があまりよくなかったので（スリープしたり、時間枠の概念だったり）悩んでいました。

👇 Vercel + Heroku 構成の紹介
https://zenn.dev/jun1123/articles/deploy-frourio

そこで知人が Docker ベースでの VPS 運用を勧めてくれて、その知人の手厚いサポートの末になんとか構築できました。本当にありがとうございます！

(単純比較できるものではないですが、Heroku Hobby は $7/月、Vultr は $5.5/月 なので自分で運用したほうが安い！)

# 完成図

![](https://storage.googleapis.com/zenn-user-upload/nw3ojxjaf3qg6bxazkblzndtgtvk)

GitHub Actions で rsync 転送後、VPS にてイメージ作成とコンテナ作成・起動をしています。

イメージを GitHub Actions で作ってから VPS にデプロイするべきなのですが、それはもう少し理解を深めてからということで……（いつか続編を書くかも）

フロントエンドは Vercel でホストしています。フロントエンドのデプロイも少しコツが要るのですが、それについても Vercel + Heroku 記事をお読みいただけると…！
[最近話題の「frourio」を無料でサクッとデプロイする方法（Vercel \+ Heroku） #フロントエンドのデプロイ](https://zenn.dev/jun1123/articles/deploy-frourio#%E3%83%95%E3%83%AD%E3%83%B3%E3%83%88%E3%82%A8%E3%83%B3%E3%83%89%E3%81%AE%E3%83%87%E3%83%97%E3%83%AD%E3%82%A4)

# VPS を借りる

## Vultr について

今回は [Vultr](https://www.vultr.com/?ref=8765771) で $5/mo の SSD Cloud Instance を借りてみます。

![](https://storage.googleapis.com/zenn-user-upload/fhk7fphb26jmqmp7n1lymxkm6mz5)

この記事をお読みの方のほとんどは消費税 10% の国にお住まいだと思いますので、$5 に 10% の消費税が加算されてタイトルの $5.5 となるわけです。

Vultr は海外運営の VPS ですが、なんと Tokyo リージョンがあります。
私は知人におすすめされて知ったのですが、Tokyo リージョンがあることと UI の綺麗さで気に入りました（こら）

👇 バナー。クリックしてくれると私が喜びます😊
[![](https://storage.googleapis.com/zenn-user-upload/biwjgu8j5u8um2womkxahdn9hrgj)](https://www.vultr.com/?ref=8765771)

トップページでメールアドレスとパスワードを入れて、名前とか住所とかを入れて、メールの認証をして決済情報まで入れておきましょう。

ちなみに管理画面はこんな感じ。

![](https://storage.googleapis.com/zenn-user-upload/tc6naxhu38eq3dt4wr8l265tnbb8)

## Deploy New Instance

新規インスタンスを作ります。

High Frequency だと 2.4 倍くらいのお値段で NVMe SSD になる…？そこまで性能が要るわけではないので Cloud Compute を選びます。
Location はもちろん Tokyo で、Type は好みでいいと思います。
Size はタイトルに従って $5/mo。一番下に `Additional 10% Consumption Tax applicable to your account` とあるように、アカウントの設定によって消費税が加算されます。

| 項目 | 内容 |
| :-: | :-: |
| Choose Server | **Cloud Compute** |
| Server Location | **Tokyo** |
| Server Type | **Ubuntu 20.04** |
| Server Size | **25 GB SSD ; $5/mo** |
| Additional Features | ここはお好みで |
| Startup Script | あるなら登録するとよさそう |
| SSH Keys | root に適用されます |
| Server Hostname & Label | 付けなくても作れます |

:::details スクリーンショット（長い）
![](https://storage.googleapis.com/zenn-user-upload/yekw66cxlzj0y8tm865lzm3y6wb2)
:::

## セットアップ

VPS をセットアップするのが久々すぎて、ほぼこの記事のとおりにやってました。
この場を借りて感謝 🙏

https://nishinatoshiharu.com/initial-vps-security-settings/

ここでは SSH Port は変更せず、MySQL 用に 3306 を開けておきます。

## バックアップ

### Snapshots

手動で保存する Snapshot は現在無料のようです。保存個数・期間も無制限なのかな…？

> Stored snapshots are currently free - pricing subject to change.

👇 Snapshots の画面はこんな感じ

![](https://storage.googleapis.com/zenn-user-upload/l13dm75071a46y9t2mucttvk26e6)

### Automatic Backups

定期的に保存してくれる Backup は、サーバー代金の 20% が追加徴収されます。

注意すべき点として、Backup として保存されるのは最新の 2 つまでのようです。
ただ、Backup を Snapshot に変換すると、先述した通り無期限？保存されます。

実行頻度は 毎日, 1 日おき, 毎週, 毎月 の中から選べます。
私は週に 1 回、UTC 日曜日 18 時 = JST 月曜日 午前 3 時 に設定しています。

👇 Backups の画面はこんな感じ（あいにくまだ一度も取られていません…）

![](https://storage.googleapis.com/zenn-user-upload/289o7ym9t2szijkyvrndj65z82ee)

これを設定しても $6.6/月 なのでお財布に優しい！
私はいつ環境を壊すかわからないので自動バックアップを頼ることにしました＞＜

# nginx-proxy を構築する

ディレクトリ構成はこんな感じ。

```
/
└ var/
　 └ docker/
　 　 ├ proxy/
　 　 │ └ docker-compose.yml
　 　 └ app/
　 　 　 ├ docker-compose.yml
　 　 　 ├ kostl-back/
　 　 　 │ ├ docker-compose.yml
　 　 　 │ └ Dockerfile
　 　 　 └ kostl-next/
　 　 　 　 ├ docker-compose.yml
　 　 　 　 └ Dockerfile
```

![](https://storage.googleapis.com/zenn-user-upload/nw3ojxjaf3qg6bxazkblzndtgtvk)

## docker, docker-compose をいれる

Vultr のドキュメントに従っていれます。

1. [Install Docker CE on Ubuntu 18\.04 \- 20\.04 \- Vultr\.com](https://www.vultr.com/docs/install-docker-ce-on-ubuntu-18-04)
2. [Install Docker Compose on Ubuntu 20\.04 \- Vultr\.com](https://www.vultr.com/docs/install-docker-compose-on-ubuntu-20-04)

ユーザーがドキュメントに寄稿できる（報奨制度有り）ので、大量の記事があります。
ここもうれしいポイント。

### 自分を docker グループに入れる

```sh
$ sudo usermod -aG docker subaru
$ id subaru
# uid=1000(subaru) gid=1000(subaru) groups=1000(subaru),27(sudo),998(docker)
```

## ディレクトリを用意

```sh
sudo mkdir /var/docker
sudo chown -R subaru:docker /var/docker
chmod 775 -R /var/docker
cd /var/docker
mkdir app && mkdir proxy
```

## network の作成

あとで使うことになる `front_bridge`, `back_bridge` を先に作成しておきます。

```sh
$ docker network create front_bridge
$ docker network create back_bridge
```

一覧を見たり、詳細を見たり……

```sh
$ docker network ls
$ docker network inspect front_bridge
```

## docker-compose.yml を書く

### nginx-proxy, Let's Encrypt

```yml:/var/docker/proxy/docker-compose.yml
version: "3"

services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    restart: on-failure
    labels:
      - com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy=jwilder/nginx-proxy
    container_name: nginx-proxy
    privileged: true
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - proxy:/etc/nginx/vhost.d
      - proxy:/usr/share/nginx/html
      - /var/run/docker.sock:/tmp/docker.sock:ro
      - ./certs:/etc/nginx/certs:ro
    networks:
      - front_bridge

  letsencrypt:
    image: jrcs/letsencrypt-nginx-proxy-companion
    restart: on-failure
    container_name: letsencrypt-nginx
    depends_on:
      - nginx-proxy
    volumes:
      - proxy:/etc/nginx/vhost.d
      - proxy:/usr/share/nginx/html
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./certs:/etc/nginx/certs:rw
    networks:
      - front_bridge

networks:
  front_bridge:
    external: true

volumes:
  proxy:
```

↓ほぼこの記事そのままです。この場を借りて感謝 🙏

https://riina-k.net/programming/docker-compose-nginx-reverse-proxy-network/

### MySQL

```yml:/var/docker/app/docker-compose.yml
version: "3"

services:
  mysql:
    image: mysql:5.7
    restart: on-failure
    container_name: docker-mysql
    ports:
      - "3306:3306"
    volumes:
      - ./mysql-5.7-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: your_root_password
    networks:
      - back_bridge

networks:
  back_bridge:
    external: true
```

データベースは全アプリ共通で 1 つの MySQL を用意して、3306 を開放しています。
本当はポート変えるべきかもしれない ~~けど私は覚えられないので変えてません~~

## nginx-proxy, MySQL を起動

```sh:/var/docker/proxy
$ docker-compose up -d
```

```sh:/var/docker/app
$ docker-compose up -d
```

初回起動ですし、ログも追っておきましょう。

```sh
$ docker-compose logs -f
```

# アプリを準備する

まずはほぼ共通の `docker-compose.yml` から。

nginx-proxy と Let's Encrypt の設定は各アプリの環境変数で行います。

```yml:/var/docker/app/hoge-dir/docker-compose.yml
version: "3.8"

services:
  hoge.example.com:
    build: .
    restart: on-failure
    container_name: hoge.example.com
    environment:
      VIRTUAL_HOST: hoge.example.com
      LETSENCRYPT_HOST: hoge.example.com
      LETSENCRYPT_EMAIL: mail@exmaple.com
    networks:
      - front_bridge
      - back_bridge

networks:
  front_bridge:
    external: true
  back_bridge:
    external: true
```

サービス名・コンテナ名は他とかぶらないものであればお好みでよさそう。

## frourio

### Dockerfile

[公式ドキュメント](https://frourio.io/docs/installation/gui) を参考にしながらプロジェクトを作成し、こんな感じの `Dockerfile` を用意します。

```docker:/var/docker/app/kostl-next/Dockerfile
FROM node:15

RUN mkdir /src
RUN mkdir /src/server

WORKDIR /src

COPY /server/package.json /server/yarn.lock ./server/
RUN yarn install --cwd ./server

COPY . .

EXPOSE 8080
CMD yarn build:server && yarn start:server
```

`yarn install` は server 側だけで問題ありません（ルートディレクトリでの install は不要）

↓ほぼこの記事そのままです。この場を借りて感謝 🙏

https://zenn.dev/jun1123/articles/deploy-frourio#docker%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%81%AE%E7%94%A8%E6%84%8F

### 環境変数

`server/.env` を作成します。

```ini:/var/docker/app/kostl-next/server/.env
SERVER_IP=0.0.0.0
BASE_PATH=/api
DATABASE_URL=mysql://kostl:your_password@mysql:3306/kostl-next
API_ORIGIN=https://api.next.kostl.info
JWT_SECRET=your_jwt_secret
USER_ID=id
USER_PASS=password
```

`DATABASE_URL` のホストは、サービス名（今回は `mysql`）で名前解決されます。

## Lumen

👇 本題から逸れるので折りたたみ & Dockerfile のみ

:::details Dockerfile
```docker:/var/docker/kostl-back/Dockerfile
FROM php:7.4-apache

WORKDIR /var/www/html
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public

# install
RUN apt-get update \
  && apt-get install -y libonig-dev zlib1g-dev libfreetype6-dev libjpeg62-turbo-dev libpng-dev git zip unzip
RUN docker-php-ext-install pdo_mysql

# gd
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd

# configure docroot
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf \
  && sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
COPY ./php.ini /usr/local/etc/php

# prepare composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER 1

# composer install
COPY composer.json /tmp/composer.json
COPY composer.lock /tmp/composer.lock
RUN composer install --no-scripts --no-autoloader -d /tmp

# prepare lumen
COPY . .
RUN mv -n /tmp/vendor ./ \
    && composer dump-autoload
RUN php artisan migrate && php artisan db:seed --force

# enable module
RUN a2enmod rewrite
```
:::

## アプリを起動

```sh:/var/docker/app/hoge-dir
$ docker-compose up -d
```

初回起動ですし、ログも追っておきましょう。

```sh
$ docker-compose logs -f
```

# ドメインに A レコードを設定する

ドメイン関連でやることは 1 つ。A レコードを設定するだけです。

VPS の IP アドレスを設定すればドメイン設定は完了！

![](https://storage.googleapis.com/zenn-user-upload/tdhjkykdrzwwg3kl0zj5mjt95x2m)

# GitHub Actions でデプロイを自動化

## デプロイ用のユーザーを作る

まずはサーバー側で `deploy` ユーザーを作り、`docker` グループをつけて Docker の操作権限を与えます。

```sh:ssh remote
$ sudo useradd -m deploy
$ sudo passwd deploy
$ sudo usermod -aG docker deploy
$ id deploy
```

そしてローカルで `deploy` の SSH 鍵を用意しておきます（あとで GitHub の Secrets に登録する）

```sh:local
$ mkdir ~/.ssh/vultr-deploy
$ ssh-keygen -t rsa -b 4096 -C "your_key_comment" -f ~/.ssh/vultr-deploy/id_rsa
$ ssh-copy-id -i ~/.ssh/vultr-deploy/id_rsa.pub deploy@198.51.100.0 # example
```

## workflow を書く

lint & type check → deploy with rsync → Docker build & up
の workflow がこんな感じです。

lint & type check には create-frourio-app で CI に GitHub Actions を選択した場合に生成されるテストを使用しています。

rsync の exclude も frourio に合わせて `server/.env` を指定しています。

👇 めちゃくちゃ長いので折りたたみ

:::details .github/workflows/deploy.yml
```yml:.github/workflows/deploy.yml
name: deploy
on:
  push:
    branches:
      - master

jobs:
  test:
    name: lint and type check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: setup Node.js
        uses: actions/setup-node@v1
        with:
          node-version: 14
      - uses: actions/cache@v2
        id: client-yarn-cache
        with:
          path: "node_modules"
          key: client-yarn-${{ hashFiles('yarn.lock') }}
      - uses: actions/cache@v2
        id: server-yarn-cache
        with:
          path: "server/node_modules"
          key: server-yarn-${{ hashFiles('server/yarn.lock') }}
      - run: yarn install
        if: steps.client-yarn-cache.outputs.cache-hit != 'true'
      - run: yarn install --cwd server
        if: steps.server-yarn-cache.outputs.cache-hit != 'true'
      - run: yarn lint
      - run: |
          sudo systemctl start mysql.service
          echo "DATABASE_URL=mysql://root:root@localhost:3306/test" > server/prisma/.env
      - run: yarn typecheck

  deploy:
    name: deploy with rsync
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v2
      - name: prepare .ssh dir
        run: mkdir -p .ssh && chmod 700 .ssh
      - name: prepare ssh key
        run: echo "$SSH_KEY" > .ssh/id_rsa && chmod 600 .ssh/id_rsa
        env:
          SSH_KEY: ${{ secrets.SSH_KEY }}
      - name: set permission
        run: chmod -R 775 * && sudo groupadd -f docker && sudo useradd deploy -g docker && sudo chown -R deploy:docker *
      - name: push with rsync
        run: |
          rsync -rlptgoD -O --delete --exclude ".git/" --exclude "server/.env" \
          -e "ssh -i .ssh/id_rsa -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null -p ${SSH_PORT}" \
          * $SSH_USER@$SSH_HOST:$DIR
        env:
          DIR: ${{ secrets.DEPLOY_DIR }}
          SSH_HOST: ${{ secrets.SSH_HOST }}
          SSH_USER: ${{ secrets.SSH_USER }}
          SSH_PORT: ${{ secrets.SSH_PORT }}

  docker:
    name: build and up
    runs-on: ubuntu-latest
    needs: deploy
    steps:
      - name: build & up -d
        uses: appleboy/ssh-action@master
        with:
          host:     ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key:      ${{ secrets.SSH_KEY }}
          port:     ${{ secrets.SSH_PORT }}
          envs: DIR
          script_stop: true
          script: |
            cd $DIR
            docker-compose build
            docker-compose up -d
        env:
          DIR: ${{ secrets.DEPLOY_DIR }}
```
:::

## Secrets を登録する

GitHub リポジトリの Settings > Secrets に Secrets を登録します。
ここで登録した情報は workflow 内で `secrets.SSH_PORT` のように参照できます。
<!-- textlint-disable ja-technical-writing/max-comma -->
`DEPLOY_DIR` (`/var/docker/app/hoge-dir`),
`SSH_HOST` ( IP Address ),
`SSH_PORT` (設定したやつ),
`SSH_USER` (`deploy`),
`SSH_KEY` (さっき作った秘密鍵)
を入れておきましょう。

![](https://storage.googleapis.com/zenn-user-upload/qxbbeeb5bjaokyeuonfpyssvupj9)

## 動かしてみる

workflow を commit して push すると実際に走ります。

ちゃんと順番に実行されることが確認できれば OK！

![](https://storage.googleapis.com/zenn-user-upload/xssm9fb8mwtusmkn7cao70imng8b)

build, up も問題なく行われていることが確認できます 🎉

![](https://storage.googleapis.com/zenn-user-upload/qnurskb8sr5ik74wxel017iupsy3)

# おわりに

最後までお読みいただきありがとうございました！
この記事が少しでもお役にたてれば幸いです😊

良き nginx-proxy & frourio ライフを～！

# 参考

大変参考になりました！ありがとうございます！

## VPS

- [【チュートリアル】VPSを借りたらやるべき最低限のセキュリティ初期設定 \| Enjoy IT Life](https://nishinatoshiharu.com/initial-vps-security-settings/)
- [お前らのSSH Keysの作り方は間違っている \- Qiita](https://qiita.com/suthio/items/2760e4cff0e185fe2db9)
- [ssh\-copy\-idで公開鍵を渡す \- Qiita](https://qiita.com/kentarosasaki/items/aa319e735a0b9660f1f0)

## Docker

- **Docker 基礎**
  - [Docker入門（第四回）～Dockerfileについて～ \| さくらのナレッジ](https://knowledge.sakura.ad.jp/15253/)
  - [docker\-compose \`up\` とか \`build\` とか \`start\` とかの違いを理解できていなかったのでまとめてみた \- Qiita](https://qiita.com/tegnike/items/bcdcee0320e11a928d46)
  - [Dockerイメージの理解を目指すチュートリアル \- Qiita](https://qiita.com/zembutsu/items/24558f9d0d254e33088f)
  - [Docker利用時のソースコードの変更反映 \- Qiita](https://qiita.com/tearoom6/items/326895f0a35774d70729)
- **ネットワーク**
  - [いい加減docker\-composeでlinksを使うのをやめてnetworkでコンテナ間名前解決をする \- Qiita](https://qiita.com/dyoshikawa/items/05d627b962da35f7d5b6)
  - [Docker Compose入門 \(3\) ～ネットワークの理解を深める～ \| さくらのナレッジ](https://knowledge.sakura.ad.jp/23899/)
  - [Docker Compose入門 \(4\) ～ネットワークの活用とボリューム～ \| さくらのナレッジ](https://knowledge.sakura.ad.jp/26522/)
- **nginx-proxy**
  - [Docker Compose と nginx でリバースプロキシを作ろうとしたお話（ネットワーク編） \| riina\-k\.net](https://riina-k.net/programming/docker-compose-nginx-reverse-proxy-network/)
  - [VPSにdockerで複数サイトをホスティングするには？](https://suin.io/561)

## frourio

- [最近話題の「frourio」を無料でサクッとデプロイする方法（Vercel \+ Heroku）](https://zenn.dev/jun1123/articles/deploy-frourio)
- [Fast and type\-safe full stack framework, for TypeScript \| frourio](https://frourio.io/)

## GitHub Actions

- [ConoHa WING に GitHub Actions でデプロイしよう！](https://zenn.dev/su8ru/articles/deploy-to-conoha-wing)
  (ダイレクトマーケティング)

## GitHub Repos

- [nginx\-proxy/nginx\-proxy: Automated nginx proxy for Docker containers using docker\-gen](https://github.com/nginx-proxy/nginx-proxy)
- [nginx\-proxy/docker\-letsencrypt\-nginx\-proxy\-companion: LetsEncrypt companion container for nginx\-proxy](https://github.com/nginx-proxy/docker-letsencrypt-nginx-proxy-companion)
- [frouriojs/frourio: Fast and type\-safe full stack framework, for TypeScript](https://github.com/frouriojs/frourio)
