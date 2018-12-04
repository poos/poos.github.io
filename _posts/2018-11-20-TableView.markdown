---
layout:     post                       # 使用的布局（不需要改）
title:      TableView 的终极优化：滑动，布局，事件，更新              # 标题
subtitle:   图片和视图优化，数据处理优化，自动布局和异步加载 Texture（ASDK），复杂 table 和人性化 UI，cell 事件哪里响应，数据同步更新            #副标题
date:       2018-08-08                 # 时间
author:     poos                         # 作者
header-img: img/post-bg-2015.jpg     #这篇文章标题背景图片
catalog: true                         # 是否归档
tags:                                #标签
- 优化
---

问题：

1. UIView 优化：图片优化？层优化？圆角优化？背景和透明度优化？
2. 自动布局：是选择 Frame 布局优先性能，auto layout 牺牲性能，如何权衡。有没有二者兼备的解决方案。
3. 数据优化：需要限制行数，需要调整图片？
4. 事件处理：cell 中的多个事件如何处理：点赞，关注，评论，分享，跳转个人页，跳转问题页，跳转回答页等
3. 状态处理：TableView 存在多个状态：冷启动，顶部推荐，正常显示，中间插入广告？如何同步，如何准确的，有条理的显示
4. cell 更新：已经加载的 cell 数据如何更新：点赞数，关注关系等


### UIView 优化

此部分优化有一定作用，但是质的提升是在第二部分 **使用 Texture（ASDK）**

1. 当 table 滑动卡顿时候，我最先检测了 **各个 UI 控件** 在 模拟器的层显示, 通过对这些检测优化了一些性能

```
Color Blended Layers

Color Copied Images

Color Misaligned Images

Color Offscreen-Rendered

```

2. **减少视图层级**

简言之：将所有的视图都放在 content 上。

可想而知会造成代码结构混乱和不容易维护

3. 透明和圆角

**透明确实可以提高性能**

**圆角问题着重说一下**

