---
layout:     post                       # 使用的布局（不需要改）
title:      swift下封装转场动画，三方库解决方案          # 标题
subtitle:   TabBar ，异形 NavigationBar，特殊界面的跳转方案      #副标题
date:       2018-07-02                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 动画
---

处理界面专场动画已经有很多库，究其根本就是继承专场动画的协议，然后实现动画的实现即可。


### 一个优秀的框架： [transitiontreasury](https://transitiontreasury.com/)

[github项目地址：DianQK/TransitionTreasury](https://github.com/DianQK/TransitionTreasury)

我在 [使用 Tip ： TabBar 的滑动切换效果](https://blog.dianqk.org/2016/03/07/TransitionTreasury%20%E4%BD%BF%E7%94%A8%20Tip%20%EF%BC%9A%20TabBar%20%E7%9A%84%E6%BB%91%E5%8A%A8%E5%88%87%E6%8D%A2%E6%95%88%E6%9E%9C/) 这篇博客了解到了这个库，另外提一下，作者有很多优秀的博客文章，我也是爬着读了很多。以为最近在使用 RxSwift 开发项目，所以爬到了作者的文章。

> 因为博客文章是讨论 TabBar 滑动转场的，所以提到了 **WXTabBarController**。我也看了看，但是因为自己项目的 TabBar 前两个都有多个tag，而且那个项目已经多年没有更新了， 所以就暂时放弃没有深入探究了。还有 wxtabbar 作者的 [MVVMReactiveCocoa](https://github.com/leichunfeng/MVVMReactiveCocoa) ，OC 的 Reactive 项目，相当全面完整的教程。

TransitionTreasury 是一个开放的项目，有一些作者提供了一些更多的动画，所有的动画都可以在 [transitiontreasury](https://transitiontreasury.com/) 上查看。

如果项目使用了自定义的 专场动画，这个框架会是一个不错的选择


### 轻量级，自定制

**我们在项目中使用了原生的 navBar，和原生的 tabBar，并且希望有一个好的跳转动画**

我的项目中用到了异形的 NavigationBar ，这样在跳转界面的就会有撕裂的感觉，对此进行了优化，使用自定制的跳转动画。

为了项目的通用性，我封装了这个动画，这样在需要使用的时候调用即可。先看一下代码


继承 UINavigationController 然后重新 push 和 pop 的方法。在代理中使用自己的动画。

```swift

class NavigationViewController: UINavigationController {
    let navigationAnimatedTransitioning = NavigationAnimatedTransitioning()

    override func viewDidLoad() {
        super.viewDidLoad()
    }

    override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
        super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
    }

    override init(rootViewController: UIViewController) {
        super.init(rootViewController: rootViewController)
        interactivePopGestureRecognizer?.delegate = self
        delegate = self
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func pushViewController(_ viewController: UIViewController, animated: Bool) {
        self.navigationAnimatedTransitioning.isPush = true
        if viewControllers.count > 0 {
            viewController.hidesBottomBarWhenPushed = true
        }
        super.pushViewController(viewController, animated: animated)
    }

    override func popViewController(animated: Bool) -> UIViewController? {
        let vc = super.popViewController(animated: animated)
        self.navigationAnimatedTransitioning.isPush = false
        return vc
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}

extension NavigationViewController: UINavigationControllerDelegate, UIGestureRecognizerDelegate {

    func navigationController(_ navigationController: UINavigationController, animationControllerFor operation: UINavigationController.Operation, from fromVC: UIViewController, to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        //按需调用
        if fromVC.isKind(of: UIViewController.self) {
            return self.navigationAnimatedTransitioning
        }
        return nil
    }
}

//模仿系统的push,处理了状态栏的问题
class NavigationAnimatedTransitioning: NSObject, UIViewControllerAnimatedTransitioning {
    var isPush = true

    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.25
    }

    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        let width = UIScreen.main.bounds.size.width
        let height = UIScreen.main.bounds.size.height
        let vc = UIApplication.shared.keyWindow?.topMostWindowController()
        //截图
        let screenView = vc?.view.snapshotView(afterScreenUpdates: false) ?? UIView()

        guard let fromVC = transitionContext.viewController(forKey: .from),
            let toView = transitionContext.view(forKey: .to) else { return }

        transitionContext.containerView.addSubview(toView)

        func setAnimationAndFire() {
            if self.isPush {
                fromVC.navigationController?.view.window?.insertSubview(screenView, at: 0)
                fromVC.navigationController?.view.window?.transform = CGAffineTransform.init(translationX: width, y: 0)
                screenView.center = CGPoint.init(x: -width/2, y: height / 2)
            } else {
                fromVC.navigationController?.view.window?.addSubview(screenView)
                fromVC.navigationController?.view.window?.transform = CGAffineTransform.init(translationX: -width / 4, y: 0)
                screenView.center = CGPoint.init(x: width - width / 4, y: height / 2)
            }
            UIView.animate(withDuration: 0.25, delay: 0, options: .curveEaseOut, animations: {
                fromVC.navigationController?.view.window?.transform = CGAffineTransform.init(translationX: 0, y: 0)
                screenView.center = CGPoint.init(x: self.isPush ? width / 4 : width * 3 / 2, y: height / 2)
            }, completion: { (_) in
                screenView.removeFromSuperview()
            })
        }
        UIView.animate(withDuration: 0, animations: {
            transitionContext.completeTransition(true)
        }, completion: { (_) in
            setAnimationAndFire()
        })
    }
}

/** 摘自IQKeyboardManager-IQUIWindow+Hierarchy.swift */
public extension UIWindow {
    /** @return Returns the current Top Most ViewController in hierarchy.   */
    public func topMostWindowController() -> UIViewController? {
        var topController = rootViewController
        while let presentedController = topController?.presentedViewController {
            topController = presentedController
        }
        return topController
    }
}

```

可以看到 实现的方式是 用了 截图 + Window 动画的方式，这是因为需要处理 tabbar 的情况，而 tabbar 在不同的 iOS 版本还是有不同的实现方法，所有为了统一就直接使用 window 实现动画。

### 最后


这个是 Swift 下好用的转场解决方案，如果在 OC 下有更多的选择，在 Swift 下建议尝试一下。

如果碰到问题，去 github 上找一找，有很多前辈已经有了好的通用解决方案。
