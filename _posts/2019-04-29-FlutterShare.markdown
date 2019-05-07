---
layout:     post                          # 使用的布局（不需要改）
title:      Flutter 技术分享 - 了解            # 标题
subtitle:   跨平台开发方案介绍，Flutter，Dart，Widget，Stream    #副标题
date:       2019-04-29               # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- Flutter
- 总结
---

整理了一些资料用来分享给小伙伴们。分享内容大概4部分。

- 跨平台方案的简介

- Flutter 和 Dart 的 简介

- 安装 和 Flutter Demo 简介

- Stream 介绍


##  跨平台开发方案

#### 为什么要跨平台开发

因为原生开发有一些优劣：

**优点：**

1.可获取平台全部开放功能，比如摄像头，蓝牙等;

2.用户访问应用的感受通常是速度快、性能高、体验好;

**缺点：**

1.为特定平台开发，综合成本高，不同平台维护需要人力成本;

2.动态化能力弱，大多数情况下，新功能更新只能发版;


如果使用跨平台开发就可以避免一些问题，同时往往能节省人力，节省开发时间。

#### 有哪些跨平台开发方案

可归为以下3个阶段

- 1.青铜时代。采用 WebView 容器(广 义) + JavaScript Bridge ;
- 2.白银时代。用JavaScript开发，绘制交由 Native 接管（JavaScript VM + Native UI）;
- 3.黄金时代。某开发语言 + engine（每个平台单独的绘图引擎）。


第一个阶段，Web与Native分离, 使用 JavaScript Bridge 协议桥接 WebView（具体来说就是在网页加载完成后插入 js 代码，实现网页与原生的通信）。这个阶段的跨平台只是伪命题，实际上就是 WebView。


第二个阶段，使用虚拟机平台将 js 代码转换为原生控件。具体来说一版使用各个开发语言，（RN-javaScript，Weex-Vue）各种语言，打包成 JS 代码，根据提供的（framework，core，Bridge），最终映射成为操作系统原生库展现 app。


第三个阶段，搞一个系统，对接操作系统 sdk 底层。具体来说是一个自绘引擎，使用开发语言（例如 dart）通过自绘引擎（针对不同的操作系统，开发专用的自绘引擎），直接在操作系统绘制图形界面。这样来实现顶层统一，产物不同。因为不使用系统 sdk 提供的组件，所以比较自由，官宣可以做到一个像素点可控。同时因为直接绘制界面，流畅性非常高。

不过也有缺点：涉及到平台层时还是需要原生开发介入实现相应插件。

#### 对比第二和第三方案

