---
title: "予約･受付･統制システムの仕組み"
---

来場者が予め予約し、当日の入退場・また各展示の入退室を管理する、予約･受付･統制システムについての概要を記す。

# 1-1. 予約の流れ

1. 文化祭公式ウェブサイト [afes.info](https://afes.info) 上に設置された予約フォームから予約をする。
2. 予約時に、個人情報として「氏名」「住所」「緊急連絡先の電話番号」「メールアドレス」を取得する。
3. 当日の受付時に使用する QR コードをメールアドレス宛に送信する。

[→ 2-1. 個人情報の取り扱いについて](#2-1-個人情報の取り扱いについて)

# 1-2. 受付・入場の流れ

1. 入場口に受付用のタブレットを設置する。
2. 入場口の担当者が、予約者の提示した QR コードとリストバンドに印刷されている QR コードを読み取り、予約情報とリストバンドの紐付けをする。

## 1-2-1. リストバンドについて

- 予約・入退室管理に、QR コード付きのカラーリストバンドを使用する。価格面や衛生面を考え紙製のものを採用する。
- 在校生には予め全員に配布し、来場者に対しては当日受付にて配布する。
- リストバンドは入場時刻/退場時刻ごとに違う色のものを使用する。ただし入場時間の制限を 120 分、30 分ごとの入場を予定しているので実際は 6 色使用し 3 時間おきに同じ色のリストバンドを使用する。

# 1-3. 展示教室への入退室の流れ

1. 展示入室時に、展示員が来場者のリストバンドの QR コードを読み取る。
2. この時、教室内の定員が超過していた場合、また退場時間が迫っている・退場時間を過ぎた場合に警告を表示するか入室を拒否するメッセージを表示する。
3. 退室時も同様に QR コードの読み取りをする。

# 1-4. その他管理システムを用いて行う事

- 各展示や統制局などに対して、各展示や学校全体での 滞在者数 / 滞在者上限 をリアルタイムに表示する機能を用意する。
- 来場者の入退室情報は全て記録し、後日陽性者が出てしまった場合に、感染者の入退室記録を確認して感染者との濃厚接触者が居なかったかどうか確認や、感染者と同じ時間帯に同じ展示にいた人物に対する注意の連絡をすることができる。