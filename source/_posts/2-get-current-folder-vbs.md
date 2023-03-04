---
title: 【VBS】カレントフォルダを取得する
date: 2023-02-23 21:22:49
tags: VBS
categories: VBS
---

VBSでカレントフォルダのパスを取得する関数のサンプルソースです。

___
目次
<!-- toc -->

___

### サンプルソース

```
Option Explicit

Msgbox GetCurrentDirectory()

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
    GetCurrentDirectory = objWshShell.CurrentDirectory

End Function
```

### 解説

VBSファイル（実行ファイル）が存在するカレントフォルダを取得する。
※カレントフォルダ＝カレントディレクトリとも呼ばれる。
