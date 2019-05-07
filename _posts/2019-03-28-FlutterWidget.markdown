---
layout:     post                          # 使用的布局（不需要改）
title:      Flutter 《三》常用组件 & Week Video           # 标题
subtitle:   官方 Widget 介绍，Week Video 特殊用途的 Widget     #副标题
date:       2019-03-28                # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- Flutter
- 语法
---

## 常用的 Widgets

### 布局 Widgets

[tutorials/layout](https://flutterchina.club/tutorials/layout/)


#### 标准 widgets
- Container
添加 padding, margins, borders, background color, 或将其他装饰添加到widget.
- GridView
将 widgets 排列为可滚动的网格.
- ListView
将widget排列为可滚动列表
- Stack
将widget重叠在另一个widget之上.


#### Material Components
- Card
将相关内容放到带圆角和投影的盒子中。
- ListTile
将最多3行文字，以及可选的行前和和行尾的图标排成一行

### 交互 Widgets

[tutorials/interactive](https://flutterchina.club/tutorials/interactive/)

不再展开

#### 标准 widgets:
- Form
- FormField
#### Material Components:
- Checkbox
- DropdownButton
- FlatButton
- FloatingActionButton
- IconButton
- Radio
- RaisedButton
- Slider
- Switch
- TextField


### 其他 widget
Decoration  装饰

Scaffold  脚手架，提供 navightion bar

Stateful <-> Stateless  有无生命周期状态

列 Column    	ListView

行 Row  		ListTitle

**两个页面写下来就基本掌握了使用方法，没达到效果就看一下 api 即可。**

**flutter 写起来基本就是 widget 套 widget。一切皆是 widget。**


### StatefulWidget、StatelessWidget

区分要点是 **是否需要用户交互**。

stateful widget 是指有状态变化的组件，例如系统提供的 Checkbox, Radio, Slider, InkWell, Form, TextField 都是 stateful widgets, 他们都是 StatefulWidget的子类。

stateless widget 是没有内部状态变化的组件，例如 Icon、 IconButton, 和Text 都是无状态widget, 他们都是 StatelessWidget的子类。


#### 实现方式

```dart
class XXXX extends StatefulWidget {
  @override
  _XXXXState createState() => _XXXXState();
}

class _XXXXState extends State<XXXX> {
  @override
  Widget build(BuildContext context) {
    return Container(
      
    );
  }
}

class LLLL extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      
    );
  }
}
```

实现方式也略微不一样，StatefulWidget 为了遵循私有变量的保密性，特别分了一个私有类来管理私有成员变量。




## week video

[Flutter](https://www.youtube.com/channel/UCwXdFgeE9KYzlDdR7TG9cMw)

flutter 官网在 YouTube 上的 3-5 分钟小视频，介绍一些特殊的用法。

### SafeArea
Flutter的SafeArea工具可以避免讨厌的消息通知栏和不规则的手机屏幕干扰您的应用程序的正常显示。 它使用MediaQuery来检查屏幕的尺寸，并使用一个子工具来匹配您的应用程序，确保它能在iOS和Android上都正常显示！

### Expanded
使用Expanded小工具扩展您对Flutter的了解！ Flutter的Expanded将改变发送给行和列的子项的约束，指示它们填补可用的空间。因此，将您的子项包裹在Expanded小工具中，并观察它的增长！
[了解更多有关Expanded→](http://bit.ly/2vLeScd)

### Wrap
当你用完横向和纵向的布局空间时，试试Wrap吧！Wrap可以为小图标进行纵向或横向的布局，且当空间用完时，它便会新增一行。Direction属性、alignment属性和spacing属性，可以帮助你准确的获得你所需要的布局效果。

### AnimatedContainer
你可以用显式动画来自己写出活动的变化，你也可以让Flutter帮你让它们动起来！利用AnimatedContainer微件，你只需用某个特定的属性（如色彩）来实施一次建造，再用另一个值来再次实施建造。Flutter会自动地让两个值之间的变化动起来！而且你可以指定动画长度和个性化曲线。

### Opacity
如果你想让一个微件在屏幕中不可见，且原来的页面布局保持不变？那就请试一试我们的Opacity微件吧！你只要设置一个透明度的数值，它就能让你的子褪色以便匹配。你也可以用它来混合不同子部件，或是用AnimatedOpacity来创建动画。

### FutureBuilder
有一个Future并且需要一些小部件来显示其价值？试试FutureBuilder吧！给它一个Future和一个构建方法，它将根据未来的状态创建小部件，并在它发生变化时更新它们。

### FadeTransition
当您只是寻找简单的转场并且不需要使用 Flutter 中更强大的动画时，请尝试使用FadeTransition 小部件！ FadeTransition 可让您轻松淡入淡出小部件，并且可以直接放入Flutter 应用程序。

### FloatingActionBar
如果您想在底部的导航栏中嵌入一个按钮，Flutter可以帮助您！通过在您的脚手架中组合FloatingActionBar（FAB）和BottomAppBar，您可以轻松地将按钮嵌入导航栏中的任何位置。

### PageView
如果你喜欢在 Flutter 上拥有好看的屏幕，想像一下拥有可滑动的屏幕会有多棒！使用 PageView 和 PageController 能更容易创建一系列使用者可以自由垂直向与水平向滑动的屏幕。现在就试一试！

### table
表格非常适合创建您不想滑动的小部件网格，尤其是如果您拥有不同大小的小部件。嵌套的行和列可能会有点乱，但表格小部件提供了一致性并为您调整子窗口小部件的大小。试一试吧！

### SliverAppBar
当您的滚动时，会发生变化或消失的花哨动画标题十分流行！使用SliverAppBar，您可以轻松地在您的应用中使用它。与CustomScrollView一起使用，您可以在应用栏中获得自定义滚动行为！

### SliverList
如果你想要同时滚动一个列表和一个网格，或想要一个很酷和复杂的滚动效果，SliverList和SliverGrid正是您需要的！ Slivers可以轻松滚动网格或列表中的大量子项，并允许您毫不费力地或提前构建子项。

### FadeInImage
为避免图像在加载时弹出到屏幕上的刺眼效果，请尝试使用FadeInImage！您可以指定本地占位符图像并设置高度和宽度参数，以确定图像加载后应占用的空间大小。

### StreamBuilder
今日的应用程序是高度不同步的，飞镖靶串流是管理非同步数据的好方法。 如何构建可以跟上串流的连续数据串流的小部件？ 试试StreamBuilder吧！ 只需给它一个串流和一个构建器，它就会在串流发出新的数据事件时重建它的子节点。

### InheritedModel
访问子子子子类的特定部分

### ClipRRect
对于那些你希望你可以在应用程序中的方块内容上有圆角的时候，有ClipRRect（额外的R实际上代表圆形）！ 用剪刀包裹你的小部件，设置半径，剩下的就完成了！ 您甚至可以自定义外观并尝试使用类似小部件的不同形状。 试试看！

### Hero
“Hero过渡”是一种常见的UI模式。他们让用户知道他们改变了屏幕，同时让交互的焦点保持在前沿和中心。 Flutter 的 Hero微件将自动在两个导航路线之间创建一个Hero过渡。

### CustomPaint
想要自定义的用户界面吗？Flutter可以使你快速高效的访问低级别的绘画调用。绘制一切。

### ToolTips
应用程序通常使用图标来传达意思。使用工具提示包裹您的图标和图像，以附加提高可访问性和提供更多上下文的工具提示消息。

### FittedBox
超出边界时如何适应。

### LayoutBuilder
在您最终确定它的外观之前，LayoutBuilder可以帮助您了解小部件的大小。它的构建器函数具有构建上下文和传入框约束的参数。

### AbsorbPointer
AbsorbPointer为您的小部件提供了一种禁止输入的好方法。如果您有一个复杂的小部件子树，并且需要一种方法将它们全部从触摸事件中移除，请尝试使用AbsorbPointer。

### Transform
使用“Transform”小部件将Flutter应用程序转换为令人惊叹的东西。各种动画转场。
[“Flutter的观点”文章→](https://bit.ly/2TYvc31)
[被旋转的菜单图标”由Raouf Rahiche提供→](https://bit.ly/2RE8fFC）
[“折叠动画的过渡”，“有弹性的滚动”，“酷的卡菜单”由FaoB提供→](https://bit.ly/2Rzq1tE）
[“3D透视动画”由Hung HD提供→ ](https://bit.ly/2sx6fjy）

### BackdropFilter
使用ImageFilter类和BackdropFilter微件将效果应用于Flutter应用程序支持的图像！
[BackdropFilter类文档→](https://bit.ly/2CQYGsR)
[ImageFilter类文档→](https://bit.ly/2CMAbwO)

### Align
Align 小部件能让你将小部件置放在其父小部件中的明确位置。你也可以用 Align 来将小部件叠起来。Align 是个小部件自定义组合的好工具！
[Align 课程记录 → ](https://bit.ly/2UF1wZl)

### AnimatedBuilder
Flutter具有出色的动画支持 - 使用AnimatedBuilder小部件创建动画。您还可以使用StatefulWidget管理AnimatedController。
[AnimatedBuilder文档→](https://bit.ly/2V9XVCB)
[动画和动态小部件→](https://bit.ly/2XeATfO)

### Dismissible
Dismissible小部件可通过向左或向右滑动来清除列表项。对于多方向滑动，它支持两种背景，如果您的UI需要用户垂直滑动，则有一个方向属性！
[从Dismissable课堂文档了解更多信息 → ](https://bit.ly/2Cnhmki)
