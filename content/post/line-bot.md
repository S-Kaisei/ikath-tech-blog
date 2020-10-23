---
title: "画像を自動で Google Drive に保存する Line Bot を作成したので紹介"
date: 2020-10-24T06:29:56+09:00
draft: true
---

友人と連絡を取り合う時によく LINE を使うのですが、送信された画像をその時のイベントごとに自動で保存できたら便利だなと考え、line bot を作ることにしました。また、グループメンバーはみんな情報系出身のためコマンドライン風に実装してみました。

{{< picture "eg.png" "eg.png" "example" >}}



## Requirement

- Line Developer の登録
- Google Account の所持



​       

## GAS (Google App Scripts) でコーディング

> GAS でコーディングをするときは、1つの Google Account のみでログインしていないとだめみたいです。複数アカウントでログインしている場合は一度全てのアカウントをログアウトさせてから対象となるアカウントでログインしましょう。

