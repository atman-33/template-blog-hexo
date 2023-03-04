---
title: 【VBS】Oracle DBからSELECTした結果をCSVファイルに保存
date: 2023-02-25 15:50:59
tags:
- VBS
- Oracle
- DB
categories: VBS
---

データベース（DB）に格納されたデータを別アプリケーションで扱いたい時はないでしょうか？

そのような場合、ひとまずDBデータをCSVファイルに抽出し、そのCSVファイルを別アプリケーションから読み込む方法があります。

{% asset_img db_to_csv.PNG %}

今回は、VBSを使用してOracle DBから抽出したデータをCSVファイルに保存するプログラムについて説明します。

ソースコードはこちらからダウンロード可能です。
↓
[template-vbs/ExportCsvOracle/](https://github.com/atman-33/template-vbs/tree/master/ExportCsvOracle/ "Github")

___
目次
<!-- toc -->

___

## プログラム動作フロー

DBから抽出したデータをCSVファイルに保存するプログラムですが、大きな流れは下記となります。

① 必要な設定情報の読み込み
　↓
② Oracle DB に接続
　↓
③SELECT文を記載したSQLを実行
　↓
④実行したSQL結果をCSVファイル保存
　↓
⑤Oracle DB の切断

## パッケージ構成

今回のプログラム動作はソースファイル単体ではありません。フォルダとファイルを含めた構成となります
```
ExportCsvOracle/
├csv/
├sql/
├Config.ini
└ExportCsvOracle.vbs
```

|フォルダ/ファイル|説明|
|:--|:--|
|ExportCsvOracle.vbs|実行プログラム|
|Config.ini|DB接続やフォルダパスの設定情報|
|sql（フォルダ）|SELECT文が格納されたファイルを格納|
|csv（フォルダ）|DBから抽出したCSVファイルを格納|

## ソースコード解説

プログラム実行に設定情報はiniファイルから読み込むことにします。
▼Config.ini
```
[source_db]
provider=OraOLEDB.Oracle
data_source=TESTDB
user_id=system
password=sys
[path]
sql_folder=sql
csv_folder=csv
```

今回のiniファイルには下記情報を格納しました。

- DB（Oracle）への接続情報
- SQL文の格納フォルダ名（パス）
- 生成するCSVの格納フォルダ名（パス）

このiniファイルの情報を読み込んで、VBSスクリプト上で扱えるよう変数に格納しておきます。
詳細は別記事にしていますので、参考にして下さい。

[【VBS】iniファイルのデータを取得する](/2023/02/23/3-get-ini-data-vbs/)

## Oracle DBに接続
ここでは、Oracle DB の各操作に必要な関数を用意しておきます。

```
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : DB接続（オラクル）
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Sub OpenDBOracle(ByRef objAdoCon, provider, dataSource, user, pass)

    If DEBUG_MODE = 1 Then
        WScript.Echo "DBに接続します。"
    End If

    Dim constr

    Set objAdoCon = WScript.CreateObject("ADODB.Connection")

    constr = "Provider=" & provider & ";Data Source=" & dataSource _
                & ";User ID=" & user & ";Password=" & pass

    If DEBUG_MODE = 1 Then
        WScript.Echo constr
    End If

    objAdoCon.ConnectionString = constr
    objAdoCon.Open

    If DEBUG_MODE = 1 Then
        WScript.Echo "DBに接続しました。"
    End If

End Sub

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : トランザクション開始
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Sub BeginTrans(ByRef objAdoCon)

    If DEBUG_MODE = 1 Then
        WScript.Echo "トランザクションを開始します。"
    End If
    objAdoCon.BeginTrans

End Sub

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : コミット
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Sub CommitTrans(ByRef objAdoCon)

    If DEBUG_MODE = 1 Then
        WScript.Echo "コミットします。"
    End If
    objAdoCon.CommitTrans

End Sub

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : ロールバック
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Sub RollbackTrans(ByRef objAdoCon)

    If DEBUG_MODE = 1 Then
        WScript.Echo "ロールバックします。"
    End If
    objAdoCon.RollbackTrans

End Sub

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : DB切断
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Sub CloseDB(ByRef objAdoCon)

    If DEBUG_MODE = 1 Then
        WScript.Echo "DBを切断します。"
    End If
    objAdoCon.Close

End Sub
```

今回は、上記のDB接続およびDB切断のみ使用します。

先ほどのiniファイルから読み込んだ変数を使えば、下記のようなDB処理が可能となります。

```
Dim objAdoCon       ' ADO接続

' 1. DB接続
OpenDBOracle objAdoCon, SDB_PROVIDER, SDB_DATA_SOURCE, SDB_USER, SDB_PASS

・
・
・

' 4. DB切断
CloseDB objAdoCon
Set objAdoCon = Nothing
```

## SELECT文を記載したSQLを実行
ここまでで、DB（Oracle）への接続は可能となりました。

次に、DBからデータを抽出するためのSQL実行まで説明します。ここでの動作は下記のようなフローとなります。

SQL格納ファイル群を取得　※SQL１ファイルにつき１SELECT文
　↓
SQL格納ファイルからSQL文の取得
　↓
SQL文を実行してレコードセットを取得

まずは各処理に必要な関数を準備します。

```
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : SQL SELECT文を実行してレコードセットを取得
' note  : 戻り値 -> レコードセット
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Function ExcuteSQLgetRS(objAdoCon, strSQL)

    Dim objAdoRS  ' レコードセット

    ' Msgbox "ExcuteSQLgetRS -> SQL: " & strSQL    

    Set objAdoRS = objAdoCon.Execute(strSQL)

    ' Msgbox objAdoRS(0).value
    ' Msgbox "EOF:" & objAdoRS.EOF

    Set ExcuteSQLgetRS = objAdoRS   ' Object のため Set を忘れないこと

End Function

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : フォルダ内の各ファイル名を取得して配列で戻す。
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Function GetFileNames(folderName)

    Dim objFileSys, objFolder, tmpFile, objFile

    ' ファイルシステムを扱うオブジェクトを作成
    Set objFileSys = CreateObject("Scripting.FileSystemObject")

    ' 引数 crrDir のフォルダのオブジェクトを取得
    Set objFolder = objFileSys.GetFolder(folderName)

    ' ファイルが無い場合
    IF objFolder.Files.Count = 0 then
        GetFileNames = -1
        Exit Function
    End IF

    ' FolderオブジェクトのFilesプロパティからFileオブジェクトを取得
    tmpFile = ""
    For Each objFile In objFolder.Files

        ' 取得したファイルのファイル名格納
        IF tmpFile = "" then
            tmpFile = folderName & "\" & objFile.Name
        Else
            tmpFile = tmpFile & "|" & folderName & "\" & objFile.Name
        End IF
    Next

    GetFileNames = split(tmpFile, "|")

    Set objFolder = Nothing
    Set objFileSys = Nothing

End Function

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : ファイル内のテキストを全取得
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Function GetFileText(fileName)

    On Error Resume Next

    Dim objFSO      ' FileSystemObject
    Dim objFile     ' ファイル読み込み用

    GetFileText = ""    
    ' Msgbox "GetFileText -> fileName: " & fileName
    Set objFSO = WScript.CreateObject("Scripting.FileSystemObject")
    If Err.Number = 0 Then
        Set objFile = objFSO.OpenTextFile(fileName)
        If Err.Number = 0 Then
            GetFileText = objFile.ReadAll
            WScript.Echo getSQL
            objFile.Close
        Else
            WScript.Echo "ファイルオープンエラー: " & Err.Description
        End If
    Else
        WScript.Echo "エラー: " & Err.Description
    End If

    Set objFile = Nothing
    Set objFSO = Nothing

End Function
```

これらの関数を使用することで、SQL文を実行してレコードセットを取得するところまで進められます。
詳細の説明は省きますが、レコードセットにはSQLのSELECT文で示した抽出結果の情報が格納されています。

各関数を呼び出すコードは、メインプログラム側に追記します。

```
Dim objAdoCon       ' ADO接続
Dim strSQLFiles     ' 実行するSQLのファイル群
Dim strSQLFile      ' 実行するSQLのファイル
Dim strSQL          ' 実行するSQL
Dim objAdoRS        ' ADOレコードセット

' 1. DB接続
OpenDBOracle objAdoCon, SDB_PROVIDER, SDB_DATA_SOURCE, SDB_USER, SDB_PASS

' 2. SQLファイル群の読込
strSQLFiles = GetFileNames(SQL_FOLDER_PATH)

' 3. CSV生成
For Each strSQLFile In strSQLFiles                      ' 各SQLファイルを繰り返し処理
    WScript.Echo strSQLFile
    strSQL = GetFileText(strSQLFile)                    ' SQL文の取得
    ' Msgbox strSQL
    Set objAdoRS = ExcuteSQLgetRS(objAdoCon, strSQL)    ' SQL文を実行してレコードセットを取得
　・
　・
　・
Next
```

## 実行したSQL結果をCSVファイル保存

先ほどまででSQLの結果をレコードセットに格納しました。
しかし、CSVファイルでデータを保存しておくためには、レコードセットをCSV形式のテキストに変換させる必要があります。

ここでの動作は下記のようなフローとなります。

①レコードセットをCSV（テキスト）に変換
　↓
②CSVファイルに変換したテキストを書き込み

```
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : レコードセットをCSVに変換
' note  : 戻り値 -> CSV形式のテキスト
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Function GetCSVTextFromRS(ByRef objAdoRS)

    Dim csvText
    Dim i

    csvText = ""
    Do While objAdoRS.EOF <> True

        'フィールドの数ループ
        For i = 0 to objAdoRS.fields.count -1
            If i <> objAdoRS.fields.count -1 then
                csvText = csvText & objAdoRS(i).value & ", "
            Else
                csvText = csvText & objAdoRS(i).value
            End If
        Next

        'テキスト改行
       csvText = csvText & vbCrLf

       objAdoRS.MoveNext
    Loop

    objAdoRS.Close
    Set objAdoRS = Nothing

    If DEBUG_MODE = 1 Then
        WScript.Echo csvText
    End If
    GetCSVTextFromRS = csvText

End Function

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : 指定したファイルにテキストデータを書き込む。
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Sub WriteFile(fileBaseName, data, extension)

    Dim objFSO      ' FileSystemObject
    Dim objFile     ' ファイル書き込み用
    Dim fileName

    fileName = fileBaseName & "." & extension
    'Msgbox "witeFile.fileName: " & fileName
    'Msgbox "witeFile.data: " & data

    On Error Resume Next
    Set objFSO = WScript.CreateObject("Scripting.FileSystemObject")
    If Err.Number = 0 Then
        Set objFile = objFSO.OpenTextFile(fileName, 2, True)
        If Err.Number = 0 Then
            objFile.Write(data)
            objFile.Close
        Else
            WScript.Echo "ファイルオープンエラー: " & Err.Description
        End If
    Else
        WScript.Echo "エラー: " & Err.Description
    End If

    On Error Goto 0

    Set objFile = Nothing
    Set objFSO = Nothing

End Sub
```

メインプログラム側には、関数を呼び出す処理を追記します。

```
Dim objAdoCon       ' ADO接続
Dim strSQLFiles     ' 実行するSQLのファイル群
Dim strSQLFile      ' 実行するSQLのファイル
Dim strSQL          ' 実行するSQL
Dim objAdoRS        ' ADOレコードセット
Dim csvText         ' SQLでSELECTしたCSVテキスト内容

' 1. DB接続
OpenDBOracle objAdoCon, SDB_PROVIDER, SDB_DATA_SOURCE, SDB_USER, SDB_PASS

' 2. SQLファイル群の読込
strSQLFiles = GetFileNames(SQL_FOLDER_PATH)

' 3. CSV生成
For Each strSQLFile In strSQLFiles                      ' 各SQLファイルを繰り返し処理
    WScript.Echo strSQLFile
    strSQL = GetFileText(strSQLFile)                    ' SQL文の取得
    ' Msgbox strSQL
    Set objAdoRS = ExcuteSQLgetRS(objAdoCon, strSQL)    ' SQL文を実行してレコードセットを取得
    csvText = GetCSVTextFromRS(objAdoRS)                ' レコードセットをCSV形式のテキストに変換

    WriteFile CSV_FOLDER_PATH & "\" & GetBaseName(strSQLFile), csvText, "csv"   ' CSVファイル生成
Next
```

これでCSVファイルの作成まで進みました。

## Oracle DB の切断

最後にOracle DBとの切断処理を追記して完了となります。（ 関数は 、先ほど説明したOracle DB 接続の部分で記載）

メインプログラム側のソースコード最終版は、下記となります。

```
Option Explicit

Const DEBUG_MODE = 1                ' 1:デバッグモード, 0:通常モード
Const INI_FILE = "Config.ini"     ' iniファイル名

Dim SQL_FOLDER_PATH, CSV_FOLDER_PATH
Dim SDB_PROVIDER, SDB_DATA_SOURCE, SDB_USER, SDB_PASS

Dim ini
ini = GetCurrentDirectory() & "\" & INI_FILE

' #### #### #### #### #### #### #### #### #### #### #### #### #### #### ####
' 0. iniファイルの読み込み　※実行VBSファイルと同じカレントフォルダに保存
SDB_PROVIDER = GetIniData(ini, "source_db", "provider")
SDB_DATA_SOURCE = GetIniData(ini, "source_db", "data_source")
SDB_USER = GetIniData(ini, "source_db", "user_id")
SDB_PASS = GetIniData(ini, "source_db", "password")

SQL_FOLDER_PATH = GetCurrentDirectory() & "\" & GetIniData(ini, "path", "sql_folder")
CSV_FOLDER_PATH = GetCurrentDirectory() & "\" & GetIniData(ini, "path", "csv_folder")
' #### #### #### #### #### #### #### #### #### #### #### #### #### #### ####

Dim objAdoCon       ' ADO接続
Dim strSQLFiles     ' 実行するSQLのファイル群
Dim strSQLFile      ' 実行するSQLのファイル
Dim strSQL          ' 実行するSQL
Dim objAdoRS        ' ADOレコードセット
Dim csvText         ' SQLでSELECTしたCSVテキスト内容

' 1. DB接続
OpenDBOracle objAdoCon, SDB_PROVIDER, SDB_DATA_SOURCE, SDB_USER, SDB_PASS

' 2. SQLファイル群の読込
strSQLFiles = GetFileNames(SQL_FOLDER_PATH)

' 3. CSV生成
For Each strSQLFile In strSQLFiles                      ' 各SQLファイルを繰り返し処理
    WScript.Echo strSQLFile
    strSQL = GetFileText(strSQLFile)                    ' SQL文の取得
    ' Msgbox strSQL
    Set objAdoRS = ExcuteSQLgetRS(objAdoCon, strSQL)    ' SQL文を実行してレコードセットを取得
    csvText = GetCSVTextFromRS(objAdoRS)                ' レコードセットをCSV形式のテキストに変換

    WriteFile CSV_FOLDER_PATH & "\" & GetBaseName(strSQLFile), csvText, "csv"   ' CSVファイル生成
Next

' 4. DB切断
CloseDB objAdoCon
Set objAdoCon = Nothing

If DEBUG_MODE = 1 Then
    WScript.Echo "処理が完了しました。"
End If
```

以上となります。

今回のサンプルをベースにすれば、DBのあるテーブルから1時間置きに最新ログを抽出することなど容易に実施できます。

繰り返しの自動実行には Windows であればタスクスケジューラを利用してください。
