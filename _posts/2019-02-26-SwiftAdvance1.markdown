---
layout:     post                          # 使用的布局（不需要改）
title:      Swift 开发必备 tips            # 标题
subtitle:   Swift新元素， 从Objective-C/C到Swift，Swift与开发环境及一些实践   #副标题
date:       2019-02-26                # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 语法
---


### Swift新元素

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

```swift
protocol AnyProtocol {
    mutating func printName()
}
```

#### Sequence

你使用某个类型遵守 􏰑􏰤􏰡􏰄􏰃􏰤􏰝􏰄􏰟􏰄􏰝􏰤􏰝􏰧􏰝􏰣􏰑􏰤􏰡􏰄􏰃􏰤􏰝􏰄􏰟􏰄􏰝􏰤􏰝􏰧􏰝􏰣IteratorProtocol 协议，那么这个类型即可使用 for ... in 方法了；同时这个类型也可以使用 map，filter，reduce 这些高阶函数。

究其根本的话 for in 是这样的

```swift
var iterator = arr.makeIterator( )
while let obj = iterator .next() {
    print(obj)
}
```


#### tuple

元组的使用，例如交换两个值，例如返回两个值

```
//1
(a, b) = (b, a)

//2
func calc() -> (Int, Int)
```


#### @autoclosure和??

例如一个函数接受一个闭包，虽然可以使用尾随闭包的特性：

> logIfTrue{2 > 1}

但是，要么是书写起来十分麻烦，要么是表达上不太清晰，看起来都让人生气。于
是@autoclosure登场了。我们可以改换方法参数，在参数名前面加上@autoclosure 关键字:
```
func logIfTrue(_ predicate: @autoclosure () -> Bool) {
if predicate() {
print("True" )
```
这时候我们就可以直接写:

> logIfTrue(2 > 1)


#### @escaping

逃逸闭包的用法。注意适时使用 [weak self]。

#### Optional Chaining

注意一点，连续可选值调用时候，判断最终调用是否成功容易发生错误：


这是错误代码

> let playClosure = {(child: Child) -> () in child.pet?.toy?.play()}

这样的代码是没有意义的!
问题在于对于 play() 的调用上。定义的时候我们没有写play() 的返回，这表示这个方法返回
Void (或者写作一对小括号()， 它们是等价的)。但是，经过 Optional Chaining以后
我们得到的是一个Optional的结果。也就是说，我们最后得到的应该是这样一个 closure :

> let playClosure = {(child: Child) -> ()? in child. pet? . toy?. play()}

这样调用的返回将是一-个()? (或者写成Void? 会更清楚- -些)， 虽然看起来挺奇怪的，但这就是
事实。使用的时候我们可以通过Optional Binding来判定方法是否调用成功:

```
if let result: () = playClosure(xiaoming) {
    print("好开心~")
} else {
    print("没有玩具可以玩:(")
}
```


#### 操作符

可以自定义操作符。可能跟 **柯里化** 擦出不一样的火花。

**注意** 尽量明了，避免歧义和可能的误解。因为一个不被公认的操作符是存在冲突风险和理解难度的， 所以我们不应该滥用这个特性。

#### func的参数修饰

```
func increaseIt(_ number: inout Int) {
    number += 1
}
```

#### 字面量表达

除了参数初始化时候可以省略类型初始化。

另一种方式 **通过实现协议即可使用 字面量初始化 自定义类型**，例如用字符串初始化 enum 等。

#### 下标

作为一门代表了先进生产力的语言，Swift 是允许我们自定义下标的。这不仅包含了对自己写的类
型进行下标自定义，也包括了对那些已经支持下标访问的类型(没错就是Array 和 Dictionay)进
行扩展。

> subscript (index: Int) -> T

> subscript ( subRange: Range<Int>) -> Slice<T>


#### 函数嵌套

如果一个函数只是被另一个函数调用，不妨尝试一下。单一功能的代码块会整合到一起。

#### 命名空间

在 Swift 中，由于可以使用命名空间了，即使是名字相同的类型，只要是来自不同的命名空间的 话，都是可以和平共处的。和 C# 这样的显式在文件中指定命名空间的做法不同，Swift 的命名空 间是基于 module 而不是在代码中显式地指明，每个 module 代表了 Swift 中的一个命名空间。也 就是说，同一个 target 里的类型名称还是不能相同的。在我们进行 app 开发时，默认添加到 app 的主 target 的内容都是处于同一个命名空间中的，我们可以通过创建 Cocoa (Touch) Framework 的 target 的方法来新建一个 module，这样我们就可以在两个不同的 target 中添加同样名字的类型 了。

