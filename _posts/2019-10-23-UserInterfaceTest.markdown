---
layout:     post
title:      UI automated testing.
subtitle:   Frame, UIViewAutoresizing, NSLayoutAnchor, Flexbox, SwiftUI, FlutterUI, VFL
date:       2019-10-23
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Summary
- RE
---

### 背景

在自动化测试中，脚本可以使用每个 UI 控件的唯一 ID 去检查显示的内容，或者去进行功能切换。本文就作一个简单的介绍：

### Apple Inc Document

```
import Foundation
import UIKit
import _SwiftUIKitOverlayShims

//
//  UIAccessibilityIdentification.h
//  UIKit
//
//  Copyright 2010-2018 Apple Inc. All rights reserved.
//

public protocol UIAccessibilityIdentification : NSObjectProtocol {


    /*
     A string that identifies the user interface element.
     default == nil
    */
    @available(iOS 5.0, *)
    var accessibilityIdentifier: String? { get set }
}

extension UIView : UIAccessibilityIdentification {
}

extension UIBarItem : UIAccessibilityIdentification {
}

/*
 Defaults to the filename of the image, if available.
 The default identifier for a UIImageView will be the identifier of its UIImage.
 */
extension UIImage : UIAccessibilityIdentification {
}

```

看到几乎所有的 UI 控件都可以支持设置 accessibilityIdentifier，也就是说，通过设置它，自动化测试可以检查大部分的文字，布局等显示问题～


### XCUITest

其实使用的话，最简单的就是在 UITests Target 里面使用，通过查找 Identifier 来拿到 XCUIElementQuery ，进行验证～

```
class testSUITests: XCTestCase {

    override func test() {
        let navigationBar = XCUIApplication().navigationBars["identifier"]
        navigationBar.title


        testClicksOnRadioButtons() {
        let app = XCUIApplication()
        app.radiobutton[0].tap()
        app.radiobutton[1].tap()
        app.radiobutton[2].tap()
        app.staticTexts[“Simple Test”]
        app.button[0].tap()
    }
}


}

```

https://blog.novoda.com/ui-testing-part-2/


### 其他测试方案

1. Appium

Appium因其在Android和iOS上的灵活性和可用性而广受欢迎，并且可在本机，混合和Web应用程序上运行。对于iOS测试，它使用JSONWireProtocol通过Selenium WebDriver与iOS应用程序进行交互。

```
driver.findElement(By.id("com.example.app:id/radio0")).click();
driver.findElement(By.id("com.example.app:id/radio1")).click();
driver.findElement(By.id("com.example.app:id/radio2")).click();
driver.findElement(By.id("com.example.app:id/editText1")).click();
driver.findElement(By.id("com.example.app:id/editText1")).sendKeys("Simple Test");
driver.findElement(By.name("Answer")).click();
// or alternatively with
driver.findElement(By.id("com.example.app:id/button1")).click();
```

2. XCTest / XCUITest

XCTest和XCUITest是用于iOS应用程序测试的两个不可或缺的测试自动化框架。如上。

3. Detox
Detox是用于移动应用程序的灰盒端到端测试和自动化框架。像Appium和Calabash一样，最大的优势之一是它在Android和iOS上都支持跨平台。唯一的问题是Detox尚不支持在真正的iOS设备上运行。

```
it('should have bitbar question element visible', async () => {
await waitFor(element(by.id('question_screen'))).toBeVisible().withTimeout(6000);
await device.takeScreenshot('app-open');
await expect(element(by.id('question_screen'))).toBeVisible();
await expect(element(by.id('question_screen').withAncestor(by.id('question_text'))));
});
it('should have text input field visible', async () => {
await waitFor(element(by.id('text_input_field'))).toBeVisible().withTimeout(6000);
await expect(element(by.id('text_input_field'))).toBeVisible();
});
```

4. Calabash
Calabash是另一个出色的跨平台框架，可与Android和iOS应用完美配合。与其他框架的主要区别之一是Calabash测试是用Cucumber编写的。这意味着该测试就像规范一样编写，即使对于非技术人员来说也很容易阅读，但是仍然可以由自动化系统执行。

```
Feature: Answer the Question feature
Scenario: As a valid user I want to answer app question
I wait for text "What is the best way to test application on hundred devices?"
Then I press Radio button 0
Then I press Radio button 1
Then I press Radio button 2
Then I enter text "Simple Test" into field with id "editText1"
Then I press view with id "Button1"
```

5. EarlGrey
在某种程度上，EarlGrey是“ iOS浓缩咖啡”。它也是由Google开发和开源的。Google使用此测试框架来测试许多iOS本机应用程序，包括Google Calendar，YouTube等。随着代号的发展，在Espresso和EarlGrey之间可以发现很多相似之处。例如，EarlGrey测试将在尝试与UI交互之前自动等待事件（动画，网络请求等）。

```
- (void)testBasicSelectionAndAction {
[[EarlGrey selectElementWithMatcher::grey_accessibilityID(@"ClickHere")]
performAction:grey_tap()];
// Example of long press with EarlGrey matchers
- (void)testLongPress {
[[EarlGrey selectElementWithMatcher::grey_accessibilityLabel(@"Box")]
performAction:grey_longPressWithDuration(0.5f)];
[[EarlGrey selectElementWithMatcher::grey_accessibilityLabel(@"One Long Press")]
assertWithMatcher:grey_sufficientlyVisible()];// Example of multi-select, visible click on items
- (void)testCollectionMatchers {
id visibleSendButtonMatcher =
grey_allOf(grey_accessibilityID(@"Box"), grey_sufficientlyVisible(), nil);
[[EarlGrey selectElementWithMatcher:visibleSendButtonMatcher]
performAction:grey_tap()];
}
```


https://bitbar.com/blog/top-5-ios-testing-frameworks-with-examples/

尽管iOS应用测试与Android应用测试完全不同，但是您可以使用Appium或Calabash创建可用于进行跨平台测试的测试脚本。话虽如此，每个框架都有其优势，并且每个人对您正在从事的项目都有不同的需求。建议做一些进一步的研究，了解每种框架的更多信息，然后选择自己喜欢的框架。

### 最后

本文只是一个简短的分析，在简单使用上，通过 XCTest / XCUITest 即可，毕竟很少有公司做单独的部门去协调 iOS 和 Android 的自动化测试。
有机会的话在做进一步分析。
