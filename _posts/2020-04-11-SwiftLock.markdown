---
layout:     post                       # 使用的布局（不需要改）
title:      Swift 中的锁和线程安全                   # 标题
subtitle:   再次认识一下各种锁, 线程理解, 场景解析           #副标题
date:       2020-04-11                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 语法
- Code
---

### 背景

随着业务的复杂，发现 Event Report 的公共数据存取发生一切线程问题，所以再复习一下相关的知识，同时修一下陈旧的 issue...

#### 锁

锁（`lock`）或者互斥锁（`mutex`）是一种结构，用来保证一段代码在同一时刻只有一个线程执行。它们通常被用来保证多线程访问同一可变数据结构时的数据一致性。主要有下面几种锁：

- 阻塞锁（`Blocking locks`）：常见的表现形式是当前线程会进入休眠，直到被其他线程释放。
- 自旋锁（`Spinlocks`）：使用一个循环不断地检查锁是否被释放。如果等待情况很少话这种锁是非常高效的，相反，等待情况非常多的情况下会浪费 CPU 时间。
- 读写锁（`Reader/writer locks`）：允许多个读线程同时进入一段代码，但当写线程获取锁时，其他线程（包括读取器）只能等待。这是非常有用的，因为大多数数据结构读取时是线程安全的，但当其他线程边读边写时就不安全了。
- 递归锁（`Recursive locks`）：允许单个线程多次获取相同的锁。非递归锁被同一线程重复获取时可能会导致死锁、崩溃或其他错误行为。


#### Swift 可用的锁的 Api

- `pthread_mutex_t` 是一个可选择性地配置为递归锁的阻塞锁；
- `pthread_rwlock_t` 是一个阻塞读写锁；
- `DispatchQueue` ~dispatch_queue_t~ 可以用作阻塞锁，也可以通过使用 barrier block 配置一个同步队列作为读写锁，还支持异步执行加锁代码；
- `OperationQueue` 可以用作阻塞锁。与 dispatch_queue_t 一样，支持异步执行加锁代码。
- `NSLock` 是 Objective-C 类的阻塞锁，它的同伴类 NSRecursiveLock 是递归锁。
- `OSSpinLock` 顾名思义，是一个自旋锁。

最后，Objective-C 中的 `@synchronized` 是一个阻塞递归锁。


因为 `pthread` API 在 Swift 中不太好用，而且功能并不比其它 API 多。因此可能更多的在 C 和 Objective-C 中使用，因为它们又好用又高效。所以本文就不讨论前两个了。

`NSLock` 是一个简单的锁定类，易于使用且效率很高。如果需要显式锁定和解锁，那可以用它替代 `DispatchQueue` 。但在大多数情况下不需要使用它。

`OSSpinLock` 对于经常使用锁定、竞争较少且锁定代码运行速度快的用户来说，是一个很好的选择。它的开销非常少，有助于提升性能。如果代码可能会在很长一段时间内保持锁定或竞争很多，那最好不要用这个 API，因为这会浪费 CPU 时间。通常来说，你可以先使用 `DispatchQueue` ，如果这块出现了性能问题，再考虑换成 `OSSpinLock` 。

#### DispatchQueue

~dispatch_queue_t~已经重新更名为 `DispatchQueue` 其受欢迎程度可见一斑。

除了常用的 `DispatchQueue.main.async {}` 它能有各种各样的使用：

- 默认提供了 `sync{}` `async{}` 这些api，所以在操作属性时候不需要显式调用 `lock` `unlock` 的函数。
- 提供了许多其他有用的 API，如使用单个 `dispatch_async` 在后台执行被锁定的代码
- 设置 `定时器` 或其他作用于 `queue` 的事件源，以便它们自动执行锁定
- 作为 `NotificationCenter` 观察者，因为对 `Observer` 的要求是 `Any`

```
NotificationCenter.default.addObserver(self, selector: #selector(oA), name: NSNotification.Name.init("haha"), object: nil)

NotificationCenter.default.post(name: NSNotification.Name.init("haha"), object: nil)
```

- 使用 `OperationQueue` 的属性 `underlyingQueue` 作为 `NSURLSession` 代理。

