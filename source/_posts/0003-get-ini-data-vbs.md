---
title: 【VBS】iniファイルのデータを取得する
date: 2023-02-23 21:46:02
tags: VBS
categories: VBS
---

Settings.ini ファイルに格納されたデータを取得する方法を説明します。
下記のように Settings.ini と GetIniData.vbs を同フォルダに格納して、Settings.ini に含まれたデータを読み取ります。

## フォルダ構造

``` bash
任意のフォルダ
 ├ Settings.ini
 └ GetIniData.vbs
```

## サンプルソース

▼Settings.ini

``` bash
[test1]
data1=あいうえお
data2=かきくけこ
[test2]
data3=abcde
data4=fghij
```

▼GetIniData.vbs

``` bash
Option Explicit

Msgbox GetIniData(GetCurrentDirectory() & "\Settings.ini", "test1", "data1")

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : カレントフォルダを取得する。
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Function GetCurrentDirectory()

    Dim objWshShell     ' WshShell オブジェクト

    Set objWshShell = WScript.CreateObject("WScript.Shell")
    If Err.Number <> 0 Then
        WScript.Echo "エラー: " & Err.Description
        wscript.quit(1)
    End If
    getCurrentDirectory = objWshShell.CurrentDirectory

End Function

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' breif : iniファイル、セクション名、パラメータ名からデータを取得する。
' note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Function GetIniData(strIniFileName, strSection, strKey)

    Dim objFSO, objIniFile, objSectionDic, strReadLine, objKeyDic, arrReadLine
    Dim strTempSection, objTempdic

    ' ファイル入出力の定数
    Const conForReading = 1, conForWriting = 2, conForAppending = 8
    Set objFSO = CreateObject("Scripting.FileSystemObject")

    ' ファイルのOPEN
    Set objIniFile = objFSO.OpenTextFile(strIniFileName, conForReading, False)
    If Err.Number <> 0 Then
        ' エラーメッセージを出力
        wscript.echo "INIファイル名: " & strIniFileName
        wscript.quit(1)
    End If

    ' 格納先Dictionaryオブジェクトの作成
    Set objSectionDic = CreateObject("Scripting.Dictionary")

    ' ファイルのリードREAD
    strReadLine = objIniFile.ReadLine
    Do While objIniFile.AtEndofStream = False
        ' ステートメント開始行を検索
        If (strReadLine <> " ") And (StrComp("[]", (Left(strReadLine, 1) & Right(strReadLine, 1))) = 0) Then
            ' セクション名を取得
            strTempSection = Mid(strReadLine, 2, (Len(strReadLine) - 2))
            ' キー用Dictionaryオブジェクト作成
            Set objKeyDic = CreateObject("Scripting.Dictionary")
            ' ファイルの最終行になるまでLoop
            Do While objIniFile.AtEndofStream = False
                strReadLine = objIniFile.ReadLine
                If (strReadLine <> "") And (StrComp(";", Left(strReadLine, 1)) <> 0) Then
                    ' 次のステートメント開始行が出現したら、Loop終了
                    If StrComp("[]", (Left(strReadLine, 1) & Right(strReadLine, 1))) = 0 Then
                        Exit Do
                    End If
                    ' １セクション内の定義をDictionaryオブジェクトに格納する
                    arrReadLine = Split(strReadLine, "=", 2, vbTextCompare)
                    objKeyDic.Add UCase(arrReadLine(0)), arrReadLine(1)
                End If
            Loop
            ' オブジェクトに格納する
            objSectionDic.Add UCase(strTempSection), objKeyDic
        Else
            strReadLine = objIniFile.ReadLine
        End If
    Loop
    ' ファイルのCLOSE
    objIniFile.Close

    ' ini配列から指定したセクション、パラメータに対応するデータを取得
    strSection = UCase(strSection)  ' 大文字化
    strKey = UCase(strKey)          ' 大文字化

    If objSectionDic.Exists(strSection) Then
        Set objTempdic = objSectionDic.Item(strSection)
        If objTempdic.Exists(strKey) Then
            getIniData = objSectionDic(strSection)(strKey)
            Exit Function
        End If
    End If
    getIniData = ""

End Function

```

## 解説

ini ファイルからデータを取り出す際、Settings.ini の各データを辞書オブジェクトに格納し、そこからセクション（ex.[test1]）とパラメータ名（ex.data1）を引き出します。

上記のサンプルでは、カレントディレクトリの ini ファイルを読み出すようにしています。別フォルダを参照する場合は、ini ファイルの読み込み先を書き換えて使用してみて下さい。
