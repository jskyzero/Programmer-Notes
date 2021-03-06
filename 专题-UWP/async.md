---
layout: post
title: UWP：多线程问题及其解决
header-img: "img/posts/uwp.jpg"
tags:
    - UWP
    - C#
thumbnail: "/img/thumb/uwp.jpg"
---

本文将讲述UWP应用中多线程的具体问题，将结合具体界面阻塞/后台线程访问UI线程问题同时给出具体解决方案。

<!--more-->

# UWP内的多线程

## 从第一次接触角度开始

可能最开始接触UWP的异步，是在使用C#的某些函数时候看到提示让我们添加await，然后又会提示我们修改函数为添加修饰async，从字面意思来讲，如果一个函数需要较长的执行时间，为了不阻塞应用的继续执行，一般是还是要提供UI的反馈的，那么我们就不能让程序卡在这里，这时候如果这个函数是async型的，程序就会另起一个线程来执行这个函数，如果这个函数需要返回数据，那么就用await来等待返回的值。


### UI的阻塞

可能有同学不太能理解什么是UI的阻塞，举例来说。

我们随意给界面上加个东西
```xml
<Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
    <TextBlock x:Name="output" Text="计算中" Style="{StaticResource HeaderTextBlockStyle}"></TextBlock>
</Grid>
```
然后把这个虽然没什么意义但是很耗时间的东西放在页面的初始化里
```csharp
int ans = 0;
for (var i = 0; i < Int32.MaxValue; i++)
{
    ans++;
}
output.Text = ans.ToString();
```
这样页面是需要很长时间才能加载完成的，我们是看不到“计算中”这样的输出的，这个例子还好，计算的时间可能会停留在加载界面，但是如果是界面上还有其他按钮，此时UI线程阻塞了，我们对其他按钮的事件的触发就会没有反馈。这自然不是我们想要的结果。

## 初步的尝试
那上面都说了有异步，那我把这个计算的环节写成异步函数不就好了吗？
```csharp
async void CalAns()
{

    int ans = 0;
    for (var i = 0; i < Int32.MaxValue; i++)
    {
        ans++;
    }

    output.Text = ans.ToString();
}
```
道理是这个道理，这样我们是能看到“计算中”的，但是很快计算结果出来的时候就会出错，报错的提示大概是使用了另一个线程的接口。


第一次遇见这个现象的时候我是很迷茫，现在看来也就是后台线程是不能直接访问UI线程的，进程之间的通讯是需要使用给定的接口的，如果我们非要访问，那可以使用Dispatcher，比如
```csharp
async void CalAns()
{

    int ans = 0;
    for (var i = 0; i < Int32.MaxValue; i++)
    {
        ans++;
    }

    await this.Dispatcher.RunAsync(Windows.UI.Core.CoreDispatcherPriority.Normal, () => {
        output.Text = ans.ToString();
    });
}
```
嗯如果你没有上面的this怎么办？嗯这是个问题，可以考虑开个静态属性存下page，或者使用MVVM Light，MVVM Light也有给的解决方法。

## 如果我把值给返回回来
那你可以这么写
```csharp
async void CalAnsAsync()
{
    Func<int> CalAnsFunc = () =>
    {
        int ans = 0;
        for (var i = 0; i < Int32.MaxValue; i++)
        {
            ans++;
        }
        return ans;
    };

    output.Text = (await Task.Run(CalAnsFunc)).ToString();
}
```
用Task.Run()开新线程跑一个本来不是异步的函数，然后等待跑完把值返回来更新值。

## 参考
> 上面的大概只能起参考左右，如果有兴趣可以看下面的这些博文。

+ [Windows 10 UWP开发：如何不让界面卡死](http://edi.wang/post/2016/2/18/windows-10-uwp-async-await-ui-thread)

+ [在MVVM中的多线程问题](http://www.jianshu.com/p/8a44075e66f8)