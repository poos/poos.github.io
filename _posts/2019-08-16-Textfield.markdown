---
layout:     post
title:      Textfield +
subtitle:   错误UI, 可输入字符, 最大字符, 正则匹配等
date:       2019-08-16
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Design
- Code
---

### 背景

接触到一些新的项目，大佬的一些写法还是比较有趣的～ 其中就有对 UITextfiled 的封装，确实封装了很多的功能。用起来也避免了很多 delegate 的写法，可以达到简洁代码的作用。

- maxCount 限制了最大字符数
- floatingText，placeholder，errorMessage 用于设置各个 message
- allowedCharacterSet，matchingRex 用于设置可输入的字符串
- format 用于格式化输入的字符

本文就先简要介绍，并且准备建一个 Pods 来详细的理解和使用一下。// TODO

### 步骤

通过背景介绍，可以看出来定制还是比较高的。那么如何入手呢，那就借鉴一下思想吧：

- 继承 UIView，这样就是规划一个蓝图，在这个地方可以随意发挥，最后注意一下 Layout 就好
- 重写 TextFieldDelegate，通过重写，可以拿到 TextField 的生命周期的每一个节点，把我们想要的逻辑插入即可
- 格局放大，通过 **枚举和协议** 进一步规范可以扩展，丰富功能

下边的话，就先预想一下可能的 enum 和 protocol 并且列一个架子～

### 简单设计

可玩的点还是非常多的：

```
public enum ErrorMode {
    case always
    case whenEndEditing
}

public enum MatchingRex {
    case count(Int)
    case countRange(Int, Int) //Default Int.min ~ Int.max
    case numberValue(Float, Float) //Default Float.min ~ Float.max
    case email(String)
    case rex(String)
}

public enum Format {
    case amount
    case mobile
    case format(String)

    var matchingRex: MatchingRex
}


@objc public protocol CustomTFDelegate {
  //...
}

@IBDesignable
public class CustomTF: UIView {
    @IBOutlet public weak var delegate: CustomTFDelegate?

    public var errorMode: ErrorMode?

    public var format: Format?

    public var floatingText: String?
    public var placeholder: String?
    public var errorMessage: String?

    public var allowedCharacterSet: String?
    public var matchingRexString: MatchingRex?
    public var maxNumberOfCharacters: Int?

}

extension GDTextField : UITextFieldDelegate {
  //...
}
```

### 分享一些正则

作为 **正则爱好者** 的我，怎么能够错过 Textfield 这个绝好的机会～

以业务为基础，可以定制不同的正则表达式～

精心写了一下几个正则，有不明白可以留言交流一下：

```
count:"^.{2}$"
countRange: "^.{0,38}$"
email: "^\w*@\w*\.\w{2,3}$"
password: "^[a-zA-Z]\w{5,17}""
amount: "^\$?(0|([1-9]\d*)|([1-9]\d{0,2}(,\d{3})*))(.\d{1,2})?$"
```

### 总结

本文理论多些，到了实践往往还有很多需要雕琢的地方，等完成之后在完善一下本文的细节～

那么就到这里OK了，期待空下时间，made it。
