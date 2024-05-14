---
layout:     post
title:      iOS 应用内存调试与优化
subtitle:   内存增长类型，大对象，泄露，oom如何监控治理
date:       2022-07-14
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- APM
- Memeory
---


### 序

摘要：本文展示了 Apple 平台iOS App 内存的计算、分配、和调优技巧。全文分四个部分：
第一部分讲解了内存的基本概念；
第二部分展示如何使用 Instruments 工具，通过内存快照来监测当前内存使用情况；
第三部分介绍了使用 Xcode Memory Debugger 和命令行工具进行分析优化；
第四部分整体总结，简单介绍一些常见的内存问题。
通过本文的探索，您可以更好地理解应用的内存构成和优化应用的内存使用。


### 一、了解应用内存

随着摩尔定律不断发展，沿用冯·诺伊曼计算机结构的操作系统，在运算器（CPU）上的效率越来越快，和存储器的读写效率差距越来越大，于是诞生了多级缓存的概念。体现在硬件设备上，出现了随机访问存储器（RAM），简称内存，来匹配 CPU 高速数据吞吐的需求。

苹果平台上的内存机制沿用了经典操作系统内存相关概念，同其他系统一样，也具备物理内存和虚拟内存、内存分页、内存交换机制等概念，下面进行简单地回顾：

1. 物理内存和虚拟内存

在 iOS/macOS 系统上，每个应用程序拥有一块私有的、连续的虚拟内存空间地址，这些地址会映射到物理内存，通过这样的方式，系统中的应用程序实现共享物理内存。

这里的实际使用内存不等于分配内存。实际使用内存是指物理内存的使用，而分配内存是应用在虚拟内存地址空间上请求的内存，称之为虚拟内存（Virtual Memory），同时不同类型的分配内存是单独计算的。

当您的应用分配内存时，这些新分配的内存不会立即或直接占用物理内存的空间，相反，它们会在系统为每个进程提供的虚拟内存地址空间上保留一些空间。当程序稍后实际使用此分配时，系统将在物理内存上准备空间。

2. 内存分页

物理内存的最小单位被称为帧（Frame），而虚拟内存的最小单位被称为页（Page）。为了方便虚拟内存和物理内存的映射及管理，目前主流操作系统内存都采用了分页管理。

在苹果平台上，同一类型的内存分配按类别 Categories 分组，并占用非连续的虚拟地址空间。这些类别可能包括：

- App 可执行二进制文件
- 堆空间，即动态分配内存区域
- 栈空间，用于存储本地和临时变量以及一些函数参数
- 类实例及手动创建分配的内存
- 动态 Frameworks
- 只读资源映射（如应用资产文件）


这些类别称之为不同的区域（Region）。内存操作在底层以内存页（memory page）为粒度，在现代苹果设备上（A7 之后）每个内存页为 16 KiB。这意味着每个 region 占用一个或多个 memory page，并且至少有 16 KiB 大。随着应用的运行，其内存状态不断演变：新对象被分配，旧内容被销毁，导致内存区域也在不断变化。但只有 region 上使用的 memory page 会在物理内存上，系统像任何其他应用程序一样将这部分内存计入应用使用内存。

3. 内存足迹（Memory Footprint）

内存足迹（Memory Footprint）是苹果平台提出的一个衡量内存使用情况的主要指标，它基于上面提到的内存分页，包括三种类型：

脏内存（Dirty Memory Pages）
压缩内存（Compressed Memory Pages）
干净内存（Clean Memory Pages）

#### Dirty Memory

指应用程序写入的任何内存。分配的内存页有写操作和使用，称之为 Dirty Memory。主要包括：

- 所有的堆内存分配
- Frameworks 进行了变量或符号修改之后的内存页
- 苹果芯片设备加载的 Metal 资源
> 在使用苹果芯片的设备上，访问的 Metal 资源也属于这一类，这是因为 CPU 和 GPU 共享同一个快速统一内存池。通过使用相同的内存池，M1 上的所有特殊的协同处理器都可以彼此快速交换信息，从而可以显著提高性能，这也是 M 系列处理器强悍的原因之一。

#### Compressed Memory

应用已经使用的 Dirty Memory 中，如果一些 Dirty Pages 长时间不使用，系统可能会通过压缩这些内存页或将其存储在闪存或磁盘上来减少它们的物理内存占用，这个过程称之为交换（Swapping），这样可以允许系统运行更多应用程序和服务。