网上有很多添加圆角的方案：[iOS高效简易添加圆角](https://juejin.im/post/5a30dab36fb9a045204c35f3) 等等。

但是我在ios 11 +的设备测试性能时候发现圆角对性能的影响几乎没有了，当然针对我的工程，也有可能是我的工程圆角不是醉消耗性能的地方。

这部分不是重点，所以就展开这么长的篇幅。

### 自动布局 与 frame 布局

不得不承认 frame 布局的效率真的是高。在对比以前的项目（frame 布局，优化缓存 cellHeight），和初步优化的 SnpKit 布局，性能真的查了好多。

但是 autolayout 有着太多的好处：**布局方便**，**适配多尺寸方便（横屏、iPad）**，**label 等 UI 空间自动撑满尺寸**

所以最后当然要披荆斩棘的去使用更为先进的 autolayout 了。

至于带来的开销问题，就去找另外的方式去优化了

**神器 [Texture / ASDK](http://texturegroup.org)**

神器在手，如果使用
[Texture(ASDK)的理解和使用 、主要是使用方面：包含详细的使用方法，UI 类（ASTableCellNode，ASScollNode），布局（FlexBox布局），优化（ASImageNode，对接本地Kingfisher图片下载）等 ](https://poos.github.io/2018/08/08/Texture/)


[Texture(ASDK)自定义 Node 和 优化 Tab 框架 、自定义 ASDisplayNode ；拆分复杂的 Node 为简单的 Node 组件 等 ](https://poos.github.io/2018/08/18/Texture1/)





但是通常我们只需要关心下面几个方法

```
//重写初始化方法以传入需要的参数
init() {
}


//主线程调用，在 didLoad 做一些事件绑定的操作
override func didLoad() {
    super.didLoad()
}

//后台线程调用，做布局的最后调整
override func layout() {
    super.layout()
}


//后台线程调用，实现布局
override func layoutSpecThatFits(_ constrainedSize: ASSizeRange) -> ASLayoutSpec {
    return ASInsetLayoutSpec.init(insets: UIEdgeInsets.init(space: 0), child: moreButton)
}
```

#### 例子：九宫格图片展示器

话不多说，来代码中继续讨论：

**需求** ：类似微信中的图片显示器，**1图按照当前尺寸，多图安装9宫格排列**

**分析** ：一种更加方案化的解决方式是使用 CollectionView；通过不同的 type 使用不同的 size 即可。

##### 创建类型

可以通过枚举方便的确定类型

```
private enum PicturesStyle {
    case larage
    case small

    init(_ count: Int) {
        if count == 1 {
            self = .larage
        } else {
            self = .small
        }
    }
}
```

##### 创建9宫格单个 Cell

单个 cell 创建比较简单，传入图片 url 和 需要在 collection 中显示的方格的自身宽带

```
private class CollectionCellNode: ASCellNode {
    let imageNode = NetworkImageNode().then { (node) in
        node.defaultImage = UIImage.init(named: "cardPlaceholder")
    }
    let width: CGFloat

    init(_ path: String, width: CGFloat) {
        self.width = width
        super.init()
        automaticallyManagesSubnodes = true

        imageNode.url = URL.init(string: path)
        imageNode.isLayerBacked = true
    }

    override func layout() {
        super.layout()
        imageNode.layer.borderWidth = 0.5
        imageNode.layer.borderColor = UIColor.hex("ececec").cgColor
    }

    override func layoutSpecThatFits(_ constrainedSize: ASSizeRange) -> ASLayoutSpec {
        imageNode.style.preferredSize = CGSize.init(width: self.width, height: self.width)
        return ASInsetLayoutSpec.init(insets: UIEdgeInsets.init(space: 0), child: imageNode)
    }
}
```

##### 创建图片展示器

这个视图虽然是个复杂的视图，但是分析优化即可。

如果是单个图就直接返回图片；如果是多个图片就返回 collectionView，并且设置大小为 contentSize 即可完整显示。

```
class PicturesView: ASDisplayNode {

    let imagePaths: [String]
    let picStyle: PicturesStyle
    let width: CGFloat

    lazy var layout = CollectionLayout()

    var collection: ASCollectionNode!
    lazy var imageNode: ASCollectionNode = NetworkImageNode().then { (node) in
        node.defaultImage = UIImage.init(named: "cardPlaceholder")
    }

    init(_ images: [String]) {

        imagePaths = images
        picStyle = PicturesStyle.init(images.count)

        switch picStyle {
        case .larage:
            imageNode.url = URL.init(string: images[0]!)
        case .small:
            collection = ASCollectionNode.init(collectionViewLayout: layout)
            width = (AppDefaults.screenW - 40 - 5) / 3
        }

        super.init()
        automaticallyManagesSubnodes = true
        view.isUserInteractionEnabled = false
    }

    override func didLoad() {
        super.didLoad()
        collection.delegate = self
        collection.dataSource = self
        collection.autoresizesSubviews = true
    }

    override func layoutSpecThatFits(_ constrainedSize: ASSizeRange) -> ASLayoutSpec {
       guard picStyle == .small else {
           return ASInsetLayoutSpec.init(insets: UIEdgeInsets.init(space: 0), child: imageNode)
       }
        var size = CGSize.init(width: AppDefaults.screenW, height: 0)
        switch picStyle {
        case .larage:
            size.height = width
        case .small:
            let line = Int((imagePaths.count - 1) / 3 + 1)
            size.height = width * CGFloat(line) + 2 * CGFloat(line - 1)
        }
        collection.style.preferredSize = size
        return ASInsetLayoutSpec.init(insets: UIEdgeInsets.init(space: 0), child: collection)
    }
}



extension PicturesView: ASCollectionDelegate, ASCollectionDataSource {

    func collectionNode(_ collectionNode: ASCollectionNode, constrainedSizeForItemAt indexPath: IndexPath) -> ASSizeRange {
        let size = CGSize.init(width: width, height: width)
        return ASSizeRange.init(min: CGSize.zero, max: size)
    }

    func collectionNode(_ collectionNode: ASCollectionNode, numberOfItemsInSection section: Int) -> Int {
        return imagePaths.count
    }

    func collectionNode(_ collectionNode: ASCollectionNode, nodeBlockForItemAt indexPath: IndexPath) -> ASCellNodeBlock {

        let path = imagePaths[indexPath.row]
        let cellNodeBlock = { () -> ASCellNode in
            let cellNode = CollectionCellNode.init(path, width: self.width)
            cellNode.neverShowPlaceholders = true
            DispatchQueue.main.asyncAfter(deadline: .now() + 0.2, execute: {
                cellNode.neverShowPlaceholders = false
            })
            return cellNode
        }
        return cellNodeBlock
    }

    func collectionNode(_ collectionNode: ASCollectionNode, didSelectItemAt indexPath: IndexPath) {
        tapClose(indexPath.row)
    }
}
```

### 问题2： cell 中的事件如何处理

在我们的产品 芝麻问答 中的实践和研究（有兴趣可以下载查看一下 **芝麻问答的 关注页Feed流** ），确立了原则用以保证代码质量：

**1. 内部事件内部消化** ，我们的产品cell 中有很多事件，例如点赞，关注，评论，分享，跳转个人页，跳转问题页，跳转回答页等。

第一版我们做了 enum 类型，将所有事件发回 vc ，vc 根据type进行处理，这样导致了 vc 含有大量的代码，且事件耦合太多。统计也会不方便。

因此我们重构了一些功能：**所有跳转使用 NavigationMap 链接跳转**；**所有用户事件发送给了单独的用户类去处理**

最终所有的 cell 内部事件在内部进行了处理。

**2. 保留 cell 的 didSelected 方法** ， cell 的点击事件不要过度的处理。例如一个问题的 cell 就没有必要在问题行添加事件，而应当在 选择 cell 时候跳转。

**3. 抽象父类** ，通过抽象父类，解决一些方法的调用，避免重复的代码和错误。

**4. 组件最小化** ，最小的组件不是一个 View 包括几个 按钮，最小的组件可能是一个定义的按钮。例如 项目中的点赞，关注按钮等

### 问题3： table 中有多个状态，如何设计

**背景** ：

1. **状态多**： 项目中关注的状态有很多种：没有动态会显示39人推荐；39人推荐中可能会显示通讯录好友；有动态会检查是否有新动态；无新动态会显示4人推荐；关闭4人推荐可能会显示10人推荐；且还有上下加载等。

2. **Cell复杂**： 整个 table会包含多种 cell 样式，甚至非服务端数据的 样式。


**分析** ： 多想少做。

1. 使用单独的状态来控制 UI 显示，方便单元测试。 使用某个状态例如 success 进行刷新 tableView ，使用 finish 结束刷新 UI，方便单元测试； 例如 begin 开始加载动画。

2. 数据一定是在 vm 中合成的，所以在 vm 中单独创建中间层模块，用于合并最终数据。通过对数据追加类型，拼接本地（例如通讯录，例如多人推荐）类型，组合最终 SectionData。

3. 在 cell 事件内部消化下，提炼父类分发，保证代码清洁。

**实现** ： 逐步推进，先大后小

对应于分析的步骤：

1. UI 状态在加载前中后有不同的显示，定义一个加载状态枚举，这样在 vc 中根据状态进行处理

```
enum FollowTabViewModelLoadState {
    case begin
    case noNet
    case normal
    case noMore
}
```

除此之外会 **定义加载类型的枚举**, 这样 根据这两个状态就可以准确的显示 UI 了

```
enum FollowTabViewModelLoadType {
    case suggestUser
    case feed
    case feedPullDown
    case feedPullUp
    case feedSuggestUser
}
```

**可以使用一个单独的中间层类来 输入类型， 输出 UI** , 也是为了方便测试

2. 创建 sectionData 类

像创建数据模型一样，对各个 section 创建类型，通过 section 区分当前组的数据类型

```
struct SectionData {
    let type: FollowingSectionType

    var suggestUsers = [SuggestModel]()
    var feeds = [ListItemModel]()
    var feedSuggestUsers = [SuggestModel]()
    init(type: FollowingSectionType) {
        self.type = type
    }
}
```

通过这种方式，可以保证数据的准确，可测试


### 4. 已经加载的 cell 数据如何更新

数据更新同步碰到的问题比较多，实现的方案也比较多，简单列了几种

```
项目中点赞数，评论数，关注关系同步的设计

排除 多设备登录同一账号的同步问题

方案B--

用本地数库保存关系，界面刷新的最后一步去从本地追加关系更新

缺点： 需要在登录下载关注关系，维护数据表
      需要刷新Table

优点： 不需要关心元数据的用户关系状态

方案B-

如果服务端可以返回最新的关注关系，那么不需要登录时候下载所有，只需要本地关注关系发生变化时候进行更新

缺点： 要维护数据表
      需要刷新Table

优点： 数据库只需要保存修改过的不需要保存所有

方案B

如果只是需要记录保存变化的数据，每次请求可以已经得到最新关注关系的话，没必要使用数据库，只需要使用内存数据库

缺点： 需要刷新Table

优点： 内存数据表开销更小，更准确

方案A

另一种思路，完全可以使用全局广播的方式告诉其他元数据对象和Ui，令其更新状态和数据

优点： 不用维护数据库 可以保证点赞数量保持服务器最新数据
      不需要刷新 table

缺点： 通知地方比较多


方案之外：

注意关注存在批量关注的问题处理

在 VM 中处理好 Model 数据，最后才给 view
```

可以看出 A 和 B 是两种比较成熟的方案, 另外一种方案是 **重数据库设计方案** 介邵如下：

```
对应于这些需要实时显示的关系

1. 创建关注表，用户表，点赞表，评论表

2. 在所有拿到数据的地方进行本地数据的更新

3. 在所有展示的 VM 进行查询，一次性查询出所有，然后互相匹配（复杂度O(n)），计算出最终的结果
```

这样就通过增大开销的方式，最大限度的保证数据的正确性。缺点也是需要刷新Table


**总结**

对比所有方案，最终选择的是 A 方案，这个方案能够准确的更新最新的数据，同时避免较大的系统开销（一个关注，引起所有相关 table reload，和 VM 相关计算）。

同时， **使用的 RxSwift 进行消息通知** ，在一些数据绑定上更加方便了一些。
