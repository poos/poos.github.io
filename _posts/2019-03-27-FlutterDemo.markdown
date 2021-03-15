---
layout:     post
title:      Flutter 《二》项目 Demo 运行，Dart 语言简单学习
subtitle:   列表，页面跳转，请求数据 Demo；Dart 语法
date:       2019-03-27
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Flutter
- 语法
---

## Demo

**[本文所有代码 Demo 地址](https://github.com/poos/BlogDemo)**


#### 介绍


项目代码主要是在 **lib/** 文件夹，在 main.dart 中导入相应的文件包即可调用跑起来。


```swift
// main.dart

import 'package:flutter/material.dart';


//import 'package:flutter_app_layout/add_example.dart';
//void main() => runApp(AddExampleApp());

import 'package:flutter_app_layout/fetch_data.dart';
void main() => runApp(FetchDataApp());

//import 'package:flutter_app_layout/net_json.dart';
//void main() => runApp(NetJsonApp());


```

然后 分别创建其他 demo 文件；单独编写测试即可。


```swift
//net_json.dart

import 'dart:async';
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;


//https://github.com/flutterchina/dio
import 'package:dio/dio.dart';


Future<Post> fetchPost() async {
  final responce = await http.get('https://jsonplaceholder.typicode.com/posts/1');
  final responseJson = json.decode(responce.body);

  return Post.fromJson(responseJson);
}

class Post {
  final int userId;
  final int id;
  final String title;
  final String body;

  Post({
    this.id,
    this.userId,
    this.title,
    this.body});

  factory Post.fromJson(Map<String, dynamic> json) {
    return Post(
      userId: json['userId'],
      id: json['id'],
      title: json['title'],
      body: json['body'],
    );
  }
}

class NetJsonApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    getHttp();

    // TODO: implement build
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: Text('Fetch data demo'),),
        body: Center(
          child: FutureBuilder<Post>(
            future: fetchPost(),
            builder: (context, snapshot) {
            if (snapshot.hasData) {
              return Text(snapshot.data.title);
            } else if (snapshot.hasError) {
              return Text('${snapshot.error}');
            }
            return CircularProgressIndicator();
          }),
        ),
      ),
      title: 'fetch data examole',
    );
  }

  // 测试 Dio
  void getHttp() async {
    try {
      Response response = await Dio().get('https://raw.githubusercontent.com/poos/poos.github.io/master/test/example.json');
      print(response);
    } catch (e) {
      print(e);
    }
  }

}

```

#### pubspec.yaml

项目的第三方包配置页，例如上边的 **http** 和 **Dio** 即需要导入包方可使用。

```swift
dependencies:
  flutter:
    sdk: flutter

  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.2

  http: any
  dio: 2.1.0

```


## 学习 Dart 语法

基本上敲了3个 demo 之后，对 Dart 的语法已经有了一些理解：

函数：

- main 函数为起点

- 单行函数简写

- 两种函数返回值类型

- 一些异步函数

- 一些关键词

常，变量：

- 字面量

- 字符串，引用字符串

以及

- 代码风格，非常多的，缩进


### 系统阅读 Dart 文档

上边了解到的毕竟还是比较少，强烈建议去 [A Tour of the Dart Language](https://www.dartlang.org/guides/language/language-tour)，通读一下代码的语法和规则。


#### important 一些重要特性

- 您可以放在变量中的所有内容都是一个对象，每个对象都是一个类的实例。偶数，函数和 null对象。所有对象都继承自Object类。

- 尽管Dart是强类型的，但类型注释是可选的，因为Dart可以推断类型。在上面的代码中，number 推断为类型int。如果要明确说明不需要任何类型，请使用特殊类型dynamic。

- Dart支持泛型类型，如List<int>（整数列表）或List<dynamic>（任何类型的对象列表）。

- Dart支持顶级函数（例如main()），以及绑定到类或对象的函数（分别是静态和实例方法）。您还可以在函数内创建函数（嵌套函数或本地函数）。

- 类似地，Dart支持顶级变量，以及绑定到类或对象的变量（静态和实例变量）。实例变量有时称为字段或属性。

- 与Java，Dart不具备关键字public，protected和private。如果标识符以下划线 _ 开头，则它对其库是私有的。有关详细信息，请参阅 库和可见性。标识符可以以字母或下划线 _ 开头，后跟这些字符加数字的任意组合。

- Dart有表达式（具有运行时值）和 语句（不具有）。例如，条件表达式 condition ? expr1 : expr2的值为expr1或expr2。将其与if-else语句进行比较，该语句没有任何值。语句通常包含一个或多个表达式，但表达式不能直接包含语句。

- Dart工具可以报告两种问题：警告和错误。警告只是表明您的代码可能无法正常工作，但它们不会阻止您的程序执行。错误可以是编译时或运行时。编译时错误会阻止代码执行; 运行时错误导致 代码执行时引发异常。

#### 数据类型

- 字符串

值得一提的是 字符串支持 单引号 ’ 和 双引号 “ 两种，好处如下：

```
var s3 = 'It\'s easy to escape the string delimiter.';
var s4 = "It's even easier to use the other delimiter.";
```

${  }  来进行取值

- Runs

Dart 使用 utf-16 编码，32位的编码表示如下：笑的表情符号（😆）是\u{1f600}。

```
Runes input = new Runes(
      '\u2665  \u{1f605}  \u{1f60e}  \u{1f47b}  \u{1f596}  \u{1f44d}');
print(new String.fromCharCodes(input));
```

- Symbols

如果某个 api 使用了关键字，你可以使用 # 来进行使用，例如： #radix

#### 函数

- 支持可选值和默认值

- 顶级函数和 main()

- 匿名函数

- 嵌套函数，作用域

- 函数式

- 判断函数相等

```swift
void foo() {} // A top-level function

class A {
  static void bar() {} // A static method
  void baz() {} // An instance method
}

void main() {
  var x;

  // Comparing top-level functions.
  x = foo;
  assert(foo == x);

  // Comparing static methods.
  x = A.bar;
  assert(A.bar == x);

  // Comparing instance methods.
  var v = A(); // Instance #1 of A
  var w = A(); // Instance #2 of A
  var y = w;
  x = w.baz;

  // These closures refer to the same instance (#2),
  // so they're equal.
  assert(y.baz == x);

  // These closures refer to different instances,
  // so they're unequal.
  assert(v.baz != w.baz);
}
```

- 返回值，无返回值 等价 返回值为 null

```
foo() {}
assert(foo() == null);
```

#### 操作符

- 数学操作符

- 关系判断操作符

- 类型判断操作符，as，is，is！

- 包含赋值的 ~/=

- 逻辑取反，与，或

- 移位操作符

- if-esle 表达式

- Cascade notation （层叠标记）

```swift
querySelector('#confirm') // Get an object.
  ..text = 'Confirm' // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));

//等价于
var button = querySelector('#confirm');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
```

- 其他符号()[]?.

#### Control flow statements 控制流


- if and else
- for loops
- while and do-while loops
- break and continue
- switch and case
- assert 如果表达式的值为true，则断言成功并继续执行。如果为false，则断言失败并抛出异常（ **仅测试环境下抛出** ）。

#### Exceptions

- throw

- try - catch

- try - finally

#### 类

- 成员变量 ，使用 .runtimeType 获取 Type 对象。

- 构造函数

- 方法

- 抽象方法，类，抽象方法只能存在抽象类中

- 隐式接口

- 扩展

- 枚举类

- 向类添加功能：mixins （相当于代理）

```swift
mixin Musical {
  bool canPlayPiano = false;
  bool canCompose = false;
  bool canConduct = false;

  void entertainMe() {
    if (canPlayPiano) {
      print('Playing piano');
    } else if (canConduct) {
      print('Waving hands');
    } else {
      print('Humming to self');
    }
  }
}

class Maestro extends Person
    with Musical, Aggressive, Demented {
  Maestro(String maestroName) {
    name = maestroName;
    canConduct = true;
  }
}
```

- 类变量和方法

#### 泛型

正确指定泛型类型会产生更好的生成代码。您可以使用泛型来减少代码重复。不再展开了，用到再去文档中学习即可。

#### 使用库

- 如果库名重复可以指定别名。

- 可以只使用库的某个文件。

- 延迟加载库。

#### 异步

- 异步函数 async， await

- 使用try，catch和finally 处理使用await以下代码的错误和清

- 声明异步函数，例如： Future < String > lookUpVersion （）async => '1.0.0'

- 处理流：使用async和异步for循环（await for）

#### Generators 生成器函数

用于懒惰的生成一系列值。支持同步和异步。


#### Callable classes 可调用的类

如下，直接调用  WannabeFunction("Hi","there,","gang");

```
class WannabeFunction {
  call(String a, String b, String c) => '$a $b $c!';
}

main() {
  var wf = new WannabeFunction();
  var out = wf("Hi","there,","gang");
  print('$out');
}
```

#### 分离

使用多核CPU。

#### 类型声明定义

目前，typedef仅限于函数类型。我们希望这会改变。

#### 元数据

使用元数据提供有关代码的其他信息。元数据注释以字符开头@，例如 @override。

元数据可以出现在库，类，typedef，类型参数，构造函数，工厂，函数，字段，参数或变量声明之前以及导入或导出指令之前。您可以使用反射在运行时检索元数据。

#### 注释 // , / * * /

#### Documentation comments


```
/// A domesticated South American camelid (Lama glama).
///
/// Andean cultures have used llamas as meat and pack
/// animals since pre-Hispanic times.
```
