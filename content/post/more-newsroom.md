---
title: "newsroom テーマと HUGO を活用してサイトをカスタマイズ"
date: 2020-10-24T08:08:16+09:00
draft: false
---



## Requirement

- [Hugo × GitHub Pages でブログを公開するまでの手順を紹介](https://s-kaisei.github.io/Ikath-Tech-Blog/post/blog-introduce/) を終えている

{{< mylist >}}

kaisei

Kaisei

{{< /mylist >}}



​      

## 画像・動画を表示する

画像は static/images に保存します。lightMode と darkMode で画像を切り替えることができます。特に気にしないのであれば同じでいいと思います。動画は youtube を公開できます。※ コードを書くと実際に画像や動画が表示されてしまうので便宜上スクリーンショットを利用させていただいてます。

{{< picture "shortcode.png" "shortcode.png" "shortcode" >}}



## コードブロックに番号をつける

config.toml の 16行目と19行目を true に変更

```
guessSyntax = true 
lineNos = true
```

なお、番号とコードを分割して表示したい場合は、20行目を true にします。

```
lineNumbersInTable = true
```

また、上記の設定をした場合 22行目を 4 に設定した方がコードが見やすくなります。

```
tabWidth = 4
```



## フッターにメニューを追加

data/ に menu.yml を追加する。

```
- item: About
  url: about/
- item: Blog
  url: post/
```

About を追加したいときは

```
hugo new about.md
```

> 公開する時に draft を false にすることを忘れずに 





