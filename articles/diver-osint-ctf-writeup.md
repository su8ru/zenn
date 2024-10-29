---
title: '[HUIT] DIVER OSINT CTF Writeup'
emoji: '🤿'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['ctf', 'writeup', 'osint']
published: false
---

# はじめに

2024 年 6 月に開催された [DIVER OSINT CTF](https://github.com/diver-osint-ctf) にチーム「HUIT」で参加し、484 チーム中 17 位でした！
このチームは大学サークル [北大 IT 研究会](https://www.huitgroup.net/) の有志から構成されています。

![](https://storage.googleapis.com/zenn-user-upload/1e169c6888a0-20241029.png)

本記事は、以下のチームメンバーが各々「自分がやった部分」を書いた文章の集合体です。

- Slephy
- Reso
- Auth0r
- su8ru

Writeup を書いたものの公開していなかったので、大変遅れての公開です 🙇（もう 4 か月前！？）

## 公式 Writeup

https://github.com/diver-osint-ctf/writeups

# welcome

## ✅ welcome

Discord サーバー、ルールに記載。

> Diver24{ganbarou!}

提出は惜しくも First Blood ならず 3 番目でした。

# misc

## ✅ number

> この車両の持ち主に連絡を取りたい。電話番号を調べてもらえないだろうか。

車両のナンバープレートは「外-4906」
https://ja.wikipedia.org/wiki/%E6%97%A5%E6%9C%AC%E3%81%AE%E3%83%8A%E3%83%B3%E3%83%90%E3%83%BC%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88#%E5%A4%96%E4%BA%A4%E5%AE%98%E8%BB%8A%E4%B8%A1_%EF%BC%88%E5%A4%96%E5%8B%99%E7%9C%81%EF%BC%89
数字の百の位から上の桁の部分「49」は派遣元国「クウェート」を意味する
https://www.kuwait-embassy.or.jp/

> Diver24{0334550361}

## ✅ label

> この荷物の宛先として考えられる施設の郵便番号を教えてほしい。

送り主の情報は断片的にわかるが、宛先の情報はわからない。

正方形型の QR コードを読み取ると、TRACKING の下に記載された数字が表示された。
これより、バーコードは DATE、長方形型の QR コード（rMQR コード）は住所を示すと予想。

[rMQR コードを読み取るアプリ](https://www.denso-wave.com/ja/system/qr/product/reader.html)をインストールし読み込むと

> SHIPMENT ADDRESS:
> Baba-cho 14-1, Tsuruoka City, Yamagata, Japan

が表示された。これより、

> Diver24{997-0035}

## ✅ wumpus

> とある Discord サーバに Flag が投稿されたぞ！

`discord.com/channels/1244302408402735114/1244302408402735117`
は
`discord.com/channels/サーバーID/チャンネルID`
で
サーバー ID でググると`https://discordservers.com/server/1244302408402735114`(削除済)
が見つかり実際にサーバーに参加できるので
チャンネルに投稿されている QR コードを読み込むと flag をゲットできる

> Diver24{Discord_1s_m0s7_u5efu1_t001}

## ✅ timestamp

> "53" と塗装されている航空機の写真が撮影された日時を答えよ。

プレスを見に行くと、pdf 上で画像の上に「航空自衛隊撮影」が載っており、その下に撮影時刻が隠れてた

> Diver24{2014-03-12T15:33}

## ✅ howmany

マンホールの手がかりが無くて困っていたら、[番号から場所を特定するやつ](https://ekikaramanhole.whitebeach.org/ext/0A0A/?ms=google&c=y&s=62&n=0I&e=3F&order=cap2id&id=201412062)を誰かが見つけてくれた
世の中にはいろいろな人がいる……

手元のバイクシェアアプリを立ち上げると、この付近にポートは 1 つしかない
道路の形も合致するので、「D6-01.新宿区役所本庁舎」で確定

ドコモ・バイクシェア（とハロサイ）の情報は ODPT で GBFS が公開されている
https://ckan.odpt.org/ja/organization/d-bikeshare

`station_information.json` から探してポートの id `00010184` は手に入るけど、`station_status.json` はリアルタイムデータなので過去のデータが得られない

定番の Wayback Machine やウェブ魚拓などを探し、[見つけた](https://archive.is/mGgLI)

> Diver24{41_00010184}

# military

## ✅ osprey1

> 2023 年 11 月 29 日、アメリカ軍のオスプレイ（V-22）が日本の屋久島沖で墜落した。この機体の番号と、墜落時のコールサインは何か。

最初に見つけたのはレジ番 `10-0054` なので、これのコールサインを探すも見当たらない

探し方を変えて、コールサインは `GUNDAM22` で決定、しかしレジ番が `12-0065` 説が浮上
https://x.com/oldconnie/status/1730235904592134469
https://x.com/YokotaAirBase/status/1735421572578693233

わからないのでどっちも試したら `12-0065` で正解

> Diver24{12-0065_GUNDAM22}

## ✅ osprey2

> 2024 年 2 月 15 日、ある米軍基地でこの事故に関する追悼式典が実施された。16:46:37 ごろ、その式典はどこで実施されていたか。OpenStreetMap の Way 番号で答えよ。

式典が行われたのは横田基地
https://x.com/m38ztree/status/1755014480529154552

ヒントを開ける癖がなかなか付かず、横田基地・横田飛行場を提出するも WA

[公式によるニュース](https://www.yokota.af.mil/News/Yokota-News/Article-Display/Article/3688950/yokota-honors-the-fallen-crew-of-gundam-22/)を発見

サムネイルの画像をダウンロードすると EXIF が残っており、撮影日時が問題と概ね一致するのでこれっぽい

後ろの × を手がかりに建物を探すと、[Yokota Main Exchange](https://www.shopmyexchange.com/exchange-stores/Japan/AP/apo/Yokota--1751007) を発見

画像中央の空き地と推測するも、OpenStreetMap 上でウェイが存在しない

仕方がないので画像右の運動場？を提出、AC

> Diver24{400923454}

## ✅ osprey3

> 墜落した機体は 2018 年 11 月 15 日の夜、ある空港に駐機していたらしい。その地点の標高（フィート）を整数で答えよ。なお、各種データソースの時刻にズレはないと仮定してよい。また、空港に関する情報は最新のものを参照してよい（2018 年時点のデータを用いる必要はない）。

レジ番 `12-0065` で検索してると、jetphotos で[当該日の写真](https://www.jetphotos.com/photo/9144357)を発見

ICAO EPKK で、[英語版 Wikipedia](https://en.wikipedia.org/wiki/Krak%C3%B3w_John_Paul_II_International_Airport) から `791` を提出したけど WA

ヒントを開けると（また忘れてただけ）、具体的なスポットの高度を出してほしいとのこと（でもこれは開ける必要なかった気もする……）

背後のタワーとハンガーの位置関係からおおよそのスポットは特定でき、Google Earth から高度を得るも WA

Chart がほしいけど、AIS JAPAN にあるわけないので、[野生のサイト](https://pl.ivao.aero/en/epkk/)を見つける

[Aerodrome Chart](https://www.ais.pansa.pl/aip/pliki/EP_AD_2_EPKK_1-1-1_en.pdf)より `774` ft とわかる

> Diver24{774}

# introduction

## ✅ 246

> 画像が撮影された場所から最も近い交差点名を答えよ。

黄色背景なので一般都道府県道とわかる

https://www.skr.mlit.go.jp/road/hyousiki/h_annai/guidesign/crossing.html#:~:text=%E4%BA%A4%E5%B7%AE%E9%81%93%E8%B7%AF%E3%81%AE%E7%95%AA%E5%8F%B7%E3%82%92,%E8%89%B2%E3%81%A8%E5%90%8C%E3%81%98%E8%89%B2%E3%81%A7%E3%81%99%E3%80%82

何県？

- 山口県、火の山ロープウェイ付近？
- 山口県道 246 号線の終点が正解
- https://ja.wikipedia.org/wiki/%E5%B1%B1%E5%8F%A3%E7%9C%8C%E9%81%93246%E5%8F%B7%E9%95%B7%E5%BA%9C%E5%89%8D%E7%94%B0%E7%B7%9A

> Diver24{前田}

## ✅ office

> ファイルから得られる Flag を入力してください。

とりあえず、file コマンドでファイルを調べる。
zip ファイルだったので、unzip すると画像が出現。
画像内に flag が書かれていた。

> Diver24{World Ocean Day}

## ✅ chain

> この看板の店の電話番号を答えよ。

鳥貴族の店舗一覧を「4F」等で検索したいところだが、表記揺れがひどそうなので目 grep。

東京は店舗が多そうなので一旦地方から探していくも見つからず、重い腰を上げて東京を見ると閉店済みの広尾店がそれっぽい。

一生分の鳥貴族を見た気がする……行ったことないのに……

> Diver24{0364593392}

## ✅ dream

> 画像に写っているパイプオルガンがある施設の郵便番号を答えなさい。

画像検索したらヒット

- https://ameblo.jp/jissi/entry-12316645389.html
- https://opera-seasons.jp/pages/19/

> Diver24{574-0046}

## ✅ serial

> これらの動画の背景に映っている航空機のシリアル番号は何か？

映像から「JA222A」が読み取れるので、検索するとシリアルナンバー（製造番号）が分かる

https://flyteam.jp/registration/JA222A

> Diver24{9580}

## ✅ ad_directiare

> この名刺の人物が、東京出張時に食べた昼ご飯の値段を答えよ。

`yone.jun.0727@gmail.com` を[Ghunt](https://github.com/mxrch/GHunt)すると
https://www.google.com/maps/contrib/104607974422086075165/reviews
が得られる

> Diver24{4400}

# geo

## ✅ imagetrack

> 画像の撮影された郷土料理屋の店名を答えよ。

EXIF から緯度経度が 34°41'41.2"N 135°11'35.6"E とわかる

雑居ビルでいろいろ入っているが、「郷土料理屋」は 1 つしかない

> Diver24{郷土料理屋 からす}

## ✅ chiban

`サンディ`と`岡田歯科医院`から撮影場所は
https://maps.app.goo.gl/X67o2tP44MhZibwC8 付近であることがわかる
[地番検索ができるサービス](https://chiban-kensaku.com/)を利用すると

![](https://storage.googleapis.com/zenn-user-upload/f6f9a1e24b34-20241029.png)

> Diver24{筆界未定地-6}

## ✅ championships

> 2010 年 10 月、この場所には大会の広告が貼られていた。その大会の優勝者のニックネームを答えよ。

看板より[光華数位新天地](https://www.google.com/maps/place/%E5%85%89%E8%8F%AF%E6%95%B0%E4%BD%8D%E6%96%B0%E5%A4%A9%E5%9C%B0/@25.0452655,121.5320331,20z/data=!3m1!5s0x3442a9635ab4f561:0xef6e1c7be7c3c9aa!4m6!3m5!1s0x3442a9634439ce7d:0x4cd6385059c7f187!8m2!3d25.0452719!4d121.5320331!16s%2Fm%2F02px8h1?entry=ttu)とあたりをつける。

建物入り口付近に ASUS の店舗が見えるため、光華数位新天地の区画内の ASUS の店舗をストリートビューで巡回し、[画像の場所付近](https://www.google.com/maps/@25.0447935,121.53154,3a,75y,69.18h,95.08t/data=!3m6!1e1!3m4!1sTES9f6cDYwwBJXfS98Tumg!2e0!7i16384!8i8192?coh=205409&entry=ttu)を特定した。

壁があって店舗入り口が見えないが、ストリートビューの撮影時期 2010 年 10 月に変更することで、看板を発見。

看板の左上に「GIGABYTE」、中央部に「Fight(?)」のような文字を発見。
画像検索では難しかったため、「GIGABYTE esports sponsor 2010」で検索。
該当の[大会の記事](https://www.gigabyte.com/Press/News/937)を発見。看板と後ろのゲームタイトルの一致を確認し、大会優勝者「Matose」を発見。

> Diver24{Matose}

## ✅ power

> 写真右側に写っている送電線の運用容量値（単位: MW） はいくつだろうか。整数で答えよ。

サンダーバードっぽいので、JR 京都線や湖西線沿いを[塔マップ](https://10map.net/)で眺めていると、桂川駅付近に[それっぽい場所](https://www.google.com/maps/@34.9627313,135.710036,3a,51.1y,185.65h,91.85t/data=!3m6!1e1!3m4!1sA7tnHdGUXSw5di2p03bzTQ!2e0!7i16384!8i8192?coh=205409&entry=ttu)が見つかる。

塔マップから「西院線」という名称が手に入るが、よくわからないので[関西電力の資料](https://www.kansai-td.co.jp/interchange/takusou/pdf/154kv_less_mapping_08.pdf)を見ると、北 172 というナンバリングが手に入る。

ところで「運用容量値」がよくわからないので調べていると、どうやら電力会社が一覧を出しているらしく、またまた[資料](https://www.kansai-td.co.jp/interchange/takusou/pdf/154kv_less_space.pdf)をぐっと睨むと 84 MW とわかる。

> Diver24{84}

## ❌ island (unsolved)

> 「四方ぎり島」という名前の島における、最高地点の標高を整数（メートル）で答えよ。

- 小豆島
- ギリ島

- 日本の島一覧から頑張る
  https://ja.wikipedia.org/wiki/%E6%97%A5%E6%9C%AC%E3%81%AE%E5%B3%B6%E3%81%AE%E4%B8%80%E8%A6%A7
- 日本語訳で「四方ぎり島？」
  - 三方限 さんぽうぎり って概念はあるらしい
  - https://kagoshima-city-kotsu.com/category1/entry42.html
  - https://ameblo.jp/harukimasaya/image-11284719171-12042505304.html
- Diver24{1936} ×
- Diver24{1629} ×
- Diver24{0} ×

## ❌ construction (unsolved)

> あるアナリストによると、37.669, 120.691 付近に新たな施設が建設されたことが判明した。この施設の名称を答えよ（現地の公用語表記）。

中国・山東省

https://www.google.com/maps/place/37%C2%B040'08.4%22N+120%C2%B041'27.6%22E/@37.6690042,120.6884251,17z/data=!3m1!4b1!4m4!3m3!8m2!3d37.669!4d120.691?entry=ttu
https://j.map.baidu.com/09/NUpi

Google Maps と Baidu で地図の形が違う場所がある、まさにそこ…？

![](https://storage.googleapis.com/zenn-user-upload/f52c74bdb350-20241029.png)
![](https://storage.googleapis.com/zenn-user-upload/562718cee2bf-20241029.png)

`Diver24{好机会农场}` ×
`Diver24{农村供水水厂}` ×
`Diver24{农村供水水厂水质提升工程}` ×

## ❌ public_service (unsolved)

> ある男が「20 年前の 11 月だったかな、この交差点について市に文句を付けてやったことがあるんだよ。最近の行政は酷いもんだがね、あの頃は良かったよ。」と言っていた。

場所はわかったものの、手がかりがない……

https://www.google.com/maps/@40.7480821,-73.8540728,3a,75y,24.51h,86.84t/data=!3m6!1e1!3m4!1snUDif3q1IrjsAUrelqIxfg!2e0!7i16384!8i8192?coh=205409&entry=ttu

# crypto

## ✅ leak

> 先月、日本の会社で大規模な BTC の不正流出が起きた。流出先となっているウォレットアドレスを答えよ。

[DMM の件](https://bitcoin.dmm.com/news/20240531_01)だと推測

Twitter で DMM BTC について 5 月 31 日頃の情報を調べると、BTC の[Alert 情報](https://x.com/BeosinAlert/status/1796567785394565514)を発見

> Diver24{1B6rJRfjTXwEy36SCs5zofGMmdv2kdZw7P}

## ❌ Akeome (unsolved)

> 以下の人物が 2017 年に始めた youtube チャンネルを見つけてください。
> https://bitcointalk.org/index.php?action=profile;u=44692

# history

## ❌ promoter (unsolved)

> 画像に写っている池沼を開拓した発起人の父親の命日を答えよ。

- 入鹿池 ここ
  https://ja.wikipedia.org/wiki/%E5%85%A5%E9%B9%BF%E6%B1%A0
- 善左衛門宗度 (父) の没年がみつからない...
- https://www.jsidre.or.jp/wordpress/wp-content/uploads/2016/03/49-7.pdf

## ❌ paddy (unsolved)

> この地域では地形の人工的な大規模改変が 2 回行われたという。郡司としてこれにかかわった人物の名は？日本語表記で答えよ。

画像の場所は[このあたり](https://www.google.com/maps/@38.4981986,141.2136294,14.06z?entry=ttu)。付近の史跡を検索し、

- 糠塚城跡
- 塩野田城跡
- 石碑群
- 桃生城跡（やや遠方、画像から北東部に位置）
  を発見。
  「城主」or「旧地名」と「開拓」or「開墾」などで検索し、いくつかの資料を見つけ候補となる人物を何人か発見したが、いずれも不正解。

## ❌ protest (unsolved)

> 23 歳の環境保全運動家の殺害場所となった道路の現在の路線番号は何か。

`environmentalist`, `activist`, `23`, `killed`などのキーワードで検索してみるもヒットしない

そこで道路上で殺害されたとの情報から道路封鎖活動を行っていたのではないかと考え
[X](https://x.com/search?q=%E9%81%93%E8%B7%AF%E5%B0%81%E9%8E%96+23%E6%AD%B3&src=typed_query)で`道路封鎖`, `23歳`と検索すると
https://www.afpbb.com/articles/-/3201942
がヒット
関連する情報を探していくと
https://www.leparisien.fr/faits-divers/a-avignon-un-gilet-jaune-de-23-ans-tue-apres-avoir-ete-percute-par-un-camion-13-12-2018-7967346.php

> sortie Avignon Sud de l'A7 「A7 号線の Avignon Sud 出口近くのロータリー」
>
> https://www.charentelibre.fr/societe/gilets-jaunes/avignon-un-gilet-jaune-de-23-ans-tue-sur-un-rond-point-par-un-camion-6093877.php?csnt=18ff6992c28
> par un poids lourd à un rond-point près d'une sortie d'autoroute à Avignon (Vaucluse) アヴィニョン（ヴォークリューズ）の高速道路出口付近のラウンドアバウトで大型車による衝突事故。
> Bonpas à Avignon sud ボンパス交差点

https://www.francebleu.fr/infos/faits-divers-justice/un-gilet-jaune-est-mort-apres-avoir-ete-percute-par-un-camion-a-avignon-1544682480

殺害場所は[ここ](https://www.google.com/maps/@43.8898575,4.9134385,3a,75y,281.14h,82.21t/data=!3m6!1e1!3m4!1sGivEe9FqSuMMnBs-TIeQRw!2e0!7i16384!8i8192?coh=205409&entry=ttu)であると考え付近の路線番号を提出するも incorrect

:::message
writeup を書きながら振り返ってみると[黄色いベスト運動](https://ja.wikipedia.org/wiki/%E9%BB%84%E8%89%B2%E3%81%84%E3%83%99%E3%82%B9%E3%83%88%E9%81%8B%E5%8B%95)はフランス政府への抗議運動であり`環境保全活動家`という前提から外れていたような気がする
:::

# transportation

## ✅ youtuber

[動画](https://web.archive.org/web/20231025043459/https://mustsharenews.com/youtuber-japan-without-paying/)を我慢して見ると、リレーかもめからさくら 572 号に乗り換えていることがわかる。

> 同氏は諫早駅から西九州新幹線に乗車し、乗務員に乗車券の提示を求められると体調不良を訴え、隙を見て逃走した。有人改札を突破し、新鳥栖駅を経由して岡山駅まで乗車したとみられる。
> https://news.yahoo.co.jp/articles/bc10510d5fed2d71ed80c26b7c6b64eb80aefc16

報道から簡単にわかる新鳥栖乗り換えは罠で武雄温泉を疑っていたが、その必要はなかった。

> Diver24{新鳥栖\_572A}

## ✅ accident

> 画像に映るバスと似たような車の観光バスが 2023 年に起こした事故の時刻と場所を探してください。

[事故の報道](https://www.taipeitimes.com/News/taiwan/archives/2023/10/22/2003808051)は誰かが特定してくれたので、地名と道路番号から[事故現場を特定](https://www.google.com/maps/@23.7026986,120.6031039,3a,75y,73.74h,85.15t/data=!3m6!1e1!3m4!1smqRuI1q5weWlRuDpnINzUA!2e0!7i16384!8i8192?coh=205409&entry=ttu)。

また前述記事には

> around 9:40 a.m. on Saturday (Oct. 21)

とあるが、詳細な日時がわからないので、YouTube で右往左往すると[映像](https://www.youtube.com/watch?v=RwjyU5dXo1s)が見つかる。

**なお、時刻はバスが事故を起こしてから完全に停止した時刻を指します。** を見落として 1 ペナ。

> Diver24{2023-10-21 09:43:00_23.702N120.603E}

## ✅ italy

> 2023 年 10 月 27 日、新潟大学に通う学生が「珍しいことにイタリアのヘリコプターがキャンパスの上空を通った」と話していた。(中略)彼の主張を裏付けるため、以下のものを調べてほしい
>
> - ヘリコプターの保有者（社名）
> - キャンパス上空を通過した時刻（JST）と、そのときの時の気圧高度

ドクターヘリの存在を誰かが示してくれたものの、周辺に配備されてそうな `JA15AC` や `JA32KC` は FR24 で当該日のログが見当たらない……（そもそも `JA32KC` は Eurocopter なのでフランス製）

頭を抱えていたところ、新潟空港に `Alidaunia` 社の `I-LIDI` とやらが PR のために飛来していたことが[判明](https://x.com/neet_KY/status/1717856013712576863)。
問題文の「珍しいことに」とも噛み合い、当該日に富山から新潟に移動しているので、この機体で間違いなさそう。

FR24 Gold なのでログを表示できて、前後の時刻から `13:45:45` 頃と推定。

![](https://storage.googleapis.com/zenn-user-upload/c0aefb5e52f7-20241029.png)

> Diver24{Alidaunia_13:45:45_1600}

## 🚫 container (unsolved?)

[場所](https://www.google.com/maps/@35.6933085,139.8261222,3a,75y,230.56h,94.57t/data=!3m6!1e1!3m4!1s1yT5crE7S63OuuTjTx-JTA!2e0!7i16384!8i8192?coh=205409&entry=ttu)は誰かが見つけてくれてた。

ビルの建設時期から 2021 年以降であることがわかり、「勝矢祭り」は 2022 年まで中止だったため、2023 年か 2024 年に絞られるが、道路の自転車マークの新しさから 2024 年と推測できる。

コンテナのナンバリングから追跡をかけるとこんな感じで、APL の結果から 2024/04/22 午後に東京港に入っており、それ以前っぽい。

![](https://storage.googleapis.com/zenn-user-upload/750508497441-20241029.png)

![](https://storage.googleapis.com/zenn-user-upload/52df47f75a35-20241029.png)

路面が濡れているのでアメダスの記録を見ながら以下の日時に絞り込んだものの、これ以上の手がかりがなく放置。終了間際に提出するも得点ならず。

- `2024-04-20-PM`
- `2024-04-21-PM`
- `2024-04-22-AM`

（実はこの中に正解が含まれています）

運営に問い合わせたところ（お手を煩わせてしまいすみません！）、時間順に以下のフラグが提出されていたことがわかった。

- `Diver24{2024-04-21-PM}`
- **`2024-04-22-AM`**
- `2024-04-20-PM`

フラグ形式が違う……
CTF 経験の浅さ（+ 徹夜による頭の回転の悪さ）が出てしまい、チームメイトには大変申し訳無い限りです……

:::message
この問題は 496 pt (12 solves) なので、正しく通していれば公式 writeup の順位表（～ 15 th）には載れたことになります。悔しい……
:::

# investigation_request

## ❌ mapper (unsolved)

> あなた方は極めて高い調査スキルを持っていると聞いた。我々の身分は明かせなくて申し訳ないのだが、一つ調査依頼を受けてくれないだろうか。我々はある男を追っている。
> 情報が見つからず困っていたのだが、彼が撮ってアップロードした写真を見つけた。現地時間でいつ撮影されたのか特定してほしい。

[画像の位置](https://www.google.co.jp/maps/@35.411964,136.7567623,3a,75y,169.83h,76.9t/data=!3m7!1e1!3m5!1sX76YmRTRhc0TnxDEgwr1Nw!2e0!5s20221101T000000!7i16384!8i8192?hl=ja&coh=205409&entry=ttu)はここ。（誰かが見つけてくれた）

- 試したアプローチ

  1. exif からの位置情報の特定

     - Special Instructions : FBMD+(94 桁の数字) を発見
     - いくつかの記事から Facebook が付加する metadata であることを把握
     - metadata から位置情報の復元が可能か調べたが失敗に終わる。（Crypto ではなく OSINT をしろ）

  2. ストリートビューで撮影された周辺に時計などが存在していないか散策
     - 「我々が追っている男」＝「ストリートビューの撮影者」と予想
     - （今冷静になると、問題では写真は彼がアップロードした言っているのでおかしなことを考えている。）
     - ストリートビューに時計などが写っていれば特定できると考えた。
     - 周辺を調べてみたがが見つからず。
     - Twitter で岐阜にストリートビューの車が巡回していた日時は大まかに把握。
     - 座標と影（太陽の傾き）から時刻を特定できないかと考えたが失敗に終わる。（OSINT をしろ）

# おわりに

改めてにはなりますが、開催ありがとうございました！次回作にも期待しています！

個人的感想として、私自身が飛行機オタクであり、これが主催の ryo-a さんと噛み合ったことにより解ける問題が多くて楽しかったです。
FR24 Gold を使ったり若干ずるい部分はあったので、正攻法でも試してみたいなと思います。
container を通せなかったのは悔しいですが、逆に私が CTF 初心者である証明になったのかもしれません。
