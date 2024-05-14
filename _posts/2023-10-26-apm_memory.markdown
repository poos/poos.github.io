---
layout:     post
title:      卡顿、内存、wakeup监控 - 2
subtitle:   小全的内存监控
date:       2023-10-26
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Summary
- APM
- Memory
---

# 内存监控

app的不当内存使用，一般不会导致应用在某个特定的点崩溃，但是会在某个特殊场景下导致应用被系统清除。


## 分析内存问题：

内存问题归类：

1、内存泄露，如阶梯一般上升
2、内存堆积，随使用时间累积上升，没有明显阶梯
3、资源过大，内存瞬间突增
4、持续高位，虽然不会立即oom，但是在后台，很容易被系统清理

前三个易发生前台OOM。
 - 其中1，2一般不会在特定业务场景发生FOOM，更具有隐蔽性。
 - 3，一般会在特定的加载资源等场景出现问题，产生FOOM。
第四个易发生后台OOM（BOOM）。


## 数据收集

### 内存泄露
对于内存泄露，页内一般采用页面级的监控，可以在页面销毁后，检查页面以及页面vc的属性是否被成功释放。常见的 MLeakFinder，Doraemonkit。

通过hook dismiss，pop，popTo等方法，在页面退出时候进行添加检查list(weak)，数秒之后检查，list里面的childVC，oresentedVC，views，touchs是否成功释放。

另外，可通过 FBRetainCycleDetector 检查是否有循环引用的情况发生。
```c
            Class FBRetainCycleDetectorClass = NSClassFromString(@"FBRetainCycleDetector");
            id detector = [[FBRetainCycleDetectorClass alloc] init];
            [detector addCandidate:object];
            NSSet *retainCycles = [detector findRetainCyclesWithMaxCycleLength:20];
```
同样的，根据Xcode的 memory graph，Profile-Leak等，可以方便的定位内存问题。

> DoraemonKit 里面的一些模块化做的还是不错的。

### 内存堆积
内存堆积其实是个比较难处理的问题，对于缓慢堆积的情况，最有效的方式可能是通过在内存较大的情况下，dump一下当前app的内存，然后进行分析。分析出头部的问题进行处理。涉及几个点：

1、大内存时候进行dump内存
2、分析最占用内存的头部问题
3、记录内存采集的堆栈

这部分其实也有一些开源方案，例如 Matrix：
> 内存记录的方案在17年之前主流的方案是使用 fishhook 工具 hook malloc/free 等接口监控堆内存分配，每隔1秒，把当前所有OC对象个数、TOP 200最大堆内存及其分配堆栈，用文本 log 输出到本地。记录上报。不过此收集方案又一些问题，例如记录不精细，hook 方案能力有限，仅能 hook 自身 app 的 C 接口调用，对系统库不起作用；打 log 间隔不可控，过长可能丢失中间峰值情况，间隔过短会引起耗电、io频繁等性能问题。

> 新的方案核心是参考 Xcode 的 Instruments 工具中 malloc stack 的实现原理，参考 malloc/free、vm_allocate/vm_deallocate 等接口，记录存活对象内存分配/释放等信息，尽量实现和 Instruments 相似的结果。
> 因为 malloc/free、vm_allocate/vm_deallocate 是私有 api，无法带上 App Store。所以采用修改 malloc_default_zone 函数返回的 malloc_zone_t 结构体里的 malloc、free 等函数指针，也是可以监控堆内存分配，效果等同于 malloc_logger；而虚拟内存分配只能通过 fishhook 方式。

### 资源过大

加载到内存的对象过大，属于大对象监控。

我们app的大对象吗监控是基于图片的大对象监控，监控修改SDWebImage的delegate回调，在合适的时机进行监控。还有系统image加载方法。主要是这些：

```c
// sd web image
imageManager: didLoadImageForURL:

// uikit image
imageNamed:inBundle:compatibleWithTraitCollection:
imageWithContentsOfFile:
```

#### 大对象监控
以上监控主要针对图片资源，实际上还可以监控其他资源，或者直接监控大的内存申请。

可改动 Matrix，记录大对象的 alloc 时机，进行上报。


### 持续高位

我们进行了页面 init - dealloc， 多次 appear dismiss 时候的内存变化（包含峰值变化），衡量页面的内存使用。另一方面，我们记录了app使用时长的内存记录。通过两个维度，来分析和定位没那么重要的持续高位内存占用（一般引起BOOM）。

### OOM上报
除了对以上内存使用特点的定向处理，还有一个必不可少的采集，就是app发生OOM时候的一个异常上报。

因为OOM需要随时记录alloc堆栈，对象，势必对性能的影响，一般采用灰度的形式进行。那么另外一些用户发生OOM如何处理呢。

我们使用了memory warning进行了能力补足。通过memory warning count进行一个简单count计数算法（当前），通过检测memory warning，但是3s后app还活着，即认为发生了OOM，用以补足剩下的统计数据。

```c
    // plus & storage
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), _storeQueue, ^{
        // minus & storage
    });
```

## 数据上报

像其他apm指标一样，主要分统计数据，异常数据。

**统计数据**
- 基于分钟内存占用，基于页面内存（及峰值）占用
- 页面使用内存过大的异常告警。

**异常数据**
- 内存泄露
- 大对象
- OOM，附加记录当前时刻内存对象list、对象内存使用、对象alloc堆栈
- 基于 memory warning 的 oom

## 总结

内存问题一般会比较隐晦，发生了也未必能真正定位到问题，但是内存是性能监控，甚至是Crash监控绕不开的一个话题。为了定位问题，不仅需要线下的memory graph，内存泄露检测。也需要线上的OOM监控，真实记录当时用户的现场，不至于面对用户发生的问题，一无所知，无从下手。


参考

[快影iOS端如何实现OOM率下降80%+](https://mp.weixin.qq.com/s/IvATFGU_bOph-WX5ZYLYew)
[火山引擎/应用性能监控全链路版/App端监控/内存优化](https://www.volcengine.com/docs/6431/68858)
[iOS学习——内存泄漏检查及原因分析](http://t.zoukankan.com/mukekeheart-p-8144742.html)