![img](https://poos.github.io/img/flutter01.png)

虽然这是比较早的时候的对比，但是可以看出，第三方案跟第二方案对比还是很有优势的。

## Flutter 和 Dart 方案介绍

进入主题。

### Flutter 


#### 有多少产品在使用它


alibaba 闲鱼，  百老汇音乐剧，  tencent NOW Live直播项目，  Abbey Road的Topline应用程序可帮助艺术家录制歌曲，  JD 金融项目，  Google ad 等

在官方宣传网址 [itsallwidgets.com](https://itsallwidgets.com/) 上还有非常多的产品用到了Flutter。

网站上还有很多开源的项目，例如 Sandwhich三明治 。


在 github 上的项目个数大概如下：

|preject|个数|
|---|---|
|flutter| 3w|
|RN | 10w|
|OC | 12w|
|swfit | 14w|
|android | 85w|

#### 社区如何

这么多人使用一方面反映了google 老大哥的带动效应。有非常多的社区，来发展这个比较年轻的语言。

|flutter|社区||flutter|社区|
|---|---|---|---|---|
|flutterchina.club| 中文网站||stackoverflow||
|GITT ER | github的即时聊天||Google Groups|Google网上论坛|
|Medium | 博客平台||slack|协作聊天|
|YouTube | 视频||Discord|实时通话软件|

#### 历史

当前规模的现状，Flutter 用了很长时间吗？

- Flutter的第一个版本被称为“Sky”，运行在Android操作系统上。它是在2015年Dart开发者峰会上亮相的，其目的是能够以每秒120帧的速度持续渲染。

- **Beta1版本于2018年2月27日在2018 世界移动大会公布。**

- **1.0版本于2018年12月5日(北京时间)发布。**

- **2019年2月发布1.2版本。目前最新为1.5版本。**


Beta1版本是从2018年才开始发展，到现在，基本才只有一个多年头。如果从1.0版开始算起，正式版的话才半年，真的非常年轻。

#### 发展 - Roadmap 2019

***在 github 上，flutter 更新频率非常高，最新版已经到1.5+。对于2019年的 flutter 计划，官网上也做出了规划。***

- Fundamentals 基本（bug，性能，测试，api文档）

- Ease of Adoption （模板，文档，组件）

- Ecosystem（C/C++，生态插件，通知，数据存储）

- **Support for platforms beyond mobile（键鼠，Web，macOS，Windows）**

- Tooling（开发工具的改进）

- **Change（在2019年删除了对动态更新的支持）**


最新的master 分支已经加入了macOS 的代码。

有三个原因使得 flutter 对热更新暂时不准备付出太多的精力：因为系统平台（iOS、Android）对热更新的态度尚且不明确；因为热更新使用下发代码的方式更新，所以不可避免存在安全问题；第三是还没有一个统一平台来做代码下发这件事，如果实现不得不由各个项目开发团队自己搭建。


### Dart

Flutter 最终选择了 Dart 作为开发语言，Dart 是什么，为什么选择 Dart 呢？

#### Dart简介


**时间线如下**

- 2011年10月 - Google Chrome（V8引擎）

- 2015年被移出 Chrome

- Sky 诞生，服务器，前端全面发展

- 2018年8月 发布重构版 Dart 2.0

Dart 的历史也是有波折的。Dart 最早在2011年被 Google Chrome（V8引擎）团队创立，使用于 Web 网页，立志要取代 JavaScript 这个繁重的 Web 应用。

然而，2014-2105年，由 NodeJS 开源项目引领，诞生了一系列明星项目如React、React-Native、Vue 等基于 JS 的前端开发框架，当时还流行一句话，“凡是能被JS实现的，最终都要被JS实现”。

于是 Dart 在2015年被移出 Chrome。

同时在2015年Dart开发者峰会上，Sky 诞生，运行在Android操作系统上。其的的是能够以每秒120帧的速度持续渲染。它也是 Flutter 改名之前版本。像 Javascript 一样，Dart在服务端和浏览器前端也在不遗余力的发展：服务端可以编写命令行程序，前端可以编译成Javascript运行在浏览器中。Google的另一个前端大杀器Angular也有对应的Dart版本。在Google的未来操作系统Fuchsia中，Dart被指定为官方的开发语言。这些发生在2015-2017年。

船小好调头，也正是使用者较少，没有历史包袱，Dart的创造者们总结了Dart1.0版本的优缺点，决定打造一个运行更快、更加安全的强类型语言Dart2.0。2018年8月正式发布 Dart 2.0 ，目前稳定版为 2.2(19年02月27发布)。


**目前，Dart 在移动端有 Flutter，web 有 Dart for the web，服务端有 Server-side Dart。**

#### Dart 适合移动端吗

目前看比较适合。

- 1) Dart是AOT (Ahead Of Time)编译的，可编译成快速、可预测的本地代码。
- 2) Dart也支持JIT (Just In Time)编译，开发周期快（状态热重载）;
- 3) Dart可以更轻松地创建以60fps运行的流畅动画和转场。它的启动速度也快得多。
- 4) Dart使Flutter不需要单独的声明式布局语言，如JSX或XML。
- 5) 开发人员发现Dart特别容易学习，因为它具有静态和动态语言用户都熟悉的特性。