#### typealias

```
// This is OK
typealias Worker<T> = Person<T>


class Person<T> {}
typealias WorkId = String
typealias Worker = Person<WorkId>


protocol Cat { ... }
protocol Dog { ... }
typealias Pat = Cat & Dog

```

#### associatedtype

```
// 相当于一个抽象父协议，不能被直接遵守实现
protocol Animal {
    associatedtype F: Food
    func eat(_ food: F)
}

struct Tiger: Animal {
    func eat(_ food: Meat) {
        print("eat (meat)")
    }
}

struct Sheep: Animal {
    func eat(_ food: Grass) {
        print("eat (food)")
    }
}
```

#### 可变参数函数

不必像 OC 一样拘泥于最后一位。

```
func myFunc (numbers: Int..., string: String) {
    numbers.forEach {
        for i in 0..<$0 {
        print("'(i + 1): (string)")
        }
    }
}
myFunc (numbers: 1, 2, 3, string: "hello" )
```


#### 初始化方法顺序
- 先初始化子类的成员变量常量
- 调用 super.init
- 视情况修改父类的成员变量

#### Designated, Convenience 和Required
- designated，不加修饰的初始化方法，必须对非可选的属性进行设置，并且会显式或者隐式调用父类的方法。
- convenience，便利的初始化方法，必须调用当前类的 designated 方法。**如果在子类重写了当前调用的 designated 方法，那么子类就可以使用父类中相关的 convenience 方法了**
- 对于某些我们希望子类中一定实现的 designated 初始化方法，我们可以通过添加 required􏰄􏰡􏱇􏰞􏰖􏰄􏰡􏰛 关键字进行限制，强制子类对这个方法重写实现。这样做的最大的好处是 **可以保证依赖于某个 designated 初始化方法的 convenience􏰧􏰝􏰕􏰪􏰡􏰕􏰖􏰡􏰕􏰧􏰡 一直可以被使用。**

#### 初始化返回nil
> convenience init?(string URLString: String)

在新版本的Swift中，对于可能初始化失败的情况，我们应该始终使用可返回nil 的初始化方法，而不是类型工厂方法。

#### static和class
#### 多类型和容器
#### default参数
#### 正则表达式
#### 模式匹配
#### ..和..<
#### AnyClass,元类型和.self
#### 协议和类方法中的Self
#### 动态类型和多方法
#### 属性观察
#### final
#### lazy修饰符和lazy方法
#### Reflection和Mirror
#### 隐式解包Optional
#### 多重Optional
#### Optional Map
#### Protocol Extension
#### where和模式匹配
#### indirect和嵌套enum


### 从Objective-C/C到Swift

#### Selector
#### 实例方法的动态调用
#### 单例
#### 条件编译
#### 编译标记
#### @UlApplicationMain
#### @objc和dynamic
#### 可选协议和协议扩展
#### 内存管理，weak 和unowned
#### @autoreleasepool
#### 值类型和引用类型
#### String还是NSString
#### UnsafePointer
#### C指针内存管理
#### COpaquePointer和C convention
#### GCD和延时调用
#### 获取对象类型
#### 自省
#### KVO
#### 局部scope
#### 判等
#### 哈希
#### 类簇
#### 调用C动态库
#### 输出格式化
#### Options
#### 数组enumerate
#### 类型编码@encode
#### C代码调用和@asmname
#### delegate
#### Associated Object
#### Lock
#### Toll-Free Bridging和Unmanaged



### Swift与开发环境及一些实践
#### Swift命令行工具
#### 随机数生成
#### print和debugPrint
#### 错误和异常处理
#### 断言
#### fatalError
#### 代码组织和Framework
#### 安全的资源组织方式
#### Playground延时运行
#### Playground与项目协作
#### Playground可视化开发
#### 数学和数字
#### JSON
#### NSNull
#### 文档注释
#### 性能考虑
#### Log输出
#### 溢出
#### 宏定义define
#### 属性访问控制
#### Swift中的测试
#### Core Data
#### 闭包歧义
#### 泛型扩展
#### 兼容性
#### 列举enum类型
#### 尾递归
