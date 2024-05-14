---
layout:     post
title:      
hide-in-nav: true
subtitle:   Note - 从安装包瘦身 到 Project Setting, App Thinning, Executable file, 静态库, Assets.car
date:       2021-05-10
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Code
---


https://segmentfault.com/a/1190000019203452

### 1. App Thinning

Apple 会尽可能，自动降低分发到具体用户时，所需要下载的 App 大小。其中包含三项主要功能：Slicing、Bitcode、On-Demand Resources。

#### 1.1 Slicing

当向 App Store Connect 上传 .ipa 后，App Store Connect 构建过程中，会自动分割该 App，创建特定的变体（variant）以适配不同设备。然后用户从 App Store 中下载到的安装包，即这个特定的变体，这一过程叫做 Slicing。

**Slicing 是创建、分发不同变体以适应不同目标设备的过程**

#### 1.2 Bitcode

Bitcode 是一种程序中间码。包含 Bitcode 配置的程序将会在 App Store Connect 上被重新编译和链接，进而对可执行文件做优化。

> 对于 iOS 而言，Bitcode 是可选的（Xcode7 以后创建的新项目默认开启），watchOS、tvOS 则是必须的。

> Bitcode. When you archive for submission to the App Store, Xcode will compile your app into an intermediate representation. The App Store will then compile the bitcode down into the 64 or 32 bit executables as necessary.

#### 1.3 on-Demand Resources
on-Demand Resource 即一部分图片可以被放置在苹果的服务器上，不随着 App 的下载而下载，直到用户真正进入到某个页面时才下载这些资源文件。

> https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/On_Demand_Resources_Guide/index.html


### 2.包体积

.ipa (iOS Application Package)：iOS 应用程序归档文件，即提交到 App Store Connect 的文件
.app （Application）：应用的具体描述，即安装到 iOS 设备上的文件

下载大小：通过 WiFi 下载的压缩 App 大小
安装大小：此 App 将在用户设备上占用磁盘空间的大小

**下载大小**

#### 2.1.Architectures

如果不支持32位以及 iOS8 ，去掉 armv7 ，可执行文件以及库会减小，即本地 .ipa 也会减小

#### 2.2.Resources
资源的优化也就是平时的细心与审查。

图片、内置素材、Bundle、多语言、Json、字体、脚本、Plist、音频

图片：Assets.car
Bundle: 非放在 Asset Catlog 中管理的图片资源。包括 Bundle，散落的 png、jpg 等

瘦包具体的方式：

- 无用资源的删除
- 重复文件的删除
- 大文件压缩
- 图片管理方式规范
- on-Demand Resource（游戏的、前置关卡依赖、滤镜App 等的依赖资源，建议用这种方式动态下载图片资源）

> 使用 imageNamed 创建的 UIImage 会被立即加入到 NSCache 中（解码后的 Image Buffer），直到收到内存警告的时候才会释放不使用的 UIImage。而 imageWithContentsOfFile 会每次重新申请内存，相同图片不会缓存，所以 xcassets 内的图片，加载后会产生缓存 **综上：常用的、较小的图建议存放在 Images.xcassets 内管理。大图放在 Bundle 内管理。**

### 3. Executable file

#### 3.1 编译选项优化
##### 3.1.1 Generate Debug Symbols
Enables or disables generation of debug symbos. When debug symbols are enabled, the level of detail can be controller by the build 'Level of Debug Symbols' Setting.
调试符号是在编译时形成的。当 Generate Debug Symbols 选项为 YES 的时，每个源文件在编译成 .o 文件时，编译参数多了 -g 和 -gmodules 两项。打包会生成 symbols 文件。设置为 NO 则 ipa 中不会生成 symbol 文件，可以减少 ipa 大小。但会影响到崩溃的定位。保持默认的开启，不做修改。

##### 3.1.2 Asset Catalog Compiler
optimization 选项设置为 space 可以减少包大小
默认选项，不做修改。

##### 3.1.3 Dead Code Stripping
For statically linked executables, dead-code stripping is the process of removing unreferenced code from the executable file. If the code is unreferenced, it must not be used and therefore is not needed in the executable file. Removing dead code reduces the size of your executable and can help reduce paging.
删除静态链接的可执行文件中未引用的代码

Debug 设置为 NO， Release 设置为 YES 可减少可执行文件大小。

Xcode 默认会开启此选项，C/C++/Swift 等静态语言编译器会在 link 的时候移除未使用的代码，但是对于 Objective-C 等动态语言是无效的。因为 Objective-C 是建立在运行时上面的，底层暴露给编译器的都是 Runtime 源码编译结果，所有的部分应该都是会被判别为有效代码。

