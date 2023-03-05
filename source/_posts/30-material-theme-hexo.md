---
title: 【Hexo】マテリアル（material）テーマの適用・設定方法
date: 2023-03-04 17:53:26
tags:
- Hexo
- マテリアルデザイン
categories: Hexo
thumbnail: /img/thumbnails/thumbnail-30.png
---

ブログをWordpressからHexoに変更し、material テーマを適用しました。
material テーマは、Gogleが推奨しているマテリアルデザインをベースとした美しいデザインです。

HexoはテーマをGithubからダウンロードして実装しますが、material テーマにはバグが残っており、そのまま利用する事が出来ませんでした。

今回、material テーマ適用に必要な修正版のテーマファイル提供と設定手順を記載しておきます。

<!-- {% asset_img 1.png %} -->

___
目次
<!-- toc -->

___

## 前提
- OS:Windows

## Hexo環境構築

### Node.jsをインストール

Hexoをインストールするための **npmコマンド** を使えるようにする必要があります。
そのために、まずはNode.jsをインストールします。

下記リンク先から **Node.jsをダウンロード** して下さい。

https://nodejs.org/ja/

次に、ダウンロードした**Windows Installer(.msi)をダブルクリックしてインストール**します。

### Hexoをインストール

コマンドプロンプトで以下を実行します。
```
npm install -g hexo-cli
```

これで、**Hexoのインストール完了**です。

### Hexoの環境構築

ブログ用のディレクトリ（フォルダ）を準備します。
例として`「C:\hexo-blog」`をHexoブログ用ディレクトリとします。

コマンドプロンプトで以下を実行します。
```
npx hexo init {Hexoブログ用ディレクトリ}

例）
npx hexo init C:\hexo-blog
```

Hexoブログ用の各種ディレクトリが生成されていれば完了です。

## material テーマ環境構築

### material テーマをダウンロード

