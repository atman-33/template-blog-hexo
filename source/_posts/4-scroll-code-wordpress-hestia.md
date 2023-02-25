---
title: 【WordPress】テーマHestiaの記事に埋め込んだコードを横スクロールさせる
date: 2023-02-25 15:28:19
tags: WordPress,css
categories: WordPress
---

WordPressのテーマ”Hestia”を使用している方への記事となります。

投稿ページにソースコードを組み込んだ場合、折り返されて見づらくなってしまうことはないでしょうか？

今回は追加CSSを使った対処方法について説明します。

2020/3/22 追記:
『①（ソースコードの）ブロックを選択』して、『②ブロックタイプの変更から整形済みを選択』することでスクロール化を実現できることが分かりました。この機能で十分です。

{% asset_img seikeizumi.png %}

## Webに表示したソースコードが見辛い場合

下記のような横に長いソースコードのサンプルであれば、自動的に折り返されて見づらくなってしまいます。
```
def __line(self, message):
        if len(self.line_notify_token) > 0:
            requests.post('https://notify-api.line.me/api/notify', headers={'Authorization': 'Bearer ' + self.line_notify_token}, data={'message': '\n' + message})
```

## 追加CSSの設定

それでは、追加CSSを設定してソースコードの横スクロール化を実現してみましょう。
まず、WordPressのカスタマイズから追加CSSの編集画面に移動します。

{% asset_img main.PNG %}

次に下記のコードを入力してください。

{% asset_img add_css.PNG %}

```
pre.scrollable-code {
  overflow-x: auto;
  margin-bottom: 1em;
  white-space: pre;
  max-width: 770px;
  word-wrap: normal;
}
```

## 投稿画面でコード編集

追加CSSを反映させるために、投稿画面でコードを直接編集します。
投稿の記事編集画面で、”コードエディター”表示に切り替えてください。

{% asset_img code_editer.PNG %}

HTML表示されますので、その中からソースコードを埋め込んだブロックを探します。
``<pre class=”wp-block-code”><code>・・・</code></pre>``に囲まれている部分です。

```
<!-- wp:code -->
<pre class="wp-block-code"><code>    def __line(self, message):
        if len(self.line_notify_token) > 0:
            requests.post('https://notify-api.line.me/api/notify', headers={'Authorization': 'Bearer ' + self.line_notify_token}, data={'message': '\n' + message})
</code></pre>
<!-- /wp:code -->
```

上記コードに対して、以下の2つを実施します。
1 pre classの変更：wp-block-code　→　scrollable-code
2 ``<wp:code></wp:code>``の削除

下記のようなコードになれば完成です。
```
<!-- wp:code -->
<pre class="scrollable-code">    def __line(self, message):
        if len(self.line_notify_token) > 0:
            requests.post('https://notify-api.line.me/api/notify', headers={'Authorization': 'Bearer ' + self.line_notify_token}, data={'message': '\n' + message})
</pre>
<!-- /wp:code -->
```

横スクロールに対応したソースコードのサンプル
{% asset_img sample.png %}

以上です。
