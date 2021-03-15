---
layout:     post
title:      Flutter 技术分享 - 深入
subtitle:   State Manage 五大方案，打包和自动化，与原生项目互嵌
date:       2019-05-07
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Flutter
- 总结
---


## Demo

**[本文所有代码 Demo 地址](https://github.com/poos/BlogDemo)**


系列一共2篇，第一篇在这里
[Flutter 技术分享 - 跨平台开发方案，Flutter，Dart，Widget，Stream ](https://poos.github.io/2019/04/29/FlutterShare/)
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


简略来说：从上到下方案分别是：直接更新；根节点存储；独立 Reducer 中心处理；明显分层, 使用 Bloc 中心处理信息流；事件影响状态，状态改变一切。


**其实除了前两种方案，三四五 的方案都是使用了分层。**


#### setState

点击直接更新。

第一个使用 setState 某个点的更新。缺点是如果相隔的层比较多，需要操作多个节点，那就没法使用。


![img](https://poos.github.io/img/flutter_state_1.png)

#### inheritedWidget & Scoped model

根视图节点存储。

针对一的问题，解决方案之一是在顶级节点保存共有数据，所有需要数据的节点就去顶级节点拿。

其实 Flutter 中全局 Theme 的样式就是使用这种方式存储的。


![img](https://poos.github.io/img/flutter_state_2.png)

#### redux

独立 Reducer 中心处理。只读 State 存储。

Reducer 中心只是一些纯函数，它接收先前的 state 和 action，并返回新的 state。刚开始你可以只有一个 reducer，随着应用变大，你可以把它拆成多个小的 reducers。

整个应用的 state 被储存在一棵 state tree 中。好处是 state 可以从服务端读取；还有就是方便依托于 State 事情都会很容易，例如查询之前的状态等。

[redux 的三大原则](https://www.redux.org.cn/docs/introduction/ThreePrinciples.html) ：单一数据源，State 是只读的，使用纯函数来执行修改。

redux 讲究独立出一个类来保存状态，提供入口事件来更新状态。

![img](https://poos.github.io/img/flutter_state_3.png)


#### Bloc / Rx

明显分层, 使用 Bloc 中心处理信息流。

事件流通过 Bloc 中心，会产出 状态流。 UI 监听状态流更新。

首先定义事件和状态的枚举； 单个状态的转变是通过 Bloc 定义一些协议，在协议方法中处理状态的转变，异步返回新状态。

[bloc 核心概念](https://felangel.github.io/bloc/#/coreconcepts) ：事件，状态，转变，流，Bloc 中心和代理。

![img](https://poos.github.io/img/flutter_state_4.png)

#### MobX

事件影响状态，状态改变一切。

MobX 讲究  **任何源自应用状态的东西都应该自动地获得。**

事件去修改 state，导致 state 的更新。state 更新会影响所有相关的细小的事件，会产生一系列事件。

MobX 提出 **Derivations(衍生)** 的概念，用来描述 State 改变产出的一系列影响。（定义是，源自状态并且不会继续可变的东西都是衍生。例如: 用户界面；衍生数据，比如剩下的待办事项的数量；后端集成，比如把变化发送到服务器端。）

MobX 区也分了两种类型的衍生，并且提出了黄金法则: 如果你想创建一个基于当前状态的值时，请使用 computed。

- **Computed values(计算值)** - 它们是永远可以使用纯函数(pure function)从当前可观察状态中衍生出的值。

- **Reactions(反应)** - Reactions 是当状态改变时需要自动发生的副作用。需要有一个桥梁来连接命令式编程(imperative programming)和响应式编程(reactive programming)。或者说得更明确一些，它们最终都需要实现I / O 操作。

使用上通常三步走：**定义状态并使其可观察，创建视图以响应状态的变化，更改状态**。

[MobX 概念与原则](https://cn.mobx.js.org/intro/concepts.html) ：State(状态)，Derivations(衍生)，Actions(动作)。

![img](https://poos.github.io/img/flutter_state_5.png)


### 总结一下

做非常简单的变化就使用 setState；

依赖顶级节点，或者单一的的数据源，就使用 inheritedWidget & Scoped model；

redux 更适合用户数据中心等设计；

Bloc / Rx 更适合页面设计；

MobX 是一种比 Bloc / Rx 消耗资源更小的设计，不过意味着更多的代码和逻辑。适合页面，更适合点一个按钮，触发 n 多事件的场景。


## 打包和自动化

#### 打包
打包非常简单，但是还是提一下。

分别给 iOS 和 Android 配置相应的 icon，启动图（在各自应用文件中）。iOS 设置好证书，Android 设置好签名就可以打包了。

- iOS 进入 Xode，打包。

- Android 使用命令行 ***flutter build apk*** 即可。


#### 自动化

Flutter 可以使用 fastlane 进行持续交付。文档中介绍了本地和服务端部署的方法，就不多说了。

[使用带有Flutter的fastlane进行持续交付](https://flutter.dev/docs/deployment/fastlane-cd)


## 与原生项目互嵌

主要是 flutter 使用原生包；原生应用插入 Flutter 页面。

### 现有 Flutter 项目接入原生包

等价于，开发和使用本地 flutter package。

[开发包和插件](https://flutter.dev/docs/development/packages-and-plugins/developing-packages)

以 iOS 为例，分为3步。

- 创建 适用于 Flutter 的 package 包。使用命令行创建，会生成相应的文件目录，iOS 和 Android 的 package 文件在不同的地方，第二步制作好之后放到相应的地方即可。
- 制作 iOS cocoaPod 库，验证在原生项目是正常运行，然后可以放到指定位置。
- 发布 package 库，或者引用本地 package。

[使用cocoapods打包静态库](https://www.jianshu.com/p/9096a2eb2804)


**对于 Android 第二步相应调整即可。**

#### 包示例

**互嵌使用不同的消息通知的方式来区分调用的内容，示例如下**

```dart
//package lib/package.dart

import 'dart:async';

import 'package:flutter/services.dart';

class Package {
  static const MethodChannel _channel =
      const MethodChannel('package');

  static Future<String> get platformVersion async {
    //调用方法并且等待返回
    final String version = await _channel.invokeMethod('getPlatformVersion');
    return version;
  }
}
```

```dart
//package ios/Classes/Plugin.m

#import "LoginPlugin.h"

@implementation LoginPlugin
+ (void)registerWithRegistrar:(NSObject<FlutterPluginRegistrar>*)registrar {
  FlutterMethodChannel* channel = [FlutterMethodChannel
      methodChannelWithName:@"login"
            binaryMessenger:[registrar messenger]];
  LoginPlugin* instance = [[LoginPlugin alloc] init];
  [registrar addMethodCallDelegate:instance channel:channel];
}

- (void)handleMethodCall:(FlutterMethodCall*)call result:(FlutterResult)result {
    if ([@"getPlatformVersion" isEqualToString:call.method]) {
        //调用你需要调用的原生方法
        //resulut 返回你需要返回的信息
    } else {
    result(FlutterMethodNotImplemented);
  }
}
@end
```

#### 调用示例

首先引用这个包。使用本地包时候注意路径正确。

```shell
# pubspec.yaml
dependencies:
  package_name:
    path: ../package_name/
```
然后即可调用。

```dart
//some file
void _onPressed() async {
    var batter = await Login.getBatteryLevel;
    print(batter);
  }
```


### 现有原生项目嵌入 Flutter

[flutter/wiki/Add-Flutter-to-existing-apps](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps)

#### Android

- 创建 Flutter 包

- Host app requirements，即设置 compileOptions

- 设置 settings.gradle

```shell
cd some/path/
flutter create -t module my_flutter
```

```
compileOptions {
  sourceCompatibility 1.8
  targetCompatibility 1.8
}
```
```shell
// MyApp/settings.gradle
include ':app'                                     // assumed existing content
setBinding(new Binding([gradle: this]))                                 // new
evaluate(new File(                                                      // new
  settingsDir.parentFile,                                               // new
  'my_flutter/.android/include_flutter.groovy'                          // new
))      
```
```shell
// MyApp/app/build.gradle
:
dependencies {
  implementation project(':flutter')
  :
}
```


#### iOS

- 创建 Flutter 包

- 将包配置进入 Podfile，导入

- 设置项目 bitcode，项目 shell 命令

```shell
cd some/path/
flutter create -t module my_flutter
```

```shell
#Podfile, install
flutter_application_path = 'path/to/my_flutter/'
  eval(File.read(File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')), binding)
```

```shell
#TARGET -> Build Phases -> New Run Script Phase
"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" build
"$FLUTTER_ROOT/packages/flutter_tools/bin/xcode_backend.sh" embed
```


#### 通信方式

**Flutter 包的 main 入口通过传入的 message 区分调用不同的函数，返回不同的 widget。**

```dart
import 'dart:ui';
import 'package:flutter/material.dart';

// void main() => runApp(MyApp());

void main() => runApp(_widgetForRoute(window.defaultRouteName));

Widget _widgetForRoute(String route) {
  switch (route) {
    case 'route1':
      return MyApp();
    case 'route2':
      return Container(color: Colors.red, width: 100, height: 100,);
    default:
      return Center(
        child: Text('Unknown route: $route', textDirection: TextDirection.ltr),
      );
  }
}
```


**原生调用如下：**

Android

```java
// MyApp/app/src/main/java/some/package/MainActivity.java
fab.setOnClickListener(new View.OnClickListener() {
  @Override
  public void onClick(View view) {
    View flutterView = Flutter.createView(
      MainActivity.this,
      getLifecycle(),
      "route1"
    );
    FrameLayout.LayoutParams layout = new FrameLayout.LayoutParams(600, 800);
    layout.leftMargin = 100;
    layout.topMargin = 200;
    addContentView(flutterView, layout);
  }
});
```

iOS - swift

```swift
    @objc func handleButtonAction() {
//        let flutterEngine = (UIApplication.shared.delegate as? AppDelegate)?.flutterEngine;
//        let flutterViewController = FlutterViewController(engine: flutterEngine, nibName: nil, bundle: nil)!;
        let flutterViewController = FlutterViewController.init();
        flutterViewController.setInitialRoute("route1")
//        self.present(flutterViewController, animated: false, completion: nil)
        guard let view = flutterViewController.view else {
            return
        }
        view.frame = CGRect.init(x: 0, y: 200, width: 414, height: 500)
        self.view?.addSubview(view)
        
    }
}
```

## 最后

第一部分内容比较广泛，理解起来也能加深对状态处理的认识。第二部分和第三部分涉及编码，最好能够结合项目来看。



推荐一篇闲鱼的嵌入分享：[闲鱼基于 Flutter 的移动端跨平台应用实践](https://www.infoq.cn/article/xianyu-cross-platform-based-on-flutter)