---
title: 【Oracle】21c XE 構築手順
date: 2023-03-11 21:54:17
tags:
- DB
- Oracle
categories: Oracle
---
Windows PCに、「Oracle Database 21c Express Edition (XE)」をインストールして利用する方法を記しておきます。

___
目次
<!-- toc -->
___

## Oracle XE とは

特徴は以下の通りです。

- 無償かつ本番環境でOracle Database XEを使用可能
- 最大12GBのユーザー・データ
- 最大2GBのデータベースRAM
- 最大2つのCPUスレッド

私はアプリ開発用で、個人PCにインストールしています。

※公式HPのOracle XE説明
https://www.oracle.com/jp/database/technologies/appdev/xe.html

## 手順

### 1. ダウンロード

Webから「Oracle Database XE」をダウンロードします。

https://www.oracle.com/jp/database/technologies/xe-downloads.html

{% asset_img 1.png %}

「Oracle Database 21c Express Edition for Windows x64」をクリックし、ダウンロードを開始します。

### 2. インストール

ダウンロードした「OracleXE213_Win64.zip」ファイルを解凍します。

解凍して作成されたフォルダ内の「setup.exe」を実行します。

「次へ」ボタン押下、「使用許諾条項の受け入れ」にチェックを入れて進めます。

宛先フォルダ画面で、インストール先を設定します。私は下記に変更しました。

`宛先フォルダ： C:\oracle\product\21c\`

上記の場合、Oracle環境変数も合わせて変更されます。

`Oracleホーム： C:\oracle\product\21c\dbhomeXE\`
`Oracleベース： C:\oracle\product\21c\`

Oracle Database情報で、データベース・パスワードを設定します。
（SYS、SYSTEM、PDBADMINアカウントのパスワード）

`パスワード例）sys`

正常にインストール完了後、DB関連情報が表示されます。

{% asset_img 2.png %}

これでインストール完了です。

### 3. ユーザー作成

コマンドプロンプトもしくは power shell で以下のコマンドを実行していきます。

Oracle 21c XEに接続
```
sqlplus / as sysdba
```

接続先を確認
```
SQL> show con_name

↓

CON_NAME
------------------------------
CDB$ROOT
```
コンテナデータベース（CDB）に接続されているため、プラガブルデータベース（PDB）に接続を切り替えます。

存在するプラガブルデータベースを確認します。
```
SQL> show pdbs

↓

    CON_ID CON_NAME                       OPEN MODE  RESTRICTED
---------- ------------------------------ ---------- ----------
         2 PDB$SEED                       READ ONLY  NO
         3 XEPDB1                         READ WRITE NO
````

既に存在する「XEPDB1」を利用する事とします。

```
SQL> ALTER SESSION SET CONTAINER=XEPDB1;

↓

セッションが変更されました。
```

```
SQL> show con_name

↓

CON_NAME
------------------------------
XEPDB1

```
「XEPDB1」に接続しました。

続いてユーザーを作成します。

- ユーザー: atman
- パスワード: atman

プロファイルは「DEFAULT」を設定してパスワード期限を無期限にします。

```
SQL > 
CREATE USER atman IDENTIFIED BY atman;
GRANT CONNECT, RESOURCE TO atman;
ALTER USER atman QUOTA UNLIMITED ON users;
ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
```

プロファイルを確認してみます。
```
SQL >
select username, profile from dba_users
where username = 'ATMAN';
```
プロファイルは「DEFAULT」となっていました。

```
SQL >
SELECT * FROM DBA_PROFILES
WHERE RESOURCE_NAME = 'PASSWORD_LIFE_TIME';
```
プロファイルDEFAULTのパスワードが無期限となっていることを確認します。

これでユーザー作成は完了です。

補足ですが、表領域は下記になっています。

- 表領域: USERS
- 一時表領域: TEMP

確認用のSQLです。
```
SELECT username,default_tablespace,temporary_tablespace 
FROM DBA_USERS WHERE username='ATMAN';
```


### 4. tnsnames.ora 編集

最後に、今回利用したプラガブルデータベースに接続するために、tnsnames.oraを編集します。

___
【補足】
tnsnames.ora の保存場所を確認する方法です。

コマンドプロンプトを開いて、**tnsping をわざと失敗**させます。
例えば `tnsping aaa`と実行します。（aaaは仮。存在しないTNS名でなければ何でもOK）
```
C:\Repos\computing-atman> tnsping aaa

↓

TNS Ping Utility for 64-bit Windows: Version 21.0.0.0.0 - Production on 11-3月 -2023 22:52:44

Copyright (c) 1997, 2021, Oracle.  All rights reserved.

パラメータ・ファイルを使用しました:
C:\oracle\product\21c\homes\OraDB21Home1\network\admin\sqlnet.ora

TNS-03505: 名前の解決に失敗しました。
```

上記の**sqlnet.oraファイルの保存ディレクトリに、tnsnames.oraも保存**されています。
___

tnsnames.oraの最後に、以下の記述を追加します。

```
XEPDB1=
(DESCRIPTION=
  (LOAD_BALANCE=off) 
  (ADDRESS=
    (PROTOCOL=tcp)  
    (HOST=localhost)  
    (PORT=1521)
  ) 
  (CONNECT_DATA=
    (SERVICE_NAME=XEPDB1) 
    (FAILOVER_MODE=
      (TYPE=select) 
      (METHOD=basic)
    )
  )
)
```

コマンドプロンプトで、tnspingで接続確認します。
```
tnsping XEPDB1
```

OKと表示されれば、問題無しで完了です。