不同于 macOS，受限于移动端存储的读写速度、使用寿命，iOS 系统没有传统的桌面操作系统一样的 swap in/out 内存交换机制。为了优化内存管理，iOS 7 开始引入了内存压缩机（memory compressor）机制：内存压缩机以 CPU 换空间，将长期占用且未访问的内存页压缩清空，来腾出更多的内存空间。当应用稍后再次访问这些页面时，系统将从重新解压或分页这些内存页。这部分压缩还原的内存称之为 Compressed Memory。

它在节省内存的同时提高了系统的响应速度，其特点可以归结为：

- Shrinks memory usage 减少了不活跃内存占用
- Improves power efficiency 改善电源效率，通过压缩减少磁盘IO带来的损耗
- Minimizes CPU usage 压缩/解压十分迅速，能够尽可能减少 CPU 的时间开销
- Is multicore aware 支持多核操作

> 在 iOS 应用上内存压缩时，部分内存会被直接从物理内存移除，而不是备份到磁盘和后续通过磁盘还原到内存，后续需要访问时需要通过内存重新加载。

> 然而，今年新发布的 iPadOS 16 系统带来虚拟内存交换特性（Virtual Memory Swap）开始支持 swap in/out 内存交换功能，不过需要 M1 及以上 iPad 设备。这对于 iPad 的多任务功能是一个福音，最多可以解锁 16GB 的 RAM。

#### Clean Memory

没有对其内容进行修改的内存页。包括内存映射文件，例如纹理或音频资产，以及加载到进程中的 Frameworks 等。系统可以随时收回或从磁盘中重新加载，因此它们不会计入应用的内存占用。然而，它们可能长驻于内存中，过度使用会减慢系统和应用应用的运行速度。

Dirty Memory 和 Compressed Memory 结合在一起，我们称之为内存足迹。系统通过它来衡量实际内存使用情况，管理和限制内存使用。

4. 小结

本部分介绍了内存的起源，以及苹果平台内存的基本概念和特点：

1. 和主流操作系统类似，iOS / macOS 上也是 page mapping 机制实现物理内存映射虚拟内存，通过交换（Swapping）机制实现物理内存的共享和调度。
2. 在虚拟内存空间上按类型（Category）分区（Region），同一个区域由分页（Page）构成，每页大小为 16 KiB。
3. 内存的分配是指在虚拟内存空间上的分配。
4. 苹果通过称之为内存足迹的指标，来分析内存实际使用情况。内存足迹主要包括 Dirty Memory 和 Compressed Memory，即虚拟内存空间上实际读写操作使用过的内存。

### 二、内存剖析工具

在前面介绍、了解了内存的基本概念和原理之后，是不是就可以着手分析内存性能问题了呢？

子曰："工欲善其事，必先利其器"。内存优化的第一步，还得通过一系列工具来收集性能数据，而苹果对于内存的使用情况提供了多个角度的工具展示。本部分将介绍 Xcode、Instruments 和终端命令行工具剖析内存数据和内存增长，从不同的角度和工具剖析内存之谜。

#### 1. 内存占用概览

下面以苹果提供 Xcode 示例代码 Modern Rendering with Metal 为例演示内存概览页面。下载和启动该示例应用项目，在 Xcode 的调试导航器中打开内存报告，仪表上的数字显示了应用当前内存的使用情况，这是应用当前内存使用情况的概览和第一视角：



除了 Xcode 内存仪表盘外，Mac 上的活动监视器也可以查找内存占用。此外也可通过 API 来快速查询当前足迹和可用内存：通过调用 os_proc_available_memory 查看 iOS、iPadOS 或 tvOS 应用的可用系统内存：



对于任何苹果平台上的内存占用，您可以通过 proc_pid_rusage 获取，并从 get pid、rusage_info_current（目前为版本 6）和数据存储中获取进程 ID。并获取其物理内存占用、生命周期最大物理内存占用属性：



#### 2. 内存使用详情

当从 Xcode 运行应用时，内存报告可以显示一段时间内的内存占用汇总。通过在 Instruments 中通过一系列工具分析应用，则可以更详细地了解内存使用情况。