[オリジナル](https://github.com/iblh/hexo-theme-material)はバグが残っていますので、修正版をダウンロードして下さい。

↓修正版
https://github.com/atman-33/hexo-theme-material

Code > Download ZIP でzipファイルをダウンロードします。
{% asset_img 2.png %}

ダウンロードが完了すれば、zipを解凍し、中身を{Hexoブログ用ディレクトリ}直下の**themes/materialフォルダ**に格納します。

```
{Hexoブログ用ディレクトリ}\
|-themes\
   |-material\
      ↑この中にzipファイルの中身が保存されていればOK

```
{% asset_img 3.png %}

### material テーマを設定

次に、material テーマ を読み込むように修正します。
{Hexoブログ用ディレクトリ}内の「_config.yml」を テキストエディタ で開き、theme を material に変更します。

▼{Hexoブログ用ディレクトリ}/_config.yml
{% asset_img 4.png %}

この時点でHexoブログをローカルサーバーで確認可能です。
確認するには、コマンドプロンプトで以下を実行します。

```
cd {Hexoブログ用ディレクトリ}
hexo server -g
```

ブラウザでURLに`「http://localhost:4000」`を入力すれば、Hexoブログを確認できます。

## Hexo/material の config 設定

Hexo 及び material テーマは初期設定のままでは扱い辛いため、設定を変更します。

Hexoブログ用ディレクトリ内には、2つの_config.ymlファイルが存在します。
説明時にどちらの_config.ymlを示しているのか明確にするため、ここからは以下の名称で記載する事とします。
- ①ルート_config.yml
- ②テーマ_config.yml

```
{Hexoブログ用ディレクトリ}\
|-_config.ml・・・①ルート_config.yml
|-themes\
   |-material
      |-_config.yml・・・②テーマ_config.yml
```

### 言語設定

Hexoブログ用ディレクトリから、テキストエディタで ルート_config.yml ファイルの language プロパティを設定します。

▼ルート_config.yml
{% asset_img 5.png %}

language を ja にして、日本語にします。

### 記事に対応した画像保存用フォルダ

ルート_config.yml の post_asset_folder の値を変更しておきます。

`post_asset_folder: `**`true`**

post_asset_folder の値を true にすると、_postフォルダ直下に新規記事作成時に記事に対応したフォルダが自動的に生成されます。
記事で使用する画像を、そのフォルダに保存しておけば、記事中に参照する事が可能です。

▼記事内でassetフォルダの画像を読み込む場合の記述
```
{% asset_img 画像ファイル名.jpg %}
```

### ブログのアイコン・画像

ここからは、テーマ_configy.ymlを設定していきます。
ブログのアイコンや画像は以下の部分で設定しています。

▼テーマ_config.yml
{% asset_img 6.png %}

▼テーマ_config.yml
{% asset_img 7.png %}

参照元の画像ファイル置場は下記にありますので、必要に応じて修正します。
**`{Hexoブログ用ディレクトリ}\themes\material\source\img`**

### SEOを最適化
この設定を有効にすると、構造化データがページのヘッダーに生成され、Google などの検索エンジンの SEO を改善するのに役立ちます。
ただし、`hexo g` に 問題がある場合は `false` に設定してみてください。

▼テーマ_config.yml
{% asset_img 8.png %}

### スローガンと背景色

スローガン（トップページの概要文）と背景色は下記から設定します。

▼テーマ_config.yml
{% asset_img 9.png %}

### ページのJavaScriptエフェクト

▼テーマ_config.yml
{% asset_img 10.png %}

### 投稿ページ要旨の単語数

▼テーマ_config.yml
{% asset_img 11.png %}

### 投稿ページのサムネイル

material テーマは 19 枚のシンプルな画像が準備されています。投稿ページにサムネイルが定義されていない場合、テーマはランダムフォルダ（{Hexoブログ用ディレクトリ}\themes\material\source\img\random）からランダムに写真を選択します。

ランダムに表示する画像数は下記から設定します。

▼テーマ_config.yml
{% asset_img 12.png %}

投稿ページにサムネイルを設定する場合は、投稿ページのmdファイルの上部に**thumbnail**を設定すればOKです。
サンプルは下記となります。

▼投稿ページのファイル.md
{% asset_img 13.png %}

上記の例では、「{Hexoブログ用ディレクトリ}\themes\material\source\img\thumbnails」フォルダを作成し、そこに保存したファイルをサムネイルとして利用しています。

### フォント

フォントは、初期設定だと日本語の見栄えが良くなかったため、下記のように変更しました。

▼テーマ_config.yml
{% asset_img 14.png %}

### コードブロックの強調表示

material テーマは、2 種類のコードの強調表示を提供しています。
- prettify
- hanabi

上記を有効にするには、ルート_config.yml 内のコードの強調表示をオフにする必要があります。（そうしなければ競合してしまいます。）

以下は、habani の設定例です。

▼ルート_config.yml
{% asset_img 15.png %}


▼テーマ_config.yml
{% asset_img 16.png %}


hanabi 表示効果のサンプルを載せてきます。
{% asset_img 17.png %}

### SNS設定

必要無いリンクは空白でOKです。

▼テーマ_config.yml
{% asset_img 18.png %}

### サイドバー設定

サイドバーをカスタマイズします。

▼テーマ_config.yml
{% asset_img 19.png %}

この設定の場合、下記のように表示されます。

{% asset_img 20.png %}

アイコンは、<u>Google の Material Icon</u> を利用しています。
　↓
https://fonts.google.com/icons?icon.set=Material+Icons

利用したいアイコンを見つけて、サイドバーの「icon: **ココ**」に設定して下さい。

**※Google の material icon が全て利用できるわけではなく、表示が反映されないアイコンもあります**のでご注意下さい。理由は不明です。

サイドバーの「タグ」・「リンク」・「問い合わせ」は専用のページを作成する必要があります。方法は後述します。

### タグページ作成

▼テーマ_config.yml
{% asset_img 21.png %}

コマンドプロンプトを開き、Hexoブログ用ディレクトリに移動して、以下のように入力します。

▼コマンドプロンプト
```
cd {Hexoブログ用ディレクトリ}
hexo new page "tags"
```

`{Hexoブログ用ディレクトリ}\source\tags` フォルダ内の `index.md` ファイルを、以下のように編集します。

▼{Hexoブログ用ディレクトリ}\source\tags\index.md

```
---
title: tags
date: 2023-02-26 15:50:54
layout: tags
---
```

**`layout: tags` が必須**です。

### リンクページ作成

▼テーマ_config.yml
{% asset_img 22.png %}

コマンドプロンプトを開き、Hexoブログ用ディレクトリに移動して、以下のように入力します。

▼コマンドプロンプト
```
cd {Hexoブログ用ディレクトリ}
hexo new page "links"
```

`{Hexoブログ用ディレクトリ}\source\links` フォルダ内の `index.md` ファイルを、以下のように編集します。

▼{Hexoブログ用ディレクトリ}\source\links\index.md

```
---
title: links
date: 2023-02-27 18:03:34
layout: links
---
```

**`layout: links` が必須**です。

また、`Hexoブログ用ディレクトリ\source` フォルダ内に「**\_data**」フォルダ を作成し、その直下に「**links.yml**」ファイルを作成して編集します。

編集内容のサンプルは下記となります。リンクを複数に増やす場合は、続けて記載していけばOKです。

▼{Hexoブログ用ディレクトリ}\source\\_data\links.yml
```
Github atman-33: 
    link: https://github.com/atman-33
    avatar: https://avatars.githubusercontent.com/u/41929192?v=4
    descr: "Github トップページ"

Google Material Icons: 
    link: https://fonts.google.com/icons?icon.set=Material+Icons
    avatar: https://www.gstatic.com/images/icons/material/apps/fonts/1x/catalog/v5/favicon.svg
    descr: "Google マテリアルアイコン"


```

### 問い合わせページ作成

▼テーマ_config.yml
{% asset_img 23.png %}

コマンドプロンプトを開き、Hexoブログ用ディレクトリに移動して、以下のように入力します。

▼コマンドプロンプト
```
cd {Hexoブログ用ディレクトリ}
hexo new page "about"
```

`{Hexoブログ用ディレクトリ}\source\about` フォルダ内の `index.md` ファイルを編集します。

▼{Hexoブログ用ディレクトリ}\source\about\index.md

```
---
title: 問い合わせ先について
date: 2023-02-27 18:14:12
---

ご閲覧頂きありがとうございます。

質問・要望などありましたら、Twitterで連絡ください。

※ここにTwitterのURLなど

```

このページは、通常の固定ページです。
そこにTwitterなどのURLを掲載すれば、問合せ先として利用可能です。

___

以上です。