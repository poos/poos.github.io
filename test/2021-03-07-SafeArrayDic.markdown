---
layout:     post                       # 使用的布局（不需要改）
title:      Swift 的 Array 和 Dictionary 源码欣赏，和创建一个线程安全的 Array 和 Dictionary #重回 Layout                  # 标题
subtitle:   Array 源码，线程讨论，          #副标题
date:       2019-09-13                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 语法
- Code
---


### 序

本文的主线是探讨创建一个跟 Swift Array/Dictionary api 一样，并且保证线程安全的的类。还有，为了尽量保持 Swift Collection 的友好函数式体验，也参考一下 Array 和 Dictionary 源码了，尝试理解一下。中间碰遇到的种种问题～


> 有关 Swift 下的线程安全，之前已经有一些探究 [Swift 中的锁和线程安全](https://poos.github.io/2020/04/11/SwiftLock/)，讲了一些锁的基础知识，有兴趣可以去看下。

#### 线程安全

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

可能有人发现了其中一个使用的的是 `struct`, 另一个是 `class`，这并不是笔误，因为 `async` 是逃逸闭包，`struct` 是 **值类型**。

然后在闭包中修改 `self` 又需要 `mutating` ，可是现行语法不支持，所以编译器报错了。虽然有各种曲线救国的方法，但是最简单的修改就是改为 `class` 了。

> ~~为了解决这个问题，可以尝试使用 delegates 把调用抛出，然后让 delegate 调用当前 struct 的 mutating函数（fileprivate）。把整个 DispatchQueue 放到 delegate 去处理，使用 delegate 的 DispatchQueue 作为唯一队列。~~ 并没有用，因为 逃逸闭包 并不能操作 inout。所以这条路是堵得死死的。

> 另外一种就是尝试使用指针操作，个人觉得还是不要对栈内存的数据进行指针操作了，本来使用栈就是为了轻松愉快的，搞得太复杂了多不好。

其实使用 `class` 并没有太大的问题，因为存储数据本质上还是在一个 `var Array` 里面放着，上层封装只是为了解决线程问题。

#### 为 Safe 类添加其他函数

为了让我们做出的类跟 swift 默认的 Array 和 Dictionary 一样方便好用，以 Dictionary 为例：

- 支持 subscript 下标，👆上面的一部分已经涉及到
- 支持 Sequence，序列，使类的对象可以遍历读出
- 支持 Collection，集合， `first(where:), filter(), reduce, map ...`
- 支持 ExpressibleByDictionaryLiteral， `字面量` 初始化
- 支持 Equatable
- 支持 Hashable

除了以上比较关键的值，还有一些 debug 时候用的：

- 支持 CustomStringConvertible, CustomDebugStringConvertible，description print
- 支持 CustomReflectable，format 的 print 值

还有一些 Dictionary 支持的关于指针的操作：

```
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







```swift
public class SafeArray<T>: RangeReplaceableCollection {
    public typealias Index = Int
    public typealias Element = T
    public typealias SubSequence = SafeArray
    let serialQueue = DispatchQueue(label: "SafeArrayDispatchQueue", attributes: .concurrent)

    fileprivate var elements: [Element]

    public var startIndex: Int { return elements.startIndex }

    public var endIndex: Int { return elements.endIndex }

    public subscript(_ index: Index) -> Iterator.Element {
        get {
            serialQueue.sync { elements[index] }
        }
        set(newValue) {
            serialQueue.async(flags: .barrier) { self.elements[index] = newValue }
        }
    }

    public subscript(bounds: Range<Index>) -> SubSequence {
        return SafeArray(elements[bounds])
    }

    required public init() {
        elements = [Element]()
    }

    public init(_ incomingE: [Element]) {
        elements = incomingE
    }

    public func index(after i: Index) -> Index {
        return serialQueue.sync { elements.index(after: i) }
    }

    public func replaceSubrange<C>(_ subrange: Range<SafeArray.Index>, with newElements: __owned C) where C : Collection, Element == C.Element {
        serialQueue.async(flags: .barrier) { self.elements.replaceSubrange(subrange, with: newElements) }
    }
}
```

class
`DispatchQueue(label: "SafeArrayDispatchQueue", attributes: .concurrent)`
`serialQueue.async(flags: .barrier) { self.elements[index] = newValue }`


struct
`DispatchQueue(label: "SafeArrayDispatchQueue")`
`serialQueue.sync { self.elements[index] = newValue }`


```swift
public class SafeArray<T>: RangeReplaceableCollection {
    public typealias Index = Int
    public typealias Element = T
    public typealias SubSequence = SafeArray
    let serialQueue = DispatchQueue(label: "SafeArrayDispatchQueue", attributes: .concurrent)

    fileprivate var elements: [Element]

    public var startIndex: Int { return elements.startIndex }

    public var endIndex: Int { return elements.endIndex }

    public subscript(_ index: Index) -> Iterator.Element {
        get {
            serialQueue.sync { elements[index] }
        }
        set(newValue) {
            serialQueue.async(flags: .barrier) { self.elements[index] = newValue }
        }
    }

    public subscript(bounds: Range<Index>) -> SubSequence {
        return SafeArray(elements[bounds])
    }

    required public init() {
        elements = [Element]()
    }

    public init(_ incomingE: [Element]) {
        elements = incomingE
    }

    public func index(after i: Index) -> Index {
        return serialQueue.sync { elements.index(after: i) }
    }

    public func replaceSubrange<C>(_ subrange: Range<SafeArray.Index>, with newElements: __owned C) where C : Collection, Element == C.Element {
        serialQueue.async(flags: .barrier) { self.elements.replaceSubrange(subrange, with: newElements) }
    }
}
```
