---
title: 【Power Query】動的SQLの結果を読み込む方法（Power BI）
date: 2023-02-25 16:52:40
tags:
- PowerBI
- PowerQuery
categories: PowerQuery
---

Power Queryでは、DBに接続してSQL結果をテーブルに取り込むことがよくあります。

その際、SQLの抽出条件を簡単に変えたい場合はないでしょうか？

今回、SQLの抽出条件（WHERE内など）をPower Queryのパラメーター設定と連動させることにより、動的にSQLを実行する方法を説明します。

___
目次
<!-- toc -->

___

## 動的SQLを実行するクエリ

動的SQLを実行するサンプルクエリを下記に記載します。

```
let
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
// SQL ここから
sql1 =
"
SELECT
  *
FROM
  test_table
WHERE
  start_date >= '__start_date__' AND start_date <= '__end_date__'
",
// SQL ここまで
// ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----

    // SQL の変数部（ex. __start_date__ など）を変換
    sql2 = Text.Replace(sql1, "__start_date__", START_DATE),
    sql3 = Text.Replace(sql2, "__end_date__", END_DATE),

    sql = sql3, // 最後にsqlという名称に定義しなおすことでミス抑制

    ソース = Oracle.Database(DB_NAME, [HierarchicalNavigation=true, Query=sql])
in
    ソース
```

このサンプルでは、下記のような動作の流れとなります。

### ①sql1

```
// SQL ここから
sql1 =
"
SELECT
  *
FROM
  test_table
WHERE
  start_date &gt;= '__start_date__' AND start_date &lt;= '__end_date__'
",
// SQL ここまで
```

ベースとなるSQLを記載（上記 // —- で囲まれている範囲）
※動的に変化させたい部分は任意の変数名を記載しておきます。
　（例では、__start_date__、__end_date__を使用）

### ②sql2

```
sql2 = Text.Replace(sql1, "__start_date__", START_DATE),
```

sql1に対して、__start_date__をSTART_DATE（パラメーター）に置換
※例のSTART_DATEには、文字列のYYYYMMDD形式を格納

### ③sql3

```
sql3 = Text.Replace(sql2, "__end_date__", END_DATE),
```

sql2に対して、__end_date__をEND_DATE（パラメーター）に置換
※例のEND_DATEには、文字列のYYYYMMDD形式を格納

### ④sql、ソース

```
    sql = sql3, // 最後にsqlという名称に定義しなおすことでミス抑制

    ソース = Oracle.Database(DB_NAME, [HierarchicalNavigation=true, Query=sql])

```

sql3をsqlに格納して、Oracle.Database関数の引数に適用
※例のDB_NAMEには、データベース名（スキーマ）を格納

___

試してみる際は、必要なパラメーターを準備したうえで、空のクエリ→詳細エディタからコードを貼り付けてみてください。

これでパラメーター設定を変更することにより、SQL抽出結果を動的に変更できます。

以上です。
