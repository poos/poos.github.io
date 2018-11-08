---
layout:     post                       # 使用的布局（不需要改）
title:      Swift 函数式编程（1）              # 标题
subtitle:   《函数式 Swift》书籍学习，函数式编程介绍，实现，应用           #副标题
date:       2018-11-05                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog:    true                         # 是否归档
tags:                                #标签
- 语法
---

**[本文所有代码 Demo 地址](https://github.com/poos/BlogDemo)**

Swift 是一门有着合适的语言特性来适配函数式编程方法的优秀语言。函数在 Swift 中是一等公民，意味着我们可以将函数作为各种变量，参数，返回值等使用，这种特性使得 Swift 可以完美的支持函数式编程思想。

## 什么是函数式

避免使用程序状态和可变对象，是降低程序复杂度的有效方式之一，而这也正是函数式编程的精髓。

摘自 [swift 函数式 引言篇章](https://objccn.io/products/functional-swift/preview/)

很难给出函数式的准确定义 — 其实同样地，我们也很难给出面向对象编程，亦或是其它编程范式的准确定义。因此，我们会尽量把重点放在我们认为设计良好的 Swift 函数式程序应该具有的一些特质上：

**模块化**： 相较于把程序认为是一系列赋值和方法调用，函数式开发者更倾向于强调每个程序都能够被反复分解为越来越小的模块单元，而所有这些块可以通过函数装配起来，以定义一个完整的程序。当然，只有当我们能够避免在两个独立组件之间共享状态时，才能将一个大型程序分解为更小的单元。这引出我们的下一个关注特质。

**对可变状态的谨慎处理**： 函数式编程有时候 (被半开玩笑地) 称为“面向值编程”。面向对象编程专注于类和对象的设计，每个类和对象都有它们自己的封装状态。然而，函数式编程强调基于值编程的重要性，这能使我们免受可变状态或其他一些副作用的困扰。通过避免可变状态，函数式程序比其对应的命令式或者面向对象的程序更容易组合。

**类型**： 最后，一个设计良好的函数式程序在使用类型时应该相当谨慎。精心选择你的数据和函数的类型，将会有助于构建你的代码，这比其他东西都重要。Swift 有一个强大的类型系统，使用得当的话，它能够让你的代码更加安全和健壮。


## 函数式思想

“函数在 Swift 中是一等值 (first-class-values)，换句话说，函数可以作为参数被传递到其它函数，也可以作为其它函数的返回值。如果你习惯了使用像是整型，布尔型或是结构体这样的简单类型来编程，那么这个理念可能看来非常奇怪。”


上面一段话再次强调了，你可以使用函数类型就像一个简单类型一样

## 学习使用

书籍中介绍了一个 **战舰类游戏** 的 可攻击计算函数，危险（可被攻击）函数，友军危险 等函数。有兴趣的同学可以到上边书籍链接查看。

在书的下一个章节作者更进一步，实现了 CoreImage 的函数式封装，我也同步尝试对大家最常用的 NSMutableAttributeString 进行封装。先贴出 [Demo 地址](https://github.com/poos/BlogDemo)


### NSMutableAttributeString 封装（函数式实践）

NSMutableAttributeString 是 Foundation 中处理属性字符串的一个类。设置属性通过 [NSAttributedStringKey: Any] 来进行设置。那么使用方便期间，我们进行封装。

**封装结果**

封装了不同 label 使用的样式，在使用的时候直接讲变量作为函数使用即可

```
// 组合函数
let titleOne = font(.systemFont(ofSize: 13)) >>> color(.red)

let titleTwo = titleOne >>> color(.black)

let contentOne = font(.systemFont(ofSize: 11)) >>> color(.red) >>>  alignment(.center) >>> color(.blue)

...

// 函数使用
let attS = NSMutableAttributedString.init(string: "title content")

label.attributedText = titleOne(attS)

```

#### 函数实现

#### 实现各个组合的单个函数

定义一个函数类型，并不是必须的。但是定义之后代码会更加清晰。

```
typealias MutableAttributeFunc = (NSMutableAttributedString) -> NSMutableAttributedString

```

**实现第一个 font 函数**

函数传入一个 font， 返回一个函数类型。

```
func font(_ font: UIFont) -> MutableAttributeFunc {
    return { attString in
        attString.addAttribute(.font, value: font, range: NSRange.init(location: 0, length: attString.length))
        return attString
    }
}

```

事实上实现这个 函数 之后，我们就可以安装如下方式调用了。

```
//通过调用上边的函数，生成一个函数
let font = font(.systemFont(ofSize: 12))

//给字符串添加一个 font 属性
let attS = NSMutableAttributedString.init(string: "title content")
let titleOne = font(attS)
```

**继续实现其他函数**

作为例子示例了 color 和 alignment 的实现

更多的 lineSpace, 插入 image, underLine, 大小字符串等，本文作为例子暂时没有实现。

```
func color(_ color: UIColor) -> MutableAttributeFunc {
    return { attString in
        attString.addAttribute(.foregroundColor, value: color, range: NSRange.init(location: 0, length: attString.length))
        return attString
    }
}


func alignment(_ align: NSTextAlignment) -> MutableAttributeFunc {
    return { attString in
        print("🖲 is nil = \(attString.attribute(.paragraphStyle, at: 0, effectiveRange: nil) == nil))")
        let paragraphStyle = attString.attribute(.paragraphStyle, at: 0, effectiveRange: nil) ?? NSMutableParagraphStyle()
        (paragraphStyle as! NSMutableParagraphStyle).alignment = align
        attString.addAttribute(.paragraphStyle, value: paragraphStyle, range: NSRange(location: 0, length: attString.length))
        return attString
    }
}

```

#### 函数进行组合

如果简单调用是这个样子的

```
//     color(      font   (attS)      )
let title = color(.black)(font(.boldSystemFont(ofSize: 14))(attS))

```

显然有太多的括号，太多的嵌套，已经影响阅读代码和书写代码程度了

**创建一个复合函数**

```
func  composeFilters(_ filter1: @escaping MutableAttributeFunc, _ filter2: @escaping MutableAttributeFunc) -> MutableAttributeFunc {
    return { attString in filter2(filter1(attString)) }
}

// 使用上边的组合函数之后

let colorAndFont = composeFilters(color(.black), font(.boldSystemFont(ofSize: 14)))

let title = colorAndFont(attS)

```

**复合函数符号化**

[资料](https://www.jianshu.com/p/4f025476701a)

上边已经能够明显提示代码清晰度了，但是还有更好的方式，那就是引入运算符：

```
precedencegroup ATPrecedence {
    associativity: left
    higherThan: AdditionPrecedence
    lowerThan: MultiplicationPrecedence
}

infix operator >>> : ATPrecedence

func >>>(filter1: @escaping MutableAttributeFunc, filter2: @escaping MutableAttributeFunc) -> MutableAttributeFunc {
    return { attString in filter2(filter1(attString)) }
}

```

定义了运算符之后，就可以通过运算符进行组合了。到这一步，已经能实现本文开头的功能了。

```
let titleTheme = font(.systemFont(ofSize: 13)) >>> color(.red)

let title = colorAndFont(attS)
```

**其实我们完全可以更进一步**

用泛型实现 >>> 的符号化

```
func >>> <A, B, C>(f: @escaping (A) -> B, g: @escaping (B) -> C) -> (A) -> C {
    return { x in g(f(x)) }
}
```

这样只要是函数 **返回类型一致** 的，我们都可以拼接这些函数，而不限于是 MutableAttributeFunc 类型。


### 理论背景：柯里化（Currying）

“将一个接受多参数的函数变换为一系列只接受单个参数的函数，这个过程被称为柯里化 (Currying)，它得名于逻辑学家 Haskell Curry。”


```
func add1(_ x: Int, _ y: Int) -> Int {
    return x + y
}

func add2(_ x: Int) -> (Int) -> Int {
    return { y in return x + y }
}

add(1, 2)
add2(1)(2)

```
上述代码 add2 就是 add1 的 Currying 版本。

优点就是可以单独调用，所以支持各种组合。


## 总结

Swift 下的很多高阶函数都是，例如 Map，Fliter，Reduce等。

本文只是仿照跟书中对 Core Image 的封装，简单的实践和体验一下。区别于分类，直接用 函数式 思想封装组合，以后在使用 NSMutableAttributeString 又有了一个新的选择。更进一步，如果用于 APP 的 Theme 模块，函数组合无论是直接使用，还是动态切换都可以十分方便，这个方案都可以完美的支持。

函数式提供了区别于面向对象的面向函数编程，这个新大陆有无限可能，记着 **函数是一等公民** 就好。


#### 资料：

[Haskell 纯函数式编程语言](https://baike.baidu.com/item/Haskell/1152799?fr=aladdin)

[ep1-functions](https://www.pointfree.co/episodes/ep1-functions)
