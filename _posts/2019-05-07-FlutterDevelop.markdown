---
layout:     post                          # 使用的布局（不需要改）
title:      Flutter 技术分享 - 深入            # 标题
subtitle:   State Manage 五大方案，打包和自动化，与原生项目互嵌    #副标题
date:       2019-05-07                # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- Flutter
- 总结
---



系列一共2篇，第一篇在这里
[Flutter 技术分享 - 跨平台开发方案，Flutter，Dart，Widget，Stream ](https://blog.csdn.net/chunqiuwei/article/details/87855312)
，第一篇属于大家都可以听听的，第二篇就比较深入一些，讲一讲前端的设计模式，Flutter与原生项目的互相调用。

- State Manage 五大方案

- 打包和自动化

- 与原生项目互嵌

## State Manage 五大方案

State Manage 是 Flutter 中 UI 更新的独有方案，但是同样可以类比到其他移动端，前端等方面。其实也是对应界面更新的几个方案。

- setState
- inheritedWidget & Scoped model
- redux
- Bloc / Rx
- MobX

第一个为需要更新则准备好数据，然后使用 setState 进行界面的更新。缺点是如果相隔的层比较远就没法使用。

第二种使用顶级节点保存数据，然后在子节点读取和操作数据。

第三种 redux，独立出一个 **只读的 State**，State 来自一个 **单一的数据源**，**接收事件，改变自身**。

第四种 Bloc 类似 MVVM，通过协议约束，接受事件，输出状态。与上个不同点之一是有数据流动，一切都是以数据流的形式传播。

第五种 MobX 中，Reactions(反应) 对应 界面，**Reactions(反应)->Actions(动作)->Observable state(可观察的状态)->Reactions(反应)** 形成一个回路。，


**其实除了前两种方案，三四五 的方案都是使用了分层。**


#### setState

点击会把相应节点更新。但是如果点击需要操作多个节点，跨级节点就无法使用。

![img](https://poos.github.io/img/flutter_state_1.png)

#### inheritedWidget & Scoped model

针对一的问题，解决方案之一是在顶级节点保存共有数据，所有需要数据的节点就去顶级节点拿。

![img](https://poos.github.io/img/flutter_state_2.png)

#### redux

[redux 的三大原则](https://www.redux.org.cn/docs/introduction/ThreePrinciples.html) ：单一数据源，State 是只读的，使用纯函数来执行修改。

redux 讲究独立出一个类来保存状态，提供入口事件来更新状态。

![img](https://poos.github.io/img/flutter_state_3.png)


#### Bloc / Rx

[bloc 核心概念](https://felangel.github.io/bloc/#/coreconcepts) ：活动，状态，改变，流。

bloc 中心接受事件流，输出状态流。UI 根据状态流更新。UI 事件发起状态流。

![img](https://poos.github.io/img/flutter_state_4.png)

#### MobX

[MobX 概念与原则](https://cn.mobx.js.org/intro/concepts.html) ：State(状态)，Derivations(衍生)，Actions(动作)。

MobX 支持单向数据流，也就是动作改变状态，而状态的改变会更新所有受影响的视图。

虽然与上边类似，但是 **Derivations(衍生)** 是关键的一步，源自状态并且不会继续可变的东西都是衍生。例如: 用户界面；衍生数据，比如剩下的待办事项的数量；后端集成，比如把变化发送到服务器端。

MobX 区也分了两种类型的衍生，并且提出了黄金法则: 如果你想创建一个基于当前状态的值时，请使用 computed。

- **Computed values(计算值)** - 它们是永远可以使用纯函数(pure function)从当前可观察状态中衍生出的值。
- **Reactions(反应)** - Reactions 是当状态改变时需要自动发生的副作用。需要有一个桥梁来连接命令式编程(imperative programming)和响应式编程(reactive programming)。或者说得更明确一些，它们最终都需要实现I / O 操作。


使用上通常三步走：**定义状态并使其可观察，创建视图以响应状态的变化，更改状态**。

![img](https://poos.github.io/img/flutter_state_5.png)


### 总结一下

做非常简单的变化就使用 setState；

依赖同一数据源，就使用 inheritedWidget & Scoped model；

redux 更适合用户数据中心等设计；

Bloc / Rx 更适合页面设计；

MobX 是一种比 Bloc / Rx 消耗资源更小的设计，不过意味着更多的代码和逻辑。适合页面，也适合一个数据操作中心。例如图片滤镜等设计。


## 打包和自动化

## 与原生项目互嵌