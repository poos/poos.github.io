---
layout:     post
title:      Module的发展历史
subtitle:   代码构件引入的方式变迁，工程使用方式变化，针对swift混编的module处理。
date:       2022-05-25
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Summary
- Project
---

## 背景
技术在进步，需求在进步，代码复杂度也在进步。程序员架构时候经常说的一句话是，如果过于复杂的话，那么可以考虑加一个中间层。 #include，#import 到 @improt 就是这样一个过程。这篇文章简单聊聊这个前后的时间节点。
然后给swift二进制支持Modules简单做个说明。后续查阅可以使用可以更加清晰。
### #include
相当于直接包含
```c
#include <stdio.h>
int main()
{
printf("hello, world");
return 0;
}
```
#### 存在多次导入的隐患
```c
#include <stdio.h>
```
如果源文件包含下面的 #include 命令，就会两次包含 stdio.h，一次是直接包含，另一次是间接包含：
```c
#include <stdio.h>
#include "myProject.h"
```

### #import
#import是#inlcude的增强版，能防止同一个文件被多次包含。
本质上是采用条件式编译的命令，方便地避免多次包含相同的文件。
```c
#ifndef INCFILE_H_
#define INCFILE_H_
/* ...实际的头文件incfile.h的内容写在这里... */
#endif /* INCFILE_H_ */
```
```c
#import <UIKit/UIKit.h>
#import "CustomUIKit.h"
```
> 尖括号，编译器会在系统文件目录下查找。
> 双引号引用，编译器首先会在用户目录下查找，然后去安装目录中查找，最后在系统文件目录中查找。

#### #import和#include的弊端
因为 #import并没有改变本质，所以还是存在很多问题
- 编译时长：每次包含时，都要重复的检查依赖关系，所以检查需要M*N的时间。
- 宏冲突: 因为是简单包含，所以在两个模块使用相同的宏时候会发生冲突，极端情况下需要检查并且调整improt顺序解决问题。
- Conventional workarounds：使用公共约定避免编译问题。
- 模块不明确：不能区分是本模块的头还是其他模块的头？在OC如果滥用了了pch，工程中模块界限更加小，更容易出错了。
> 被迫的公约: 例如，绝大多数标头都需要包含保护，以确保多个包含不会破坏编译。宏名称使用LONG_PREFIXED_UPPERCASE_IDENTIFIERS编写，以避免冲突，一些库/框架开发人员甚至在标题中使用__underscored名称，以避免与（根据惯例）甚至不应该是宏的“正常”名称发生冲突。
### @class
不属于这个范畴，不讨论，因为它是@开头，列出来。个人理解@也是表示了一个从M*N到M+N的一个转变。

