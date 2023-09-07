---
title: 【Unity小技巧】Tooltip（提示窗口）和解决鼠标滑入闪烁问题
date: 2022-01-01 11:35:22
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
Tooltip（提示 UI）是当游戏的某个功能需要详细介绍时使用的。
当鼠标滑入指定区域后，就会显示出一个窗口，用来介绍此功能。

以 B 站为例，把鼠标滑入人名上面，过一会就会弹出人物资料卡：

![b 站人物资料卡](https://pic.imgdb.cn/item/61cfccd02ab3f51d915a5f54.jpg)
## Image 和 Panel 的区别
制作 UI 不可避免的会发现存在 Image 和 Panel 两个组件。
其实这两个组件是一样的，只是锚点不同。

![Image 和 Panel 的区别](https://pic.imgdb.cn/item/61cfce542ab3f51d915b5456.jpg)

另外，默认的 Panel 透明度是 100，把它调成 255，并且修改锚点，那它就变成 Image 了。
两者其实是一模一样的东西，只不过 Unity 把上面的操作封装起来，方便直接使用罢了。

例如一些固定位置的 UI，不会变化位置，就可以直接使用 Panel 快捷创建出来。
而一些动态的 UI，则推荐使用 Image 来实现。

Tooltip 包含一个背景窗口，并且它会跟随鼠标指针而改变位置，因此选用 Image。
## 实现 Tooltip(自适应宽高)
创建一个 Image 作为窗口背景。
然后在添加 Text 用来显示文本。

为了让窗口可以自适应内容而改变宽高需要设置一些特殊的组件：
![装备详情窗口](https://pic.imgdb.cn/item/61cfcf302ab3f51d915be68c.jpg)

`Vertical Layout Group`：以竖直排列的方式显示元素。
如下图所示，设置好边距，并且将宽度和高度设置为 `Control Child Size`（以子类大小为准）。
![Vertical Layout Group](https://pic.imgdb.cn/item/61cfcfd32ab3f51d915c6096.jpg)

接着还需要设置 `Content Size Fitter` 约束窗口内容大小。
![Content Size Fitter](https://pic.imgdb.cn/item/61cfd1142ab3f51d915d5227.jpg)
将宽度和高度都设置为：`Preferred Size`（首选项）。

最后再设置 `Layout Element` 用来约束窗口内元素的最大宽度和最小宽度：
![Layout Element](https://pic.imgdb.cn/item/61cfd1712ab3f51d915d97df.jpg)

演示效果：
![自适应的窗口](https://pic.imgdb.cn/item/61cfd24d2ab3f51d915e6309.gif)
## 滑入事件
鼠标滑入事件只需要继承 `IPointerEnterHandler, IPointerExitHandler` 即可实现。
然后将继承的类挂在需要显示的物体上，鼠标滑入的时候生成详情窗口即可。

接着修改窗口类的 Update 方法，使其跟随鼠标位置：

```
private RectTransform rectTransform;

private void Awake()
{
    rectTransform = GetComponent<RectTransform>();
}

// 前面一篇文章《限制UI范围》有介绍此方法
private void Update()
{
    rectTransform.anchoredPosition = ObjectBuilder.MousePositionToUIPosition(Input.mousePosition);
}

// 工具类转换鼠标位置为 UI 位置
public static Vector2 MousePositionToUIPosition(Vector2 position)
{
    float xDistance = Screen.width / 2;
    float yDistance = Screen.height / 2;

    return position - new Vector2(xDistance, yDistance);
}
```

这里会出现一个问题，显示的窗口会闪烁。
这是因为鼠标滑入物体生成了详情窗口，详情窗口遮挡了鼠标，导致触发 `IPointerExitHandler` 将窗口隐藏，接着窗口消失了，鼠标又变得没有被遮挡，因此又触发了 `IPointerEnterHandler`，如此反复就会变成闪烁的样子了。
## 解决闪烁问题
之所以会这样是因为 Canvas 组件有射线检测，只要把射线检测的组件移除就可以。
但是我们不能直接在 UI 的 Canvas 上面移除，否则别的 UI 就没办法触发点击事件。
我们可以重新创建一个 Tooltip 专属的 Canvas，并且它的层级比普通的 UI 要高，让它永远显示在 UI 上层不会被遮挡。

![Tooltip Canvas](https://pic.imgdb.cn/item/61cfe0b72ab3f51d91685bb5.jpg)

把 Tooltip Canvas 上面的 Graphic Raycaster 组件删掉就可以。
然后让装备详情窗口挂在 Tooltip Canvas 底下，这样闪烁问题就解决了。
## 小技巧
现在窗口的中心是在鼠标指针的位置，这样不美观。
我们可以通过修改锚点的方式让窗口显示在鼠标的左侧。
将 Pivot 的 x 改成 1 即可。

![让窗口显示在鼠标左侧](https://pic.imgdb.cn/item/61cfe5b02ab3f51d916c0209.jpg)
## 推荐视频
视频教程：[Tooltip](https://www.bilibili.com/video/BV1Kb4y1m7ed?spm_id_from=333.1007.top_right_bar_window_history.content.click)

很多 Unity 的教程在 YouTube 上面找得到，不过都是英文的。
所以，学好英语还是很有必要的，否则没有字幕就无法学习了。
## 追加补充
> 本内容为 2022-01-27 补充。

其实后面我发现了可以不需要转换 UI 的坐标。
我们知道 UI 是需要摄像机来渲染的，因此摄像机的喧嚷方式决定了 UI 的坐标类型。

![摄像机渲染方式](https://pic.imgdb.cn/item/61f2755c2ab3f51d91c3a00b.jpg)

将摄像机的渲染方式修改为：`Screen Space - Overlay`（屏幕空间覆盖）。
这样上面修改 UI 坐标的代码就可以修改为：

```
// 前面一篇文章《限制UI范围》有介绍此方法
private void Update()
{
    // 这是原来的，摄像机模式为：Screen Space - Camera（以当前摄像机为准）
    // rectTransform.anchoredPosition = ObjectBuilder.MousePositionToUIPosition(Input.mousePosition);

    // 现在修改了渲染方式为 Screen Space - Overlay 就可以直接这样：
    var position = Input.mousePosition;
    transform.position = position;
}
```

当 UI 的对象在不同的摄像机渲染模式下，坐标的计算方法也不一样。
同时节点的缩放也会不一样，很容易踩坑。