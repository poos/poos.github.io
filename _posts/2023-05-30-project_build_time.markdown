---
layout:     post
title:      优化build时长
subtitle:   优化过程中碰到的二进制问题，swift framework问题如何解决；一些最佳实践
date:       2023-05-30
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Project
---

最近打包耗时从 500s 优化到了大概 300s。
看下具体如何做到的。

## 先上干货：
如何减少build耗时？几大核心：

- 编译-更多的库打成二进制，含Swift库代码编译
- 依赖治理-简单依赖关系，同步build
- 链接-时长优化
- 单点-runtime，build 向前声明等黑科技




## 如何查看build耗时？

通过 Xcode - Build History - Build - Assistant

### 分析build耗时图

一般~~不可优化~~优化空间小的时间包括：
- prepare
- compute dependency
- create build description
- link

这部分时间是随着项目规模扩大，随之增长的，且单位时间在15s一下的。

那除上述之外，最大的就是build时间了，也是重点优化的目标。

## 优化 build 时间

优化build时间最快的方式就是提前二进制化。

### 库内自治

站在上帝视角，二进制化可能会遇到一些问题：

#### OC 库的预编译宏，枚举，结构体

集团的二方库更新频率高，我们使用了 d.d.d-alpha.d d.d.d-beta.d  d.d.d 的通用语义化规范，但是仍然难以保证正式版本都是稳定的，宏定义不可改变的。一旦发生改变，那么可能出现库版本兼容问题。

- 宏的问题在于，一些预编译宏 DEBUG RELEASE 的处理。

编译的二进制无法同时兼容支持这种处理，例如一个debug log。这种只能通过编译两个二进制库来解决。
所以方向一定是杜绝使用这类宏。
也会在lint进行检测，避免一些预编译类的宏使用。

- 枚举的问题在于，值判断问题

一旦使用方进行等值判断，而没有使用枚举进行使用，呢么就会出现代码逻辑错误。
这些方向是稳定化，即不变；扩展化，即通过default，custom扩展，不直接修改枚举。
也会在lint进行检测，枚举值不可改，可增加。

- 结构体的问题在于，调整结构体的属性顺序可能造成问题

结构体即属性集合，一般申请一段连续内存，属性可以通过内存指针偏移找到。一旦进行二进制编译，使用方通过基址加偏移方式查找解析属性情况下，一旦调整顺序，造成代码错误甚至崩溃。
这个方向是，避免使用；完全可以使用类进行替换。对外 public 避免暴露。
也会在lint进行检测，杜绝使用。


以上问题的处理通常三步走，**避免pubilc**，**规范使用**，**lint检查**。处理这些问题，也不仅仅是为了减少打包时长，对于大家对一些工程规范化，开闭原则的理解也是有提高。也是提高软件的可维护性扩展性。

#### OC - Swift 混编，二进制依赖链条问题

包含 Swift 代码的编译有几个特点：

编译慢：编译一个模块的时候，OC 代码可以单个文件，通过 import 依赖进行并行编译；但是涉及 Swift 代码，所有swift代码需要整体编译，且是在当前模块的 OC 代码编译完成之后。这种情况，对编译时长的影响其实很大的。那么就急需进行二进制化。

整体依赖：不像 OC 可以依赖库内的某个文件，含 Swift 代码的引用，是作为整体引用的，具体是通过库编译之后的 -Swift.h，swiftmodule 整体开放给 OC 和 Swift 代码使用的。

环境兼容：不同版本的 Xcode 对于 Swift 代码的二进制产物是不一致的，通过 modulemap 进行映射。那么就需要进行兼容性处理。

那么为了实现 Swift 库的二进制化，就需要进行 `-Swift.h, swiftmodule` 的保存，在使用 .a 的情况下，进行 这两个文件的正确路径 copy。在使用静态framwork 的情况下，将这两个文件正确放置，写好 podspec。我们使用了 cocoapods 工具， 同时定制了 cocoapods-bin 插件（后话，实现了一些快速发库等操作）。

##### 二进制调试

通过设置 debug 的源码路径，即可在调试时候将二进制断电的汇编地址转换为文件地址，方便进行调试。二进制需要使用 DEBUG 方式构建哦。

LLDB 调试工具，查看 `settings list target.source-map`，会显示支持的设置信息，其中就有 `-fdebug-prefix-map=/path/to/build_dir=` 可用于设置 debug 的查找路径。通过 Other C Flags 设置即可。

other c flags `-fdebug-prefix-map=/path/to/build_dir=`


#### 其他

