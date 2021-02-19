---
layout:     post                       # 使用的布局（不需要改）
title:      "在 M1 电脑上安装命令行" # #重回 Layout                  # 标题
subtitle:   brew, Cocoapods...            #副标题
date:       2020-12-31                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 总结
---

### 简介

新推出的 M1 电脑相当惊喜：

`M1 mbp 13 8g/256G` vs `mbp 16-inch 2019 16G/500G`

- M1 的 build 速度要快比16寸 mac 快 1/3 到 1/4。
- 在大约 10h 轻度 run 项目，连接手机 build，调试 project ，14h 待机情况下，电脑还有 35% 电量。

那么除了这些惊喜之外，对于 command line 和 在 M1 运行iOS app 的具体表现怎么样，那就看看吧：

- brew
- cocoapods

本文主要介绍 command line，对 项目运行的可以直接通过下边连接直达。

[在 M1 电脑上运行，调试项目和打包ipa](https://poos.github.io/2019/11/03/FeatureControl/)

### Command line 安装和运行

通过兼容模式即可运行， 通过在命令行前添加 `arch -x86_64` 即可使应用在兼容模式运行。

例如：
```
$: arch -x86_64 brew  
```

### Cocoapods

有些命令行不兼容 arm，那么可以使用**Rosetta安装**，使用兼容模式安装大概的步骤：

1. Duplicate the Terminal application in the Utilities folder.
2. Right click on the app and choose Get Info.
3. Rename the other version of the app as you like.
4. Check the option "open with Rosetta".
5. Install Cocoapods with the command "sudo gem install cocoapods"
6. Type the command line "sudo gem install ffi" to fix the ffi bundle problem. Now you can do a "pod install" without problem.


[Error on M1 Mac](https://github.com/CocoaPods/CocoaPods/issues/10287)

### Homebrew

**安装 arm 版本**

Homebrew 已经支持了两种方式，安装 arm 版的 或者 安装兼容版的。

[Homebrew 2.6.0](https://brew.sh/2020/12/01/homebrew-2.6.0/)

[github/issue/Handle macOS Homebrew on ARM](https://github.com/Homebrew/brew/pull/9117) 目前推荐的方式还是使用 Rosetta 2 安装。


1. 下载安装

arm 版本的 Homebrew 命令行是安装在 `/opt/homebrew`, 而兼容版是在 `/usr/local/Homebrew`。

```
cd /opt
mkdir homebrew && curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
```

使用上面的命令即可下载安装。

> 但是安装完不能使用，这是因为没有配置到 zsh 的配置中。

2. 设置配置文件

M1 电脑命令行貌似默认安装了 zsh，所以需要在 `.zshrc` 修改环境变量。

```
vi ~/.zshrc
path=('/opt/homebrew/bin' $path)
export PATH
```

### 目前可以用的命令行

要想知道命令行是否已经支持 M1，这里有个直达连接：

[macOS 11 Big Sur compatibility on Apple Silicon](https://github.com/Homebrew/brew/issues/7857)

> 大眼看来，大概有 60% 已经支持了。

### 总结

本文基本上是安装命令行的干货了，按照命令行即可安装运行。

另外别忘了[在 M1 电脑上运行，调试项目和打包ipa](https://poos.github.io/2019/11/03/FeatureControl/)。

觉得好的话，不妨去 github 点个赞。哈哈哈，谢啦。
