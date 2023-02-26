---
title: 【Power Query】パラメータ設定で指定した期間のカレンダー作成（Power BI）
date: 2023-02-25 16:40:22
tags:
- PowerBI
- PowerQuery
categories: PowerQuery
---

Power BI を使っていると日付をキーにしたカレンダーを作成しておきたい場合があります。
そのような時に便利な、指定した期間のカレンダーを自動で作成する方法を説明します。

※追記：
カレンダーはDAXから作成した方が、列追加の融通が効くため便利と感じました。
この記事は、クエリでパラメータ利用する手順として参考になれば幸いです

## カレンダー作成の流れ

手順は下記のような流れとなります。

①パラメーター設定（複数モード考慮）
　↓
②モード別対応の日付を取得
　↓
③カレンダーの作成

## パラメーター設定

今回、設定するパラメーターは下記の通りです。

{% asset_img create_calendar.PNG %}

日付の期間するだけなのですが、利便性を考えて複数のパターンに対応できるようにモード選択機能を設けました。

### ①日付指定モード

指定した開始日付と終了日付に対してカレンダーを作成するモード

▼パラメーター設定（前提）
A_DATE_MODE = 0

▼パラメーター設定（設定が必要な項目）
B_START_DATE = 任意のYYYYMMDD[ex. 20200101]
C_END_DATE = 任意のYYYYMMDD [ex. 20200131]

### ②日数を遡るモード
今日を基準として、指定日数だけ遡った日付からカレンダーを作成するモード

▼パラメーター設定（前提）
A_DATE_MODE = 1

▼パラメーター設定（設定が必要な項目）
D_DAYS_AGO = 任意の今日から遡る日数[ex. -5（5日前）]
E_DAYS_LATER = 任意の今日から進める日数[ex. 2（2日後）]

## モード別対応の日付を取得

上記の各モードに対応させた日付を計算する方法は、下記の通りです。
後の Calendar クエリで扱う START_DATE と END_DATE を準備します。

{% asset_img template_query.PNG %}

▼クエリ：START_DATE

```
let
    // A_date_mode = 0 の場合（日付指定）
    start_date_0 = Date.FromText(Text.Start(B_START_DATE,4) & "," & Text.Middle(B_START_DATE,4,2) & "," & Text.End(B_START_DATE,2)),

    // A_date_mode = 1 の場合（指定日数を遡った期間）
    now_1 = DateTime.LocalNow() ,
    start_date_1 = DateTime.Date(Date.AddDays(now_1,D_DAYS_AGO)),

    // A_date_mode = 2 の場合（指定月数を遡った期間）
    now_2 = DateTime.LocalNow() ,
    start_month_2 = DateTime.Date(Date.AddMonths(now_2,F_MONTHS_AGO)),
    start_date_2 = DateTime.Date(Date.AddDays(start_month_2,-Date.Day(start_month_2)+1)),

    // mode に合わせた日付取得
    start_date_ori = if A_DATE_MODE = 0 then start_date_0 else (if A_DATE_MODE = 1 then start_date_1 else start_date_2)

in
    start_date_ori
```

▼クエリ：END_DATE

```
let
    // A_date_mode = 0 の場合（日付指定）
    end_date_0 = Date.FromText(Text.Start(C_END_DATE,4) & "," & Text.Middle(C_END_DATE,4,2) & "," & Text.End(C_END_DATE,2)),

    // A_date_mode = 1 の場合（指定日数を遡った期間）
    now_1 = DateTime.LocalNow() ,
    end_date_1 = DateTime.Date(Date.AddDays(now_1,E_DAYS_LATER)),

    // A_date_mode = 2 の場合（指定月数を遡った期間）
    now_2 = DateTime.LocalNow() ,
    end_date_2 = DateTime.Date(Date.AddDays(now_2,E_DAYS_LATER)),

    // mode に合わせた日付取得
    end_date_ori = if A_DATE_MODE = 0 then end_date_0 else (if A_DATE_MODE = 1 then end_date_1 else end_date_2)

in
    end_date_ori
```

空のクエリ作成　→　詳細エディター　より、上記コードをコピペすれば使用可能です。

## カレンダーの作成

カレンダーは、パラメーター設定で行ったモードに対応した指定期間だけ作成します。

▼クエリ：Calendar

```
let
    Source = List.Generate(
                    ()=>[日付=START_DATE, 曜日=Date.DayOfWeekName(日付)],
                    each [日付]<=END_DATE,
                    each [日付=Date.AddDays([日付],1),曜日=Date.DayOfWeekName(日付)]
             ),
    Custom = Table.FromRecords(Source),
    追加されたカスタム = Table.AddColumn(Custom, "年月", each Number.ToText(Date.Year([日付])) & Text.PadStart(Number.ToText(Date.Month([日付])),2,"0")),
    挿入された月の通算週 = Table.AddColumn(追加されたカスタム, "月の通算週", each Date.WeekOfMonth([日付]), Int64.Type)
in
    挿入された月の通算週
```

このクエリ結果は、下記のようになります。

{% asset_img calendar_sample.PNG %}

これで特定期間のカレンダー生成が可能となりました。

以上です。