**Dart 支持两种编译模式，AOT 和 JIT**。开发环境使用 JIT，借助虚拟机，随时修改代码，随时看到效果，可以实现亚秒级的热重载。发布环境使用 AOT，打包出经过各种编译优化的包，不论是在安装包安装速度，和包内程序运行速度都要很多优势。

Dart 最初就被设计目标就是**持续渲染，不卡顿**。**1）** AOT 编译就像原生代码的 AOT 编译，与“JavaScript桥梁”方案相比，天然少了桥的过程。**2）** Dart 在 抢占式调度、时间分片和共享资源 方面权衡诸多条件，选用最适合界面特点的方案：单线程，程显式让出（使用 async/await、Future 和 Stream）CPU。**3）** 访问共享资源（内存）的情况下，在很多语言中都需要使用锁。但在回收可用内存时， 锁会阻止整个应用程序运行
。Dart使用先进的 **分代垃圾回收和对象分配方案**，该方案对于分配许多短暂的对象（对于Flutter这样的反应式用户界面来说非常完美，Flutter为每帧重建不可变视图树）都特别快速。

Dart的另一个好处是**热重载**，Flutter不需要从程序中拆分出额外的模板或布局语言，如JSX或XML，也不需要单独的可视布局工具。借助热重载，可视化编辑就不需要了。体验下来，开发中，**从页面开始创建，到添加资源，请求，封装模型，修改 UI，添加事件，页面最终完成，你都不需要重新编译，一切都是所写即可见**。


##  安装 和 Flutter Demo 简介

这里就不多讲了，参考之前的博客。

#### 安装 和  Demo

