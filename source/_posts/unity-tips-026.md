---
title: 【Unity小技巧】安装 Tilemap 必备插件 TileEditor
date: 2022-01-04 10:41:39
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
昨天在开发新游戏的场景系统。
因为没有能力原创素材，所以我选用的是 RPG Maker 系列的 Tilemap 图块制作场景。

关于 Tilemap 的教程可以参考：[https://www.bilibili.com/video/BV1J4411b7Mg](https://www.bilibili.com/video/BV1J4411b7Mg)
需要注意的是视频最后部分介绍的功能图块，如 Rule Tile，Animated Tile 等。

默认情况下，图块就只是一张图片。
如果直接绘制的话，就会像下面这样：
![不连续的图块](https://pic.imgdb.cn/item/61d3b5eb2ab3f51d91fd5cb1.jpg)

但我们希望的结果是连续的图块：
![连续的图块](https://pic.imgdb.cn/item/61d3b75c2ab3f51d91ffdb06.gif)

Unity 默认是不提供功能图块的功能，需要安装扩展插件 2d-extras。
## 2d-extras
先来说下安装这个插件的坑……
昨天晚上弄到一两点都没搞定。

网上的教程一般会给你这两个地址：

```
https://github.com/Unity-Technologies/2d-extras
https://github.com/Unity-Technologies/2d-techdemos
```

第一个是 2d-extras 包文件。
如果你把它下载并且拷贝到项目目录会直接报错（可能是 Unity 版本的问题）。

然后参照文档的第二个方法：

> The following line needs to be added to your Packages/manifest.json file in your Unity Project under the dependencies section:

在项目的 `Packages/manifest.json` 添加一行配置：

```
"com.unity.2d.tilemap.extras": "https://github.com/Unity-Technologies/2d-extras.git#master"
```

你需要从文件管理找到这个配置文件：

![manifest.json](https://pic.imgdb.cn/item/61d3b9b22ab3f51d910360c5.png)

然后打开配置，把上面的配置信息加到里面。
然后打开 Unity 就会自动识别安装包文件。

这里也有一个坑就是 GitHub 是国外的网站，所以受到「墙」的监管。
如果报 443 错误，多试几次或者尝试开 VPN 就可以，直到安装成功。

安装了 2d-extras 之后在 Tilemap 画板会多出几个笔刷。
但是我尝试右键看了下，没发现有 Tiles 的选项。

> 后来我才发现创建功能 Tile 是需要在文件夹里面右键的，不知道当时有没有这个选项。

## TileEditor
上面的方法应该是适用于旧版的 Unity，但我用的是最新的 2021.2.7 版本。
网上搜得到的教程几乎都是过时的，已经不能用在最新版本上面了。
直到我看无意中到官方文档上面的一段话：

![Tilemap官方文档](https://pic.imgdb.cn/item/61d3bac22ab3f51d91040e19.png)

原来从  2020.1 版本开始就已经不再随 Unity 安装了。
所以一些旧的教程按照我上面的步骤可以创建功能 Tile。
新版本的 TileEditor 需要手动添加包。

打开 Unity 打开包管理工具：`Window - Package Manager`，选择 `Unity Registry`：

![Window - Package Manager](https://pic.imgdb.cn/item/61d3bb692ab3f51d91048494.png)

然后选择 2D，将旁边的 7 Packages 展开，选择下方的 2D Tilemap Editor，右下角有一个 install：

![TileEditor](https://pic.imgdb.cn/item/61d3bb9f2ab3f51d9104aa28.png)

安装好之后，install 按钮会变成 remove（移除），如果不想用这个包可以点击进行卸载。
这样我们就安装好 TilemapEditor 了。
再次返回 Unity 场景编辑器，在文件夹中右键就可以看到功能 Tile 了：

![Tiles](https://pic.imgdb.cn/item/61d3bcce2ab3f51d910577e9.png)

注意：是要在「文件夹中」右键，而不是场景中！
很可能我第一次安装 2d-extras 就是弄错了。

创建一个功能图块，配置完成之后是这样的。
![功能图块](https://pic.imgdb.cn/item/61d3bea02ab3f51d910746e7.png)

这个图块实际上是一个预制体文件。
![预制体文件](https://pic.imgdb.cn/item/61d3bec72ab3f51d91075e80.png)

将创建好的功能图块预制体拖到画板中即可：

![拖到画板](https://pic.imgdb.cn/item/61d3bf752ab3f51d9108a67a.gif)
## 动态图块
RPG Maker 的默认图块有一些地形是动态的。
例如池水和岩浆：
![RPGMaker 动态图块](https://pic.imgdb.cn/item/61d3c1392ab3f51d910b8c34.png)

右键创建一个 Animated Tile 图块，设置三张岩浆的图块为动画图块：
![岩浆图块](https://pic.imgdb.cn/item/61d3c1ae2ab3f51d910c4040.png)

测试效果：
![动态岩浆](https://pic.imgdb.cn/item/61d3c1f42ab3f51d910cf7df.gif)

## 连续图块
在 RPG Maker 里还有一些连续的图块，例如地毯：

![地毯](https://pic.imgdb.cn/item/61d3e7772ab3f51d91324dcc.jpg)

以红色地毯为例，如果使用这种图块，在 RPG Maker 里面会整块都连在一起。
这种效果就是 Rule Tile 实现的。

![连续的图块](https://pic.imgdb.cn/item/61d3b75c2ab3f51d91ffdb06.gif)

这种图块需要先设定好「规则」，当满足规则的时候，图块就会切换成另外一种形态。
在 Unity 需要根据九宫格来设定图块的规则：

![九宫格](https://pic.imgdb.cn/item/61d3e8612ab3f51d913306b9.jpg)

Tilemap 本身即是格子的结构。
中间的空位代表自身，箭头代表上下左右，以及左上、右上、左下、右下，一共 8 个方向。

一张地毯的 Rule Tile 设置如下：

![地毯的 Rule Tile](https://pic.imgdb.cn/item/61d3e8ce2ab3f51d91335f74.jpg)

default 是显示在画板中的图像，也是默认的图块。
如果有空缺的图块，就会用 default 来填充。

以中间部分的图块和左上角的图块示例：

![图块示例](https://pic.imgdb.cn/item/61d3e9442ab3f51d9133b34e.jpg)

中间的图块，上下左右必然有相邻的图块。
也就是说，满足上下左右都有图块的话，当前图块就是这张。

![地毯中间的图块](https://pic.imgdb.cn/item/61d3e9962ab3f51d9133f52f.jpg)

然后是左上角的图块，既然都是左上角了，那肯定是「最上面」、「最左边」。
那么，它上面就不会再有其他图块了，左边也不会有图块，所以我们把这两个方向 x 掉。

![地毯左上角的图块](https://pic.imgdb.cn/item/61d3e9f02ab3f51d91344196.jpg)

同理设置好剩下的几个图块。
最后的效果如下：

![完整的地毯](https://pic.imgdb.cn/item/61d3ea382ab3f51d913474c8.jpg)

为什么这里不能连在一起呢？
因为是「九宫格」，而 RPG Maker 提供的素材只有「五宫格」：

![五宫格](https://pic.imgdb.cn/item/61d3eb512ab3f51d913545b3.jpg)

将它们拼接起来：

![拼接效果](https://pic.imgdb.cn/item/61d3eb622ab3f51d91354edb.jpg)

可以发现，它还缺少「上下」、「左右」两个部分的图块。
缺失的图块被设置为 default 的图块填充了：

![缺失的图块](https://pic.imgdb.cn/item/61d3ebad2ab3f51d9135879d.jpg)

所以，很遗憾 RPG Maker 的素材不能生成连续的图块。
（但是 RPG Maker 内部是如何实现的？？？）

一张完整的连续图片必须是九宫格的形式。
下面这张水管的图片可以用来练手：

![水管](https://pic.imgdb.cn/item/61d3c22e2ab3f51d910d834d.png)

最终效果：
![水管最终效果](https://pic.imgdb.cn/item/61d3c2812ab3f51d910e14d2.gif)

可以发现水管居然十分神奇的连接在一起了！

## 补充
文章的疑问一直困扰着我，如果不解决 RPG Maker 图块的问题，游戏的场景就没法绘制。
经过一番搜寻，终于知道 RPG Maker 为什么只有 5 个 图块还能拼接起来了。
参考链接：[https://blog.csdn.net/u013412391/article/details/105021909](https://blog.csdn.net/u013412391/article/details/105021909)

实际上是需要代码来计算的。
图块的一部分切掉，然后用另一个图块的部分替换，以此实现拼接效果。

比起这样，还不如直接把原图切分成九宫格……