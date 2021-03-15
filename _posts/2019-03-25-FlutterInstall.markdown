---
layout:     post
title:      Flutter 《一》 安装和配置，QA
subtitle:   macOS 配置 Flutter 环境，一些值得注意的 QA
date:       2019-03-25
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Flutter
---

![img](https://github.com/dart-lang/site-shared/raw/master/src/_assets/image/flutter/icon/64.png?raw=1)

### 安装


#### 安装教程

所有的安装文档可以在 [macOS 安装指南](https://flutterchina.club/setup-macos/) 查看.

- 命令行安装

- 项目下载，安装必备命令行工具

- android studio 安装


#### tips

android studio 等第三方软件可以使用 **cask** 工具包安装。可以使用 **brew install cask** 安装。方便的使用命令行安装程序。

另外的； 有一个 **mas** 用于管理 AppStore 的应用，可以在 命令行下载当前用户已经购买的应用。具体就不再展开了。

#### 错误

文档基本有所有的错误说明，碰到错误对应处理即可，一般可以一遍过。 偶尔需要重启终端。

- flutter doctor ，对应的错误一般都有给出错误提示。

- android sdk ，这个需要到打开 ide 安装。

- android studio plugins ，这个需要到 ide 的设置中搜索安装，需要重启 ide。

- 全局的环境变量要设置好 。

#### 在中国使用

将以下代码放入 .bash_podfile，或者直接在终端运行即可。

export FLUTTER_STORAGE_BASE_URL=https://mirrors.sjtug.sjtu.edu.cn
export PUB_HOSTED_URL=https://dart-pub.mirrors.sjtug.sjtu.edu.cn

export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

#### 总结

文档比较全，按文档走基本没有问题。


建议浏览 下一篇 QA 的相关资料。



### [FAQ](https://flutterchina.club/faq/)

#### Flutter 是什么?

Flutter是一款移动应用程序SDK，包含框架、widget和工具，为开发人员提供了一种在Android和iOS上构建和部署精美移动应用程序的简单高效的方式。

#### 是什么让Flutter如此独一无二?
Flutter与用于构建移动应用程序的其它大多数框架不同，因为Flutter既不使用WebView，也不使用操作系统的原生控件。 相反，Flutter使用自己的高性能渲染引擎来绘制widget。

另外，Flutter的不同是因为它核心只有一层轻量的C/C++代码。Flutter在 [Dart](https://www.dartlang.org/tools)（一种现代的、简洁的、面向对象的语言）中实现了其它大部分系统（组合、手势、动画、框架、widget等）， 开发人员可以轻松地进行读取、更改、替换或移除。这为开发人员提供了对系统的巨大可定制性。


#### Flutter SDK里面有什么？
- 深度优化了的、移动优先的2D渲染引擎
- 现代、响应式框架
- 丰富的Android和iOS套件
- 单元和集成测试的API
- 连接到系统和第三方SDK的Interop和插件API
- 无头的测试运行器，用于在Windows、Linux和Mac上运行测试
- 用于创建、构建、测试和编译应用程序的命令行工具

#### Flutter使用什么技术？
Flutter使用C、C ++、Dart和Skia（2D渲染引擎）构建。请参阅此 [架构图](https://docs.google.com/presentation/d/1cw7A4HbvM_Abv320rVgPVGiUP2msVs7tfGbkgdrTy0I/edit#slide=id.gbb3c3233b_0_162) 以更好地了解主要组件。

#### Flutter支持哪些设备和操作系统版本？
移动操作系统：Android Jelly Bean，v16,4.1.x或更新的版本以及iOS 8或更新版本。

移动硬件：64位iOS设备（从iPhone 5S和更新的iPhone型号开始）以及ARM Android设备。

请注意，我们目前不支持：

- ARM32 iOS设备（iPhone 4，iPhone 5;问题＃2089）
- x86 Android设备（问题＃9253）
我们支持使用Android和iOS设备(包括Android模拟器和iOS模拟器)来开发测试Flutter应用。

我们测试了各种低端到高端手机，但我们还没有官方设备兼容性保证。

我们不会在平板电脑上进行测试，因此平板电脑上的某些widget可能会出现问题。我们尚未在我们的widget中实施针对平板电脑的改动。


#### Flutter引擎有多大？

2018年12月，我们测量了一个最小的 Flutter 应用（不含 Material 组件，仅有一个 Center 控件，使用 flutter build apk 构建）的下载大小，并打包为 release 版本，大小约为 4.06 MB.

对于这个简单的应用，核心引擎大约为 2.7MB(已压缩)，框架和应用程序代码大约 820.6KB (已压缩)，LICENSE 文件为 53.5KB(已压缩)，必要的 Java 代码 (classes.dex) 为 61.8KB(已压缩)，此外还有大约 450.4KB(已压缩)的 ICU 数据。

这些数据是通过 apkanalyzer 测量得到的，它也是 Android Studio 内置的分析工具。

在 iOS 上，同一个应用的 release 版本 IPA 文件在 iPhone X 上的下载大小为 10.8MB，数据通过苹果的 App Store Connect 得到。IPA 比 APK 更大的主要原因是苹果加密了 IPA 中的二进制文件，这使得压缩的效果有所折扣（参见苹果 QA1795 的 iOS App Store特定注意事项章节）。

当然，数据因人而异，我们也推荐你测量你自己的应用。要测量 Android 应用，请执行 flutter build apk 并将 APK (build/app/outputs/apk/release/app-release.apk) 导入 Android Studio 中以查看具体的尺寸报告。要测量 iOS 应用，将发行版的 IPA 上传到 App Store Connect 中并在那里获取尺寸报告。


#### Flutter框架使用什么编程范式？
Flutter是一个多范式编程环境。在Flutter中使用了过去几十年中开发的许多编程技术。我们使用的每一个范式都是我们相信该它的优势特别适合Flutter：

- **组合**：Flutter使用的主要范例是使用小对象，然后将它们组合在一起以获得更复杂的对象。Flutter widget库中的大多数widget都是以这种方式构建的。例如，Material FlatButton 类是使用MaterialButton 类构建， 该类本身使用IconTheme、InkWell、Padding、Center、Material、AnimatedDefaultTextStyle和ConstrainedBox组合 构建。该InkWell 使用内置GestureDetector。Material 是使用内置AnimatedDefaultTextStyle、NotificationListener和AnimatedPhysicalModel。等等，它们都是widget。
- **函数式编程**：整个应用程序可以仅使用StatelessWidget来构建 ，这些函数本质上是描述参数如何映射到其他函数的函数。在计算布局或绘制图形的底层（这些应用程序不拥有状态，因此通常是非交互式的）例如，Icon widget本质上是一个将其参数（颜色、 icon、size）映射到布局基本单元的函数。此外，大量使用的是不可变数据结构，包括整个Widget类层次结构以及许多支持类，如 Rect和 TextStyle也都是。Dart中的Iterable API经常用来处理框架中的值列表，它大量使用了函数式（map，reduce，where等）方法。
- **事件驱动**：用户交互由事件对象表示，这些事件对象被分派给注册了事件处理程序的回调。屏幕刷新也由类似的回调机制触发。监听类是动画系统的基础，它确立了与多个监听事件订阅模式。
- **基于类的面向对象编程**：框架的大部分API都是使用继承类来构建的。我们使用一种方法来在基类中定义非常抽象的API，然后在子类中迭代地对它们进行定制化。例如，我们的渲染对象有一个与坐标系无关的基类（RenderObject），然后我们有一个子类（RenderBox），它引入了基于笛卡尔坐标系（x / width）和Y /高度）。
- **基于原型的面向对象编程**： ScrollPhysics 类将实例链接起来组成适用于在运行时动态滚动的physics。这使得系统可以编写包含特定平台physics的分页physics，而无需在编译时选择平台。
- **命令式编程**：直接命令式编程通常与对象内部封装的状态配对，用于提供最直观的解决方案。例如，测试是以一种强制性风格编写的，首先描述测试中的情况，然后列出测试必须匹配的不变量，然后根据测试需要推进时钟或插入事件。
- **响应式编程**：widget和元素树有时被描述为响应式的，因为在widget的构造函数中提供的新输入会立即作为widget的构建方法对较低级别widget的更改传播，并在较低widget中进行更改（例如，作为响应到用户输入）通过事件处理程序传播回widget树。根据widget的需求，功能向应和命令响应两方面都存在于框架中。具有构建方法的widget仅由一个表达式组成，该表达式描述了widget如何对其配置中的变化做出反应的功能响应widget（例如，Divider类）。 构建方法通过几个语句构建子项列表的widget，描述了widget如何对其配置中的更改作出反应，这些都是命令性响应widget（例如 Chip类）。
- **声明式编程**：widget的构建方法通常是具有多层嵌套构造函数的单一表达式，使用Dart的严格声明子集编写。这样的嵌套表达式可以机械地转换成任何适合表达的标记语言或从任何适合表达的标记语言转换。例如， UserAccountsDrawerHeader widget具有很长的构建方法（20行以上），由单个嵌套表达式组成。这也可以与命令式风格相结合来构建用纯粹声明式方法难以描述的UI。
- **泛型**：类型可用于帮助开发人员及早发现编程错误。Flutter框架使用泛型编程来处理这个问题。例如， State类根据其关联widget的类型进行参数化，以便Dart分析器可以捕获状态和widget的不匹配。类似地， GlobalKey类需要的类型参数，以便它可以访问远程widget的状态下在一个类型安全的方式（使用运行时检查）， 路由接口是参数化时，它是预期使用类型 pop和集合，例如List、Map和 Set都是参数化的，这样可以在分析过程中或在调试期间的运行时提前捕获不匹配的元素。
- **并发**：Flutter大量使用 Future和其他异步API。例如，动画系统通过Future来完成动画完成时的通知。图像加载系统同样使用Future在加载完成时进行报告。
- **约束**：Flutter中的布局系统使用弱形式的约束编程来确定场景的几何形状。约束（例如，对于笛卡尔盒子，最小和最大宽度以及最小和最大高度）从父母传递给孩子，并且孩子选择生成的几何结构（例如，对于笛卡尔盒子，大小，特别是宽度和高度）满足这些限制。通过使用这种技术，Flutter通常可以通过一次遍布整个场景。


#### 我可以在我现有的原生应用程序中使用Flutter吗？
可以，您可以在现有的Android或iOS应用中嵌入Flutter，但是我们的工具目前尚未完全针对此用例进行优化（请参阅此问题以了解详细信息）。

目前的两个示例是 [platform_view](https://github.com/flutter/flutter/tree/master/examples/platform_view) 和 [flutter_view](https://github.com/flutter/flutter/tree/master/examples/flutter_view) 示例。一些开始文档可以在wiki页面中 [添加Flutter到现有的应用程序](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps)。

wiki文档显示添加到现有项目 需要关闭 bitcode 。
