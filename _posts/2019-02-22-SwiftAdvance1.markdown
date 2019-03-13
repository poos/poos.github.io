---
layout:     post                          # 使用的布局（不需要改）
title:      Swift 开发必备 tips 《一》            # 标题
subtitle:   Swift新元素   #副标题
date:       2019-02-22                # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 语法
---

#### 柯里化(Currying)

Swift里可以将方法进行柯里化(Currying)，这是也就是把接受多个参数的方法进行- -些变形，使
其更加灵活的方法。

之前的博客已经介绍过，不再赘述。

```swift
func addTo(_ adder: Int) -> (Int) -> Int {
    return { num in
        return num + adder
    }
```


#### 将protocol的方法声明为mutating
Swift 的 mutating􏰅􏰞􏰤􏰃􏰤􏰖􏰕􏰗 关键字修饰方法是为了能在该方法中修改 struct􏰰􏰤􏰄􏰞􏰧􏰤 或是 􏰡􏰕􏰞􏰅enum 的变量，所以如果你没在协议方法里写 mutating 的话，别人如果用 struct􏰰􏰤􏰄􏰞􏰧􏰤 或是 􏰡􏰕􏰞􏰅enum 来实现这个协议的话，就不能在方法里改变自己的变量了。

```
protocol AnyProtocol {
    mutating func printName()
}
```

#### Sequence

你使用某个类型遵守 􏰑􏰤􏰡􏰄􏰃􏰤􏰝􏰄􏰟􏰄􏰝􏰤􏰝􏰧􏰝􏰣􏰑􏰤􏰡􏰄􏰃􏰤􏰝􏰄􏰟􏰄􏰝􏰤􏰝􏰧􏰝􏰣IteratorProtocol 协议，那么这个类型即可使用 for ... in 方法了；同时这个类型也可以使用 map，filter，reduce 这些高阶函数。

究其根本的话 for in 是这样的

```
var iterator = arr.makeIterator( )
while let obj = iterator .next() {
    print(obj)
}
```


tuple
@autoclosure和??
@escaping
Optional Chaining
操作符
func的参数修饰
字面量表达
下标
方法嵌套
命名空间
typealias
associatedtype
可变参数函数
初始化方法顺序
Designated, Convenience 和Required
初始化返回nil
static和class
多类型和容器
default参数
正则表达式
模式匹配
..和..<
AnyClass,元类型和.self
协议和类方法中的Self
动态类型和多方法
属性观察
final
lazy修饰符和lazy方法
Reflection和Mirror
隐式解包Optional
多重Optional
Optional Map
Protocol Extension
where和模式匹配
indirect和嵌套enum
