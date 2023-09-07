---
title: 【Unity小技巧】UI基础及如何将UI限制在屏幕范围内
date: 2021-12-26 11:55:40
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 预备知识
在 Unity 中，UI 跟场景中的普通物体是不一样的。
而我第一次从 cocos creator 转到 Unity 时不知道这一点，踩了许多坑。
需要在了解 UI 的一些基础知识，然后才能编写与 UI 相关的代码。

## UI 系统
Unity 的 UI 系统有两大类，第一个是直接在场景创建 Canvas（画布），在画布上可以添加 Image、Text 之类的，这种叫做 UGUI；第二个是 NGUI（Next-Gen-UI）译为中文即“下一代的 UI”，不过这个好像是收费的插件。

NGUI 商店页面：[https://assetstore.unity.com/packages/tools/gui/ngui-next-gen-ui-2413](https://assetstore.unity.com/packages/tools/gui/ngui-next-gen-ui-2413)

一般而言，我们都是用 Unity 自带的 UGUI，即直接在场景右键创建 UI 组件。
UGUI 一定需要一个 Canvas（画布），所有的 UI 组件都必须挂在画布上面。
场景可以创建很多个画布，画布有一个统一的 Order In Layer。

![Order In Layer](https://pic.imgdb.cn/item/61c7ea182ab3f51d91a19221.jpg)

当设定了 Order In Layer 之后，挂在该 Canvas 下面的组件就统一使用此 Order In Layer。
Order In Layer 是 Unity 中层级显示的依据。

UI 组件本身并没有 Order In Layer 字段，必须借助 Canvas 来设置。
如果你需要复杂的层级关系，可以创建多个 Canvas 分别设置 Order In Layer，然后将不同层级的组件挂在不同的 Canvas 上面。

## 普通物体和 UI 组件
需要注意普通物体和 UI 组件不能混用。
最开始就是因为我不知道 UI 是不一样的，结果存在各种缩放的问题。
务必将 UI 组件和普通物体分开挂在不同的根节点，普通物体就别挂在 Canvas 里面了。

下面来说一下它们的区别：

![普通2D游戏对象](https://pic.imgdb.cn/item/61c7eb912ab3f51d91a240e4.jpg)

在场景创建一个 2D 的精灵（图片），观察它的属性：

![2D 精灵的属性](https://pic.imgdb.cn/item/61c7eb912ab3f51d91a240e7.jpg)

在 Unity 中，所有的 GameObject 都有 Transform 属性，用来记录它在场景中的位置、翻转以及变换。
但是，UI 组件并非直接继承 Transform，而是有 UI 特殊的 RectangleTransform：

![UI 组件](https://pic.imgdb.cn/item/61c7ec5e2ab3f51d91a284e0.jpg)

可以发现，UI 组件的 Transform 为 RectTransform，并且它没有 Order In Layer 属性可以设置。
因为 UI 的层级关系只能依赖 Canvas，而不能单独设置。

因为 Unity 所有物体都要继承 Transform，UI 组件同样属于 GameObject，它自然也是需要继承 Transform 的，RectTransform 就是 Transform 的子类，它也有 position 属性，但是 UI 并非使用此属性，而是要使用 anchoredPosition 这个属性。

这两个属性虽然都是记录坐标，但完全不一样，请参考下文说明。

除此之外，物体的渲染层级也存在差别：

![普通2D精灵Layer默认为Default](https://pic.imgdb.cn/item/61c7f1002ab3f51d91a43b11.jpg)
![Image的Layer默认是UI](https://pic.imgdb.cn/item/61c7f1472ab3f51d91a457a3.jpg)

还有，UI 组件还有普通组件没有的锚点属性，锚点。
锚点改变之后，属性也会发生变化，通常我们不用去修改锚点，默认为中心位置即可：

![UI锚点](https://pic.imgdb.cn/item/61c7f1742ab3f51d91a46699.jpg)

只有在制作一些血条或者其他需要拉伸变化的才会修改锚点，这边就不展开了。
不过，分享一篇写的很不错的文章：[Unity进阶技巧 - RectTransform详解](https://www.jianshu.com/p/dbefa746e50d)
此文对锚点的介绍十分详细，有兴趣可以阅读。
## UI 的渲染
无论玩家走到哪，UI 肯定不能跟随玩家移动的，除非开发者手动使用代码操控位置，否则 UI 一定是显示在整个屏幕中。
UI 的渲染与开发者的设置有关，有三种渲染方式：

![Canvas 的渲染方式](https://pic.imgdb.cn/item/61c7ed6c2ab3f51d91a2eae9.jpg)

点击场景的 Canvas，在右侧属性栏即可找到 Render Mode。

- Screen Space - Overlay：覆盖模式，Canvas 始终覆盖在屏幕上方，也就是说 UI 会遮挡其他元素。
- Screen Space - Camera：Canvas 始终显示在设定的摄像机（Camera）前方，一般 2D 游戏选择这个即可。
- World Space：世界坐标模式，此模式下 UI 与其他元素没有区别，也就是说场景的物体可以遮挡 UI。

我做的是 2D 游戏，摄像机拍摄的也就只有一个平面而已。
这里选择 `Screen Space - Camera：Canvas` 模式即可。
设置该模式后，要把场景中的摄像机拖到 Canvas 的 Render Camera 属性里面。

接着我把 Canvas 的 Sort In Layer 设置为最大值：32767，让 UI 显示在场景最顶层，防止被场景的元素遮挡。
## 坐标问题
将一个普通的 2D 精灵，向右移动，观察坐标变化：

![2D精灵坐标](https://pic.imgdb.cn/item/61c7f29d2ab3f51d91a4deb1.jpg)

可以发现，坐标的变动十分小，再拖动一个 Image：

![Image坐标](https://pic.imgdb.cn/item/61c7f2e72ab3f51d91a4fab7.jpg)

可以看到，UI 坐标的量级比普通物体大多了。
UI 在场景的坐标范围如下：

![UI坐标系](https://pic.imgdb.cn/item/61c7f3922ab3f51d91a538d6.jpg)
以屏幕中心点为（0，0），向右 x 轴增加，向上 y 轴增加。

## 限制 UI 范围
现在终于可以进入正题了——限制 UI 不离开屏幕外面。
详情窗口是 UI 中最常见的一个，如下是我正在开发中的游戏 UI：

![详情窗口](https://pic.imgdb.cn/item/61c7fec62ab3f51d91a9345d.jpg)

当鼠标移动到人物的出生天赋上面，就会在旁边显示一个 UI 窗口，用来描述天赋的说明。
而这个窗口的坐标是跟随鼠标指针变化的，也就是说可能存在下面的情况：

![UI出界](https://pic.imgdb.cn/item/61c7ff822ab3f51d91a9719b.jpg)

UI 应该被限制在屏幕内，而不能让它出界，因而需要对 UI 的坐标进行限制。
`Mathf.Clamp(value, min, max)` 方法可以将 value 限制在 min 与 max 之间。

我们只要让 UI 的 x 坐标在 `[-640, 640]`，y 坐标在 `[-360, 360]` 之间即可。
计算坐标的时候，还需要计算 UI 的宽度和高度，然后计算出距离边缘位置的距离。

![1280*720的屏幕](https://pic.imgdb.cn/item/61c8000b2ab3f51d91a9a8c6.jpg)

计算实际的 UI 坐标：

![计算实际的 UI 坐标](https://pic.imgdb.cn/item/61c800c42ab3f51d91a9e27d.jpg)

其实就是一道初中的数学题而已。
假设屏幕是 1280 * 720 的，那屏幕中心就是（0,0），屏幕左侧边缘的坐标就是（-640,y）。
一个宽为 100 的 UI 物体，x 坐标的最小值就是：-640 + 100 / 2 = 590。
同理可以计算出 xMin，xMax，yMin，yMax 四个极限坐标。


```
// 获取场景的 Image 对象
public Image img;

// 在 Update 方法里改变它的坐标
float xDistance = Screen.width / 2;
float yDistance = Screen.height / 2;

float x = Mathf.Clamp(pos.x, -xDistance + size.x, xDistance - size.x);
float y = Mathf.Clamp(pos.y, -yDistance + size.y, yDistance - size.y);

img.transform.position = new Vector2(x,y);
```

但其实上面的方法并不能修改 image 的位置，因为——
**UI 的坐标是继承了 Transform 的 RectTransform，获取 UI 坐标不能直接使用 `transform.position`，应该使用 `rectTransform.anchoredPosition`。**

所以，真正的代码应该如下：

```
public Image img;
private RectTransform rectTransform;

private void Awake()
{
    rectTransform = img.GetComponent<RectTransform>();
}

private void Update()
{
    // UI 的真实坐标
    var pos = rectTransform.anchoredPosition;

    // UI 的大小尺寸
    var size = rectTransform.sizeDelta / 2;

    // 计算屏幕的尺寸
    float xDistance = Screen.width / 2;
    float yDistance = Screen.height / 2;

    // 限制 UI 坐标最大最小值
    float x = Mathf.Clamp(pos.x, -xDistance + size.x, xDistance - size.x);
    float y = Mathf.Clamp(pos.y, -yDistance + size.y, yDistance - size.y);

    // 调整 UI 坐标
    rectTransform.anchoredPosition = new Vector2(x, y);
}
```

演示效果：

![限制 UI 范围](https://pic.imgdb.cn/item/61c803102ab3f51d91aaa32a.gif)