以上面示例项目为例，从 Xcode 中按住运行按钮，然后选取 Profile，打开 Instruments。Instruments 包括一系列分析工具，这些工具记录了内存不同角度、随时间变化、可视化的使用数据。

今年新增的 Game Memory Template 可以更好地了解 Metal 应用中的内存增长。此模板附带 Allocations 和 Metal Resource Events 工具，用于记录内存分配和历史记录，VM Tracker 用于记录内存占用，Virtual Memory Trace 用于记录虚拟内存活动，Metal Application 和 GPU 用于记录与 Metal 相关的事件。

这里重点介绍前三个工具：Allocations、Metal Resource Events 和 VM Tracker。

首先为应用录制一段 trace 用于分析：在 Instruments 按下录制按钮开始录制，稍后按下相同的按钮或退出应用停止录制。或者在终端使用 xctrace 命令录制，这可能在自动化工作流程中非常有用：

    xctrace record --device-name "xxx's iPhone" --template name "Game Memory" --attach ModernRenderer --output ModernRenderer.trace --time-limit 30s
##### 2.1 Allocations 工具

Allocations 工具追踪所有堆分配和匿名虚拟内存 (VM) 分配的大小和数量，可以提供内存分配、内存大小和对象引用计数的详细视图，并按类别组织和显示。通过 Allocations 统计视图，有一系列选项用于查看和分析内存追踪历史。



All Heap Allocations

All Heap Allocations 选项显示了分配的所有堆内存，代表了应用真实占用的内存。通常，可以从堆内存较大的分配来入手分析内存使用情况：

点击 Size 列，按内存大小进行排序，点击地址栏内存对象右侧小箭头以查看 Swift 和 Objective-C 对象的引用计数变化；
选中某个分配地址，右边检视面板会显示内存堆栈追踪历史；
点击右侧按钮可以显示/隐藏系统库或框架；
双击右侧方法调用栈中的某个方法，可以跳转查看源码详情。
这里以 Modern Rendering with Metal 项目为例，查看加载 Assets 资源时的分配、追踪方法调用栈：



All Anonymous VM

All Anonymous VM 则显示了系统为程序分配的虚拟内存。主要包含一些系统模块的内存占用，通常无法直接控制该部分虚拟内存的大小。

同样以 Modern Rendering with Metal 项目为例，在 Metal 应用中，IOAccelerator 和 IOSurface 类别通常占用了很大的分配大小。IOAccelerator 对应于 Metal 资源加载，从堆栈追踪信息中，可以看到在加载 Assets 资源时发生了这种分配：



IOSurface 对应于对象绘制。在这里，可以看到堆栈跟踪显示了 MetalKit 视图的绘制请求：



Allocations 视图切换

Allocations 工具默认显示可视化、随时间线变化的内存大小图。同时，也可以在 Allocations 下方下拉箭头按钮上自定义显示内存密度 Allocation Density。Allocation Density 将随时间更新执行的分配量并显示内存分配的峰值，这些峰值可能是内存增长的来源。此外，Active Allocation Distribution 也从内存开辟激活历史的角度记录了何时创建分配了内存。切换这些不同的内存追踪历史角度，可以协助更好地了解、分析、优化内存增长点。



##### 2.2 Metal Resource Events 工具

为了更好地了解分配的 Metal 资源，可以利用 Metal 资源事件（Metal Resource Events）工具。Metal Resource Events 工具是围绕 Metal 资源设计的。在资源事件视图中，可以找到 Metal 资源内存分配和释放的历史记录。在这里，可以通过标签来识别 Metal 资源，这些标签可以通过 Metal API 以代码形式指定。与 Allocations 工具类似，在检查面板中可以找到堆栈分配历史记录。此外，Metal 设备在表格中还添加了内存分配和释放追踪栏，有助于从内存随时间分配密度，即内存占用峰值角度，对 Metal 资源的内存分配过程有一个可视化的了解。



##### 2.3 VM Tracker 工具

到目前为止，Allocations 工具和 Metal Resource Events 工具可以帮助理解内存分配。然而，分配的内存并不总是实际使用的内存。因此，让我们转到 VM Tracker 工具来分析内存的实际使用情况。

VM Tracker 显示了未压缩的 Dirty Memory、Compressed/Swapped Memory。这里：

