---
title: 【Unity小技巧】Cinemachine 实现视野跟随
date: 2021-08-24 20:29:52
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
Cinemachine 是 Unity 的一个和摄像机相关的开箱即用包。
完全不需要写代码，只需要在可视化的界面进行简单的配置即可实现视野跟随的功能。

## 安装 Cinemachine
在 Unity 的顶部菜单栏中，选择 `Windows`（窗口）然后点击 `Package Manager`（包管理）。

![Package Manager](https://pic.imgdb.cn/item/6124e71f44eaada739bcdab2.jpg)

接着在弹出的包管理窗口中，点击左上角选择 `Unity Registry`（如果不选中这个是看不到 Cinemachine 的）。然后等一会，Unity 需要拉取包信息，大约十秒左右就会显示出所有的包列表了，等列表加载出来以后，找到 `Cinemachine` 点击，右侧的窗口会变成包信息，找到右下角的 `install` 按钮点击即可安装。

![Cinemachine](https://pic.imgdb.cn/item/6124e7ec44eaada739be9681.jpg)

等待安装完成以后，关掉包管理工具，可以看到顶部菜单栏的 `Component` 里多出了 `Cinemachine` 的选项。

## 2D Camera 去哪了？
然鹅，网上的教程都会告诉你要创建一个 `2D Camera`，可是……这个菜单里并没有这个选项。

![2D Camera 去哪了？](https://pic.imgdb.cn/item/6124e95e44eaada739c1ed52.jpg)

为了解决这个问题，我翻了网上所有教程，结果没一个是说明这个问题的。
难道说……只有我安装的 `Cinemachine` 有问题？

最开始以为是因为新版本更新导致功能变化，尝试了菜单中的 `Virtual Camera`（虚拟相机），结果发现并不能跟随视野。然后又去翻了一遍搜索引擎的结果，仍然没有解决，最后返回包管理工具，意外的注意到一个不起眼的地方……

目前 Cinemachine 最新版是 2.7.5 版本，网上的一些教程基本上都是 N 年前的 2.2.x 版本，升级到 2.7.x 之后，界面已经发生了一些改变，在包的描述信息中间有一行不起眼的英文：

![描述信息](https://pic.imgdb.cn/item/6124e90544eaada739c1235b.jpg)

点击 `More` 查看完整的信息：

![More](https://pic.imgdb.cn/item/6124eb7c44eaada739c6684e.jpg)

这里有一段英文：

> New starting from 2.7.1: Are you looking for the Cinemachine menu? It has moved to the GameObject menu.

意思是说：从 2.7.1 版本开始，Cinemachine 已经移动到了 `GameObject` 菜单底下。

原来新版本改变了菜单的位置，怪不得找不到！
返回 unity 场景，选择顶部菜单栏 `GameObject` 菜单，看到下拉列表里有 `Cinemachine`，点开就可以发现 `2D Camera` 了。

![GameObject菜单](https://pic.imgdb.cn/item/6124ebfb44eaada739c78df8.jpg)

## 实现视野跟随
点击创建 `2D Camera`，可以看到场景多出来这么一个东西：

![CM vcam1](https://pic.imgdb.cn/item/6124eca644eaada739c91abc.jpg)

并且 `Main Camera` 也自动添加了一个组件：

![Main Camera](https://pic.imgdb.cn/item/6124ecc044eaada739c95b32.jpg)

主摄像机不用管，我们只需要关注 `CM vcam1`，在右侧面板中，找到 `follow` 变量，把场景中的角色拖到变量里：

![follow](https://pic.imgdb.cn/item/6124ed5444eaada739caadf4.jpg)

这样就 OK 了，点击测试！

![实现视野跟随](https://pic.imgdb.cn/item/6124ee0044eaada739cc3a28.gif)

可以看到，视野可以跟随主角移动了。

## 配置最大视野范围
当角色移动到场景边界时，就会看到场景的边缘：

![移动到边缘](https://pic.imgdb.cn/item/6124ee6e44eaada739cd28c5.jpg)

但是我们希望它能限制在一个范围内，而不是走到角落时可以看到黑边。

在场景中任意一个物体上创建 `Polygon Collision2D`（多边形碰撞器），然后把 `is trigger` 勾选上，此处我选择在关卡的根节点添加一个多边形碰撞器：

![多边形碰撞器](https://pic.imgdb.cn/item/6124efaf44eaada739cfe6e2.jpg)

接着再返回场景，选择 `CM vcam1`，在右侧的界面中，点击 `Add Extension`：

![Add Extension](https://pic.imgdb.cn/item/6124f00844eaada739d0ab4a.jpg)

在弹出的下拉菜单中，选择 `Cinemachine confinder`：

![Cinemachine confinder](https://pic.imgdb.cn/item/6124f05a44eaada739d16539.jpg)

然后把添加了多边形碰撞器的根节点拖进 `Bounding Shape 2D`：

![根节点](https://pic.imgdb.cn/item/6124f0aa44eaada739d217d8.jpg)

这样视野就会限制在多边形的范围里：

![视野范围](https://pic.imgdb.cn/item/6124f0f444eaada739d2b8b2.jpg)

然后进入游戏测试：

![测试视野范围](https://pic.imgdb.cn/item/6124f14d44eaada739d38095.gif)

当角色走出多边形的范围时，摄像机就不再跟随了。

## 抖动问题修复
在没有配置的时候，摄像机的跟随范围很大，不仅是水平方向，竖直方向也会进行跟随，如下：

![抖动问题](https://pic.imgdb.cn/item/6124f1d244eaada739d4b14f.gif)

作为一个 2D 平台跳跃游戏，上下抖动会产生眩晕的感觉。

这是因为设置的多边形与摄像机的边界存在空白导致的。

![摄像机边界](https://pic.imgdb.cn/item/6124f26e44eaada739d61a27.jpg)

调整多边形的范围让它与白色边重合即可解决此问题。
除此之外，还可以调节摄像机的边缘（即上图白边部分）。

点击 `CM vcam1`，再点击 `Lens` 展开选项：

![Lens](https://pic.imgdb.cn/item/6124f34544eaada739d80355.jpg)

只要修改 `Ortho Size` 的值即可调整摄像机的大小。

## 跟随范围
可以设置跟随范围让角色一直保持在中间或者超过某个区域才开始跟随。

选择 `Body` 然后编辑 `screen X` 和 `screen Y` 的值即可：

![跟随范围](https://pic.imgdb.cn/item/6124f3ea44eaada739d9803a.jpg)

可以看到上图中有一个区域，当角色走出这个区域的时候摄像机才会开始跟随。

![区域](https://pic.imgdb.cn/item/6124f43e44eaada739da37bb.gif)

在区域中移动时，摄像机不会跟随，当走到区域边缘时摄像机才开始跟随，直到走出多边形的最大区域才停止。
（可以看到上图中角色脸上有一个点，当那个点碰到边缘的时候便会开始跟随）

如果要让角色一直处于视野中心的位置，只要将上面两个值设置为 0 即可。

## 后言
本文仅做简单的介绍，因为本人也还没有详细了解过这个插件的所有功能（它实在太大了！）。
根据我搜索得到的结果，这个插件可以实现包括特写等等很多效果（主要用在 3D 游戏）。
以及在 3D 游戏中，切换不同的画面以实现动画效果。

因为我现在做的是 2D 游戏，暂时还用不到那么多功能，只要视野跟随就足够了，等需要用到的时候再继续深入学习。