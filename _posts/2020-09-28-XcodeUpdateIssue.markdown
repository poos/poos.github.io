---
layout:     post                       # 使用的布局（不需要改）
title:      "Xcode update" # #重回 Layout                  # 标题
subtitle:   UIButton, XCFramework            #副标题
date:       2020-09-28                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 总结
---


### 背景

开个文章记录一下 Xcode 升级碰到的故事～ 以后不断更新。

更新了一下 Xcode 12 的故事。

#### UIButton

一个神奇的问题：

Xcode 11.5+ 使用 Xcode 直接 run 真机在（iOS 13.4+）会出现。但是关掉之后点击手机中的图标直接打开就没有问题。

使用 Xcode 11.4 以下没有问题。

https://developer.apple.com/forums/thread/652941


> 测试了在最新 Xcode 12.3 仍然没有修复。


为了能够继续愉快的debug，暂时的修改是使用 runtime 再次设置了一下 title：

```swift
#pragma mark - Debug Only -

#if DEBUG

#import <objc/runtime.h>

@interface UIButton ()
@end

@implementation UIButton (Debug)

+ (void)load {
    Method original = class_getInstanceMethod(UIButton.class, @selector(setTitle:));
    Method swizzled = class_getInstanceMethod(UIButton.class, @selector(setAllTitle:));
    method_exchangeImplementations(original, swizzled);
}

- (void)setAllTitle:(NSString *)title {
    [self setAllTitle:title];

    [self setTitle:title forState:UIControlStateNormal];
    [self setTitle:title forState:UIControlStateHighlighted];
    [self setTitle:title forState:UIControlStateDisabled];
    [self setTitle:title forState:UIControlStateSelected];
}
@end

#endif

```


#### XCFramework

这个跟 Xcode 12 息息相关。

进入到 Xcode 12 时代之后，iphone simulator 的打包也会提供 arm64 的编码了，且 arm64 不通用，所以使用原来的合并 framework 的时候就会出现问题。

但是使用 xcframework 就不会有这个问题。

https://medium.com/trueengineering/xcode-and-xcframeworks-new-format-of-packing-frameworks-ca15db2381d3


xcfamework 使用文件夹直接区分了framework。

```
Example.xcframework
|- Info.plist
|- ios-arm64
|- ios-arm64_x86_64-simulator
```

##### BUILD_LIBRARY_FOR_DISTRIBUTION

如果要设置 BUILD_LIBRARY_FOR_DISTRIBUTION YES，有可能需要到项目去设置

##### build 脚本

为了 build 的方便，也可以使用命令行去build。

```
xcodebuild archive -scheme Example_iOS -destination="iOS" -archivePath build/iphoneos -derivedDataPath build/iphoneos -sdk iphoneos SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES

xcodebuild archive -scheme Example_iOS -destination="iOS Simulator" -archivePath build/iphonesimulator -derivedDataPath build/iphonesimulator  -sdk iphonesimulator SKIP_INSTALL=NO BUILD_LIBRARIES_FOR_DISTRIBUTION=YES

rm -rf build/Example.xcframework

xcodebuild -create-xcframework -framework build/iphoneos.xcarchive/Products/Frameworks/Example.framework -framework build/iphonesimulator.xcarchive/Products/Frameworks/Example.framework -output build/Example.xcframework
```

##### 在项目的使用

有可能需要设置 `General` -> `Frameworks, Libraries, and Embedded Content` 到 `Embed and Sign`。




### 最后

More issue waiting to be discovered..