- Dirty Size - 表示未压缩的脏内存
- Swapped Size - 表示已压缩/交换至磁盘的内存
- Resident Size - 表示实际使用物理内存，包含物理内存中已加载的干净内存和脏内存


上面摘要视图详情显示了虚拟内存区域，"Type" 一栏显示 VM Region 的不同类型。这里的类型可以区分不同的内存使用来源，其中 All 和 Dirty 只是统计汇总，其他类型如 __TEXT 表示代码段的内存映射，__DATA 表示数据段的内存映射，CoreServices 表示系统库的内存映射等。而 "mapped file" 类型表示一些内存映射的资源，如应用 Assets 资源。例如这里的 Modern Rendering with Metal 示例项目中，"bistro" Asset 资源文件映射到了内存中。

#### 3. 本章小结

收集数据是性能优化的第一步。本部分介绍了苹果平台上常用的内存剖析工具和使用方法，包括传统的 Xcode 内存概览、今年 Instruments 里新增的 Game Memory Template 工具，利用这些工具查看和分析宏观、底层各个角度的内存使用情况，往往是内存优化的先决条件。

### 三、Memory Graph 调试和定位问题

上面 Xcode、Instruments 新增的 Game Memory Template 和命令行工具对于帮助理解内存使用随时间的变化非常有用。此外，您可能还想捕捉应用在给定时间的内存状态，以便更深入地挖掘该内存状态并通过不同的角度对其进行性能瓶颈挖掘和优化。为此，苹果提供了 Memory Graph 和一套相应的工具。

为了增加趣味，这里用烹饪食谱类比来说明如何使用 Memory Graph 分析内存和挖掘性能瓶颈，它包括原材料、准备部分、烹饪（分析）过程三部曲：

1. 原材料

使用内存图来分析内存，需要准备以下内容：

应用 Xcode 项目，这里以上面的 Modern Rendering with Metal 示例项目为例
开启 Malloc Stack Logging
捕获的 Memory Graph
2. 准备 Memory Graph

Memory Graph 是通过开启内存记录后生成的一个内存日志文件，可以有效地存储应用内存状态的完整快照，包括对象创建历史、引用以及任何压缩或交换内存痕迹。Memory Graph 快照可以随时创建，例如当问题发生时，或者在问题发生之前和之后创建两个快照进行比较。

开启 Malloc Stack Logging 录制

Malloc Stack Logging 记录应用过程中的内存分配信息。在 Xcode 项目 > Scheme 设置中，选择 Run，转到 Diagnostics，然后勾选 Malloc Stack Logging：



这里的 All Allocation and Free History 指记录所有已分配对象，包括已释放的内存对象；Live Allocation Only 会从其历史中删除已分配的对象。日志记录数据可能会占用更多内存，但它对于调试碎片化等问题非常有用。在大多数情况下，Live Allocation Only 是推荐的选项。

如果没有从 Xcode 启动，也可以设置环境变量。查看 malloc 手册页面，了解一些额外的内存数据录制模式。

使用 Xcode Memory Debugger 导出。在 Xcode 的 File 选项下，导出 memgraph 。Xcode 使用 memgraph 的文件格式来储存应用程序的占用信息，导出 memgraph 文件可以结合命令行工具进行分析。

Xcode 运行起 app 之后，在调试栏点击 Debug Memory Graph ，这是 Xcode 会捕获当前 app 的内存快照，此时你可以很方便的查看内存中的存活对象，以及从 app 启动到此刻产生的内存泄露（紫色的叹号代表内存泄露），你可以灵活的选择展示当前内存内所有的存活对象，内存泄露的对象，也可以屏蔽系统的存活对象只关注当前工程调用产生的对象，或者是基于上述的选择，筛选指定类型对象。筛选之后，你可以看到当前类型对象有多少个，点击某个对象可以查看它的引用关系，右侧的 inspectors 还会展示当前对象的详细信息，比如占用大小，调用堆栈等。




在上面我们已经了解了 Xcode 内置的可视化工具，虽然可视化工具已经能够直观的表现我们想要了解的内存占用信息，但是在终端中不仅可以灵活地利用各种命令和 flag 突出我们想要的内容，更可以快速的实现信息查找和文本化交互。在了解内存问题分类之前我们先简单的了解下四种常用的命令行工具。

vmmp

