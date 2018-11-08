---
layout:     post                       # 使用的布局（不需要改）
title:      Swift 函数式编程（2）                # 标题
subtitle:   Swift 中的高阶函数探究和使用           #副标题
date:       2018-11-08                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog:    true                        # 是否归档
tags:                                #标签
- 语法
---

**[本文所有代码 Demo 地址](https://github.com/poos/BlogDemo)**

“接受其它函数作为参数的函数有时被称为 **高阶函数** 。本章中，我们将在一些来自 Swift 标准库中作用于数组的高阶函数中漫游。伴随这个过程，我们将介绍 Swift 的 **泛型** ，以及展示如何将复杂计算运用于数组。”


Swift 中的 Array 和 Dictionary 具有 map，fliter，reduce 等高阶函数。用于转换，筛选和组合里面的元素


### map

转换函数
#### 自己实现一个 map 函数
Swift 标准库已经实现了 map 函数 (基于 SequenceType 协议被定义，需要了解一下生成器和序列的知识)
如果 **自己实现一个 map 函数** 大概是如下代码

```
extension Array {
    func map2<T>(transform: (Element) -> T) -> [T] {
        var result: [T] = []
        for x in self {
            result.append(transform(x))
        }
        return result
    }
}
```

代码里边的 T 即是一个 **泛型** ，在调用时候给出确定的类型即可。调用示例如下：

```
let exampleFiles = ["README.md", "HelloWorld.swift", "FlappyBird.swift"]


let map3 = exampleFiles.map2 { (item) -> Int in
    return item.count
}
print(map3)

```

为什么用 3 ，因为有更加简便的书写方式，Swift 可以用 “$0” 等价为闭包中的第一个元素，“$1”...这算一种语法糖。

```
let map2 = exampleFiles.map2 { $0.count }
print(map2)

```

#### 使用 map 函数

调用系统的 map 函数跟上边自己实现的是差不多的

```
let map = exampleFiles.map { $0.count }
print(map)
//[9, 16, 16]

```

#### 使用 flatMap 函数


>flatMap返回后的数组中不存在 nil 同时它会把Optional解包;
>flatMap 还能把数组中存有数组的数组 一同打开变成一个新的数组 ;

[Swift 数组中 Map,FlatMap,Filter,Reduce的使用](http://www.cocoachina.com/swift/20160210/15068.html)

一些例子：

为了展示，新添加一个 arr： exampleFiles2。

- 例子1：是利用 flatMap 展开了数组
- 例子2：是利用 flatMap 正常遍历转换其他类型的情况
- 例子3：展开了数组，再将每一个元素转换类型
- 例子4：利用 map 展开了并且组合，利用外部的 flatMap 展开组合后的元组数组

**注意：例子2使用了compactMap，这个如何这个 Any 对象没有 count 值，那么就回返回 nil。如果使用flatMap，则会跳过（ 可能有隐式 bug，所以不推荐）。**

**注意：例子3使用了两个高阶函数链式调用，这会使整个表达式的复杂的上升（O(n*n)）。**

```

let exampleFiles2 = [exampleFiles, [""], exampleFiles]

let flatMap1 = exampleFiles2.flatMap { $0 }
print(flatMap1)
//["README.md", "HelloWorld.swift", "FlappyBird.swift", "", "README.md", "HelloWorld.swift", "FlappyBird.swift"]

//'flatMap' is deprecated: Please use compactMap(_:) for the case where closure returns an optional value
let flatMap2 = exampleFiles2.compactMap{ $0.count }
print(flatMap2)
//[3, 1, 3]

let flatMap3 = exampleFiles2.flatMap { $0 }.map({$0.count})
print(flatMap3)
//[9, 16, 16, 0, 9, 16, 16]

let flatMap4 = exampleFiles.flatMap { (item) in
    exampleFiles.map({ (item2) -> (String, String) in
        return (item, item)
    })
}
print(flatMap4)
//[("README.md", "README.md"), ("README.md", "README.md"), ("README.md", "README.md"), ("HelloWorld.swift", "HelloWorld.swift"), ("HelloWorld.swift", "HelloWorld.swift"), ("HelloWorld.swift", "HelloWorld.swift"), ("FlappyBird.swift", "FlappyBird.swift"), ("FlappyBird.swift", "FlappyBird.swift"), ("FlappyBird.swift", "FlappyBird.swift")]
```

#### 关于时间复杂度

我们可以利用自己的 map 函数，验证组合函数的情况下，数组的遍历实际执行了几次

```
extension Array {
    func map2<T>(transform: (Element) -> T) -> [T] {
        var result: [T] = []
        for x in self {
            result.append(transform(x))
            print(x)
        }
        return result
    }
}


let map4 = exampleFiles.map2 { $0.count }.map2(transform: { $0 == 2 })
print(map4)
```


### fliter

筛选函数

```
let filter = exampleFiles.filter({ $0.hasSuffix(".swift") })
print(filter)
//["HelloWorld.swift", "FlappyBird.swift"]
```

### reduce

修改

我们可以将一个集合类型的 对象转换为其他需要的值

- 例子1：转换成用逗号分隔的字符串
- 例子2：统计文件个数

**result[key, default: 0] += 1，这个语法糖在碰到 key 的情况下将 Int 型的 Value 加 1**

```

var reduceR = exampleFiles.reduce("") { (total, item) -> String in
    if total.count == 0 {
        return item
    } else {
        return total + "," + item
    }
}
print(reduceR)
//README.md,HelloWorld.swift,FlappyBird.swift

let reduceInto = exampleFiles.reduce(into: [String: Int]()) { (result, item) in
    let index = item.lastIndex(of: ".") ?? item.startIndex
    let key = item.suffix(from: index)
    result["\(key)", default: 0] += 1
}
print(reduceInto)
//[".md": 1, ".swift": 2]

```

#### 关于时间复杂度

map - flatMap - 例子2，因为链接高阶函数导致复杂度提升的问题，我们可以用 reduce 和 map 函数组合 **修改转换** 实现。

```
let reduceInto2 = exampleFiles2.reduce(into: [Int]()) { (result, item) in
    return result += item.map({ $0.count })
}
print(reduceInto2)
//[9, 16, 16, 0, 9, 16, 16]
```



## 总结

对这些函数实现和使用，更加接触到了 泛型， Any， 时间复杂度，一些语法糖等知识，这篇文章应该是兼顾入门和使用了。但是学以致用，还是要在项目中多多使用才能巩固提升。
