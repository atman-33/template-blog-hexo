---
title: 【C#】WPF Prism 画面遷移メモリ開放
date: 2023-03-05 14:34:57
tags:
- C#
- WPF
- Prism
categories: C#
---

WPF Prism では、Regionという機能で画面遷移します。

ナビゲーションの画面遷移（画面の一部であるRegion部分が切り替わる処理）では、遷移先に INavigaitionAware インターフェースを実装します。

その際に生成される <u>IsNavigationTarget メソッド</u>は、ViewModelインスタンスを使い回すかどうかを 戻り値の bool で判断しますが、**return false としてもメモリは開放されません**。

</br>

<u>【IsNavigationTargetの戻り値】</u>
- true : インスタンスを使い回す。次回の画面起動時にコンストラクタが呼ばれない。
- false: インスタンスを使い回さない。次回の画面起動時にコンストラクタが呼ばれる。ただし、**メモリは開放されない**。

## ナビゲーション画面遷移時にメモリを開放する方法

ViewModelに**IRegionMemberLifetimeインターフェースを実装**し、**KeepAliveプロパティの値をfalse**にします。

また、**INavigationAwareインターフェースも合わせて実装**している場合、**IsNavigationTarget は True**にしておきます。

ViewModelへの実装例です。

▼SampleViewModel.cs
```
public class SampleViewModel : BindableBase, INavigationAware, IRegionMemberLifetime
{
    /// <summary>
    /// ViewModel破棄に伴いメモリ開放する際はfalse
    /// </summary>
    public bool KeepAlive { get; set; } = false;

    public SampleViewModel()
    {
    }

    public bool IsNavigationTarget(NavigationContext navigationContext)
    {
        //// RegionMemberLifetime(KeepAlive = false)でViewModelを破棄するため、こちらはTrue
        return true;
    }

    public virtual void OnNavigatedFrom(NavigationContext navigationContext)
    {
    }

    public virtual void OnNavigatedTo(NavigationContext navigationContext)
    {
    }
}
```

以上です。