---
title: 【Git】ソースコードをGitへpushする方法
date: 2023-02-26 18:00:02
tags:
- Git
categories: Git
---

ソースコードをGitへpushする手順を残しておきます。

___
#### ①プロジェクトのフォルダへ移動（folder_pathは書き換えてください）

`cd folder_path`

___
#### ②ローカルリポジトリ作成　★初回のみでOK

`git init`

___
#### ③プロジェクトフォルダの全ファイルをインデックスに保存

`git add -A`

（add した内容の確認）

`git status`

___
#### ④add したファイルをコミット

`git commit -m "first commit"`

___
#### ⑤リモートリポジトリ名 origin に登録　★初回のみでOK

github でリポジトリを作成（説明は省略）

`git remote add origin https://github.com/対象リポジトリのURL`
（登録したリモートリポジトリの情報確認）

`git remote`
`git remote -v`

___
#### ⑥Github にプッシュ（originリモートリポジトリ ← masterブランチ）

`git push origin master`

※masterブランチにpushする例です。

以上です。
