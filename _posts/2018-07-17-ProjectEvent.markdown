---
layout:     post                       # 使用的布局（不需要改）
title:      Swift项目中合适的打点方案                 # 标题
subtitle:   动态hook打点？重写控件？手动打点？      #副标题
date:       2018-07-17                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 设计
---

## Swift打点方案



首先介绍一下打点的业务需求：点可能是展示、控制、服务器返回结果

基础计数点、入屏点击、页面时长、鱼骨模型（1点进，多点出）



> 结合网上各个文档，总结一下Swift可用的打点方案。总结测试打点的方法


如果想直接看本文实现方式可以跳过三大类介绍

### 三大类的方案简介

#### 方案一：动态hook打点

[Aspects源码解析](https://www.jianshu.com/p/2c93446d86bd)

在OC可以使用Aspects动态的Get到在某个函数或者方法执行前后点，所以可以方便的使用这个运行时特性做一些打点操作

使用这个特性，你可以生成一个类似与下边的表，甚至通过服务端配置可以**随时调整打点**：
```
ViewController,click,buttonAction,trackName,trackkeys
ViewController,click,buttonActionNext:,trackName,trackkeys
TableViewController,tableSelect,tableView:didSelectRowAtIndexPath:,,
ViewController,lifeCycle,viewDidLoad,,
TableViewController,lifeCycle,viewDidLoad,,
ViewController,lifeCycle,viewWillLayoutSubviews,,
ViewController,other,buttonActionNext2:,,
,,,,
```

**那么在Swift下是否好用呢**，我尝试写了一个demo在[**干货 poos/AspectOrientedProgramming**](https://github.com/poos/SwiftEFarm)。完全是不侵入业务的，甚至 **直接移除** TrackManager 这个文件夹原项目仍然可以很好运行。

1. Demo中对常用的buttonClick，tableSelect，lifeCycle做了监听，在相应的地方会触发打印

2. Demo中使用cvs来配置打点的类，方法名，参数等

**使用的结果呢**： 它在Demo中可以完美的检测和打点.....

**然而...**：将它运行到Swift项目中，你就会发现问题：我的大部分函数和方法不是 **@objc** 的，大部分的swift方法并不支持OC的运行时，而且 **在Swift4.0之后不能简单的@objc标志类的动态性** ，这意味着你需要将所有的相关方法使用 **@objc** 暴露出来。


**方案一优劣简介**
```
- oc下的动态埋点方案，使用csv文件标志要打的的类和方法，动态hook方法打点
动态埋点，运行时，在方法执行前后插入打点方法
- oc下可解决部分打点，其他打点可以通过扩展类，传入上下文，造专属的打点方法


然而
- 4.0之后不能简单的@objc标志类（Swift下致命缺陷）
```

#### 方案二：重写所有常用的模块，赋予组件值并且打点

正如字面的意思，所有的UI控件和以及常用的Contrl操作均封装，这样在加载控件或者执行操作时候会自动打点


这个就没有去写demo了。不过可以用脚分析一下优劣


**方案二优劣简介**
```
不论oc和swift都可以使用，组件还可以做其他事情，例如数据即控件

打点会繁而具体，大量冗余数据打点
现有工程改造困难
基础组件拖开发进度
其他点可能需要处理
```
#### 方案三：手动打点

既然前两条路都堵死了，那么只能手动去打点了，不过手动打点仍然还是有很多地方优化：

1. 一些模块封装打点：生命周期，入屏点击这些可以单独分离出模块

2. 一些属于操作的点可以整合

3. 可以使用链式的方案减少打点的痛苦😄（ex： **EventManage.share.event(.click).page(.home).push()** ）


**方案三优劣简介**
```
相当准确，点点命中要害

但是每个地方都要写
```

### 手动打点的优化

1. 标记版本号来对应打点代码，日期对应本地的类名

```
base v12    select v22      page v54    180707
```
2. 制定大量枚举/使用plist或者csv来做键值映射，方便管理

```
page字典对应页面的enum
event 的 enum
key 的 enum
```

3. 使用链式编程

```swift
class EventManage {
    //事件名
    enum EventId: String {
        case tapAction
        ...
    }

    ...

    //参数
    private var parameters: [ParamKey: String] = [:]

    ...

    //方便调用的方法
    func event(_ event: EventId) -> UmengManager {
        parameters[.event] = event.rawValue
        addTrackerId(event)
        return self
    }

    ...

    func page(_ page: PageEnum) -> UmengManager {
        parameters[.page] = page.rawValue
        addTrackerId(event)
        return self
    }

    ...

    func push() {
        NSlog(...)

        push...//提交的代码

        parameters.removeAll()
    }
}

```

使用了上边的框架之后，在打点时候就可以方便的打点了

```
EventManage.share.event(.click).page(.home).push()

```


4. 特殊打点创建单独的类管理

入屏点击使用一个类来管理，新创建类创建一个新实例就好

上拉下拉等同上

一些业务相关需要传入page的，维护一个正在显示的page参数即可，只需要在 vc 父类更新 page 。



**打点嘛，重要的是管理和分配**，通过这一系列的手段就可以确保打点无误了


### 检查已经打的点

**众里寻她千百度，蓦然回首，那人却在灯火阑珊处**

当你苦苦寻找如何直接实时显示打点，让测试方便测试的时候，有没有发现这个东西
**Console**或者叫**控制台**，这是mac自带的一个程序，通过**NSLog**即可将Xcode的输出显示到终端，啰嗦一句，swift 中 print 是不可以的，所以你需要使用 **NSLog**，猜测也是跟OC运行时的关系吧

~~--懒得传图，从网上找个图片--~~
![img](http://images.macx.cn/forum/201202/27/1206273lioivz2pvlnkpz3.jpg)


### 参考资料：

```
github项目：

AOPTrack
SwiftAspects
```

#### 推荐一下自己写的最新测Demo

[**干货 poos/AspectOrientedProgramming**](https://github.com/poos/SwiftEFarm)
