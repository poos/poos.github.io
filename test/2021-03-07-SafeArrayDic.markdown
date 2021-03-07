---
layout:     post                       # 使用的布局（不需要改）
title:      "test" # #重回 Layout                  # 标题
subtitle:   Frame, UIViewAutoresizing, NSLayoutAnchor, Flexbox, SwiftUI, FlutterUI, VFL            #副标题
date:       2019-09-13                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 语法
- Code
---


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
