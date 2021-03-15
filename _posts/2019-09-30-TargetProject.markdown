---
layout:     post
title:      多 Target 工程
subtitle:   配置，资源（图片，字符串），Code，Build
date:       2019-09-30
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- 总结
---

### 背景/简介

Blogger 有幸进入某外企，也接触到一些新的工程使用方式，不妨记录和分享一下，让大家也多提提意见～

本文就主要是多 Target 的工程方式，那么在一个 Code Base 的基础上如何万花齐放呢？可以先预测一下要注意的点：

- 配置：证书配置，target 配置等
- Code/Stroyboard/图片/资源：需要勾选 ☑️ 多个target，或者使用单独的target，放入单独的 folder
- Build：Xcode run，Command Line build export 等


> 本文讨论更多的是基于同一个 Project， 创建不同 Target 打包出相应的 app。



### init Target

这部分也比较简单，Duplicate 一个 Target，然后 修改一下 Scheme 就可以了。

> 这一步就可以复制一个 App 了，然后就是相应的 config 修改。

> 也可以新建一些 Target，例如 Framework，Test，Extension，Watch等。当然我们的主角是 iOS App～

#### 准备开发环境

包含一些证书和环境的配置：

- id，环境（推送等）
- 开发证书
- 打包脚本

### Code/Stroyboard/图片/资源 管理

这部分主要是通过是否勾选 target 引用来区分。相同的类名在不同的 target 勾选使用即可～

到最后，文件夹可能会复杂的像这样：
```
Project
├── Base
│   ├── Assets
│   │   ├── icon
│   │   └── ...
│   ├── Config
│   │   ├── info
│   │   └── ...
│   ├── Storyboard
│   │   ├── main
│   │   └── ...
│   └── Strings
│       ├── ...
│       └── ...
├── Target1
│   ├── Assets

```

正常情况下，Base 里面是勾选了所有 target 的，然后下边是针对单个 targer 的。

#### UI 相同，字符串不同

通常多个 target 的页面可能相差不多，但是字符串是相差很大的。这时候我们可以利用独立的 String file 来存储字符串。就是上面讲到的区分 Resource。


**有一些自动化的脚本可以帮助更加快速便捷的创建**

[documentation/Cocoa/Conceptual/LoadingResources/Strings](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/LoadingResources/Strings/Strings.html)，除了基本的介绍，文档还介绍了自动生成 String file 的小工具，还是挺有趣的～

#### APP 之间 UI 差别比较大

UI 差别大那就放弃，不要使用同一个 VC 了，分两个 VC， 分别勾选不同的 Target 即可，还是美滋滋的～

但是 VM 一般相似。可以复用 VM。

一些情况下可能针对某个项目有特殊的 tips。这时候就需要添加 flag 。**Flag 好的命名是不关心 APP， 只关心feature。例如 `haveATips = True`**

### Build

除了使用 Xcode 打包上传之外，

这一部分主要是打包脚本和一些信息了～

> 先 TODO 一下，后边准备一个 脚本专题，完成之后会在这里 link 一下。

### 总结

复制一个 app 还是很简单的。主要还是后续的 code 和最佳实践了，如果有新的有趣的点还是得补充一下～