默认选项，不做修改。

##### 3.1.4 Apple Clang - Code Generation
Optimization Level 编译参数决定了程序在编译过程中的两个指标：编译速度和内存的占用，也决定了编译之后可执行结果的两个指标：速度和文件大小。
Build Settings -> code Generation -> Optimization Level
默认情况下，Debug 设定为 None[-O0] ，Release 设定为 Fastest,Smallest[-Os]。

None[-O0]。 Debug 默认级别。不进行任何优化，直接将源代码编译到执行文件中，结果不进行任何重排，编译时比较长。主要用于调试程序，可以进行设置断点、改变变量 、计算表达式等调试工作。
Fast[-O,O1]。最常用的优化级别，不考虑速度和文件大小权衡问题。与-O0级别相比，它生成的文件更小，可执行的速度更快，编译时间更少。
Faster[-O2]。在-O1级别基础上再进行优化，增加指令调度的优化。与-O1级别相，它生成的文件大小没有变大，编译时间变长了，编译期间占用的内存更多了，但程序的运行速度有所提高。
Fastest[-O3]。在-O2和-O1级别上进行优化，该级别可能会提高程序的运行速度，但是也会增加文件的大小。
Fastest Smallest[-Os]。Release 默认级别。这种级别用于在有限的内存和磁盘空间下生成尽可能小的文件。由于使用了很好的缓存技术，它在某些情况下也会有很快的运行速度。
Fastest, Aggressive Optimization[-Ofast]。 它是一种更为激进的编译参数, 它以点浮点数的精度为代价。
默认选项，不做修改。

##### 3.1.5 Swift Compiler - Code Generation
Xcode 9.3 版本之后 Swift 编译器提供了新的 Optimization Level 选项来帮助减少 Swift 可执行文件的大小：

No optimization[-Onone]：不进行优化，能保证较快的编译速度。
Optimize for Speed[-O]：编译器将会对代码的执行效率进行优化，一定程度上会增加包大小。
Optimize for Size[-Osize]：编译器会尽可能减少包的大小并且最小限度影响代码的执行效率。
We have seen that using -Osize reduces code size from 5% to even 30% for some projects. But what about performance? This completely depends on the project. For most applications the performance hit with -Osize will be negligible, i.e. below 5%. But for performance sensitive code -O might still be the better choice.
官方提到，-Osize 根据项目不同，大致可以优化掉 5% - 30% 的代码空间占用。 相比 -0 来说，会损失大概 5% 的运行时性能。 如果你的项目对运行速度不是特别敏感，并且可以接受轻微的性能损失，那么 -Osize 是首选。

除了 -O 和 -Osize， 还有另外一个概念也值得说一下。 就是 Single File 和 Whole Module 。 在之前的 XCode 版本，这两个选项和 -O 是连在一起设置的，Xcode 9.3 中，将他们分离出来，可以独立设置：

Single File 和 Whole Module 这两个模式分别对应编译器以什么方式处理优化操作。

Single File：逐个文件进行优化，它的好处是对于增量编译的项目来说，它可以减少编译时间，对没有更改的源文件，不用每次都重新编译。并且可以充分利用多核 CPU，并行优化多个文件，提高编译速度。但它的缺点就是对于一些需要跨文件的优化操作，它没办法处理。如果某个文件被多次引用，那么对这些引用方文件进行优化的时候，会反复的重新处理这个被引用的文件，如果你项目中类似的交叉引用比较多，就会影响性能。
Whole Module： 将项目所有的文件看做一个整体，不会产生 Single File 模式对同一个文件反复处理的问题，并且可以进行最大限度的优化，包括跨文件的优化操作。缺点是，不能充分利用多核处理器的性能，并且对于增量编译，每次也都需要重新编译整个项目。
如果没有特殊情况，使用默认的 Whole Module 优化即可。 它会牺牲部分编译性能，但的优化结果是最好的。

故，在 Relese 模式下 -Osize 和 Whole Module 同时开启效果会最好！

##### 3.1.6 Strip Symbol Information
1、Deployment Postprocessing
2、Strip Linked Product
3、Strip Debug Symbols During Copy
4、Symbols hidden by default

设置为 YES 可以去掉不必要的符号信息，可以减少可执行文件大小。但去除了符号信息之后我们就只能使用 dSYM 来进行符号化了，所以需要将 Debug Information Format 修改为 DWARF with dSYM file。

