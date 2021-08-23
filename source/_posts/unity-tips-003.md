---
title: 【Unity小技巧】横版平台游戏之跳跃手感优化
date: 2021-08-21 17:48:43
tags:
 - unity
 - 开发技术

categories:
 - 小技巧

---
## 通常的跳跃
一般的跳跃是通过下面的代码实现的：

```
rb.velocity = new Vector2(rb.velocity.x, jumpForce);
```

上述代码给刚体施加一个竖直方向的速度 `jumpForce`，从而使物体进行竖直上抛运动。
这样的运动轨迹比较单一，即物体的跳跃高度与给定的初始速度相关，初始速度越大跳的越高，玩家无法自由的控制跳跃高度，因此手感不好。

![竖直上抛跳跃](https://pic.imgdb.cn/item/6120cdd64907e2d39cc0fc84.gif)

## 优化跳跃手感
要让玩家自由控制跳跃高度，可以根据玩家按键的时间来控制。例如玩家按住跳跃键，那么角色可以跳到最大高度；如果玩家只是轻轻点一下跳跃键，那么角色也只是进行一下小跳，角色能跳多高取决于玩家按下跳跃键的“力度”，让玩家产生一种错觉就是越用力按下跳跃键角色就能跳得越高。

为了方便看到跳跃效果，我把跳跃的初始速度增加了。
在进行了一番优化之后，可以看到角色每次跳跃的高度已经不再是固定的了：

![长按跳得高，点按挑的低](https://pic.imgdb.cn/item/6120cbef4907e2d39cbca3f5.gif)

那么这个是如何实现的呢？

## 跳跃辅助脚本
不需要改动原来的操作逻辑，只需要新建一个脚本，取名为 `BestJumping`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class BetterJumping : MonoBehaviour
{
    private Rigidbody2D rb;
    public float fallMultiplier = 2.5f;
    public float lowJumpMultiplier = 2f;

    void Start()
    {
        rb = GetComponent<Rigidbody2D>();
    }

    void Update()
    {
        if (rb.velocity.y < 0)
        {
            rb.velocity += Vector2.up * Physics2D.gravity.y * (fallMultiplier - 1) * Time.deltaTime;
        }
        else if (rb.velocity.y > 0 && !Input.GetButton("Jump"))
        {
            rb.velocity += Vector2.up * Physics2D.gravity.y * (lowJumpMultiplier - 1) * Time.deltaTime;
        }
    }
}
```

将这个脚本挂在游戏场景的角色对象上就 OK 了，然后进入游戏测试即可看到效果。
只需要修改 `fallMultiplier` 和 `lowJumpMultiplier` 的值来控制长按和点按的跳跃高度。

`BestJumping` 是一个老外分享的：[前往 Github 查看](https://github.com/mixandjam/Celeste-Movement/blob/master/Assets/Scripts/BetterJumping.cs)
B 站有 UP 主搬运了其教程视频：[Unity 实现2D游戏的爬墙侧跳和空中瞬移效果
](https://www.bilibili.com/video/BV1zJ411w7TG?from=search&seid=7765245838833455508)

## 原理解析
该脚本包含两个参数：

- fallMultiplier：降落系数
- lowJumpMultiplier：小跳系数

这两个系数是如何控制大跳和小跳的呢？

```
void Update()
{
    if (rb.velocity.y < 0)
    {
        rb.velocity += Vector2.up * Physics2D.gravity.y * (fallMultiplier - 1) * Time.deltaTime;
    }
    else if (rb.velocity.y > 0 && !Input.GetButton("Jump"))
    {
        rb.velocity += Vector2.up * Physics2D.gravity.y * (lowJumpMultiplier - 1) * Time.deltaTime;
    }
}
```

第一个系数 `fallMultiplier`：

```
if (rb.velocity.y < 0)
{
    rb.velocity += Vector2.up * Physics2D.gravity.y * (fallMultiplier - 1) * Time.deltaTime;
}
```

这里有一个补正值：`Physics2D.gravity.y * (fallMultiplier - 1)`，当角色处于下落状态时，unity 预设的重力值就会乘以这个系数，也就是说这个值 越大，角色在下落的时候受到的重力就越大，掉落的速度也就越快。

这个效果其实是模仿了平台游戏《蔚蓝》里加速下落的效果。
参考视频：[《蔚蓝》的手感为何迷人？Why Does Celeste Feel So Good to Play? | GMTK](https://www.bilibili.com/video/BV1M441197sr?from=search&seid=4834462703206664027)

![蔚蓝的跳跃](https://pic.imgdb.cn/item/6120d5064907e2d39cd06242.gif)

蔚蓝的跳跃在下落过程中比较急促，`fallMultiplier` 系数就是用来增加重力的，让角色掉落到地上的时间更短一点。
这段代码是用来优化自由落体运动过程中的手感，通过增加重力系数使其下坠更加快速，使角色跳跃的过程看起不那么“飘”。

如果你想让角色像蒲公英一样飘落可以试着把该值改成负值，比如：-1。

![蒲公英一样飘落](https://pic.imgdb.cn/item/6120da8c4907e2d39cdab103.gif)

蔚蓝是一个对控制要求很高的游戏，快速的下坠有利于玩家迅速进入下一个操作指令。
同理，`lowJumpMultiplier` 也是用来增加重力系数的，但不同的是它是控制加速上升阶段的重力，该值越大玩家的最小跳跃高度就越低。

```
else if (rb.velocity.y > 0 && !Input.GetButton("Jump"))
{
    rb.velocity += Vector2.up * Physics2D.gravity.y * (lowJumpMultiplier - 1) * Time.deltaTime;
}
```

当玩家第一次按下跳跃键时，如果此时松手离开跳跃键，
那么这段代码就开始发挥作用了：`Physics2D.gravity.y * (lowJumpMultiplier - 1)`。
在角色处于跳跃的上升阶段时，如果玩家没有按住跳跃键就增加重力使其快速下降，这就控制跳跃高度的核心逻辑。

![测试效果](https://pic.imgdb.cn/item/6120dc094907e2d39cdd1678.gif)
