---
title: "solafune コンペ#1に参加しました"
date: 2020-12-06T11:01:54+09:00
draft: false
---

cf. @solafune (https://solafune.com) 

solafune 主催のコンテスト#1 に参加しました。このコンテストで私がどのようなアプローチをとり、結果に対してどう考察したのかを紹介します。ちなみに、コンテストの結果は 8/118人 でした。本記事は参加していないとイマイチ理解し難い記事となっていることをご了承ください。(画像データ等が取り扱えないため)

　　　

## 概要

衛生画像から空港利用者数を予測します。詳しくは[こちら](https://solafune.com/#/competitions/ea90cba4-3e01-42df-9516-9ac0d7a44204)。

　　　

## 始めにやったこと

始めに、それぞれの衛生画像に対して空港利用者数がどれくらいなのかを確認して考察しました。ここで気づいたのが、以下の二つです。

・ 空港が大きければ、利用者数も多い。

・ 空港の立地によって利用者数が異なる。森林が多いと少なく、海岸近くなら多い。

この二つの特徴をうまく数値化して、回帰分析すれば良い結果が出ないかなと考えました。ただ、これを適用する前に一旦定番の転移学習・ファインチューニングを試してみることにしました。

　　　

## 転移学習・ファインチューニング+アルファ

基本的に多くの参加者が試したのではないでしょうか。画像があったら CNN の転移学習・ファインチューニングって感じですよね。私も実際に試してみました。最初は特に前処理・データ拡張等は行わず、ResNet と imagenet で事前に学習した重みを使って学習させました。結果としては、train data、validation data 共に誤差がとても大きく、テストとして一度提出してみるのも躊躇うレベルでした。ここで、なんで精度が悪かったのかを考えたとき、全体画像に対して空港が小さすぎるのが問題ではないかと考えました。なので、空港部分をクロップして新たなデータセットにすることにしました。幸いにも空港を中心として衛生写真が撮られていたので、中心から一定の大きさでクロップしました。ここでの結果は、クロップ前よりは精度が大幅に上がりましたが、いまいちパッとしない精度でした。一度テストとして提出してみましたが、スコアは 2.6 程度で当時では 20 位くらいでした。次に試したのは、事前学習させた重みを使わずにスクラッチで行ってみました。特徴抽出の仕方が一般的なのより異なっていたら精度に影響するかもしれないと考えたからです。すると、結果はそこそこ良くなり、テストデータに対しては、1.8 を記録しました。CNN に関するアプローチはここで終わりにしました。

​       

## NN による回帰分析

上の方にも書きましたが、空港の大きさと空港の立地を数値化して、説明変数として用いて回帰分析ができれば、精度が上がるのではないかと踏んでこちらを試してみることにしました。どのように数値化するのかが問題になってくるのですが、私はそれぞれ以下の方法を試してみることにしました。

・ セマンティックセグメンテーションによる空港の検出

・ エントロピー 計算による数値化

​     

#### セマンティックセグメンテーションによる空港の検出

最初は画像処理技術をふんだんに使用して空港を検出しようと考えたのですが、白飛びしている画像等があり、なかなかうまくいきませんでした。なので、セマンティックセグメンテーションを使って空港を検出することにしました。セマンティックセグメンテーションの手法は DeepLabv3+ を使用しました。DeepLabv3 に関してなかなか独自のデータセットを使って学習する記事がなく、苦戦を強いられましたが、色々試行錯誤してなんとか学習まで行うことができました。validation data に対して検出を行ったところ、見た目ではあるのですが、かなり良い精度が出せていました。

​       

#### エントロピー計算による数値化

エントロピーを使えば立地の数値化ができるのではないかと考えました。例えば、海のような同色の割合が多くある場所だとエントロピーは小さくなり、山や街並みが多く見える場所は複雑ゆえエントロピー値が大きくなります。

　　　

この２つの値を説明変数として使用して、回帰分析を行いました。最初中間層のノード数を何も考えず 64 にしていたのですが、そのときの結果は、転移学習・ファインチューニングしたときの結果とほぼ同等でした。インプットデータ数が2つなのに中間層のノード数が 64 でいいのか疑問に感じ、4 に変更して試したところ結果は以下のようになりました。

{{<picture "solafune_result1.png" "solafune_result1.png" "solafune_result1">}}

　　　

## まとめ

データを把握してからどのようにアプローチするかを決めて、実行に移しました。今回は、CNN の転移学習・ファインチューニングと NN による回帰分析を試し、結果的には後者が精度がよく採用しました。NN による回帰分析の方はあまり考察ができず、結果を詰めれなかったのが少し悔いですが、自分なりに考えを出し結果まで出すことができたので良かったです。今後も積極的にコンテストに参加したいと思います。以上です。お疲れ様でした。





 

　　　