Symbols Hidden by Default 会把所有符号都定义成”private extern”，详细信息见官方文档。

故，Release 设置为 YES，Debug 设置为 NO。

##### 3.1.7 Exceptions
在 iOS微信安装包瘦身 一文中，有提到：

去掉异常支持，Enable C++ Exceptions和Enable Objective-C Exceptions设为NO，并且Other C Flags添加-fno-exceptions，可执行文件减少了27M，其中__gcc_except_tab段减少了17.3M，__text减少了9.7M，效果特别明显。可以对某些文件单独支持异常，编译选项加上-fexceptions即可。但有个问题，假如ABC三个文件，AC文件支持了异常，B不支持，如果C抛了异常，在模拟器下A还是能捕获异常不至于Crash，但真机下捕获不了（有知道原因可以在下面留言：）。去掉异常后，Appstore 后续几个版本 Crash 率没有明显上升。
个人认为关键路径支持异常处理就好，像启动时NSCoder读取setting配置文件得要支持捕获异常，等等

看这个优化效果，感觉发现了新大陆。关闭后验证.. 毫无感知，基本没什么变化。

可能和项目中用到比较少有关系。故保持开启状态。

##### 3.1.8 Link-Time Optimization
Link-Time Optimization 是 LLVM 编译器的一个特性，用于在 link 中间代码时，对全局代码进行优化。这个优化是自动完成的，因此不需要修改现有的代码；这个优化也是高效的，因为可以在全局视角下优化代码。

苹果在 WWDC 2016 中，明确提出了这个优化的概念，What’s New in LLVM。并且说在苹果内部已经广泛地使用这个优化方法进行编译。

它的优化主要体现在如下几个方面：

多余代码去除（Dead code elimination）：如果一段代码分布在多个文件中，但是从来没有被使用，普通的 -O3 优化方法不能发现跨中间代码文件的多余代码，因此是一个“局部优化”。但是Link-Time Optimization 技术可以在 link 时发现跨中间代码文件的多余代码。
跨过程优化（Interprocedural analysis and optimization）：这是一个相对广泛的概念。举个例子来说，如果一个 if 方法的某个分支永不可能执行，那么在最后生成的二进制文件中就不应该有这个分支的代码。
内联优化（Inlining optimization）：内联优化形象来说，就是在汇编中不使用 “call func_name” 语句，直接将外部方法内的语句“复制”到调用者的代码段内。这样做的好处是不用进行调用函数前的压栈、调用函数后的出栈操作，提高运行效率与栈空间利用率。
在新的版本中，苹果使用了新的优化方式 Incremental，大大减少了链接的时间。建议开启。

总结，开启这个优化后，一方面减少了汇编代码的体积，一方面提高了代码的运行效率。


#### 3.2 代码瘦身
代码的优化，即通过删除无用类、无用方法、重复方法等，来达到可执行文件大小的减小。

目前市面上的扫描的思路大致可以分为 3 种：

基于 Clang 扫描
基于可执行文件扫描
基于源码扫描
先谈几个概念。

可执行文件就是 Mach-O 文件，其大小是油代码量决定的，通常情况下，对可执行文件进行瘦身，就是找到并删除无用代码的过程。找到无用代码的过程类比找到无用图片的思路。

找到类和方法的全集
找到使用过的类和方法集合
取2者差集得到无用代码集合
工程师确认后，删除即可
LinkMap 文件分为3部分：Object File、Section、Symbols。

Object File：包含了代码工程的所有文件
Section：描述了代码段在生成的 Mach-O 里的偏移位置和大小
Symbols：会列出每个方法、类、Block，以及它们的大小

#### 3.2.2 基于可执行文件扫描（LinkMap 结合 Mach-O 找无用代码）
上面我们得知可以通过 LinkMap 统计出所有的类和方法，还可以清晰地看到代码所占包大小的具体分布，进而有针对性地进行代码优化。



### 4. App Extension
App Extension 的占用，都放在 Plugin 文件夹内。它是独立打包签名，然后再拷贝进 Target App Bundle 的。
关于 Extension，有两个点要注意：

静态库最终会打包进可执行文件内部，所以如果 App Extension 依赖了三方静态库，同时主工程也引用了相同的静态库的话，最终 App 包中可能会包含两份三方静态库的体积。

动态库是在运行的时候才进行加载链接的，所以 Plugin 的动态库是可以和主工程共享的，把动态库的加载路径 Runpath Search Paths 修改为跟主工程一致就可以共享主工程引入的动态库。

