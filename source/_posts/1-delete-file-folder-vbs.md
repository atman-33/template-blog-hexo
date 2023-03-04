---
title: 【VBS】指定したファイル、フォルダを削除する
date: 2023-02-23 20:14:32
tags: VBS
categories: VBS
---

VBSでファイル、フォルダを削除する関数のサンプルソースです。

___
目次
<!-- toc -->

___

### サンプルソース

```
Option Explicit

DeleteFolder "C:\sample"
DeleteFile "C:\test\test.txt"

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' @brief : 指定したファイルを削除する。
' @note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Sub DeleteFile (ByVal strFile)

 Dim objFso
 Set objFso = CreateObject("Scripting.FileSystemObject")

 ' フォルダを削除
 objFso.DeleteFile strFile,True ' 注意：戻り値が無い場合は引数を（）で括らないこと

 Set objFso = Nothing

End Sub

' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
' @brief : 指定したフォルダを削除する。
' @note  :
' ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ---- ----
Sub DeleteFolder (ByVal strFolder)

 Dim objFso
 Set objFso = CreateObject("Scripting.FileSystemObject")

 ' フォルダを削除
 objFso.DeleteFolder strFolder,True ' 注意：戻り値が無い場合は引数を（）で括らないこと

 Set objFso = Nothing

End Sub
```

### 解説

FileSystemObject.DeleteFile関数
FileSystemObject.DeleteFolder関数
第1パラメータ：削除するフォルダ
第2パラメータ：
　True：読み取り専用ファイルも削除
　False(規定値)：読み取り専用ファイルは削除しない
