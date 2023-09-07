---
title: 【Unity小技巧】EventSystems中IPointerEnterHandler与IPointerExitHandler同时触发导致闪烁问题的解决方法
date: 2022-09-09 21:49:23
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
在使用 EventSystems 制作 Tooltip（提示框）的时候，鼠标指针划入需要显示提示框的物体上，在旁边显示一个帮助窗口，需要用到 Unity 的 EventSystems 系统。并且在脚本组件实现两个接口：`IPointerEnterHandler` 和 `IPointerExitHandler`，这里就会出现一个很奇怪的问题，当鼠标指针滑入物体显示出提示框，但是很快又会消失，然后又瞬间出现——不断闪烁。

## 解决方法一
这是因为鼠标是根据射线来检测的，在 Canvas 组件有 `Graphic Raycaster` 的组件用于检测鼠标指针。

![射线检测组件](https://pic.imgdb.cn/item/631b45cd16f2c2beb1f4b6c1.jpg)

之所以会出现闪烁的情况是因为当鼠标移动到物体上面时，触发了显示窗口的方法，而弹出的窗口刚好遮挡了鼠标指针，导致触发 `OnPointerExit` 方法，而触发了该方法之后，一般我们会隐藏提示框，当提示框被隐藏之后，鼠标又检测到进入物体，接着又显示提示框……如此反复循环就出现闪烁的情况了。

最简单的解决方法是：错位。
即将提示框的位置进行偏移，避免遮挡到鼠标指针，或者直接修改提示框的锚点都可以，如下图所示，我将提示框的位置偏移到指针右下角，这样当提示框出现的时候就不会挡到鼠标指针了。

![进行了偏移的提示框](https://pic.imgdb.cn/item/631b46ac16f2c2beb1f5cb87.jpg)

第二种方法就是移除 Canvas 的射线检测组件，但是 Canvas 上面会挂载其他 UI 节点，为避免影响其他 UI 对象的射线检测，我们需要添加一个专门存放提示窗口的 Tooltip Canvas，如下图所示：

![ToolCanvas](https://pic.imgdb.cn/item/631b476b16f2c2beb1f724c4.jpg)

即场景中存在两个 Canvas，其中一个用来放置普通的 UI，另一个专门用来放置提示窗口的 UI。
接着将 ToolCanvas 的 `Graphic Raycaster` 组件删掉就行：

![移除ToolCanvas的射线检测组件](https://pic.imgdb.cn/item/631b47c316f2c2beb1f83aa4.jpg)

然后所有的提示框都放到这个 Canvas 下面就不会出现闪烁的情况了。

## 解决方法二
如果按照上面的方法依然出现闪烁，说明不是常规的情况。
比如一个背包的格子由两部分组成：格子和道具图标。

![](https://pic.imgdb.cn/item/631b482f16f2c2beb1f8b92f.jpg)

格子就是背后的格子背景，就是一个方框而已。

![格子背景](https://pic.imgdb.cn/item/631b486d16f2c2beb1f90b6c.jpg)

背包就是由格子和道具 Icon（图标）组成。
一般而言，我们会把脚本挂在格子背景上面，Icon 就是纯粹的图片，没有任何脚本挂在 Icon 上面。

用这种方法制作的背包格子，在鼠标滑入的时候会出现闪烁情况，而且按照方法一也无法解决，具体原因如下图所示：

![Icon与背景的间距](https://pic.imgdb.cn/item/631b498516f2c2beb1fa582d.jpg)
![鼠标触碰到Icon的问题](https://pic.imgdb.cn/item/631b498516f2c2beb1fa5839.jpg)

为了方便演示，我把 Icon 的大小调小了一些，这样我们就能清楚的看到 Icon 和格子之间的间距。
当鼠标指针滑入格子背景时就会触发 `OnPointerEnter` 方法，并且在格子与 Icon 的间距之间滑动没有任何问题，但是如果碰到 Icon 就会触发 `OnPointerExit` 方法。

这是因为我们的脚本是挂在格子背景上面，而 Icon 的层级是在格子之上的，因此鼠标会被 Icon 的图片遮挡导致触发了 `OnPointerExit`。

解决方法就是在 Icon 上挂显示提示框的脚本，或者不采用这种双层的格子，而只需要一张 Icon，不需要背后的方框，更简单粗暴的方法就是在美工层面把 Icon 和背后的格子方框通过 PS 合并成一张图，这样我们就不需要用双层节点实现了，也就不会被下面那层节点遮挡鼠标指针了。
## 解决方法三（次日补充）
最新发现的方法可以不做任何修改，即使是双层嵌套（背景框+图标）也能使用，具体做法是将 Icon 的 Image 组件里面的 `Raycaster Target`（射线检测目标）勾选去掉即可，即忽视这个图片的射线检测，这样就不会遮挡到背景框，即使把脚本挂在背景框也没问题。
![Raycaster Target](https://pic.imgdb.cn/item/631c7cd816f2c2beb116e32d.jpg)