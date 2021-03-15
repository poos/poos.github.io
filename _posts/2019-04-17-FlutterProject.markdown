---
layout:     post
title:      Flutter 《四》 项目
subtitle:   一个 flutter 项目的文件框架，基于 bloc 页面设计，模型代码
date:       2019-04-17
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Flutter
---

### pubspec.yaml

用于存放三方包的地方，可以在 [flutter/packages](https://pub.dartlang.org/flutter/packages) 查询需要的插件；常用的插件如下：udid，cacheImage，上下拉，网络请求，bloc 设计模式（下边会单独说一下）， user_agent 获取，本地存储，分享，toasts，网络连接检查等

```
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  udid: ^0.0.2
  cached_network_image: ^0.5.1
  pull_to_refresh: ^1.2.0
  dio: ^1.0.17
  flutter_bloc: ^0.9.1
  flutter_user_agent: ^1.0.1
  shared_preferences: ^0.4.0
  share: ^0.6.1
  fluttertoast: ^3.0.3
  connectivity: ^0.3.2

  # user_agent: ^2.0.0
  # flutter_webview_plugin: ^0.3.0+2
  # objectdb: ^1.0.5
  # url_launcher: ^4.0.3
  # event_bus: ^1.0.1
  # flutter_redux: ^0.5.2
```

### 基于 bloc 的列表设计

flutter_bloc: ^0.9.1

> BlocBuilder is a Flutter widget which requires a Bloc and a builder function. BlocBuilder handles building the widget in response to new states. BlocBuilder is very similar to StreamBuilder but has a more simple API to reduce the amount of boilerplate code needed. The builder function will potentially be called many times and should be a pure function that returns a widget in response to the state.

用于将用户交互从 Page Widget 拆分出来，类似于 MVP 的设计模式。例如我们有一个列表页面，有上下拉操作，再加上网络请求失败成功等，如果所有的逻辑写在一个页面，那么就会很不清晰，互相耦合不易测试。那我们就以这个例子展示一下 bloc 设计模式下的代码：

#### 首先我们总结一下页面的用户操作
```
// enmu
enum ListPageEvent { initData, nextPage, refreshData }

```
#### 总结一下页面的状态

使用一个抽象类进行约束
```
//state
abstract class PagePostState {
}

class PagePostUninit extends PagePostState {
//needed show loading
}

class PagePostError extends PagePostState {
//needed show error
}

class PagePostLoaded extends PagePostState {
//needed set data, show normal view
}
```

通常的上边的代码是不用动的，如果需要使用就新创建一个类，然后 继承 PagePostLoaded 即可使用，同时又能够实现 PagePostLoaded 的通用性

本例子中我使用了类似下边的方式，真实项目中内容更多，还有 tips，refreshEnd 等：

```
class HomePagePostLoaded extends PagePostLoaded {
  final List<NewsData> list;
  final int page;
}
```
#### 创建 bloc 类，并且在这个类将 event 处理 为 state

真实情况下并不像下边列的那么简单。例如成功之后的某次网络请求失败，应当返回 Loaded 并且附加 error tips。

```
class ThoughtListPageBloc extends Bloc<ListPageEvent, PagePostState> {
  @override
  get initialState => PagePostUninit();

  @override
  Stream<PagePostState> mapEventToState(event) async* {
    try {
      if (event == ListPageEvent.initData) {
        yield PagePostUninit();
      }

      //refresh -> request && set HomePagePostLoaded
      //nextPage -> request && set HomePagePostLoaded && append data

      var eventToState =
        await _eventToState(event);
      yield eventToState;
    } catch (e) {
      yield PagePostError();

    }
  }
}

```

#### 根据 state ，响应不同的页面

当然真实请求同样不如写的这么简单，通常需要在 loadedWidget(state) 中处理上下拉的显示。

```
class HomePageWidget {

// ...

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('xxx'),
      ),
      body: BlocBuilder(
        bloc: _postBloc,
        builder: (BuildContext context, PagePostState state) {
          if (state is PagePostUninit) {
            return LoadingWidget();
          }
          if (state is PagePostError) {
            return LoadErrorWidget(() {
              // tap to reinit
            });
          }
          if (state is ThoughtListPostLoaded) {
            // normal widget
            return SafeArea(child: _loadedWidget(state));
          }
        },
      ),
    );
  }
```



#### 关于 flutter_bloc 提供的其他工具类

- 这个函数用于根据状态展示相应的 Widge，本例中主要用到了这个

```
BlocBuilder(
  bloc: BlocA(),
  builder: (context, state) {
    // return widget here based on BlocA's state
  }
)

```

- BlocProvider ，自定义一个 bloc 和 child，以便可以向子树内的多个小部件提供单个bloc实例

```
BlocProvider(
  bloc: BlocA(),
  child: ChildA(),
);

//...
BlocProvider.of<BlocA>(context)
```

- BlocProviderTree 是一个 Flutter 小部件，可将多个 BlocProvider 小部件合并为一个小部件

```
BlocProvider<BlocA>(
   bloc: BlocA(),
   child: BlocProvider<BlocB>(
     bloc: BlocB(),
     child: BlocProvider<BlocC>(
       value: BlocC(),
       child: ChildA(),
     )
   )
 )

//...
BlocProviderTree(
  blocProviders: [
    BlocProvider<BlocA>(bloc: BlocA()),
    BlocProvider<BlocB>(bloc: BlocB()),
    BlocProvider<BlocC>(bloc: BlocC()),
  ],
  child: ChildA(),
)

```


- BlocListener是一个Flutter小部件，它接受a Bloc和a BlocWidgetListener并调用listener响应bloc中的状态变化。用于每个状态更改需要发生一次的功能。

```
BlocListener(
  bloc: _bloc,
  listener: (context, state) {
    if (state is Success) {
      Navigator.of(context).pushNamed('/details');
    }
  },
  child: BlocBuilder(
    bloc: _bloc,
    builder: (context, state) {
      if (state is Initial) {
        return Text('Press the Button');
      }
      if (state is Loading) {
        return CircularProgressIndicator();
      }
      if (state is Success) {
        return Text('Success');
      }
      if (state is Failure) {
        return Text('Failure');
      }
    },
  }
)
```
### dart model

模型解析通常可以用使用三方插件，或者代码生成器生成，我也查看了一些三方库，或者代码转换脚本，蓦然回首发现很久之前下载的 **Paste JSON as Code · quicktvoe** 竟然支持 dart 语言。

借助 Dio 和 模型类 用起来还是方便的。

```
    var response = await Dio.get(url: _Api, params: params);
    final thoughtList = modelFromJson(response.toString());
```

### 其他

#### 文件目录如下

item 用于共享 Widget

page 就是一种特殊的 Widget 了

```
lib
|
|- api
|- bloc
|- config
|- item
|- model
|- page
|- main.dart
```

#### config

项目的 config 包含的类很多，引用的三方库也很多：

```

class AppColor {
  static const Color c_999999 = Color(0xff999999);
}

class AppTheme {
  static const TextStyle fuzhu3 =
      TextStyle(color: AppColor.c_999999, fontSize: 12);
}

class AppTime {
  static String formatTime(int time) {
  }
}

class AppData {
  static String _userTokenKey = 'FlutterUserToken';
  static Size screenSize = HAHA.window.physicalSize;

  static String uuid() {
  }

  static Future<String> udid() {
  }

  static Future<String> userToken() async {
  }

  static void setUserToken(String token) async {
  }
}

class AppAlert {
}

class AppShare {
}

```

### 写在最后

拿 flutter 写项目的话还是不错的选择的。就上手来说，官网资料也有很多，遇到问题基本都能解决，最好找一个好项目学习上手2-3天。

关于扩展，对于大多数常用的扩展是都有现成可以使用的。但是也是有一些是难以解决的，packages 没有提供的就需要写三方包来扩展了，这点比较麻烦。

> Tips

推荐一个 alibaba 的 帮助开发者快速上手 Flutter 的项目 [alibaba/flutter-go](https://github.com/alibaba/flutter-go)

![img](https://camo.githubusercontent.com/b59e4856a54d720712862c763ac3fade321e9dc9/68747470733a2f2f696d672e616c6963646e2e636f6d2f7466732f5442313955616851517a6f4b31526a535a466c58586169345658612d313530302d313130362e706e67)
