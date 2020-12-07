---
title: "ThinkPad と外付け GPU で最高のおうち開発環境をつくる"
emoji: "💻"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["アドベントカレンダー", "egpu"]
published: true
---

# はじめに

この記事は、[生産性向上のための環境整備2020 【PR】 Lenovo Advent Calendar 2020](https://qiita.com/advent-calendar/2020/lenovo_env) の 8 日目の記事です。

家での開発環境を整えるにあたって大きな問題となるのが、出先と家での環境統一です。
外ではノートパソコンを持ち歩き、家ではデスクトップで開発する…なんてことをしていると、2 つのパソコンでデータを同期する必要があり、Git リポジトリが `tmp` や `yay` などといった commit で溢れてしまうことになります。^[これは私の友人がそうなだけで、一般的にはもっとまともな同期方法を取るのかもしれません。]
また、データ同期のみならず環境の違いも面倒です。

私は家と外で環境が分かれるのが嫌で、家でも外でもノートパソコンを愛用しています。
しかし家で作業するというのにディスプレイが 14 インチ 1 枚というのはいただけませんし、スペックもデスクトップには劣ります。

そこで私が 1 年半かけて築き上げた、ノートパソコンと外付け GPU を用いた WQHD トリプルディスプレイ環境をご紹介します。

# 全体のご紹介

まずは写真から。

3 枚あるディスプレイは WQHD (2560 x 1440)。デュアル用モニターアームと、シングル用モニターアームを使用しています。脚が無いだけでとてもすっきりするのでおすすめです。
キーボードは FILCO Majestouch 2 のテンキーレス + テンキーパッドの組み合わせ。フレームは諸事情により外しています。

![desk photo](https://i.imgur.com/zdhvR5uh.jpg)

(壁紙: [命と呼べば／赤を叫んだ \- 雨森ほわのイラスト \- pixiv](https://www.pixiv.net/artworks/85389067))

続いて配線図。きりがないので、電源系統は書いていません。
なんだかごちゃごちゃしていますが、最終的にノートパソコンにつながるのは Type-C 2 本だけです。

![](https://i.imgur.com/2OsInWQ.png)

# 主要ガジェット一覧

- Lenovo ThinkPad X1 Carbon 2019 (7th Gen)
  ![](https://storage.googleapis.com/zenn-user-upload/oed9qmdk4lriccdbwdkmpnt85coj)
- AORUS RTX 2070 GAMING BOX
  ![](https://storage.googleapis.com/zenn-user-upload/v4xldr0zj7q7kwpfgpq517oowggj)
- Dell P2418D x3
  ![](https://storage.googleapis.com/zenn-user-upload/9da9chtq2ore0rhnsa5i93kg9dec)
- FILCO Majestouch Convertivle 2 Tenkeyless
  キーキャップは Majestouch 2 HAKUA のものと、同社製交換用キーキャップを使用
  ![](https://www.diatec.co.jp/image_prod/FKBC91MJB2_01.jpg)
- FILCO Majestouch TenKeyPad 2 Professional
  ![](https://www.diatec.co.jp/image_prod/FTKP22MPSMW2_01.jpg)

:::details その他紹介できていないもの

- Logicool M331
  自分で塗装済 買い替えを検討中
  ![](https://storage.googleapis.com/zenn-user-upload/tjofr3hx72ie8e5i7mkuj1aim6e5)
- Logicool G233
  ![](https://storage.googleapis.com/zenn-user-upload/s71qyv281cbhc1mxcr1ygpjssdi6)
- Creative Pebble
  ![](https://storage.googleapis.com/zenn-user-upload/qwt8nze9wqoebmrn8sb8y8arenae)

:::

# ノートパソコンを壁に固定する

限られたスペースを有効活用するため、ThinkPad を**壁に固定**しています。
横からスライドして本体左側面の Type-C を接続した後、上の小さなアルミプレートで支える仕組みです。

![](https://storage.googleapis.com/zenn-user-upload/68ul97pmj0wka9vzm5d0ujriavbc)

![](https://storage.googleapis.com/zenn-user-upload/2ald6y5k5z376uo4w37f6p7sdvkd)

大きなアルミプレートの内側は下の画像のようになっていて、ケーブルをエポキシパテで固定して接続しています。^[なのでパソコンを変えると位置変更が大変…。修理に出してる間に古いのを使うぞ～ってなっても使えない……]
(USB 3.1 Gen1 / Thunderbolt 3)

![](https://storage.googleapis.com/zenn-user-upload/sx2yx7b5qa455d792602etktdfiy)

また、X1 Carbon は 底面吸気・側面排気 なので、底面側の板には空気の通り道として切り欠きを設けています。^[本体のファンのサイズを考えると、エアフローに無理がある気も…。本当はここにファンを外付けしたい]

電源は eGPU と接続している Thunderbolt 3 ケーブルで供給されています。Lenovo Vantage で確認したところ、100W 給電されているようです。

## 参考動画

@[youtube](tGX34631pdk)

# 内蔵 GPU のパワー不足を eGPU でカバーする

基本的に、**薄型ノートパソコンの iGPU^[Integrated GPU : 内蔵 / 統合 GPU] は一般的なデスクトップ用 GPU と比べて貧弱**です。^[このアドベントカレンダーの景品の [Lenovo ThinkPad P14s](https://www.lenovo.com/jp/ja/notebooks/thinkpad/p-series/thinkpad-p14s/p/20S4CTO1WWJAJP3) のようにノート用内蔵 GPU を搭載しているワークステーションなどは別ですが…]

そこで活躍するのが、**eGPU ( External GPU )** です。
eGPU とは、Thunderbolt 3 などで接続する外付け GPU のことです。ノートパソコンにデスクトップ用 GPU を外付けすることができ、多少ボトルネック^[本来 PCIe x16 で接続するものを、PCIe x4 相当で接続するため。2 ~ 3 割程度の性能低下 が通説]があるものの**一般的なデスクトップパソコンと遜色ない性能を発揮**できます。

私は [AORUS RTX 2070 GAMING BOX](https://www.gigabyte.com/jp/Graphics-Card/GV-N2070IXEB-8GC) で RTX 2070 を外付けしています。

![](https://storage.googleapis.com/zenn-user-upload/yc7lhsjftfkw0d7mgre6hi903phi)

参考:
https://mupon.net/difference-igpu-dgpu-egpu/

## iGPU と eGPU どちらを使用するかの設定

NVIDIA コントロールパネルで、3D 設定 > 3D 設定の管理 > グローバル設定 の「優先グラフィックスプロセッサ」を「高パフォーマンス NVIDIA プロセッサ」にします。

![](https://storage.googleapis.com/zenn-user-upload/4qlzcpir7tr6c7oaxiuenxcr5ptl)

これで設定は完了です。

:::details 【注意】Windows 設定 からは変更できない

> Windows OS でもグラフィックプロセッサの選択管理ができるようになりました。

とあり、Windows 側の設定でも管理できるように見えます。
しかしながら、設定アプリから変更できるのは特定のアプリの設定だけで、グローバル設定は NVIDIA コントロールパネルからしかできません。（ややこしいですね…）

![](https://storage.googleapis.com/zenn-user-upload/mm9j1ff4ck7wn38tpcfigzc0qox0)

:::

# 配線類をワイヤーネットに整理して壁に固定する

これは有名な配線整理方法ですね。既に実践している方もいるのではないでしょうか。

ノートパソコンに接続している 2 本の Type-C のうち、1 本は eGPU につなぐ Thunderbolt 3 ですから、USB はたった 1 本です。そこから大量の周辺機器に分岐するので、当然配線の量も凄まじくなります。
写真の下半分にあるように、データ通信だけでなく電源周りも大変なことになっています。

**ワイヤーネットに整理して、壁に固定**するとすっきり(?)収まります。

![](https://storage.googleapis.com/zenn-user-upload/vpv1uirue5quirsgxtrd5f7qcfzx)

ちなみにケーブルの固定・結束には、結束バンドよりも面ファスナーのほうが便利です。配線をいじるたびに切らなくていいのがとても楽。最初は結束バンドで固定していたのですが、配線整理も兼ねて全部交換しました。
「3M ワンタッチベルト」が薄くて取り回しやすいのでおすすめ。

![](https://storage.googleapis.com/zenn-user-upload/dv3dfs3bwdq88a75y8k8doj764rp)

# おわりに

以上、この 1 年半で築き上げた、最高の WQHD トリプルディスプレイ環境でした。
最後までお読みいただきありがとうございました！

まだ紹介しきれていないものもたくさんあるのですが、あまりだらだらと書いても仕方がないので、特徴的な 3 つのみのご紹介となりました。

家で過ごす時間が多くなるこのご時世^[実際、私も 1 学期の休校期間にいろいろ増えました…笑]、デスク周りの環境整備で生産性を向上してみてはいかがでしょうか？
