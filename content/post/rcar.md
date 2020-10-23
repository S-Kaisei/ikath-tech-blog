---
title: "自動走行するラジコンを作成"
date: 2020-10-24T05:43:11+09:00
draft: false
---

## 概要

Raspbeey Pi に学習させたモデルを搭載して、道路の左車線を自動で走行するラジコンカーです。今回は直線のみを対象にし、道路からはみ出さないように設計しました。見た目が戦車ですので、停止時に物体検出を行い自作砲台 (3Dプリンターで作成) を使って輪ゴムを発射するオプションも加えてみました。

[完成形]

{{< picture "rcar.png" "rcar.png" "car" >}}

[全体の流れのイメージ]

{{< picture "course.png" "course.png" "course" >}}

## 開発環境

- Raspberry Pi 3B raspbian 9.4
- Jupyter Notebook 4.4.0
- Python 3.5.3
- TensorFlow 1.11.0
- OpenCV 3.4.4

## 行ったこと

1. ラジコンの製作
   - [これ](https://www.amazon.co.jp/gp/product/B014L1CF1K/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)の組み立て

2. Raspberry Piの環境構築
   - OS のインストール
   - SSH の設定や IPアドレスの固定、その他必要なフレームワークのインストールなど

3. 3Dプリンタの使用
   - 砲台の作成
   - 砲台やモータドライバを固定するための土台の作成

4. プログラミング(Python)
   - サーボモータ、DC モータによるラジコンの制御
   - CNN による画像の分類
   - Tiny-YOLOv3 を用いた物体検出

5. データセットの収集
   - 前進、右折、左折の3クラスの画像

6. 画像分類モデルの作成

## フローチャート

{{< picture "flowchart.png" "flowchart.png" "flowchart" >}}

今回、停止は手動で行っています。何かしらをトリガーにして停止の処理も自動でできるようにしたいですね。

## アーキテクチャ

内部と外部の2段構想になっています。ラズパイは内部と外部で同様のものです。カメラは道路を撮影する用と物体を検出する用の2つに分けました。サーボモータは片方を砲台の向きを変える用に、もう片方を輪ゴムを発射させる用にしました。

[内部]

{{< picture "architecture_inside.png" "architecture_inside.png" "architecture inside" >}}

[外部]

{{< picture "architecture_outside.png" "architecture_outside.png" "architecture out" >}}

## 学習モデルの作成

[NVIDIA DIGITS](https://developer.nvidia.com/digits) を使ってモデルを作成しました。道路からはみ出さないようにする進むには、前進と右折と左折の3パターンが必要なので、その時の状況に合わせてコースからそれぞれデータを取得します。画像は全体で1300枚程用意して、トレーニング画像とバリデーション画像を 8:2 に分割しました。ネットワークはラズパイに搭載するということもあり LeNet を採用しています。

{{< picture "dataset.jpeg" "dataset.jpeg" "dataset" >}}



## デモ

[https://youtu.be/1bVmcZxtD7E](https://youtu.be/1bVmcZxtD7E)

{{< picture "result.png" "result.png" "result" >}}