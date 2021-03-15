---
layout:     post
title:      Flutter 嵌入原生的一些坑
subtitle:   运行错误或者白屏
date:       2019-05-30
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Flutter
- 总结
---

## tips

关于 Flutter 的总结。

[所有 Flutter 的资料](https://poos.github.io/tags/#Flutter)。

- 介绍和技术分享会
- dart 语法简介
- widget 和 layout
- 设计模式
- 嵌入原生项目

## 坑点

一些开发遇到的坑

### flutter channle

当需要切换分支时候你可以使用 flutter channle 命令。

我在前段时间做 flutter 嵌入 iOS 原生项目时候，发生了 image asset 没有找到的情况。Issue 有人提出切换 master，当然现在已经修复了

### use in China

不多说，可能需要修改镜像地址。尤其是 flutter channle 更换之后往往需要重新下载。

基本设置如下即可。

```shell
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

如不能解决，[flutter-io.cn/](https://flutter-io.cn/) 最底部也明显提出解决办法。

**如果 flutter doctor 遇到安卓协议 卡了很长时间，那么换个网试一下。不行了，按照 flutter 文档，重新确认一下。**

### 原生项目添加 flutter

安装官方文档添加即可：[flutter/wiki/Add-Flutter-to-existing-apps](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps)。

**但是注意如下几点：**


### 1. flutter_boost

如果你使用了三方的 页面管理方案，例如 flutter_boost。有可能不需要上述的官方文档中提到的注册一步。即省略如下相关：

```swift
    //...
    GeneratedPluginRegistrant.register(with: self.flutterEngine);
```

### 2. VERBOSE-2:engine.cc......

- 使用了非官方推荐的方式导入，需要按照你使用的例子，添加相应的 app.framework 和 flutter.framework。

- 或者你使用的是官方导入方式，升级 flutter 遇到的问题，那么升级到最新 stable channel，重新生成一个 module 再试试。

### 3. Failed to find assets path for "Frameworks/App.framework/flutter_assets"

这个 bug 已经在5月初修复，现在不需要添加 framework 了。

所以 **升级到最新 stable channel，重新生成一个 module** 再试试。

### 4. 模拟器好的，真机错误

很莫名其妙，哪里出错了。模拟器好的，真机错误，这种情况可以往证书方面考虑。

**MissingPluginException while adding to existing iOS application**

```
...
iPhone Developer: xxx (MQF8D9PK85): ambiguous (matches "iPhone Developer: xxx (MQF8D9PK85)
iPhone Developer: xxx (MQF8D9PK85): ambiguous (matches "iPhone Developer: xxx (MQF8D9PK85)
MissingPluginException while adding to existing iOS application 
```

这是因为在 keychain 中有多个开发者证书，所以产生了冲突。

**虽然在原生项目中，Xcode 会帮你处理，使用正确的证书。但是，Flutter 的脚本还没有处理这个方面的问题。** 为了保持一个干净的证书环境，还是删除没用的证书吧。

### 5. 热启动/热加载 与 Dart代码调试

命令如下
```shell
cd xx/flutter_module 
flutter attach
```
然后重新导入即可连接设备，热更新运行。

发生错误：
Oops; flutter has exited unexpectedly.

[flutter attach - Oops; flutter has exited unexpectedly.](https://github.com/flutter/flutter/issues/33035)

安照教程更新即可解决.

如果需要更新，可以查看各个 channel 的最新版本。
[flutter/sdk/releases](https://flutter.dev/docs/development/tools/sdk/releases?tab=macos)



### 6. 嵌入 android

尚在探索中

### 其他坑

尚在探索中