vmmap 能够打印出进程信息，所有分配给该进程的 VMRegions 以及 VMRegion 的种类、内存占用信息等内容。利用 --summary 则能够根据不同的 region type 打印出详细的内存占用类型和信息。这里需要注意的是 SWAPPED SIZE 在 iOS 上指的是 Compressed memory size 且其值表示压缩前的占用大小。

leaks

leaks 追踪堆中的对象，打印出进程中内存泄露情况、调用堆栈以及循环引用信息。利用 --traceTree 和指定对象的地址，leaks 还能以树形结构打印出对象的相关引用。

heap

heap 会打印出所有在堆上的对象信息，默认按类数量排序，也可以通过 -sortBySize 按大小排序，对于追踪堆中较大的对象十分有帮助。找到目标对象后，通过 -address 获得所有/指定类的地址，继而可以利用 malloc_history 寻找其调用堆栈信息。

malloc_history

malloc_history App.memgraph --fuStacks [address]
使用上述命令能够获得我们知道地址的对象的调用堆栈信息，它能够得到的比 memory inspector 中 Backtrace 更加详细。但是需要开启 Dignostics 中的 Malloc Stack 选项，才能通过 malloc_history 获得 memgraph 记录的调用堆栈信息。


排查步骤：

从leaks看起
leaks --referenceTree --groupByType App\[1286\]_post.memgraph

从summary看起
vmmap -summary iOSXCTestApp\[871\].memgraph
heap -addresses=_NSZombie_CFString[100k-] App\[1286\]_post.memgraph
> heap --addresses=_NSZombie_OS_dispatch_data\[400k-\] -diffFrom=App\[1286\].memgraph App\[1286\]_post.memgraph 
leaks --traceTree=0x117098000 App\[1286\]_post.memgraph
malloc_history -fullStacks App\[1286\]_post.memgraph 0x1248b4000




### 四、总结

总结之前再介绍Xcode两个相关的能力，Analyze he XCTest。


#### Product->Analyze
Product 中的静态分析主要分析以下四种问题：

a.) 逻辑错误：访问空指针或未初始化的变量等

b.) 内存管理错误：如内存泄漏等

c.) 声明错误：从未使用过的变量

d.) Api调用错误：未包含使用的库和框架

注意使用静态分析是基于编译器的静态检查，而 Objective-C 是具有相当强大的动态性，所以静态分析能够检查出一些内存泄露问题，一些动态执行引起的内存泄露需要其他工具来检查。

优势：静态分析是基于编译器的静态检查，且检查会涵盖多种问题的检查。

短板：静态检查本事是基于静态的检查，对应 Objective-C 这种动态性语言的检查具有一定的局限性。


#### XCTest

开发一个新功能之后，用 XCTest 写一个性能测试来监测内存,和其他系统指标。为每个测试设置 baseline。然后用测试来捕获回归，并使用收集到的 ktrace 和 memgraph 文件进行调查。

#### 常见内存问题
- 如果你使用 unsafe 类型，确保你会释它。
- 同时也要注意代码中的循环引。
- 找到一种方式来减少你的堆分配，你可以缩小它们, 并且尽量把持有它们的时间变的更短。或者完全取消不必要的分配。
- 确保把碎片化问题牢记在心，创建的对象要尽量相邻并且具有相似的生命周期。
- 使用这些最佳实践和 XCTest 工作流，您将能够检测、诊断和修复应用程序中的内存问题。

### 总结

上边的文章大多搬运自WWDC “2022/调试iOS游戏内存” 和 “2021/检测和诊断 App 内存问题”。

欲善其事，必利其器。通过两个Section的介绍，能了解到IDE提供的能力，将有助于更快准确定位问题。一点建议，可以多使用 Xcode 的 Debug Memory Graph，发现开发过程的内存问题，将性能优化做到平时。

-------相关链接 

调试iOS游戏内存：
https://developer.apple.com/videos/play/wwdc2022/10106
https://xiaozhuanlan.com/topic/4258936701
https://developer.apple.com/documentation/Xcode/analyzing-the-performance-of-your-shipping-app


检测和诊断 App 内存问题
https://developer.apple.com/videos/play/wwdc2021/10180
https://www.codingsky.com/doc/2021/12/31/671.html


Leak和iOS堆内存碎片化及如何定位优化：
https://blog.csdn.net/u014600626/article/details/122263315