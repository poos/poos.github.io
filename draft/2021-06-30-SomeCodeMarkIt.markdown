---
layout:     post
title:      
hide-in-nav: true
subtitle:   Note 一些杂七杂八的代码片段
date:       2021-06-30
author:     poos
header-img: img/post-bg-2015.jpg
catalog: true
tags:
- Notes
---


<!-- test  -->

############################################

// 去除头尾空格
.trimmingCharacters(in: .whitespacesAndNewlines)

############################################

//
DateFormatter.localizedString(from: Date(), dateStyle: .short, timeStyle: .long)
Data().description(with: .current)

############################################

//
print("🧩🧩--\(DateFormatter.localizedString(from: Date(), dateStyle: .short, timeStyle: .long))--🧩--\(Date().description(with: .current))--\(#function)--\(#line)🧩")

############################################

////dynamic///

############################################

//
extension Optional where Wrapped == ()->Void {
    mutating func append(_ c: @escaping () -> Void) {
        let base = self
        self = {
            base?()
            c()
        }
    }
}

typealias CustomC = (()->Void)?
extension CustomC {
    mutating func append(_ c: @escaping () -> Void) {
        let base = self
        self = {
            base?()
            c()
        }
    }
}
//

############################################

/////  IBInspectable

extension UIView {

    //cut irrelevant code for SO Question

    @IBInspectable
    var masksToBounds: Bool {
        get {
            return layer.masksToBounds
        }
        set {
            layer.masksToBounds = newValue
        }
    }


    // Shadow handling
    @IBInspectable
    var shadowColor: UIColor? {
        get {
            if let color = layer.shadowColor {
                return UIColor(cgColor: color)
            }
            return nil
        }
        set {
            if let color = newValue {
                layer.shadowColor = color.cgColor
            } else {
                layer.shadowColor = nil
            }
        }
    }

    @IBInspectable
    var shadowOpacity: Float {
        get {
            return layer.shadowOpacity
        }
        set {
            layer.shadowOpacity = newValue
        }
    }


    @IBInspectable
    var shadowRadius: CGFloat {
        get {
            return layer.shadowRadius
        }
        set {
            layer.shadowRadius = newValue
        }
    }

    @IBInspectable
    var shadowOffset: CGSize {
        get {
            return layer.shadowOffset
        }
        set {
            layer.shadowOffset = newValue
        }
    }


}

############################################

///UT
   XCTAssert(app.tabBars.buttons["aa"].isHittable)
        app.tabBars.buttons["aa"].tap()
        app.tabBars.buttons["aa"].swipeLeft()

############################################

    open override func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?) {
        self.becomeFirstResponder()
        self.selectedTextRange = self.textRange(from: self.endOfDocument, to: self.endOfDocument)
    }
    
    open override func gestureRecognizerShouldBegin(_ gestureRecognizer: UIGestureRecognizer) -> Bool {
        return false
    }

############################################

############################################

############################################

############################################

############################################

############################################

############################################

############################################

############################################

############################################

############################################

############################################