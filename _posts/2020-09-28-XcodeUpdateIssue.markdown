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


#### Universal Framework

进入到 Xcode 12 时代之后，iphone simulator 的打包也会提供 arm64 的编码了，且 arm64 不通用，所以使用原来的合并 framework 的时候就会出现问题。虽然可以在合并之前去掉模拟器的 arm64 符号，但是如果到了 Xcode 12.2 新的隐患就会出现。

Xcode 12.2 是最后一个默认支持 Universal Framework 的版本，如果**使用 >= Xcode12.3 在模拟器运行就会抛出错误**：

```
Building for iOS Simulator, but the linked and embedded framework 'My.framework' was built for iOS + iOS Simulator.
```

那解决这个问题的终极方法就是使用新的 xcframework，稍后介绍如果打包和使用。

但是还有一种改变 error 到 warning 的方式，就是去 `BuildSettings` -> `Validate Workspace` 设置 `Yes`。


[Xcode 12.3: Building for iOS Simulator, but the linked and embedded framework was built for iOS + iOS Simulator [duplicate]
](https://stackoverflow.com/questions/65303304/xcode-12-3-building-for-ios-simulator-but-the-linked-and-embedded-framework-wa)



#### XCFramework

使用 xcframework 一个是因为 `使用 >= Xcode12.3 在模拟器运行`, 还有一个是`M1电脑只能用 Xcode12+，且要支持模拟器运行必须打包arm64符号`。虽然这两个都有解决方法，但都不是最终的完美解决。

完美解决就是使用 xcframework ：

[xcode-xcframeworks](https://medium.com/trueengineering/xcode-and-xcframeworks-new-format-of-packing-frameworks-ca15db2381d3)


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