[developer.apple.com/documentation/foundation/nsoperationqueue/underlyingqueue](https://developer.apple.com/documentation/foundation/nsoperationqueue/1415344-underlyingqueue)

##### Event 的实践

关于本文章的背景，对于 Event 的线程冲突，最小模型和解决方法应该是下边这样：

采用并发队列解决多个读操作的性能问题，然后共享互斥锁来解决写操作的数据竞争问题。对于 iOS 来说它就是 GCD 中的写栏栅 barrier 机制。

```swift
class Dic {
    let serialQueue = DispatchQueue(label: "DicSyncQueue", attributes: .concurrent)

    private var _data = [String: String]()

    var data: [String: String] {
        get {
            return serialQueue.sync {
                return _data
            }
        }
        set {
            serialQueue.async(flags: .barrier) {
                self._data = newValue
            }
        }
    }

    func data(key: String) -> String? {
        serialQueue.sync {
            return _data[key]
        }
    }
    func set(key: String, vaule newValue: String) {
        serialQueue.async(flags: .barrier) {
            self._data[key] = newValue
        }
    }
}


let dic = Dic()

DispatchQueue.concurrentPerform(iterations: 100) { index in
    dic.set(key: "\(index % 100)", vaule: "\(arc4random() % 10)")
}

print("🧩🧩--\(dic.data.count)--🧩--\(Date())🧩")
print("🧩🧩--\(dic.data)--🧩--\(Date())🧩")

```

上面的代码将打印出正确的数据：

```
🧩🧩--100--🧩--2020-03-05 09:20:09 +0000--74🧩
🧩🧩--["87": "7", "56": "5", "7": "8", "70": "9", "60": "6", "62": "8", "47": "9", "76": "2", "4": "3", "3": "9", "63": "0", "27": "3", "72": "7", "9": "1", "11": "5", "32": "9", "22": "7", "96": "5", "44": "2", "36": "2", "29": "3", "93": "1", "19": "0", "0": "0", "85": "5", "90": "1", "38": "4", "42": "9", "33": "1", "18": "5", "16": "8", "12": "5", "69": "5", "86": "7", "78": "9", "82": "8", "84": "2", "57": "4", "53": "4", "14": "6", "8": "7", "13": "9", "46": "0", "1": "0", "54": "5", "6": "4", "40": "7", "41": "3", "66": "1", "92": "9", "15": "6", "98": "0", "31": "1", "28": "4", "64": "1", "94": "0", "67": "3", "35": "7", "99": "4", "26": "1", "83": "3", "45": "2", "30": "8", "49": "2", "37": "0", "97": "3", "51": "5", "68": "7", "21": "5", "95": "4", "55": "4", "24": "1", "10": "0", "73": "5", "65": "4", "52": "6", "81": "3", "61": "3", "77": "7", "50": "2", "48": "8", "25": "5", "75": "3", "39": "9", "17": "3", "59": "7", "43": "3", "91": "8", "5": "8", "20": "1", "23": "1", "88": "3", "79": "8", "71": "4", "80": "7", "2": "3", "34": "9", "89": "3", "74": "2", "58": "8"]--🧩--2021-03-05 09:20:09 +0000🧩
```

#### OperationQueue

这个跟 `DispatchQueue` 虽然是哥俩，但是地位应该比大哥差很多，因为他的 API 复杂，用起来不方便。

奇效在使用继承和自定义上，通过自定义 `cancel` `complete` 可以方便的控制每个节点的结束和取消，在管理依赖的方面非常方便。

*虽然 abandon 的 用处非常少，但是还是写一下。*

```
fileprivate protocol WaitToFinishOperationControlProtocol {
    func complete()
    func abandon()
}

fileprivate final class WaitToFinishOperation: Operation, WaitToFinishOperationControlProtocol {
    func complete() { self.done = true }
    func abandon() { self.abandoned = true; self.done = true }

    private var block: ((WaitToFinishOperationControlProtocol) -> Void) = { _ in }

    private var done = false {
        willSet {
            self.didChangeValue(forKey: "isExecuting")
            self.willChangeValue(forKey: "isFinished")
        }
        didSet {
            self.didChangeValue(forKey: "isExecuting")
            self.didChangeValue(forKey: "isFinished")
        }
    }

    private var abandoned = false {
        willSet {
            self.didChangeValue(forKey: "isCancelled")
        }
        didSet {
            self.didChangeValue(forKey: "isCancelled")
        }
    }

    convenience init(_ block: @escaping (WaitToFinishOperationControlProtocol) -> Void) {
        self.init()
        self.block = block
        self.done = false
    }

    override func main() { block(self) }
    override var isFinished: Bool { done }
    override var isExecuting: Bool { !done }
    override var isCancelled: Bool { abandoned }
    override var isAsynchronous: Bool { true }
}
```
### 最后

借着解决问题复习了一下理解。借用 [swift.gg/locks-thread-safety-and-swift/](https://swift.gg/2018/06/07/friday-qa-2015-02-06-locks-thread-safety-and-swift/) 的结尾吧：

Swift 语言层面并不支持线程同步，但是 Apple 的系统框架有很多好用的 API。`GCD` 和 `DispatchQueue` 非常好用，并且Swift 中的 API 也是如此。虽然 Swift 里没有 `@synchronized` 和原子属性，但我们有其他更好的选择。
