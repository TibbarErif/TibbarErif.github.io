---
title: 【Unity小技巧】改掉一些不必要的xp
date: 2022-11-14 08:13:53
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
# 前言
代码不规范有时候会让开发者十分难受，但有时候过于追求规范也一样，换句话说不管是哪个极端都不是正常的。

以个人为例，我很喜欢「分门别类」把一些功能不一样的类拆分成单个文件，像下面这样定义一些数据类型：

![数据类](https://files.catbox.moe/ex20vx.jpg)

这些数据类型是用来当做订阅模式发送消息用的，可以看到数量非常多，每个类基本都是下面这样的结构：

```
using FireRabbit;

public class SubParam_EnemyDead : ObserverParams
{
    public Format_Enemy enemyModel;
    public int level;
}
```

包含一些很简单的参数：

```
using FireRabbit;

public class SubParam_GetExp : ObserverParams
{
    public int exp;
}
```

实际上并不需要为这些数据类型单独保存一个文件，将他们合并在一起就可以了，像下面这样，直接新建一个 `SubParam` 文件，内容如下：

```

using FireRabbit;

public class SubParam_EnemyDead : ObserverParams
{
    public Format_Enemy enemyModel;
    public int level;
}

public class SubParam_GetExp : ObserverParams
{
    public int exp;
}

// ... 其他类型
```

也就是说把所有的类都放在同一个文件里面，不需要为每个类都单独创建一个文件。

研究了其他编程框架，基本上也是这么做的，这样不仅可以减少管理文件的难度，也大大加快了文件的加载和编译速度（因为读取文件是需要执行 IO 的，要读取的文件越多，表示要加载的次数也越多）最好的例子就是 PHP 制作的网页，每一名用户访问网站，PHP 解释器就要将整站的 PHP 代码加载一次，文件越多意味着要加载的次数越多，加载速度越慢。

效率往往和规范相冲突，例如以优雅著称的 Laravel 框架，实际上效率很低，至于如何取舍就取决于个人了。