---
layout:     post                       # 使用的布局（不需要改）
title:      Swift 的 Array 和 Dictionary 源码欣赏，和创建一个线程安全的 Array 和 Dictionary #重回 Layout                  # 标题
subtitle:   Array 源码, 线程安全, Collection, ExpressibleByDictionaryLiteral, RangeReplaceableCollection 等协议         #副标题
date:       2019-09-13                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 总结
- Code
---


### 序

本文的主线是探讨创建一个跟 Swift Array/Dictionary api 一样，并且保证线程安全的的类。还有，为了尽量保持 Swift Collection 的友好函数式体验，也参考一下 Array 和 Dictionary 源码了，尝试理解一下。中间碰遇到的种种问题～


> 有关 Swift 下的线程安全，之前已经有一些探究 [Swift 中的锁和线程安全](https://poos.github.io/2020/04/11/SwiftLock/)，讲了一些锁的基础知识，有兴趣可以去看下。

### 线程安全

为了线程安全，可以在不同线程调用的时候，最终都进入同一个队列等待数据处理或者返回。有两种处理方法：

- 如果队列是同步的，那么所有的任务 FIFO（先进先出，按顺序执行），即可解决问题。

```swift
public struct SafeArray<T> {
    let serialQueue = DispatchQueue(label: "SafeArrayDispatchQueue")
    fileprivate var elements: [T]
    ...

    public subscript(_ index: Int) -> T {
        get {
            serialQueue.sync { elements[index] }
        }
        set(newValue) {
            serialQueue.sync { self.elements[index] = newValue }
        }
    }
}
```

- 如果队列是支持异步的，那么所有的读操作 FIFO ，写操作异步插入到某个原子操作结束之后

```swift
public class SafeArray<T> {
    let serialQueue = DispatchQueue(label: "SafeArrayDispatchQueue", attributes: .concurrent)
    fileprivate var elements: [T]
    ...

    public subscript(_ index: Int) -> T {
        get {
            serialQueue.sync { elements[index] }
        }
        set(newValue) {
            serialQueue.async(flags: .barrier) { self.elements[index] = newValue }
        }
    }
}
```

上面的代码比较简单，用了个泛型作为数组内部的值类型，然后定义了下标，使得这个类的对象支持使用下标的方式进行存取值。然后就是队列的处理了：

第一个使用了同步队列，可以理解为只有一个通道，所有原子操作一个个依次执行。

第二个使用了支持 `.concurrent` 进行标记，那么除了正常的读操作的`sync`队列之外，可以使用 `async` 进行操作，将写操作提前放入某个原子操作完成之后。

> Summary The queue schedules tasks concurrently.
Declaration static let concurrent: DispatchQueue.AttributesIf this attribute is not present, the queue schedules tasks serially in first-in, first-out (FIFO) order.


#### Struct 和 Class


可能有人发现了其中一个使用的的是 `struct`, 另一个是 `class`，这并不是笔误，因为 `async` 是逃逸闭包，`struct` 是 **值类型**。

然后在闭包中修改 `self` 又需要 `mutating` ，可是现行语法不支持，所以编译器报错了。虽然有各种曲线救国的方法，但是最简单的修改就是改为 `class` 了。

> ~~为了解决这个问题，可以尝试使用 delegates 把调用抛出，然后让 delegate 调用当前 struct 的 mutating函数（fileprivate）。把整个 DispatchQueue 放到 delegate 去处理，使用 delegate 的 DispatchQueue 作为唯一队列。~~ 并没有用，因为 逃逸闭包 并不能操作 inout。所以这条路是堵得死死的。

> 另外一种就是尝试使用指针操作，个人觉得还是不要对栈内存的数据进行指针操作了，本来使用栈就是为了轻松愉快的，搞得太复杂了多不好。

顺便再看下 Struct 和 Class 的区别吧：[Swift中的Class和Struct](https://juejin.cn/post/6844903775816155144)


**相同点**

- 都能定义 property、method、initializers
- 都支持 protocol、extension

**不同点**

- class 是引用类型；struct 是值类型
- class 支持继承；struct不支持继承
- class 声明的方法修改属性不需要 mutating 关键字；struct 需要
- class 没有提供默认的 memberwise initializer；struct 提供默认的 memberwise initializer
- class 支持引用计数(Reference counting)；struct不支持
- class 支持 Type casting；struct不支持
- class 支持 Deinitializers；struct不支持


### 为 SafeDictionary 类添加其他函数

为了让我们做出的类跟 swift 默认的 Array 和 Dictionary 一样方便好用，以 Dictionary 为例：

- 支持 subscript 下标，👆上面的一部分已经涉及到
- 支持 Sequence，序列，使类的对象可以遍历读出 `forEach({}), first(where:), filter(),`
- 支持 Collection，支持之后能够使用跟 Sequence 一样的高阶函数，稍后再扩展
- 支持 ExpressibleByDictionaryLiteral， `字面量` 初始化
- 支持 Equatable
- 支持 Hashable

除了以上比较关键的值，还有一些 debug 时候用的：

- 支持 CustomStringConvertible, CustomDebugStringConvertible，description print
- 支持 CustomReflectable，format 的 print 值

还有一些 Dictionary 支持的关于指针的操作：

```swift
extension Dictionary: ConcurrentValue, UnsafeConcurrentValue
  where Key: ConcurrentValue, Value: ConcurrentValue { }
extension Dictionary.Keys: ConcurrentValue, UnsafeConcurrentValue
  where Key: ConcurrentValue, Value: ConcurrentValue { }
extension Dictionary.Values: ConcurrentValue, UnsafeConcurrentValue
  where Key: ConcurrentValue, Value: ConcurrentValue { }
extension Dictionary.Keys.Iterator: ConcurrentValue, UnsafeConcurrentValue
  where Key: ConcurrentValue, Value: ConcurrentValue { }
extension Dictionary.Values.Iterator: ConcurrentValue, UnsafeConcurrentValue
  where Key: ConcurrentValue, Value: ConcurrentValue { }
extension Dictionary.Index: ConcurrentValue, UnsafeConcurrentValue
  where Key: ConcurrentValue, Value: ConcurrentValue { }
extension Dictionary.Iterator: ConcurrentValue, UnsafeConcurrentValue
  where Key: ConcurrentValue, Value: ConcurrentValue { }
```

> 上两个宝库， apple 的 Array 和 Dictionary 的源码：
> [github.com/apple/swift/blob/main/stdlib/public/core/Array](https://github.com/apple/swift/blob/main/stdlib/public/core/Array.swift)
> [github.com/apple/swift/blob/main/stdlib/public/core/Dictionary](https://github.com/apple/swift/blob/main/stdlib/public/core/Dictionary.swift)

#### Sequence

Sequence 协议为我们带来了很多有用的函数，这些函数可以让我们方便的使用函数操作数组，为了能够直接使用下面的那些函数，就来看下这个协议怎么实现吧：
- shuffled
- map
- filter
- forEach
- first(where）
- split
- suffix
- dropFirst
- prefix
- enumerated
- ...

为此，我们需要实现两个协议：`Sequence, IteratorProtocol`, 先上一段代码：

```swift
public struct SafeDictionary<Key, Value> where Key: Hashable {
    let serialQueue = DispatchQueue(label: "SafeDictionaryDispatchQueue")

    private var _elements = [Key: Value]()
    private var _index: Dictionary<Key, Value>.Index?
    ...
}
```

```swift
extension SafeDictionary: Sequence, IteratorProtocol {
    public typealias Element = (key: Key, value: Value)

    public mutating func next() -> Element? {
        return serialQueue.sync {
            if _index == nil {
                _index = _elements.startIndex
            }
            let index = _index!
             guard _elements.endIndex != index else {
                defer {
                    _index = nil
                }
                return nil
            }

            defer {
                _index = _elements.index(after: index)
            }

            return _elements[index]
        }
    }
}
```

上面的代码中 `Sequence` 标记了 struct 是可以循环的内容，`IteratorProtocol` 标记了在序列上进行迭代的内容（Element）。

如果一个类型同时遵守这两个协议，那要做的就是实现一个 `next() -> Element?` 方法，用来返回下一个值，当所以的值都迭代完成，返回 nil 即可。

- 使用 `_index` 标记当前遍历到哪个地方了， 然后在每次重新遍历的时候重新标记。因为 Extension 没法使用存储属性，所以放在了类里面。
- 判断 当前 `index` 是否已经遍历到了最后一个， `_elements.endIndex` 即可书写相关逻辑代码

现在就可以愉快的使用那些高阶函数了～

#### Collection

`Collection` 协议是比 Sequence 更底层的协议，它定义了一些关于 `Index` 的操作和返回。

> 熟悉，没错，就是 `Sequence, IteratorProtocol` 具体实现使用到的 `_elements.startIndex` `_elements.endIndex` 之类的。


Swift 标准库中的所有集合都遵循该 Collection 协议，而该协议又继承自该 Sequence 协议。通过使自定义集合符合这两个协议，它可以完全免费地利用所有标准集合操作（​​例如 `forEach` 和 `filter`  和👆上面一大堆）。

看下代码：

```swift
extension SafeDictionary: Collection {
    public typealias Index = Dictionary<Key, Value>.Index
    public var startIndex: Index {
        elements.startIndex
    }

    public var endIndex: Index {
        elements.endIndex
    }

    public subscript(position: Index) -> (key: Key, value: Value) {
        elements[position]
    }

    public func index(after i: Index) -> Index {
        elements.index(after: i)
    }
}
```

需要指定 `Index` 类型，并且实现 `startIndex`, `endIndex`, `subscript(position:)->Item`,`index(after:)->Index`。

单看这几个函数就是一个完整的序列查询。没错了，现在可以把 `Sequence, IteratorProtocol` 相关的代码去掉了，毕竟看起来清爽多了！

#### ExpressibleByDictionaryLiteral

除了 ByDictionary 还有一些其他 其他的 `String`, `Int`, `Array` 等其他的字面量支持协议。

> 看到了 ExpressibleByStringLiteral 你会不会有一些sao操作的想法，解析个amout，date什么的不在话下～

不多说，这一部分比较简单。实现 `ExpressibleByDictionaryLiteral` 协议，只需实现一个指定的初始化函数 `init(dictionaryLiteral:)`。


```swift
extension SafeDictionary: ExpressibleByDictionaryLiteral {
    public init(dictionaryLiteral elements: (Key, Value)...) {
        let elements = elements.reduce(into: Dictionary<Key, Value>(), { (result, item) in
            result[item.0] = item.1
        })
        self.init(_elements: elements)
    }
}
```

上面的代码因为是个可变多参数，所以会有类型不匹配的错误。没关系，使用 `reduce` 转换一下即可。

现在我们定义的字典也能够使用字面量进行初始化或者赋值了，随便测试也不会有问题～

```swift
var dic = SafeDictionary<Int, Int>()

dic = [1: 1,
       2: 3,
       5: 8]
```


#### Equatable

`Equatable` 用于比较两个 `Collection` 相等, 可以想到的方式就是直接调用 `_elements` 相应的 Equatable 方法即可：

```swift
extension SafeDictionary: Equatable where Value: Equatable {
    public static func == (lhs: SafeDictionary<Key, Value>, rhs: SafeDictionary<Key, Value>) -> Bool {
        return lhs.elements == rhs.elements
    }
}
```

小插曲是只写 `extension SafeDictionary: Equatable` 编译器会报错 *Operator function '==' requires that 'Value' conform to 'Equatable'*，添加 ` where Value: Equatable` 即可。

现在就可以使用 `==` 开始判断了：

```swift
dic == [1: 1,
        2: 3,
        3: 6,
        9: 15]
```


#### Hashable

`Hashable` 继承自 `Equatable`，同样的，我们只是简单对接即可。

```swift
extension SafeDictionary: Hashable where Value: Hashable {
    public var hashValue: Int {
        return elements.hashValue
    }

    public func hash(into hasher: inout Hasher) {
        elements.hash(into: &hasher)
    }
}
```

### 为 SafeArray 类添加其他函数

Array 支持的协议跟 Dictionary 差不多，但是多了一个 `RangeReplaceableCollection`，可以使用 `+=` 之类的组合操作了

官方的 Array 主要支持的协议是：

- 支持 subscript 下标，👆上面的一部分已经涉及到
- 支持 Sequence，序列，使类的对象可以遍历读出 `forEach({}), first(where:), filter(),`
- 支持 Collection，支持之后能够使用跟 Sequence 一样的高阶函数，稍后再扩展
- 支持 ExpressibleByDictionaryLiteral， `字面量` 初始化
- 支持 Equatable
- 支持 Hashable
**- RangeReplaceableCollection**

除了以上比较关键的值，还有一些 debug 时候用的：

- 支持 CustomStringConvertible, CustomDebugStringConvertible，description print
- 支持 CustomReflectable，format 的 print 值

还有一些关于指针操作的协议：

```swift
extension Array: ConcurrentValue, UnsafeConcurrentValue where Element: ConcurrentValue { }
```

#### RangeReplaceableCollection

RangeReplaceableCollection 继承自 Collection 协议，在 Collection 协议实现了 `startIndex`, `endIndex`, `subscript(bounds:)->Item`,`index(after:)->Index` 的函数的基础上，实现 `replaceSubrange(::)` 即可。

```swift
extension SafeArray: RangeReplaceableCollection {
    public func replaceSubrange<C>(_ subrange: Range<SafeArray.Index>, with newElements: __owned C) where C : Collection, Element == C.Element {
        serialQueue.async(flags: .barrier) { self.elements.replaceSubrange(subrange, with: newElements) }
    }
}
```

现在对于数组的操作即可使用 `+=` 符号来复制和操作了。

```swift
var sarr = SafeArray<Int>()

sarr += [1, 2, 3]
```

### 结尾

终于写到结尾了，码了挺久的这篇文章。越写越发现 swift 的协议真的是非常强大，基于你实现的继承函数，即可提供非常多好用的高阶函数式。

还有，如果文章有哪些地方有误，可以指出一下～：

---
最后是成品代码全家桶：


```swift

import UIKit

// MARK: - SafeDictionary
public struct SafeDictionary<Key, Value> where Key: Hashable {
    let serialQueue = DispatchQueue(label: "SafeDictionaryDispatchQueue")

    fileprivate var _elements = [Key: Value]()

    public init() {}

    init(_elements: [Key: Value]) {
        self.init()
        serialQueue.sync {
            self._elements = _elements
        }
    }

    var elements: [Key: Value] {
        get {
            return serialQueue.sync {
                return _elements
            }
        }
        set {
            serialQueue.sync {
                _elements = newValue
            }
        }
    }

    public subscript(key: Key) -> Value? {
        get {
            serialQueue.sync {
                return _elements[key]
            }
        }
        set(newValue) {
            serialQueue.sync {
                self._elements[key] = newValue
            }
        }
    }
}

//extension SafeDictionary: Sequence, IteratorProtocol {
//    public typealias Element = (key: Key, value: Value)
//
//    public mutating func next() -> Element? {
//        return serialQueue.sync {
//            if _index == nil {
//                _index = _elements.startIndex
//            }
//            let index = _index!
//             guard _elements.endIndex != index else {
//                defer {
//                    _index = nil
//                }
//                return nil
//            }
//
//            defer {
//                _index = _elements.index(after: index)
//            }
//
//            return _elements[index]
//        }
//    }
//}

extension SafeDictionary: Collection {
    public typealias Index = Dictionary<Key, Value>.Index
    public var startIndex: Index {
        elements.startIndex
    }

    public var endIndex: Index {
        elements.endIndex
    }

    public subscript(position: Index) -> (key: Key, value: Value) {
        elements[position]
    }

    public func index(after i: Index) -> Index {
        elements.index(after: i)
    }
}

extension SafeDictionary: ExpressibleByDictionaryLiteral {
    public init(dictionaryLiteral elements: (Key, Value)...) {
        let elements = elements.reduce(into: Dictionary<Key, Value>(), { (result, item) in
            result[item.0] = item.1
        })
        self.init()
        self.elements = elements
    }
}

extension SafeDictionary: Equatable where Value: Equatable {
    public static func == (lhs: SafeDictionary<Key, Value>, rhs: SafeDictionary<Key, Value>) -> Bool {
        return lhs.elements == rhs.elements
    }
}

extension SafeDictionary: Hashable where Value: Hashable {
    public var hashValue: Int {
        return elements.hashValue
    }

    public func hash(into hasher: inout Hasher) {
        elements.hash(into: &hasher)
    }
}

// MARK: - SafeArray
public struct SafeArray<Element> {
    let serialQueue = DispatchQueue(label: "SafeArrayDispatchQueue")

    fileprivate var _elements = [Element]()

    public init() {}

    var elements: [Element] {
        get {
            return serialQueue.sync {
                return _elements
            }
        }
        set {
            serialQueue.sync {
                _elements = newValue
            }
        }
    }

    public func index(after i: Index) -> Index {
        return serialQueue.sync { elements.index(after: i) }
    }
}

extension SafeArray: RangeReplaceableCollection {
    public typealias Index = Int
    public typealias Element = Element
    public typealias SubSequence = SafeArray

    public var startIndex: Int { return elements.startIndex }

    public var endIndex: Int { return elements.endIndex }

    public subscript(bounds: Int) -> Element {
        elements[bounds]
    }

    public mutating func replaceSubrange<C>(_ subrange: Range<SafeArray.Index>, with newElements: __owned C) where C : Collection, Element == C.Element {
        elements.replaceSubrange(subrange, with: newElements)
    }
}


// MARK: - Test
class ViewController: UIViewController {
    var dic = SafeDictionary<Int, Int>()

    override func viewDidLoad() {
        super.viewDidLoad()
        dic = [1: 1,
               2: 3,
               3: 6,
               9: 15]

        print("🧩🧩--\(dic)--🧩--\(Date())--\(#function)--\(#line)🧩")
        let isSame = dic == [1: 1,
                             2: 3,
                             3: 6,
                             9: 15]
        print("🧩🧩--\(isSame)--🧩--\(Date())--\(#function)--\(#line)🧩")

        print("🧩🧩--\(dic)--🧩--\(Date())--\(#function)--\(#line)🧩")

        DispatchQueue.concurrentPerform(iterations: 100) { index in
            dic[index] = Int(arc4random() % 100)
        }

        print("🧩🧩--\(dic)--🧩--\(Date())--\(#function)--\(#line)🧩")

        SafeDictionary<Int, Int>() == SafeDictionary<Int, Int>()
    }


    @IBAction func bttonAction(_ sender: Any) {

        let num10 = dic.first(where: { $0.key == 10 })
        print("🧩🧩--\(num10)--🧩--\(Date())--\(#function)--\(#line)🧩")
        let first = dic.first(where: { (item) -> Bool in
            item.key == 23
        })

        print("🧩🧩--\(first)--🧩--\(Date())--\(#function)--\(#line)🧩")

        DispatchQueue.concurrentPerform(iterations: 10) { index in
            let first = dic.first(where: { (item) -> Bool in
                item.key == index
            })
            print("🧩🧩--\(first)--🧩--\(Date())--\(#function)--\(#line)🧩")
        }
        dic.enumerated()
        //        Dictionary<Int, Int>().firstIndex(where: <#T##((key: Int, value: Int)) throws -> Bool#>)

        var sarr = SafeArray<Int>()

        sarr += sarr

    }
}

```
