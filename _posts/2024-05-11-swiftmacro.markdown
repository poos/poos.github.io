---
layout:     post
title:      Swift Macro 实践，封装 OSLog
subtitle:   Xcode15后，debug控制台提供了非常有用的日志输入，其中包括了可直接定位到代码行的功能。本文的实践让我们自己的日志库可以实现类似的功能。
date:       2024-05-11
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Swift Macro
- Code
---

> 仓库已开源:[NoLogMacro](https://github.com/poos/NoLogMacro)。

# Swift Macro

Swift Macro 是 2023 WWDC 推出的。对于Swift Macro的介绍文章不少，但是大部分都是官方视频的翻译，例子和一些问题介绍处理真的是比较少。本文还真的有可能是第一个实践应用的...

## 特点

简单了解 WWDC https://developer.apple.com/videos/play/wwdc2023/10166/ 后，应该能得出一下结论：

Swfit Macro 大概分两种，表达式宏（#），和扩展宏（@）。这么一想限制还真多，但是保证了swift macro的安全性，就是说它支持类型判断，泛型等。

# Swift Macro 和 OSLog 的 碰撞


Xcode15后，debug控制台提供了非常有用的日志输入，其中包括了可直接定位到代码行的功能。

初见便想到，如果我们自己的日志库能够实现相应的功能，那调试bug不是手到擒来。

要支持的功能很简单：
- 支持日志，转发到自己的日志库处理
- 支持代码行定位

咱就是说，搞个统一的消息转发，然后在使用宏进行OSLog处理，把两个表达式用宏定义，如果验证代码行能正确定位即可。

预验证确实能正确定位到代码行，那么可以开干了。

## 火花1，仅支持单行表达式

根据WWDC的文档，搞起：
- 项目创建
- 宏定义
- 测试success
- duang，运行起来报错了

![img](https://poos.github.io/img/post-swift-macro.png)

这个报错没有有用信息，忽然想到表达式可以用于初始化变量，是不是有可能不支持多个表达式，通过修改宏，始终不能正确编译。

既然**Swift 表达式宏不支持多个表达式**，那么就用单行表达式吧。

# 火花2，不支持闭包

通过使用闭包，这样就能执行方法，然后再闭包里面执行log收归或者调用系统log即可。

方式1:
```
Logger()
.sync {
    NoLog.noLog("msg", attrs: ["a": 1])
}
.log("msg")
```
可以正确定位到代码行，**但是，NoLog消息中心收到的消息始终是第一个**。
方式2:

```
NoLog
.sync {
    Logger().log("msg")
}
.noLog("msg", attrs: ["a": 1])
```
**不能正确定位到代码行**，NoLog消息中心收到的正确。

这下闭包的路堵死了。猜测应该是宏展开，block被视为常量，最终展开时候相当于一个全局的block。

## 链式语法
条条大路，那么试试链式语法看看：
```
Logger()
.noLog("msg", attrs: ["a": 1])
.log("msg")
```
还是这个方法ok，既能够显示到代码行，也能到消息中心打出日志。剩下的就剩下封装宏了。按照文档一步步来即可。


那最后，简单看下 OC define 和 Swfit Macro 的区别：

·|OC define|Swfit Macro
---|---|---
代码编辑器类型检测|N|Y
定义单个表达式|Y|Y
定义多个表达式|Y|N
定义各种结构的扩展|Y|Y

也就是说，Swift Macro 在保证类型安全的情况下，支持单个表达式和数据结构的宏。

# 开源库

**[Not Only Log](https://github.com/poos/NoLogMacro)**，（灵感来自nosql），比较贴切的有木有。

**先看效果，常规方法的log无法定位到正确的代码行。通过本库提供的宏方法可以完美的以宏的方式支持，非常轻量级。**

![img](https://poos.github.io/img/post-swift-macro2.gif)


## 使用
库已开源。使用相当简单，但是考虑到很多小伙伴没使用过 swift package，特别录制了一些步骤：

- 右键添加 Package，通过链接 `https://github.com/poos/NoLogMacro` 添加，同时选择 Lib 和 Client
- `import OSLog` 和 `import NoLogMacro`，初始化 `NoLogger.callback`（可选）
- 使用`#noLog()` 代替 `Logger().log()`

gif:

![img](https://poos.github.io/img/post-swift-macro3.gif)

### Code

具体代码，一共就几行，起飞:

```
//import
import OSLog
import NoLogMacro

// set once
NoLogger.callback = { (type: OSLogType, message: String, attrs: Dictionary<String, Any>?) in
    print("simple type: \(type) message: \(message) dic: \(String(describing: attrs))")
}


// all log can send to `NoLogger.callback`
// **can be located to a line**

// same with `Logger().log(level: .default, "default")`
#noLog("message")
// and more custom info
#noLogError("error", attrs: ["code": 404])

// others
Logger().log(level: .info, "info")
#noLogInfo("info")
#noLogInfo("info", attrs: ["a": 2])

Logger().log(level: .debug, "debug")
#noLogDebug("debug")
#noLogDebug("debug", attrs: ["a": 3])

Logger().log(level: .error, "error")
#noLogError("error")
#noLogError("error", attrs: ["a": 4])

Logger().log(level: .fault, "fault")
#noLogFault("fault")
#noLogFault("fault", attrs: ["a": 5])
```

## TODO List

当然还有几个TODO项，just waiting 一下子。

- support `Logger(subsystem: <#T##String#>, category: <#T##String#>)`
- brew iOS 14, using print support
- using Swift Macro writing this lib