[Flutter 《一》 安装和配置，QA](https://poos.github.io/2019/03/25/FlutterInstall/)
介绍了安装问题；一些官方的 QA。


[Flutter 《二》项目 Demo 运行，Dart 语言简单学习](https://poos.github.io/2019/03/27/FlutterDemo/)
几个简单的 Demo。速学 Dart 语法。


#### Wdiget 和 布局


[Flutter 《三》常用组件 & Week Video](https://poos.github.io/2019/03/28/FlutterWidget/)
一切皆是 Widget，Widget 认识和学习。


布局方面跟之前用到的 Texture 盒式布局异曲同工，所以没有单独写博客。其实上手还是比较简单的，有兴趣参考[Texture(ASDK)的理解和使用](https://poos.github.io/2018/08/08/Texture/)

## Stream

顾名思义，Stream 就是流的意思，表示发出的一系列的异步数据。可以简单地认为 Stream 是一个异步数据源。它是 Dart 中处理异步事件流的统一 API。

使用数据流可以更好管理项目代码，设计项目模式，节约设备资源(例如持有大文件)等。

Dart 中，集合（Iterable或Collection）表示一系列的对象。而 Stream （也就是“流”）也表示一系列的对象，但区别在于 Stream 是异步的事件流。比如文件、套接字这种 IO 数据的非阻塞输入流（input data），或者用户界面上用户触发的动作（UI事件）。

集合可以理解为“拉”模式，比如你有一个 List ，你可以主动地通过迭代获得其中的每个元素，想要就能拿出来。而 Stream 可以理解为“推”模式，这些异步产生的事件或数据会推送给你（并不是你想要就能立刻拿到）。这种模式下，你要做的是用一个 listener （即callback）做好数据接收的准备，数据可用时就通知你。

### Future

第一个概念叫 Future。
Future 表示稍后获得的一个数据，所有异步的操作的返回值都用 Future 来表示。但是 Future 只能表示一次异步获得的数据。
而 Stream 表示多次异步获得的数据。

```dart
  Future<T>
  Stream<T>
```

### 获得 Stream

**数据流的发出点**

将集合（Iterable）包装为 Stream 
Stream 有3个工厂构造函数：fromFuture、fromIterable 和 periodic，分别可以通过一个 Future、Iterable或定时触发动作作为 Stream 的事件源构造 Stream。


使用 Stream 读文件 
读文件的方式有多种，其中一种是使用 Stream 获得文件内容。


下边示例一个集合转换为 steam 并进行后续操作的例子。

```dart
  //获得 Stream
  var data = [1, 2, 3, 4];
  var stream = new Stream.fromIterable(data);
```

### 订阅 Stream

**数据流的接受点**

```dart
//订阅
stream.listen((e) => print(e));
```

#### 订阅和高级订阅（onData、onError和onDone）

```dart
  //订阅 Stream
  stream.listen((e) => print(e), onDone: () => print('Done'));
  //1 2 3 4 Done

  //订阅管理 StreamSubscription
  var sub = stream.listen(null);
  sub.onData((e) {
    if (e > 2)
      sub.cancel();
    else
      print(e);
  });
  sub.onDone(() => print('done'));
  //1 2
```
区别在于高级订阅返回一个 **StreamSubscription<int>** 的对象，这个对象有 **onData，onDone，onError** 方法。

#### 单订阅模式 和 多订阅模式 

```dart
  //多订阅（broadcast）
  var bs = stream.asBroadcastStream();
  assert(bs.isBroadcast == true);
  bs.first.then(print);
  bs.last.then(print);
  //1 4
```
上边的 stream 只支持 一个数据流接受，接受完毕就不会再发出数据流了。 可以使用 **asBroadcastStream()**方法返回一个同样类型（Stream<T>）的 stream，这样就可以多次订阅了。


#### Stream 的集合特性

Stream 和一般的集合类似，都是一组数据，只不过一个是异步推送，一个是同步拉取。所以他们都很多共同的方法。

Stream 还提供一些数据转换方法。还可以自定义转换方法（StreamTransformer）。

```dart

  //常见集合方法
  //stream.first.then(print);
  //stream.firstWhere((e) => e > 3, orElse: () => 0).then(print);
  //...

//last, length, isEmpty, any, every, contains, elementAt, where, skip, take,

//map, reduce, expand, toList, toSet

//StreamTransformer
class MyTransformer extends StreamEventTransformer {
...
}
```



### 最后

这篇文章先到这里了。整理分享还是要注意些问题：既要广泛了解，让大家有个认识；又不能太过深入，否则大家不能理解，时间也不允许。不像编程，只要解决问题就好了。

还有下一篇，包含 Flutter 的 Stream，state 更新方案，原生项目和 Flutter 项目互调等精彩内容，不容错过。




### 参考

[Flutter之StatelessWidget StatefulWidget和Element关系浅析](https://blog.csdn.net/chunqiuwei/article/details/87855312)


[谷歌推出简化APP开发：Google Flutter的功能和优势](https://www.kingwins.com.cn/content-4194.html)


[理解Dart 异步事件流 Stream](https://blog.csdn.net/u012995888/article/details/82740252)

[3.2 类的继承与重写 – 《简单易懂的Dart》](https://www.blackglory.me/straightforward-dart-3-2/)


[理解Dart 异步事件流 Stream ](http://blog.sina.com.cn/s/blog_12d64892f0102vtk9.html)


[AOT,JIT区别，各自优劣，混合编译](https://blog.csdn.net/h1130189083/article/details/78302502)


[编译型与解释型、动态语言与静态语言、强类型语言与弱类型语言的区别](https://www.cnblogs.com/dzhanjie/archive/2011/07/07/2100340.html)


[Flutter笔记(二)关于Dart和Weight](https://www.jianshu.com/p/188c15064a1f)


[为什么Flutter会选择 Dart ？](https://www.colabug.com/2475126.html)


[Dart 语言的介绍](https://www.yoytang.com/dart-intro.html)


[using-streams-in-flutter](https://medium.com/@ayushpguptaapg/using-streams-in-flutter-62fed41662e4)


[官网 API 介绍](https://www.dartlang.org/tutorials/language/streams)


[理解Dart 异步事件流 Stream](http://blog.sina.com.cn/s/blog_12d64892f0102vtk9.html)