所以，如果可能的话，把相关的依赖改成动态库方式，达到共享。

### 5. 静态库瘦身
项目中都会引入第三方静态库。通过 lipo 工具可以查看支持的指令集，比如查看微博 SDK
终端切换到微博 SDK 的目录下执行下面命令

静态库指令集信息查看：lipo -info libname.a(或者libname.framework/libname)
lipo -info libWeiboSDK.a
//Architectures in the fat file: libWeiboSDK.a are: armv7 arm64 i386 x86_64
我们知道 i386、x86_64 是模拟器的指令集。所以我们可以模拟器版本的指令集。因为 armv7 也可以兼容 armv7s。所以 armv7s 也可以删除了。只保留 armv7 和 arm64

静态库拆分：lipo 静态库文件路径 -thin CPU架构 -output 拆分后的静态库文件路径
静态库合并：lipo -create 静态库1文件路径 静态库2文件路径... 静态库n文件路径 -output 合并后的静态库文件径
lipo libWeiboSDK.a -thin armv7 -output libWeiboSDK-armv7.a
lipo libWeiboSDK.a -thin arm64 -output libWeiboSDK-arm64.a
lipo create libWeiboSDK-armv7.a libWeiboSDK-arm64.a -output libWeiboSDK.device.a
通过上面的操作我们将静态库里面支持模拟器的指令集给去掉了，所以模拟器是无法跑代码的，如何解决？

平时使用包含模拟器指令集的静态库，在 App 发布的时候去掉
如果使用 Cocoapods 管理可以使用2份 Podfile 文件。一份包含指令集一份不包含，发布的时候切换 Podfile 文件即可。或者一份 Podfile 文件，但是配置不同的环境设置
补充2个说明：

dSYM 文件
符号表文件 .dSYM 文件是从 Mach-O 文件中抽取调试信息而得到的文件目录，实际用于保存调试信息的是 DWARF 文件
自动生成。Xcode 会在工程编译或者归档的时候自动生成 .dSYM 文件，在 Buld setting 设置中有开关可以设置去关掉 .dSYM 文件
手动生成。通过脚本从 Mach-O 文件中提取出来。
$ /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/dsymutil /Users/wangzz/Library/Developer/Xcode/DerivedData/YourApp-cqvijavqbptjyhbwewgpdmzbmwzk/Build/Products/Debug-iphonesimulator/YourApp.app/YourApp -o YourApp.dSYM
该方式通过 dsymutil 工具，从项目编译结果 .app 目录下的 Mach-O 文件中提取出调试符号表文件。Xcode 在归档的时候是通过它生辰的 .dSYM 文件

DWARF 文件
DebuggingWith Arbitrary Record Formats 是 ELF 和 Mach-O 等文件格式中用来存储和处理调试信息的标准格式，.dSYM 文件中真正保存符号表数据的是 DWARF 文件。DWARF 文件中不同的数据都保存在相应的 section 中。
最后的一个对比效果图：
瘦身效果图

总结：瘦身技术常见操作就这些，但是维持应用包体积的瘦身却是一个观念，从日常开发到线上发布都需要有这个意识。这样当你在写代码的时候就会考虑同样一个效果，你的具体实现手段是怎么样的。比如为了一个稍微炫酷的效果就要引入一个很大的三方库，有了“瘦身”的意识，你很大可能就是自己动手撸一个代码。比如一些无用资源的管理方式、有用的图片资源的高效管理方式等等。有了意识，行动自然会往这个方面去靠。（😂大道理一套一套的。我也不想的，毕竟是playboy）

其中遇到了一个神奇的问题。lint 的时候看到一些未使用的依赖库。见 问题

By the way：
如果在应用包瘦身方面有其他的做法，请告知，完善文章。




### 查看 Assets.car 文件

https://stackoverflow.com/questions/22630418/analysing-assets-car-file-in-ios

https://github.com/steventroughtonsmith/cartool

- open terminal
- cd /path/to/cartool
- ./cartool /path/to/Assets.car /path/to/outputDirectory

[Create asset catalogs and sets](https://hel[p.apple.com/xcode/mac/8.0/#/dev10510b1f7)
[Folders](https://developer.apple.com/library/archive/documentation/Xcode/Reference/xcode_ref-Asset_Catalog_Format/FolderStructure.html#//apple_ref/doc/uid/TP40015170-CH33-SW1)
[Adding Images](https://developer.apple.com/library/archive/documentation/ToolsLanguages/Conceptual/Xcode_Overview/AddingImages.html)