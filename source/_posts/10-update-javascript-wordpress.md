---
title: 【WordPress】更新したJavaScriptファイルを反映させる方法
date: 2023-02-25 21:43:30
tags: WordPress
categories: WordPress
---

WordPress（Webサーバー）内のJavaScriptを更新した際、上手く反映されずハマってしまいました。

同じ現象に陥らないように対処した方法を残しておきます。

## 発生した問題
下記ソースのように、別ファイルのPlyayer.jsを読み込む場合がありました。

```
<html>
<head>
    <meta charset="utf-8" />
    <script src="js/class/Player.js"></script>
　・
　・
　・
```

もしPlyaer.jsに不具合が見つかり修正した場合、jsファイルを再度アップロードするだけではWebブラウザ上に反映されません。

## 対処方法
結論から言えば、下記のようにコードを書き換えます。

```
<html>
<head>
    <meta charset="utf-8" />
    <script src="js/class/Player.js?20200306"></script>
　・
　・
　・
```

Plyer.js の後に 『?20200306』を付け加えています。この?の後はどのような数字でも構いません。更新した日付などでOKです。

こうすることで、Webブラウザ側は新しいJavaScriptファイルという認識を行うためブラウザに反映されます。

## 原因
CSSやJSファイルのデータは、Webブラウザの **キャッシュ機能** によって保存されています。そのため、WordPress内のファイルを変更しても反映されないことがあるのです。

しかし、2. 対処方法で示したように?＋数値を付け加えれば、新規ファイル（別ファイル）と認識するため、新しくデータが読み込まれるのです。

以上です。