- xib 的编译比较耗时，一个20k的文件，编译需要3s以上。可以作为单点问题消除
- 一个文件，引用越复杂，耗时越多。合理通过 h 收集（非pch，不符合模块化的规范）减少头文件引入，可以优化编译时间。
- 减少 assert 的复杂读，assert的处理也是在单个库编译阶段处理的，通过最小化资源，对于优化包体也有收益。


除了以上，一些编译选项有助于优化编译时间：

- 将Debug Information Format改为DWARF

在工程对应Target的Build Settings中，找到Debug Information Format这一项，将Debug时的DWARF with dSYM file改为DWARF。

这一项设置的是是否将调试信息加入到可执行文件中，改为DWARF后，如果程序崩溃，将无法输出崩溃位置对应的函数堆栈，但由于Debug模式下可以在XCode中查看调试信息，所以改为DWARF影响并不大。这一项更改完之后，可以大幅提升编译速度。

- 将Build Active Architecture Only改为Yes

在工程对应Target的Build Settings中，找到Build Active Architecture Only这一项，将Debug时的No改为Yes。

这一项设置的是是否仅编译当前架构的版本，如果为No，会编译所有架构的版本。需要注意的是，此选项在Release模式下必须为Yes，否则发布的ipa在部分设备上将不能运行。这一项更改完之后，可以显著提高编译速度。

 

### 简单依赖关系，同步build

以上问题可以解决单个库的编译，但二进制完成后，提高多核 cpu 的使用，增加并发才能有效缩短 build 时间。即增加多个库的同步编译。

那跟大家了解到的高内聚低耦合的一个概念相匹配。对于一个大型工程，依赖结构是非常复杂的，未经约束的工程，100个库左右，爆发式产生了超过3000个依赖关系。单个库的编译也会很容易因为没找到合适的库产生编译问题。

通过架构规划，库依赖治理，Service化。实现基础库的依赖瘦身，业务底层库的Service化，业务库的相互依赖解除，达到一个相对清晰的依赖关系。


### 其他耗时阶段

除了 build 阶段外是同时 build 多个文件，其他的阶段的都是单线程一个个来的。
例如如下：
- prepare
- compute dependency
- create build description
- link

通过以上治理之后，前三个优化空间已经有限，最耗时的阶段来到了 link 阶段。


#### link

- order_file

想到 link，可能有人会想到启动优化的二进制重排。恭喜，确实有联系。利用 ld64 的 -order_file 让 linker 按照指定顺序生成 Mach-O，可以避免启动时候的内存页中断，优化启动速度。

- LTO

Xcode Project 的 LTO 分为3种，`LLVM_LTO = YES_THIN;LLVM_LTO = YES;LLVM_LTO = NO;SWIFT_OPTIMIZATION_LEVEL;`。其中对于编译时长优化最大的是NO，取决于打包工具链的支持。

- 其他

比如 link_map, DEAD_CODE_STRIPPING, exported_symbols_list 等，都会影响 link 时间。

大部分 和 LTO 一样，跟包体大小放一起是一个权衡点；剩下的一些和 order_file 一样，跟启动时长放一起，是个权衡点。不过大部分情况我们不需要这么纠结，开发的配置可以基于编译时长最优。

- 使用其他工具链

比如爱加密工具链，完全接管了 build 底层，这样就有机会做更低层次的优化，例如复用产物等。

对链接器感兴趣可以看一下字节这篇文章，[深入 iOS 静态链接器（一）— ld64 ](ttps://mp.weixin.qq.com/s/tSj6JVEg7plJQm7aDHLyMw)。


### 一些代码的最佳实践

以上大多是一些架构，配置的优化，有一些代码的最佳实践，也一并说下。

1、podspec 减少 public

信息隐藏原则。

2、资源文件缩减

资源复用，远程请求，减少编译时长，优化启动时间。

3、代码使用前向声明

OC 的`@class, @protocal`，swift 的 `@_silgen_name`，前向声明。

[Using @_silgen_name to forward declare functions in Swift and improve build times](https://swiftrocks.com/using-silgenname-to-call-private-swift-code)

4、swift 的一些项 let，any，类型推导等，这些看看就好，没必要为了编译时长牺牲太多。

## 总结

通过包体优化一条线，其实涉及一些改造，有仓库规范推进，代码 lint 规范推进，工程配置调优，二进制化计划，工具链支持等。当然最重要的是编译时长的成果数据展示非常明显，对开发和测试的体验指标提升。

> 其实也有涉及项目推进、模块架构、软件维护等等工程性问题，不必停留于一个数据的产出。