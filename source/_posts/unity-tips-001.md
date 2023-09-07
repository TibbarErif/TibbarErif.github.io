---
title: 【Unity小技巧】横版平台游戏之单点判定法接触地板检测
date: 2021-08-20 16:20:48
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
地板接触检测实际上是一个比较复杂的事情。
因为角色可能会在地板上方，也可能在左侧或者右侧，甚至是在地板的下方（即头撞到平台上）。

![地板的碰撞检测](https://pic.imgdb.cn/item/611f66c14907e2d39c6c631f.jpg)

因此碰撞检测不能简单地使用 `OnCollisionEnter2D`，`OnCollisionEnter2D` 只能简单的判定两个物体发生了碰撞，但是不能判定具体是碰到左边还是碰到右边或者是上面还是下面。

## 单点判定法
所谓的单点判定法就是在角色脚底新建一个「判定点」。
由于判定点是一个极小的点，因此当这个点接触到地板时，就可以认为角色是站在地板上的。
同理，如果你想判断角色左侧碰到了墙壁，可以在角色身体左侧加一个判定点。

我们只要检测这个点与平台发生了碰撞即可。

## 具体实现
在角色的脚底创建一个空白的 `GameObject`（空对象）：

![创建一个空对象](https://pic.imgdb.cn/item/611f68b44907e2d39c70c58a.jpg)

这个空对象不需要添加碰撞体或者刚体组件，把它挂在角色节点里面就可以了。
然后调整位置，将这个空对象拖到角色脚底，这个空对象（可以视为一个点）就是脚部的判定点。
接着在你的角色控制器脚本中，添加一个 `Transform` 类型的公开变量用来接收刚才创建的空对象。

```
public class MiniGame_Player : MiniGame_Character
{
    public Transform groundCheck;
    public LayerMask layerMask;

    // 省略其他代码
}
```

然后把刚才新建的空对象拖进脚本的变量里：

![拖进变量](https://pic.imgdb.cn/item/611f69734907e2d39c727548.jpg)

上面的脚本还有一个 `LayerMask` 类型的变量，这个变量保存的是 `Layer`（图层）。

![图层](https://pic.imgdb.cn/item/611f6b554907e2d39c76aa94.jpg)

Unity 中任何对象都属于某个图层，这个类型的变量就是用来获取某个图层的。
我们要让脚部的判定点与地板图层的对象进行碰撞检测，因此需要创建一个新的图层。
点击上图中的 `Layers`，在下拉菜单中点击 `Edit Layers` 添加新的图层，取名为 `Ground`：

![添加图层](https://pic.imgdb.cn/item/611f6ba74907e2d39c775cf7.jpg)

接着返回角色节点，修改 `layerMask` 变量的值，选择刚才创建的 `Ground` 图层：

![选择图层](https://pic.imgdb.cn/item/611f6bed4907e2d39c77f33e.jpg)

最后回到游戏场景，选择地板，把地板的 `Layer` 改成 `Ground`：

![修改地板图层](https://pic.imgdb.cn/item/611f6c4e4907e2d39c78b9cc.jpg)

接下来只要检测角色脚部的点与这个 `Ground` 图层的物体发生碰撞即可。
具体的检测方法是利用 Unity 的物理函数 `Physics2D`：

```
public class MiniGame_Player : MiniGame_Character
{
    public Transform firePoint;
    public Transform groundCheck;

    // 是否站在地板
    private bool isGround;

    private void FixedUpdate()
    {
        isGround = Physics2D.OverlapCircle(groundCheck.position, 0.5f, layerMask);

        if(isGround) {
            // ……站在地板的处理，例如更高角色的跳跃状态
        }
    }
}
```

`Physics2D.OverlapCircle` 会在一个目标点进行画圆，相应的参数如下：

```
Physics2D.OverlapCircle(目标点位置, 圆的半径, 检测图层);
```

圆的半径越大，检测的范围就越大，可以根据实际情况修改这个值。
物理函数画出的圆与 `Ground` 图层发生接触的时候，就会返回一个 `true`（布尔值）。
