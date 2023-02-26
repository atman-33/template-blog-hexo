---
title: 【C#】WPF Prism 画面遷移（ナビゲート）
date: 2023-02-26 21:35:19
tags:
- C#
- WPF
- Prism
categories: C#
---

C#の **WPF Prism** でナビゲート画面の遷移方法について説明します。

下記のように、ボタンを押すと画面が切り替わる処理です。
（ダイアログではなく、画面内の表示が切り替わります。）

{% asset_img 1.gif %}

___
## 対象ファイル

コーディングが必要なファイルは下記の **A～C** です。

```
Views/
 |-MainWindowView.xaml（画面遷移元）　･･･A
 |-SampleNavigationView.xaml(画面遷移先)

ViewModels/
 |-MainWindowViewModel.cs（画面遷移元）　･･･B
 |-SampleNavigationViewModel.cs(画面遷移先)

App.xaml.cs　･･･C
```

___
## MainWindowView.xaml（画面遷移元）　･･･A

___
### 1. ボタンにCommandを追加

画面遷移元のViewのボタンに対して、CommandにBindingでデリゲートコマンド名称を記載します。

```
<Button Content="Sampleナビゲーション画面"
        FontSize="14"
        Margin="10"
        Padding="5"
        Command="{Binding SampleNavigationViewButton}"/>
```

___
## MainWindowViewModel.cs（画面遷移元）　･･･B
___
### 2. ViewModelにIRegionManagerの変数を追加

画面遷移元のViewModelに、IRegionManagerのプライベート変数を追加し、コンストラクタでセットします。

### 3. ボタン押下時の実行メソッドを追加

ボタン押下イベントを受け取るデリゲートコマンドのプロパティを追加し、ボタン押下時のExcuteメソッドを実装します。

上記2～3のコード例は下記となります。

```
private IRegionManager _regionManager;  //// 画面遷移（ナビゲーション）

//// コンストラクタ
public MainWindowViewModel(IRegionManager regionManager)
{
    //// 画面遷移用（ナビゲーション）
    _regionManager = regionManager;

    SampleNavigationViewButton = new DelegateCommand(SampleNavigationViewButtonExecute);
}

public DelegateCommand SampleNavigationViewButton { get; }

private void SampleNavigationViewButtonExecute()
{
   //// 画面遷移処理（ナビゲーション）
   _regionManager.RequestNavigate("ContentRegion", nameof(SampleNavigationView));
}
```

___
## App.xaml.cs　･･･C
___
### 4. RegisterTypes内でViewを登録

```
protected override void RegisterTypes(IContainerRegistry containerRegistry)
{
    //// ナビゲーション画面
    containerRegistry.RegisterForNavigation<SampleNavigationView>();
}
```

containerRegistry.RegisterForNavigationに設定したViewのみ画面遷移が可能です。
