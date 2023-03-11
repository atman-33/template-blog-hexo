---
title: 【C#】Prism Xamarin Forms 「参照アセンブリが見つかりません」 対処
date: 2023-03-11 20:36:03
tags:
- C#
- Xamarin
- Prism
categories: C#
---

Visual Studioで、Prismの「Xamarin Forms」を利用していた際、UWPプロジェクトのビルドが通らなくてエラーが発生していました。

対処方法を残しておきます。

## 現象

環境は以下

- Windows 10
- Visual Studio 2022
- Prism Template Pack インストール済み

エラーメッセージはこちらです。

{% asset_img 1.png %}

> .NETCore,Version=v5.0 の参照アセンブリが見つかりませんでした。この問題を解決するには、このフレームワーク バージョンの Developer Pack (SDK/Targeting Pack) をインストールするか、アプリケーションのターゲットを再設定してください。https://aka.ms/msbuild/developerpacks で .NET Framework Developer Pack をダウンロードできます。


## 対処

Visual Studio インストーラーから、**ユニバーサル Windows プラットフォーム開発をインストール**する必要があります。

{% asset_img 3.png %}

検証できていませんが、合わせて「.NET マルチプラットフォーム アプリの UI開発」も必要かもしれません。（今回のエラーが発生した際、マルチプラットフォームUI開発は既にインストール済みだったため）

以上です。