---
title: "DEV 主催の GitHub Actions Hackathon に参加してきました"
date: 2020-12-06T03:10:54+09:00
draft: false
---

タイトルにもある通り、DEV が主催した [GitHub Actions hackathon ](https://dev.to/devteam/announcing-the-github-actions-hackathon-on-dev-3ljn)に参加しましたので、備忘録として記事を書きたいと思います。

## GitHib Actions とは

GitHub Actions は GitHub のリポジトリ内でソフトウェア開発ワークフローを自動化することができるサービスです。(ここで、ソフトウェア開発ワークフローは、ビルド・テスト・デプロイなどが該当します。) ワークフローは、プッシュ、Issue、リリースなどの GitHub プラットフォームのイベントをトリガーとして起動することができます。GitHub Actions についてより詳しく知りたい方は[こちら](https://docs.github.com/en/free-pro-team@latest/actions)をどうぞ。

　　　

## Categories

本ハッカソンには、5つのカテゴリーがあり、それぞれ以下に示します。

・ **Maintainer Must-Haves:** 開発者のメンテナンスを楽にします

・ **DIY Deployments:** オープンソース・プロジェクトのデプロイメント・プロセスを改善します

・ **Interesting IoT:** IoT と連携させます

・ **Phone Friendly:** モバイル用に構築されたワークフロー (PWA、iOS/Android)・

・ **Wacky Wildcards:** 上記のカテゴリに当てはまらないもの (面白いもの)

> **Maintainer Must-Haves** にエントリーしました。

　　　

## 参加方法

・ #ActionsHackathon タグを使用して、プロジェクトの進行状況を投稿します。

・ テンプレートに従ってワークフロー (.yml) やコードリポジトリを共有し、プロジェクトを DEV post に提出します。

・ 投稿の一部としてリポジトリを共有する場合は、プロジェクトにオープンソースライセンスと README が含むようにします。

　　　

## Workflow

今回作成した GitHub Actions は、関連もしくは類似する issue を提案する [actions-suggest-related-links](https://github.com/peaceiris/actions-suggest-related-links) です。

ユーザーによっては、open issue や close issue に類似した新しい issue を作成することがあります。そのような issue に遭遇したら、類似した問題を見つけ、それをコメントとして投稿する必要があります。そのプロセスは面倒ですよね？この Action は、あなたの代わりにそれを行うことができます。

{{<picture "eg_suggest_related_link.png" "eg_suggest_related_link.png" "eg_suggest_related_link">}}

e.g. `.github/workflows/suggest-related-links.yml`

```
name: 'Suggest Related Links'

on:
  issues:
    types:
      - opened
      - edited
  workflow_dispatch:
  schedule:
    - cron: '13 13 * * */7'

jobs:
  action:
    runs-on: ubuntu-18.04
    steps:
      - name: Cache dependencies
        uses: actions/cache@v2
        with:
          path: ~/actions-suggest-related-links-tmp
          key: ${{ runner.os }}-action-${{ hashFiles('~/actions-suggest-related-links-tmp/training-data.json') }}
          restore-keys: |
            ${{ runner.os }}-action-

      - uses: peaceiris/actions-suggest-related-links@v1.1.1
      - uses: peaceiris/actions-suggest-related-links/models/fasttext@v1.1.1
        if: github.event_name == 'issues'
        with:
          version: v1.1.1
      - uses: peaceiris/actions-suggest-related-links@v1.1.1
        with:
          mode: 'suggest'
          repository: 'peaceiris/actions-gh-pages'
          unclickable: true
```

​     

## Workflow Overview

このアクションは、主にデータ収集、前処理、モデルの学習、類似する issue の検索、リンクの推奨の5つの部分で構成されています。

{{<picture "workflow.jpg" "workflow.jpg" "workflow">}}

​     

## Data Collection

リポジトリのすべての issue は GitHub API で収集されます。issue にはタイトル、本文、コメントが含まれます。トレーニングデータはスケジューリング機能を用いて定期的に収集され、アーティファクトとして出力され、 キャッシュとして保存されます。

​       

## Preprocessing

Markdownフォーマットは、統合されたプレーンテキストに変換されます。このとき、アルファベット以外の記号は削除されます。

・[https://github.com/peaceiris/actions-suggest-related-links/blob/main/actions/src/preprocess.ts](https://github.com/peaceiris/actions-suggest-related-links/blob/main/actions/src/preprocess.ts)

　　　

## Train Model

新しい issue が作成または更新されると、fastText モデルが学習されます。fastText は、その名前の通り、推論時間が非常に短いという利点があります。GitHub Actions を実行する際のトレーニング時間は問題にならないと考えています。GitHub Pages リポジトリの GitHub Actions の場合、トレーニング実行時間は1秒でした。

​     

## Find Similar Issues

fastText では、トレーニングデータの単語ベクトルと投稿データの単語ベクトルを計算します。コサイン類似度は、トレーニング・データのどのワード・ベクトルが投稿されたデータのワード・ベクトルに近いかを決定するために使用されます。コサイン類似度が高いほど、文はより類似しています。

　　　

## Suggest Issues

作成された issue の本文に類似するいくつかの関連リンクが、このアクションによって一覧表示されます。

{{<picture "eg_suggest_related_link.png" "eg_suggest_related_link.png" "eg_suggest_related_link">}}

[https://github.com/peaceiris/actions-suggest-related-links/issues/2](https://github.com/peaceiris/actions-suggest-related-links/issues/2)

​     

## まとめ

投稿された issue に対して、すでに投稿された issue と類似する issue を探して提案する [actions-suggest-related-links](https://github.com/peaceiris/actions-suggest-related-links) を作成しました。GitHub API を介して取得された issue は、fastText によってモデルを生成し、コサイン類似度を求めて関連性の高い issue をリストするといった内容でした。

所々、苦労した場面はありましたが、最後には形になって投稿できたことが良かったです。記事は以上となります。お疲れ様でした。