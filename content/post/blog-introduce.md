---
title: "Hugo × GitHub Pages でブログを公開するまでの手順を紹介"
date: 2020-10-24T05:35:42+09:00
draft: false
---

「自分が行なってきたことをアウトプットする場を作りたい」と前々から考えていましたので、この度 **Hugo** と **GitHub Pages** を使ってブログを公開することにしました。なぜ Hugo かというと[友人](https://peaceiris.com/) が強く勧めてくれたからです。[ここ](https://qiita.com/peaceiris/items/ef38cc2a4b5565d0dd7c) で多くを語ってくれているのでよろしければ一読ください。

​               


## Requirement

- macOS Catalina
- GitHub Account の所持: [https://github.co.jp/](https://github.co.jp/)
- Homebrew: [https://brew.sh/index_ja](https://brew.sh/index_ja)

を前提に話を進めていきます。        

​          

## Hugo のインストール

homebrew を使って hugo をインストールします。

```
brew install hugo
```



## サイトの作成

"your site name" は任意です。
生成するとその名前のディレクトリができるので移動します。

```
hugo new site <your site name>
cd <your site name>
```

ディレクトリ構成は以下のようになってます。

```
<your site name>
├── archetypes/
├── config.toml
├── content/
├── data/
├── layouts/
├── static/
└── themes/
```



## GitHub リポジトリの作成           

今後このコンテンツは GitHub にアップロードしていくので、[https://github.com/](https://github.com/) であらかじめリポジトリを作成しておきます。
Repository name を site name と同じに設定した方が分かりやすいと思います。
このとき README や .gitignore の追加はスキップします。

{{< picture "create_repo.jpg" "create_repo.jpg" "create repository" >}}

リポジトリ作成が終わったら terminal に戻り、README.md を追加して git の初期化を行います。

```
echo "# <your site name>" >> README.md
git init
```



## Hugo テーマの追加     

テーマは [https://themes.gohugo.io/](https://themes.gohugo.io/) から選ぶことができます。本サイトは、[newsroom](https://github.com/onweru/newsroom) を利用させていただいていますので newsroom を例に説明していきます。テーマごとに設定が少し異なりますので違うテーマを選んだ方はそちらを参考にしてください。

themes/ にテーマを追加して、 config.toml にテンプレートをコピーします。

```
git submodule add https://github.com/onweru/newsroom.git themes/newsroom
```

```toml
baseurl = "https://example.com/"  # Include trailing slash
title = "Newsroom"
theme = "newsroom"
author = "Weru"
copyright = "Copyright © 2019, Weru and the Hugo Authors; all rights reserved."
canonifyurls = true
paginate = 6

[markup]
  [markup.goldmark.renderer]
    hardWraps = false
    unsafe = false # change to true to enable inclusion of rawHTML and math functions
    xhtml = false
  [markup.highlight]
    codeFences = true
    guessSyntax = false
    hl_Lines = ""
    lineNoStart = 1
    lineNos = true
    lineNumbersInTable = false
    noClasses = false
    tabWidth = 2

# If you want to use disqus, uncomment the line below
# disqusShortname = "yourdiscussshortname"

# ids used below are not valid. Replace with valid ones
[params]
  # ga_analytics = "" # google analytics ID e.g UA-116386578-1
  # ga_verify = "" # google verification code e.g 5jb-rxeBfITeJQKOuUss3ud6FPGTxXkCho-ZN5qlrZg
  # twitter = "" # twitter handle e.g @weru
  # mainSections = ["blog", "docs"] see https://gohugo.io/functions/where/#mainsections
  # uncomment the line below will disable darkmode by default.
  # disableDarkMode = true  # note that the UI control for toggling darkmode will remain in place. This way, the user can decide which mode they would like to use while browsing your website
```

ここで baseurl と theme を変更します。　

```toml
baseurl = "https://<your-github-id>.github.io/<your-site-name>/"
theme = "<your-theme-name>"

例
baseurl = "https://s-kaisei.github.io/ikath-tech-blog/"
theme = "Ikath Tech Blog"
```



## 記事の作成

記事を作成する際には次のコマンドを実行します。

```
hugo new post/<filename>.md
```

ファイルは以下のようなフォーマットで content/posts/ に生成されます。ここに markdown 記法で記事を書いてくって感じです。draft は記事を作成途中で公開したくない時に true にします。公開する時には false にしましょう。

```markdown
--- 
title: "<filename>" 
date: 2019-03-26T08:47:11+01:00 
draft: true 
---
## ここから記事を書いていく
```

サイトは hugo  server を起動した後に [http://localhost:1313/](http://localhost:1313/) にアクセスすることで確認できます。

```
hugo server -D
```



## Actions-gh-pages

gh-pages という GitHub Actions を使用してデプロイを自動で行います。これを使えば、GitHub にプッシュしただけでサイトが構築されます。方法は簡単で以下のファイルを .github/workflows/gh-pages.yml に追加するだけです。この Action について詳しい内容を知りたい方は[こちら](https://github.com/peaceiris/actions-gh-pages)をどうぞ。

```yml
name: github pages

on:
  push:
    branches:
      - master

jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: true  # Fetch Hugo themes
          fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: '0.76.5'

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

ここまで来たら、一度コミットしてプッシュします。

```
git commit -m "first article"
git remote add origin git@github.com:<your github name>/<your branch name>.git
git push -u origin master
```

## GitHub Pages の設定

GitHub Pages の設定を行います。リポジトリから Setting に行き、下の方にスクロールしていくと、下の画像のような GitHub Pages があると思います。Source のところを画像のように Branch: gh-pages from /root にして保存します。

{{< picture "gh_pages.jpg" "gh_pages.png" "gh-pages setting" >}}

ここまでの設定を一通り終えたらサイトが公開されているはずです。

​      

## 設定が終わったら

以降、記事の作成からデプロイまでの流れは次のようになります。

```
hugo new posts/<filename>.md
git add .
git commit -m 'add filename article'
git push
```



以上で終わりです。お疲れ様でした!!