### @import / Modules
基于上面编译时长和模块界限的问题，clang.llvm.org提出了Module的概念。Modules相当于将框架进行了封装，然后加入在实际编译之时加入了一个用来存放已编译添加过的Modules列表。如果在编译的文件中引用到某个Modules的话，将首先在这个列表内查找，找到的话说明已经被加载过则直接使用已有的，如果没有找到，则把引用的头文件编译后加入到这个表中。这样被引用到的Modules只会被编译一次，但是在开发时又不会被意外使用到，从而同时解决了编译时间和引用泛滥两方面的问题。
Module的文件大概长这样，他指定了公开的头文件`XLib-umbrella.h`，所有需要公开的头文件在`XLib-umbrella.h`文件引入即可。另外，`module * { export * }`表示导出所有子模块。：
```ruby
module XLib {
umbrella header "XLib-umbrella.h"
export *
module * { export * }
}
```
- 如果是Xcode，可以通过设置 DEFINES_MODULE：YES 支持Modules。
- 如果是通过cocoapods导入，pod install 就会从podspec文件的public header生成一份 `XLib-umbrella.h`的文件，进而生成一份`XLib.modulemap`，一起放在Target Support Files相应的Header。然后设置Pods下相应的target的设置项。 MODULEM_MAP_FILE。
如有需要查看Modulemap的全部语法，可以到：[clang.llvm.org](https://clang.llvm.org/docs/Modules.html)。显然这套规则是从C的自下而上的支持的，所以OC可以很方便的使用。
> ModuleMap采用模块映射语言，但是到现在( 2020 年 Q3 为止)该语法依然不够稳定，所以建议：编写 modulemap 时需要尽可能使用少的关键字实现 module 功能，比如 framework、umbrella、header、extern、use。

## Swift/OC 都支持 Modules
除了OC支持Modules，Swift也天然的支持Modules。
Swift 的一些官方库正是通过 Module 方式导入的，例如UIKit。
```ruby
framework module UIKit {
umbrella header "UIKit.h"
module * { export * }
link framework "UIKit"
}
```
另外，一些自己自己制作的C库也可以通过Module方便的给swift使用。[Swift 关于 module.modulemap 使用](https://www.jianshu.com/p/ce49d8f32f77)

### Modules 在 Xcode 中的推广
有了好的方式，自然要在Xcode体现一下。
如果对于以前的工程，想要使用新的 Modules 特性，如果要把所有头文件都这样一个一个改成@import的话，会是很大的一个工作量。Apple自然也考虑到了这一点，于是对于原来的代码，只要使用的是iOS7或者MacOS10.9的SDK，在Build Settings中将Enable Modules(C and Objective-C)打开，然后保持原来的#import写法就行了。是的，不需要任何代码上的改变，编译器会在编译的时候自动地把可能的地方换成Modules的写法去编译的。

#### build setting 关于 Modules的参数
在 Xcode 中， build setting，build setting 和 Module 化相关的变量主要有这些：
对module自身的描述：
```
DEFINES_MODULE：YES/NO，module 化需要设置为 YES
MODULEMAP_FILE：指向 module.modulemap 路径
HEADER_SEARCH_PATHS：modulemap 内定义的 Objective-C 头文件，必须在 HEADER_SEARCH_PATHS 内能搜索到
PRODUCT_MODULE_NAME：module 名称，默认和 Target name 相同
```
对外部module的引用：
```
FRAMEWORK_SEARCH_PATHS：依赖的 Framework 搜索路径
OTHER_LDFLAGS：非二进制要设置依赖的framework来确定依赖关系

OTHER_CFLAGS：编译选项，可配置依赖的其他 modulemap 文件路径 -fmodule-map-file=${modulemap_path}
HEADER_SEARCH_PATHS：头文件搜索路径，可用于配置源码中引用的其他 Library 的头文件

SWIFT_INCLUDE_PATHS ：swiftmodule 搜索路径，可用于配置依赖的其他 swiftmodule
OTHER_SWIFT_FLAGS：Swift 编译选项，可配置依赖的其他 modulemap 文件路径 -Xcc -fmodule-map-file=${modulemap_path}

CLANG_ALLOW_NON_MODULAR_INCLUDES_IN_FRAMEWORK_MODULES: Xcode在默认情况下是不允许在framework中的头文件引入一个不属于任何Module的头文件
```

以上参数正确设置之后，跨库不论是OC还是Swift都可以找到相应的定义了，可以愉快的抹平差异桥代码了。

### 私有库源码方式支持 Modules
上面已经看到，Swfit支持Module，C支持Mdoules，那么自己的私有代码库如何支持Modules呢？

- OC/C
OC方法因为天然继承C，所以不论是C还是OC的，只需要生成生成一份modulemap文件即可。
如果是基于Cocoapods，使用`use_modular_headers!`参数可以在Pod的中间文件生成一份modulemap。

```ruby
module XLib {
umbrella header "XLib-umbrella.h"
export *
module * { export * }
}
```

> PS，库内混编，OC给swift用，使用`-bridging-Header.h`即可。但是没必要放入`XLib-umbrella.h`。OC使用Swift也是直接`import -Swfit.h`，等到编译过后可以用了。

- Swift
```ruby
module XLib {
umbrella header "XLib-umbrella.h"
export *
module * { export * }
}

```
这里没有-Swift.h相关文件，那么跨库使用怎么解决呢。因为强大的 Xcode 编译器，在**编译期间生成一份 `-Swfit.h`文件**，直接导入`-Swfit.h`即可。但是没必要放入`XLib-umbrella.h`。

> 真正编译完在build目录相应的文件夹有一个新的modulemap，是包含`-Swfit.h`相关信息的。`-Swfit.h`相关引用会追加到module文件上。

通过Module的定义解决了内部矛盾，确保跨库使用的一致性。

## OC/Swift私有库二进制 支持 Modules
```ruby
framework module XLib {
umbrella header "XLib.h"
export *
module * { export * }
}
```
看到，基本上是增加了`framework`关键字，用来指示这个Module是已经制作完framework的了。

### OC二进制
这里主要讨论framework，因为framework是以包的提供，里面包含资源文件，头文件。可以直接被使用。
对于纯OC库来说，如果需要支持二进制，那么提供modulemap指定Module公开的头文件即可：
```
| |____A
| | |____XLib
| | |____Headers
| | | |____XLibHeader.h
| | | |____XLib-bridging-Header.h
| | | |____XLib-umbrella.h
| |____Current
|____XLib
|____Headers
|____Modules
| |____module.modulemap
```
另外你已经看到上面的库提供了一个`-bridging-Header.h`，那么这个库也可以被swift代码方便的使用。**所以在OC上，跟非Mdoule化没有明显区别，加入一个modulemap文件即可。**

### Modules Swift二进制
首先，因为Swift模块被提前制作成了二进制，所以就需要提前处理上面提到的 `-Swfit.h`文件等问题。
事实上，除了`-Swfit.h`，需要处理还有`.swiftmodule文件夹`。而且`.modulemap`和其他的也长得不太一样。

看一下成品吧：
文件目录
```
|____Versions
| |____A
| | |____XLib
| | |____Headers
| | | |____XLibOCModule.h
| | | |____XLib-umbrella.h
| | | |____XLib-Swift.h
| | | |____XLib-bridging-Header.h
| |____Current
|____XLib
|____Headers
|____Modules
| |____module.modulemap
| |____XLib.swiftmodule
| | |____x86_64-apple-ios-simulator.swiftdoc
| | |____x86_64-apple-ios-simulator.swiftmodule
| | |____x86_64-apple-ios-simulator.swiftinterface
| | |____x86_64.swiftdoc
| | |____x86_64.swiftmodule
| | |____x86_64.swiftinterface
```
XLib.swiftmodule
```ruby
framework module XLib {
umbrella header "XLib-umbrella.h"
export *
module * { export * }
}
module XLib.Swift {
header "XLib-Swift.h"
requires objc
}
```
XLib-Swift.h
```
// 从编译文件获取
```
XLib.swiftmodule
```
// 从编译文件获取以下文件
| |____XLib.swiftmodule
| | |____x86_64-apple-ios-simulator.swiftdoc
| | |____x86_64-apple-ios-simulator.swiftmodule
| | |____x86_64-apple-ios-simulator.swiftinterface
| | |____x86_64.swiftdoc
| | |____x86_64.swiftmodule
| | |____x86_64.swiftinterface
```
**总结一句，就是因为你提前编译了，所以你要将提前编译的内容在framework准备好。**
如果成功打包了二进制库，并且支持了Module。站在这个高度上，那么凭借 Modules，可以完全抹平Swift-OC的差异了。**三方交付也更方便了。**

## 如何打包私有库的二进制

以上基本原理是通了的，以后如何打包还需要种种试错，前人栽的树可以先了解一下。Swift打包二进制下回分解～

不得不提到下面的cocoapods库：
cocoapods-generate # 通过Podspec生成模版工程
[cocoapods-packager](https://github.com/tripleCC/cocoapods-bin) # 通过Podspec生成的模版工程进行打包二进制。只支持OC，2年前archieve了。
因为swift用户的越来越多，也有越来越多的小伙伴们继续着cocoapods-packager没有走完的路。
[HamGuy/cocoapods-packager](https://github.com/HamGuy/cocoapods-packager/commit/c22fc93ea508623d265cd66635f1b995b48f6cee)在package时候复制modulemap文件，貌似没有复制swiftmodule文件。
[rubygems.org/gems/cocoapods-static-swift-framework](https://rubygems.org/gems/cocoapods-static-swift-framework) 打包swift二进制。
[cocoapods-xcframework](https://github.com/TyrantDante/cocoapods-xcframework/issues)打包xc framework。**支持M1，支持xcframework。**
跳槽去[Carthage](https://juejin.cn/post/6844904168843395079)寻求Swift二进制解决的。
还有完整解决方案的。
[cocoapods-bin](https://github.com/tripleCC/cocoapods-bin)一套成熟方案，使用本地双私有源开发模块工程。
[cocoapods-imy-bin](https://github.com/MeetYouDevs/cocoapods-imy-bin)在cocoapods-bin提供更多方便的功能，更少的lint限制，基于宿主打包二进制。

## 一些可能遇到的失败：
### Build Libraries for Distribution
build参数问题，如果没有无法生成 `.swiftinterface`文件;
`BUILD_LIBRARY_FOR_DISTRIBUTION=YES`
### 在{Development Pods}下，OC引用Swift文件找不到
解决：（建议壳工程化，可以避免此问题）
> 非壳工程， 在Targets-->Build Settings-->Swift Compiler - General-->Objective-C Generated Interface Header Name进行配置，默认文件名是工程名-Swift.h，一般不做改动。
### #import <XLib/XLib.h> 找不到
Podfile引用 use_frameworks! 与不引用 use_frameworks!的编译问题
因为这个头文件是xcode编译时自动生成的，在Products/Debug-iphonesimulator/lottie-ios/lottie-ios.framework/Headers 中，去掉**use_frameworks!**后就找不到了，解决方法：在Header Search Paths 添加对应的文件引用
### include of non-modular header inside framework module
framework module 没有 modular header
在podspec文件中加上：s.user_target_xcconfig = { ‘CLANG_ALLOW_NON_MODULAR_INCLUDES_IN_FRAMEWORK_MODULES’ => ‘YES’ }
### bitcode的错误
当pod工程中引入了.a静态库，.a静态库编写的时候bitcode是否开启和项目工程的bitcode开关不一致时，就会出现bitcode的报错。
解决办法：
我的工程中bitcode是关闭的，那么需要在podspec文件中加上：s.user_target_xcconfig = { ‘ENABLE_BITCODE’ => ‘NO’ }
这样就设置了该pod默认不使用bitcode，解决了报错。
### 使用私有库之后有些库找不到
在制作pod的过程中既要依赖一个私有库，又要依赖一个共有库，发现公有库报了找不到的错误
分析：
当我单独去依赖一个私有库：s.dependency ‘BIEncrypt’,’~> 0.2.4’
然后使用命令：pod lib lint --sources=BIEncrypt。去编译时，可以正常编译。
然后单独去依赖一个共有库：s.dependency ‘LKDBHelper’,’~> 2.5.1’
使用命令：pod lib lint。也是可以正常编译通过的。
可是一旦两种库同时依赖，就找不到公有库了。
原因：
使用–sources命令后，用到的库的地址都默认从这个命令中去找了，自然会就找不到公有库。
解决办法：
在–sources后以地址的形式依赖，包括公有库和私有库。每一个库都用地址的形式去依赖，尤其需要注意的是公有库的地址，应该用：https://github.com/CocoaPods/Specs.git，这样所有依赖库就都可以找到了。
### –skip-import-validation --skip-tests --quick命令
~~–quick命令是个坑，虽然使用它可以迅速通过验证，但是在pod repo push的时候并不存在这个命令，不能直接跳过，这样podspec自然会也推不上去。~~ 这个应该是基于imybin的。满帮自己是一的cocoapods-bin也提供了skip的能力。
–skip-import-validation --skip-tests命令组合可以常用，只要在demo中确定工程没有问题，如果想避免在编译期间的各种麻烦，可以加上这两个命令来解决。这个命令组合在push的时候也是可以使用的。
### 几个命令回顾
s.libraries = “c++”。添加pod依赖的lib文件。
s.ios.vendored_framework = “verify_engine/opencv2.framework”。添加pod依赖的第三方framework。
s.dependency ‘LKDBHelper’,’~> 2.5.1’。添加pod依赖。
s.resources = “facesdk/authorization/.bundle"。添加资源文件。
s.ios.vendored_libraries = "facesdk/lib/.a”。依赖静态库。
ss.dependency ‘BIFaceSDK/include’。依赖某个文件层级。
### 非cocoapods集成 SEARCH_PATHS 失败 || 找不到 .moudlemap 定义的框架名称
HEADER_SEARCH_PATHS = $(inherited) "${SRCROOT}/LGSwiftC/Public/LGSwiftA.framework/Headers" "${SRCROOT}/LGSwiftC/Public/LGSwiftB.framework/Headers"
// OTHER_CFLAGS：传递给用来编译C或者OC的编译器，当前就是clang
OTHER_CFLAGS="-fmodule-map-file=${SRCROOT}/LGSwiftC/Public/LGSwiftA.framework/module.modulemap" "-fmodule-map-file=${SRCROOT}/LGSwiftC/Public/LGSwiftB.framework/module.modulemap"
// SWIFT_INCLUDE_PATHS: 传递给SwiftC编译器，告诉他去下面的路径中查找module.file
SWIFT_INCLUDE_PATHS="${SRCROOT}/LGSwiftC/Public/LGSwiftB.framework" "${SRCROOT}/LGSwiftC/Public/LGSwiftA.framework"
### private.modulemap
创建是私有的modulemap文件命令中必须含有 private 比如：LGSwiftFramework.private.modulemap。并且在该文件内部LGSwiftFramework后面必须加上_Private，其中 P 必须大写。这些都是规则。
private.modulemap文件并不是真正意义上的让外部文件不能使用期私有的module，而是仅仅做了一个标识，来区分与.modulemap文件的不同而已。
### Api添加注解，OC给Swift使用的时候的迁移
wift与OC相互映射
OC映射到Swift方式：1. 宏；2. <工程名称>.apinotes。
宏配置的缺点：如果一个 SDK 使用 OC 来写的，现在需要适配 Swift。这样就需要给每一个方法或属性添加宏来适配，这样就会导致有大量工作要做，费时费力。并且要修改原有代码。
.apinotes：文件以.apinotes结尾，且该文件一定要放在 SDK 的目录里。该文件是采用yaml格式书写。官方地址：https://clang.llvm.org/docs/APINotes.html
```ruby
Name: OCFramework
Classes:
- Name: LGToSwift
SwiftName: ToSwift
Methods:
- Selector: "changeTeacherName:"
Parameters:
- Position: 0
Nullability: O
MethodKind: Instance
SwiftPrivate: true
Availability: nonswift
AvailabilityMsg: "这个不能用"
- Selector: "initWithName:"
MethodKind: Instance
```
### 制作的 Framework 启动后报错
swift 制作的 Framework 添加到 General - Embedded Binaries
### 非壳工程的编译顺序控制
~~当依赖的组件是 Library，并且包含 Swift 的源码，需将当前 Target -> Scheme -> Build 编译条件配置为非并行编译 uncheck Parallelize Build(如下图所示)，达到控制编译顺序的目的，避免因为依赖组件还未生成的 *-Swift.h 文件（依赖组件编译成功后生成），造成当前组件源码的编译错误。~~
在新版Xcode已经没有看到这个选项，默认是按照依赖build。
### swiftmodule 的传递依赖性
已知：有组件 A 依赖组件 B，组件 B 依赖组件 C 在 Objective-C 中，B 对外暴露的头文件中引用了 C 的公开头文件，我们叫组件 B 传递依赖 C，结果就是编译组件 A 时必须同时能找到组件 B 和组件 C 的头文件，否则编译失败。
然而 Swift 并没有公开头文件一说，只要组件 B import C，导致 swiftmodule 中也明确标记了 import C，当组件 A import B 时，也同时 import C ，如果组件 A 找不到组件 C 的 module，那组件 A 将编译失败。


## 参考
[百度App Objective-C/Swift 组件化混编之路（二）- 工程化](https://blog.csdn.net/olsQ93038o99S/article/details/112386043?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-1.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7Edefault-1.no_search_link)
[Objective-C Swift 混编的模块二进制化 1：基础知识](https://blog.csdn.net/weixin_34055910/article/details/91427232)
[iOS编译速度如何稳定提高10倍以上](https://www.jianshu.com/p/08cffdfa2885?hmsr=toutiao.io)
[Objective-C Swift 混编的模块二进制化 1：基础知识](https://blog.csdn.net/weixin_34055910/article/details/91427232)
[Swift编译慢？请看这里，全套开源](https://juejin.cn/post/6890419459639476237)
[Swift 二进制，静态库，动态库解决方案](https://www.jianshu.com/p/274483e2bfe4)
[iOS组件二进制化](https://juejin.cn/post/6844904168843395079)

一些Swift官方库对于Module处理代码和接口稳定性的文档：
https://github.com/apple/swift/blob/7123d2614b5f222d03b3762cb110d27a9dd98e24/test/Driver/loaded_module_trace_swiftinterface.swift
https://www.swift.org/blog/library-evolution/
https://forums.swift.org/t/update-on-module-stability-and-module-interface-files/23337

