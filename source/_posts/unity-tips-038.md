---
title: 【Unity小技巧】让动画准确的显示在地面上（附LayerMask小知识）
date: 2022-12-05 15:20:14
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
# 前言
假如有一个战争类的游戏，里面有一种燃烧瓶，玩家捡到了可以将燃烧瓶丢出去，碰到地面就会破碎爆炸，然后在地面生成一团持续燃烧的火焰，那么是怎么做到让这团火焰的动画精准的显示在地面上面的呢？

如下图所示，这是一个往前方生成「地火」的技能：

![地火技能动画](https://s2.loli.net/2022/12/05/6F21YIxSQTgelV5.gif)

不管玩家是站在地面，还是跳跃状态，使用地火的时候，这个技能都应该在“地面冒出”，而不是突然出现在空中：

![地火技能动画（错误示范）](https://s2.loli.net/2022/12/05/vA9G3dcrHPtIopa.gif)

这个技能的原理就是以玩家当前位置为坐标，按照一定的步长（固定距离）增加 x 坐标来生成地火。

![地火技能的原理](https://s2.loli.net/2022/12/05/qYHMVDfiSEyBUcl.jpg)

实现这个技能的难点在于玩家的 y 坐标是不固定的，有可能是在跳跃状态下使用技能，但是地火必须刚好出现在地板上面，以玩家为参考点显然是不准确的（物理学中一般会找一个相对静止的物体当参考系）同理，我们应该以地面为参考系来计算地火的位置。
# 思路
为了让动画完美的显示在地板上方，我们需要获得地面上点的距离，地火的位置（x坐标）以玩家所在的位置为参考，向朝向的方向递增，比如玩家的 x 坐标是 0，那么第一团地火就是 1，第二团就是 2，以此类推。但是 y 坐标我们需要动态计算，如下图：

![地火的生成坐标原理](https://s2.loli.net/2022/12/05/v3du4C2Ne6OzExm.jpg)

具体原理是：x 坐标依然是以玩家当前所在位置进行自增，而实际的生成位置是以当前点的正下方，且与地面相交的那个点。

# 实现
过一个点，发出一条竖直向下的射线，射线与地面相交的点即地火的生成位置。我们可以用 Unity 自带的射线检测机制获取这个点：

```
protected Vector2 GetGroundPoint(Vector2 origin, float offset = 0f)
{
    var hit = Physics2D.Raycast(origin, Vector2.down, 999f, Context.gameManager.groundLayer);
    var res = hit.point;
    res.y += offset;

    return res;
}
```

这里的 999f 是一个 float 类型的变量，用来指定射线检测的最大距离，`groundLayer` 为 LayerMask 类型的变量，就是在面板中右上角的“Layer”：

![Layer Mask](https://s2.loli.net/2022/12/05/JbAIhsM1TyewFNQ.jpg)

只要在脚本中声明一个变量：

```
 public LayerMask groundLayer
```

然后将这个脚本挂在物体上，就可以通过下拉选择 Layer 了。

> 注意：layer 是自定义的属性，Ground 是我自己添加的。

`Physics2D.Raycast` 方法可以以某个点为原点，朝着某个方向发出一条射线，当这条射线与某个碰撞器发生碰撞的时候，我们就可以从中获取到碰撞的交点，通过赋值 LayerMask 参数来指定这条射线只与 Ground（地板）进行检测，这个交点就是我们需要的位置。上面加入了一个 offset（偏移量）用来防止位置计算不准确的情况。

下面用伪代码示例：

```
public override void Handle()
{
    // 生成地火的数量
    int count = 10;

    // 生成地火的间距，每隔2f生成一个
    var distance = new Vector3(2f, 0);

    for (int i = 0; i < count; i++)
    {
        var bullet = CreateBullet();

        // 以施法者为参照物，向施法者朝向的地方，按照偏移量生成地火
        var origin = skillData.user.transform.position + skillData.user.GetFaceTo() * (i + 1) * distance;

        // 上面的点与地面的交点，GetYoffset 返回一个 float 类型的变量，默认返回 0 即可（根据实际的动画图片手动修改）
        var pos = GetGroundPoint(origin, bullet.GetYoffset());
        bullet.transform.position = pos;
    }
}

// 生成地火
Bullet_FireEruption CreateBullet()
{
    return skillData.user.CreateBullet<Bullet_FireEruption>("FireEruption", skillData);
}
```

演示效果：

![地火技能（修改之后）](https://s2.loli.net/2022/12/05/DUpQea6Bmouc5fI.gif)

现在当玩家处于跳跃状态时，地火也能显示在地板了，可是新的问题又来了，地火为什么会出现在地板下面一点的位置呢？这是由于动画的锚点是在中间，而当我们把动画的坐标设置为地面上方的时候：

![动画的锚点](https://s2.loli.net/2022/12/05/JwOH3xGqFvnmoi8.jpg)

可以看到，动画的下面一部分”陷入“了地板，为了解决这个问题，就需要手动设置一个偏移量了，也就是上面的方法 `GetYoffset`，这个偏移值具体要以动画的大小人工进行调节，目前没有比较好的自动计算方法，除非一张图片是完美对称的，那么就可以取图片高度的一半作为偏移量，而实际情况却很难让美工把所有的动画都做出对称图形。

所以，这里就需要我们自己在场景中算出 y 轴的偏移值了。

让锚点在图片的正下方？很遗憾，往往也很难做到，除非有一个专门的美工能统一美术风格，而我们大多数是网上找的素材，每个美工的风格都不一样，注意看下面这两张素材，其中一张图片是紧紧贴合网格线下方的，而另外一个则是跟网格线存在很大的空白：
![动画与网格线贴合，不存在空白](https://s2.loli.net/2022/12/05/tRWJQd3kBMInX9m.jpg)
![动画四周存在很多空白](https://s2.loli.net/2022/12/05/k8HpUgQj4mDyXbL.jpg)

如果是第二种图片的类型，那么这个空白部分也没办法进行计算，除非是对素材进行重新加工，把动画素材的风格统一了才行。
## 扩展
关于 LayerMask 的一个小技巧，上面我是用 public 变量手动选择 LayerMask 的方式，其实还有一种很简单的方法，首先点击一个游戏中的对象，在右侧面板右上角的位置，选择 Layer 下拉菜单：

![Layer 下拉菜单](https://s2.loli.net/2022/12/05/ncFegwJRX1afLjz.jpg)

每个 Layer 前面都有一个序号，从 0 开始不断自增，记住这个编号，还有另一种引用的方法：

```
1 << 6
```

这种写法其实就是说：“序号为 6 的 Layer”，也就是 Ground 了，我们再返回射线检测的代码：

```
// 这里直接引用了 LayerMask 变量
Physics2D.Raycast(origin, Vector2.down, 999f, Context.gameManager.groundLayer);

// 第二种写法
Physics2D.Raycast(origin, Vector2.down, 999f, 1 << 6);
```

上面这两种写法是等价的，两个左箭头 `<<` 是位运算，LayerMask 类型的变量，其实就是一个数字，而左侧的 1 也是有含义的，意思「要」检测序号为 6 的 Layer，聪明的小伙伴可能会猜出，如果「不要」检测的话，可以把 1 改成 0，即下面这样：

```
// 不检测地面
Physics2D.Raycast(origin, Vector2.down, 0 << 6);
```

如果要同时检测两个呢？同样是用位运算：

```
// 同时检测序号为6和7的layerMask
Physics2D.Raycast(origin, Vector2.down, 999f, 1 << 6 | 1 << 7);
```

关于位运算就不再多解释了，感兴趣的自行查阅或参考（网上抄的）：

```
LayerMask mask = 1 << 2; // 表示开启Layer2。
LayerMask mask = 0 << 5; // 表示关闭Layer5。
LayerMask mask = 1<<2|1<<8; // 表示开启Layer2和Layer8。
LayerMask mask = 0<<3|0<<7; // 表示关闭Layer3和Layer7。
LayerMask mask = ~(1<<3|1<<7); // 表示关闭Layer3和Layer7。(同上)
LayerMask mask = 1<<2|0<<4; // 表示开启Layer2并且同时关闭Layer4.
```