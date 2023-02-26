---
title: 【MQL】WebRequest関数でjsonデータを送信
date: 2023-02-25 23:29:15
tags:
- MQL
- WebRequest
- json
categories: MQL
---

FXの売買で用いられるMT4用の言語であるMQLに関する記事です。

今回、Webサーバーへのリクエスト送信を行うWebRequest関数を用いてjsonデータを送信する方法を紹介します。

## WebRequest関数
指定されたWebサーバーにHTTPリクエストを送信する関数です。

WebRequest関数は引数違いで2種類あり、今回紹介する方法では下記のタイプを用います。

```
int  WebRequest(
   const string      method,           // HTTPメソッド
   const string      url,              // URL
   const string      headers,          // ヘッダー
   int               timeout,          // タイムアウト
   const char        &data[],          // HTTPメッセージ本体の配列
   char              &result[],        // サーバーの応答データを含む配列
   string            &result_headers   // サーバー応答のヘッダ
   );
```
より理解したいのであれば、下記のサイトも参考にしてください。

[参考サイト](https://yukifx.web.fc2.com/sub/reference/05_common_func/cone/commonfunc_webrequest.html)

## jsonデータをPOSTする方法
WebRequest関数によりjosnデータをPOSTするステップを説明します。

### ①WebRequestを許可するURLを設定
MT4ではWebへのリクエストを送信できるURLを指定する必要があります。

まずMT4を立ち上げて、メニューバーから「ツール」→「オプション」→「エキスパートアドバイザ」の画面を開きます。

その画面に『WebRequestを許可するURLリスト』がありますので、チェックボックスに✓を入れて、リクエスト送信するURLを追加します。

{% asset_img add_url.png %}  

___
### ②WebサーバーにjsonデータをPOST
投稿系のWebサーバーにテキストを送信する例を説明します。

下記のようなテキストを送ることとします。

こんにちは。
さようなら。
これをMQLでString型の変数に格納すると下記のようになります。

```
// MQLで準備するテキスト
string text;
text = "こんにちは。" + "\n" + "さようなら。"
```
このテキストをjson形式でリクエスト送信するには、下記のように変換しておかなければなりません。

```
{"content":"こんにちは。\nさようなら"}
```

※content部分はあくまで例です。対象のWebサーバーによって異なりますのでご注意ください。

上記のようにjson形式にテキストを変換して、WebRequest関数に読み込ませることでリクエスト送信ができます。

MQLで例を示します。

```
// MQLで準備するテキスト
string text;
text = "こんにちは。" + "\n" + "さようなら。"       
string headers;

// ここからテキストをjson形式に変換してWebリクエスト送信
string data;
char post[],result[];

// ヘッダー部分の作成
headers = "Content-Type: application/json\r\n";      

// ボディ部分の作成
  // 改行がjsonとして認識されるように文字列 \n に置換
StringReplace(text, "\n", "\\n");
data = "{\"content\":\""+text+"\"}";
  // 文字エンコードををUTF-8に変換
ArrayResize(post,StringToCharArray(data,post,0,WHOLE_ARRAY,CP_UTF8)-1);

// リクエスト送信
int rest=WebRequest("POST",＊WebサーバーのURL＊,headers,5000,post,result,headers);
```

上記例のように、改行などの特殊文字はjsonで扱える文字列に変換が必要なので注意が必要です。

以上です。
