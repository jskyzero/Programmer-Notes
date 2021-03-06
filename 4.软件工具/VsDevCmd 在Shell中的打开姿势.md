---
layout: post
title: "效率提升：VsDevCmd快捷启动"
header-img: "img/site/bg1.jpg"
tags:
    - Tools
    - Microsoft
---

想在PowerShell中直接用VsDevCmd还是需要点折腾的，本文将以使用VsDevCmd为例带读者初步学习编写.bat

<!--more-->

# VsDevCmd 在Shell中的打开姿势
`jskyzero` `2017/05/31`


看完本文，您将可以在PowerShell自定义运行Visual Studio 2017 的VsDevCmd.bat，在命令行编译/调试的时候还是挺方便的。


## 简介VsDevCmd

我是在想在PowerShell中直接运行csc编译.cs文件的时候引起的这个需求，嗯我们可以在开始菜单的Visual Studio 2017文件夹中找到Developer Command Prompt for VS 2017，这个东西就提供给我们的工具了，不过直接运行系统的话会有三个问题，其一是运行方法是鼠标点击，这显然不够效率，其二是自带的运行的话是在CMD里面的，讲波道理ls指令都没有的话还是用不太顺手，其三是运行时候的目录会变成源文件的根目录里，需要我们手动再导航，这就比较尴尬。


## 初步尝试
我们可以定位下开始菜单里面的快捷方式，来看看这玩意具体是啥。
在快捷方式的目标里面就写的是 `%comspec% /k "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\VsDevCmd.bat"`，欸这样我们就找到了源文件VsDevCmd.bat的地址，那么初步来看我们有两种解决方法。
### 直接添加源文件所在路径到Path 
这个想法简单粗暴，实际上也比较实际，基本等同与直接运行.bat文件，大概等效与运行如下.PS1文件：
`cd "C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\"
.\VsDevCmd.bat`

除了暂时没找到回到上一次目录的方法，其他都还挺好的，查到的一些除了像这样写function的
```
function cd { if ($args[0] -eq '-') { $pwd=$OLDPWD; } else { $pwd=$args[0]; } $tmp=pwd; if ($pwd) { Set-Location $pwd; } Set-Variable -Name OLDPWD -Value $tmp -Scope global; }
Get-Item function:cd
```
就是这样直接回去的
```
cd $OLDPWD
```

不过在我这里用起来就会报关于path的错误。

### 修改快捷方式指令加入PowerShell

这个的想法就是我们修改原先快捷方式的指令，来达到使用PowerShell的目的，比如来说修改成`%comspec% /k ""C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\VsDevCmd.bat"  & powershell"`，嗯这个也是最好能回到原先目录就好了。

### 新建.bat文件

因为是自己新建，当然可以建到已经Path有的路径，第一个版本是模仿快捷方式写的，如下：`%comspec% /k ""C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\VsDevCmd.bat" & powershell "`然后我们加上返回上一层就好了：`%comspec% /k ""C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\Common7\Tools\VsDevCmd.bat" & cd %CD%  & powershell "`完美解决这个问题了。

不过如果是完全自己新建的话，记得把当前路径加入到Path中。

## 参考
+ [What does “&&” in this batch file?](https://stackoverflow.com/questions/28889954/what-does-in-this-batch-file)

+ [Using Visual Studio Developer Command Prompt With PowerShell](https://www.gurustop.net/blog/2014/02/01/using-visual-studio-developer-command-prompt-with-powershell/)

+ [Managing Current Location](https://msdn.microsoft.com/en-us/powershell/scripting/getting-started/cookbooks/managing-current-location)

