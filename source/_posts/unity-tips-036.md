---
title: 【Unity小技巧】限定条件与释放权限的时机
date: 2022-11-11 16:14:15
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
# 前言
在游戏中，存在许多限定条件，例如在对话状态下玩家就不能控制角色在场景左右移动，在很早之前开发《名为怪物游戏》的时候，开发菜单系统使用了「栈」这种数据结构来解决这个问题，但有的时候限定条件会存在多个触发器的问题。

名为怪物游戏菜单系统：[https://huotuyouxi.com/2021/08/17/monster-game-14](https://huotuyouxi.com/2021/08/17/monster-game-14)

以横版卷轴为例，玩家在场景中左右移动，按键释放技能等等，统一归纳为「玩家操作」，而如果玩家打开菜单或者与NPC进行对话的时候应该限制玩家不能操作，不然就太奇怪了；这种时候，使用栈不一定能很好解决问题，用栈可以很方便的解决多级菜单控制权问题，但是限制玩家操作的地方很多，例如前面的打开菜单和NPC对话，这种多对一的情况下，可能会出现乱序问题，比如玩家在打开菜单的情况下，因为某些情况发生了移动（比如站在斜面上因为重力影响而下滑）而恰好滑入了某个剧情触发器，进而自动弹出对话，那么这个时候，玩家的菜单并没有关闭，但是已经进入了对话系统，当对话结束的时候，菜单窗口还在，可是因为对话系统结束了，执行了出栈操作，解放了玩家的操作权，因而就会导致玩家明明开着菜单，却可以在地图移动的情况。

![多级菜单的控制权转移](https://i.niupic.com/images/2022/11/11/aauK.png)

多级菜单可以逐级限制，逐级释放权限，但是很多场景并不是这么有序的情况。

![玩家操作权限](https://i.niupic.com/images/2022/11/11/aauL.jpg)

玩家的操作权限受到很多其他系统的控制，如何保证有序的执行呢？
# 思路
当一个条件存在多引用的时候，何时释放权限成为了最大问题。
我想到了利用内存的垃圾回收机制——即计数器引用的方式来控制权限。
这是面试很常见的一道问题：
> 请解释一个变量是如何被系统回收的？

首先，变量在内存中有两个字段要保存，第一个是变量本身的值，第二个是变量的引用次数，每当变量被引用就会让计数器自增一次，而每当调用完毕就会让计数器自减，然后系统再根据变量的计数器来判断这个变量要不要被回收（即计数器为0的变量就会被判定为无用变量）。

由上面的思路，在写某些限定条件的时候，可以写一个如下的父类：

```
/// <summary>
/// 一个计数器模式
/// </summary>
namespace FireRabbit
{
    public class Counter
    {
        /// <summary>
        /// 引用计数器
        /// </summary>
        protected int count;

        /// <summary>
        /// 功能权限判定
        /// </summary>
        /// <returns></returns>
        public bool IsEnabled()
        {
            return count == 0;
        }

        public void Increment()
        {
            count++;
        }

        public void Decrement()
        {
            count--;
        }
    }
}
```

下面是一个简单的示例：

```
public class Test : Counter
{
    public void Handle()
    {
        if (IsEnabled())
        {
            // 执行某些逻辑
        }
        else
        {
            // 还存在未被释放的引用，无法执行的处理
        }
    }
}

public class A
{
    public void DoSomeThing(Test test)
    {
        // 开始执行，先让Test的计数器自增，令其处于被引用状态
        test.Increment();

        // ... 这里执行A的逻辑
    }

    /// <summary>
    /// 执行完毕后释放引用计数器
    /// </summary>
    /// <param name="test"></param>
    public void OnCompleted(Test test)
    {
        test.Decrement();
    }
}

public class B
{
    public void DoSomeThing(Test test)
    {
        // 开始执行，先让Test的计数器自增，令其处于被引用状态
        test.Increment();

        // ... 这里执行A的逻辑
    }

    /// <summary>
    /// 执行完毕后释放引用计数器
    /// </summary>
    /// <param name="test"></param>
    public void OnCompleted(Test test)
    {
        test.Decrement();
    }
}

public class Main
{
    public void Method()
    {
        var test = new Test();
        var a = new A();
        var b = new B();

        // 没有任何引用的时候（正常执行）
        test.Handle();

        // 依次执行a和b的方法
        a.DoSomeThing(test);
        b.DoSomeThing(test);

        // 现在就无法执行了（因为a和b还未释放引用）
        test.Handle();

        // 将A的引用释放
        a.OnCompleted(test);

        // 因为还存在b的引用未被释放，所以还是执行不了
        test.Handle();

        // 再将b释放
        b.OnCompleted(test);

        // 至此，所有引用都已经被释放，可以正常执行了
        test.Handle();
    }
}
```

好了，这就是利用了引用计数器模式解决无序权限的限制与释放时机问题。