---
title: 【Unity小技巧】如何优雅的控制游戏中的剧情事件？
date: 2021-08-06 21:54:18
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---

## 前言
本次尝试更新一篇技术型的文档，目的也是为了记录自己学习的过程。
游戏开发与普通开发不一样的地方在于逻辑的处理比较复杂。
如果没有一个良好的架构设计，处理游戏逻辑的代码就会看起来乱糟糟的。

## 糟糕的代码
这是在写这篇文档之前，《名为怪物的游戏》中处理场景事件的代码。
在火车逃亡篇中，有这样一段剧情：

![巴古与神秘人的交易](https://files.catbox.moe/f20zr5.gif)

这个剧情事件一共分为如下几个步骤：

- 神秘人走过来
- 开始第一段对话
- 神秘人走到旁边的椅子
- 开始第二段对话

这是游戏剧情中很常见的操作，控制场景 NPC 执行移动事件，然后触发对话。
但是要控制这样一连串的行为，用代码如何实现呢？

在通常情况下，代码是从上至下执行的：

```
Debug.Log("aa");
Debug.Log("bb");
```

比如上述的代码片段，首先打印出 `aa`，然后才打印出 `bb`；
这是由于打印 `aa` 的语句写在前面，所以先执行了，如果交换一下顺序，输出的顺序也会跟着改变。

这也是程序开发中很寻常的书写方法，那么如果游戏开发也用这种方法，会怎么样呢？

下面是我用来实现上述剧情事件的代码。

### 神秘人走过来
由于神秘人最开始是看不见的，所以把神秘人放到屏幕外面，不仅是神秘人，还有巴古以及后续登场的小喽喽都放在屏幕外面（玩家看不见的地方）。

![场景中的NPC](https://files.catbox.moe/s2iepm.jpg)

当玩家进入当前场景的时候，事件就开始执行了。
首先是设置登场 NPC 的坐标和朝向。

例如，下面是控制神秘人移动的代码。

```
MapManager.SetTargetLocalPosition(npcMysticMan, 7.08f, -0.62f, Direct.LEFT);

iTween.MoveTo(npcMysticMan, new Hashtable
{
    { "x", 1.37f },
    { "isLocal", true },
    { "easeType", iTween.EaseType.linear },
    { "onComplete", "Trade" },
    { "onCompleteTarget", gameObject },
    { "time", 2f },
});
```

上述代码使用了第三方插件 Itween 来控制目标的移动。
因为没有行走图只是单纯的移动坐标，所以角色看起来像是飘过来。

![神秘人移动事件](https://files.catbox.moe/lesfxz.gif)

### 触发第一段对话
神秘人移动结束之后，就要触发第一段对话。
这里需要使用 Itween 插件的回调方法 `onComplete` 来执行。

Itween 插件可以很方便的控制物体移动，并且在移动结束后执行设置好的回调方法。
编写移动结束时的回调方法：

```
private void Trade()
{
    WindowManager.CallDialog("06_bagu_room/02_trade");
}
```

现在，神秘人移动到指定位置后就会触发第一段对话了，如下。

![第一段对话](https://files.catbox.moe/d230tx.gif)

### 神秘人移动到椅子那边
对话结束后，神秘人要移动到椅子那边。
这里用到我自己设计的对话完成后回调的方法。

修改对话方法，添加移动事件。

```
private void Trade()
{
    WindowManager.CallDialog("06_bagu_room/02_trade", delegate
    {
        iTween.MoveTo(npcMysticMan, new Hashtable
        {
            { "x", -4.71f },
            { "isLocal", true },
            { "easeType", iTween.EaseType.linear },
            { "onComplete", "MysticManLeave" },
            { "onCompleteTarget", gameObject },
            { "time", 2f },
        });
    });
}
```

### 触发第二段对话
这里与上面的移动事件同理，在神秘人移动结束后调用 `MysticManLeave` 方法来执行第二段对话。

```
private void MysticManLeave()
{
    MapManager.ChangeDirect(npcMysticMan, Direct.RIGHT);
    MapManager.ChangeDirect(npcBaGu, Direct.LEFT);
    WindowManager.CallDialog("06_bagu_room/03_sitdown");
}
```

### 完成后的代码
至此，上面的小剧情事件已经用代码实现了。
回过来看看全部的代码是怎样的？

```
private void Event_001()
{
     MapManager.SetTargetLocalPosition(npcMysticMan, 7.08f, -0.62f, Direct.LEFT);

    iTween.MoveTo(npcMysticMan, new Hashtable
    {
        { "x", 1.37f },
        { "isLocal", true },
        { "easeType", iTween.EaseType.linear },
        { "onComplete", "Trade" },
        { "onCompleteTarget", gameObject },
        { "time", 2f },
    });
}

private void Trade()
{
    WindowManager.CallDialog("06_bagu_room/02_trade", delegate
    {
        iTween.MoveTo(npcMysticMan, new Hashtable
        {
            { "x", -4.71f },
            { "isLocal", true },
            { "easeType", iTween.EaseType.linear },
            { "onComplete", "MysticManLeave" },
            { "onCompleteTarget", gameObject },
            { "time", 2f },
        });
    });
}

private void MysticManLeave()
{
    MapManager.ChangeDirect(npcMysticMan, Direct.RIGHT);
    MapManager.ChangeDirect(npcBaGu, Direct.LEFT);
    WindowManager.CallDialog("06_bagu_room/03_sitdown");
}
```

看起来每个事件分明，一个方法控制一个动作，好像没有什么大问题。
但是，从逻辑上来讲，这几个方法的关系不是很明确，只是一直在用回调函数调用下一个事件而已。

很明显这种方法的弊端很大，不仅没有明确的关联关系，而且存在重复的代码，如控制神秘人移动事件。
这种即属于“面向过程”的编程方法，后期维护很不方便，写起来也很烦躁。
上面还只是不到 1 分钟的剧情，如果是更长的剧情，代码随随便便就要写上几千行。

这不仅是代码美不美观的问题，而且还严重影响效率，编写冗余的代码对程序员的身心也不友好。
为了保证后续的剧情事件能够「优雅」的开发，需要重制原来的剧情处理事件。

## 数据结构与设计模式
这两个名词是大学的时候最害怕的……
数据结构就是定义数据是如何进行排列的，比如最简单的就是数组。

数据保存在内存，内存是由一大块很大的空间构成。
于是数据就要保存在空间的其中一处，也就是地址。
根据一组数据在地址中存在的位置，分成数组和链表两种数据结构。

### 数组
1、2、3、4、5，这样按顺序排列，在内存中占用一段连续的地址。

![数组在内存中的结构](https://files.catbox.moe/xp9tss.png)

### 链表
链表就是一个结构体，也可以是一个类，一共有两个字段，一个是数据，一个是保存下一个数据的地址。
用结构体或者类都可以作为链表的数据结构。

```
public struct Data {
    int value;
    Data* next;
}
```

大概就是上面这种结构，`Data*` 是指向 `Data` 数据地址的指针。

> 注意：某启蒙 C 语言书本上写的“指针就是地址，地址就是指针”，个人以为这种说法不准确，因为这两个是不同的概念，打个比方，你给了我一张纸条，上面写着你家地址，我通过纸条就可以找到你家的位置，那么在这里——纸条就是指针，你家的地址就是地址，可以说这张纸条就是你家吗？很明显不能。指针是一种数据类型，它保存的是某种数据在内存中的地址，通过指针可以找到对应数据的地址。

链表数据的其中一个字段保存下一个数据的地址，如下。

![链表结构](https://files.catbox.moe/mwunmc.png)

### 数据结构有什么用？
其实这就是学习的最大敌人——不知道学了有什么用。
大学的时候，老师也从来不会告诉你学了知识能用在哪些地方。
对着这些枯燥无味的知识，而且还不知道以后能不能用的上，自然就没兴趣学下去了。

但是在经历了几年的开发经验之后，发现这些数据结构在进行系统设计和框架设计非常有用。
（当然，如果一直在从事普通的开发岗，基本没机会接触到这些，业务全部都是用别人写好现成的，自然也就学而无用了。）

看下面一组例子。

```
string a = "aaa";
string b = "bbb";
```

这是两个变量，不能叫做数据结构。
如果把它们放到一个数组里。

```
string items = new string { "aaa", "bbb" };

Debug.Log(items[0]);
Debug.Log(items[1]);
```

这样就是一个数据结构，因为他们是在一起的变量。
也就是说，单个的变量不能叫做数据结构，而需要一个特殊的关系把它们连接在一起才能叫做数据结构。

数组可以通过索引快速找到指定元素，如果知道数组的下标，查找数据的复杂程度是 O(1)，也就是瞬间就找到的意思。
链表没有下标的概念，查找数据只能一个个对比，找到对应的值才能返回结果，复杂度是 O(n)，也就是最坏的情况下，有多少个数据就要对比多少次。

> 如果不知道下标，只知道值，即使是数组也只能遍历对比数据，复杂度也是 O(n)。

单个的变量用途十分单一，但是把它们连接在一起功能就变得十分强大了。
不仅可以用来搜索数据，还可以定义某些具有特殊功能的结构。

### 队列和栈
队列和栈是比较常见的数据结构，其原理就是利用数组或者链表实现。
队列是“先进先出”的一种结构，而栈却相反，即“先进反而后出”。

关于栈的用处，之前在设计《名为怪物的游戏》中菜单系统就有介绍。
队列和栈如何实现就不科普了，感兴趣的可以自行搜索。

总之，所有的数据结构基本都可以由数组和链表来实现。
除了队列和栈，还有堆（树形结构），树型结构又分成很多种，比如二叉树、B 树、B+ 树等等（咱也没深入了解，有兴趣可以自己查）。

MySQL 的索引就是利用树的结构实现的。

> 如果当初老师能把数据结构实际的用途告诉我们，那我们应该会比较有兴趣学下去。

### 设计模式是什么？
普通的程序员在上班过程中几乎接触不到设计模式，因为都是用别人设计好的。

> 顶多也就是接触到单例模式、工厂模式那些。

只有在设计框架和系统结构的时候，设计模式才能大显身手。
这里暂时用不到，在游戏的开发中设计模式也是很重要的。

## 剧情事件优化
上面讲了那么多，全部都是为了优化剧情事件做的铺垫。
回到开头剧情的场景：

- 神秘人走过来
- 开始第一段对话
- 神秘人走到旁边的椅子
- 开始第二段对话

一段剧情可以分成多个部分，每一个部分就可以看做一个零散的“变量”，把它们组合在一起就是一个数据结构。

剧情的事件执行顺序是一个一个来的，比如先执行神秘人走过来的事件，然后开始第一段对话……以此类推。
换句话说，当第一个事件还没执行完成，后面的事件都得“等着”，直到第一个事件完事了，才能轮到下一个事件执行。

上面的场景描述……已经在指名道姓了，说的就是你——队列。
先加入的事件，最先处理，也就是在说队列的特点。

`c#` 自带队列数据类型 `Queue`，不用自己实现。
`Enqueue` 方法将一个数据插入队列的尾部，`Dequeue` 方法从队列头部取出一个数据并将其从队列删除。

利用上述两个方法即可轻松添加和取数据。

### 链式调用
当一个类的返回值为对象时，就可以实现链式调用。
最常见的方法就是一个类返回自身，就可以无限调用自己内部的方法了。

示例：

```

public class Link {

    public Link sayHello() {
        Debug.Log("hello");
        return this;
    }

}

Link link = new Link();
link.sayHello().sayHello().sayHello();

```

上述代码可以无限调用 `sayHello` 方法，因为调用完这个方法返回了类自身。
链式调用属于比较美观的写法，对于代码整洁有很大的帮助。

### 事件处理
首先需要定义一个用来处理剧情流程的事件系统，其实就是一个简单的结构。

第一个类是事件容器，用来保存和处理容器内事件。

```
using System;
using System.Collections.Generic;
using UnityEngine;

namespace FR_Event
{
    public class EventContainer
    {
        public Queue<EventActionAbstract> queue = new Queue<EventActionAbstract>();

        public EventContainer Append(EventActionAbstract eventAction)
        {
            queue.Enqueue(eventAction);
            return this;
        }

        public void StartEvent()
        {
            if (queue.Count == 0) return;

            var current = queue.Dequeue();
            current.Handle();
        }
    }
}
```

这里为了规范，给类加上了命名空间 `namespace FR_Event`。
命名空间是为了避免重名类的冲突，因为 Unity 在启动时就会加载所有类文件。
如果有同名的类就会报错，所以给它加上一个命名空间 `FR_Event`。
这样就可以用 `FR_Event.EventContainer` 与其他类区分开来。

面向对象有个“开放-封闭”原作，即隐藏无关的内容。
在这里，要调用事件，可以增加一个 `Manager` 类来间接调用事件容器。

```
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

namespace FR_Event
{
    public class EventManager : MonoBehaviour
    {
        public static EventContainer eventContainer;

        public static EventContainer StartNewQueue()
        {
            eventContainer = new EventContainer();
            return eventContainer;
        }
    }
}
```

这样就不用每次都 `new` 一个 `EventContainer`。

容器内接收 `EventActionAbstract` 类型的「事件动作」。
每个事件都要继承这个抽象类。

```
namespace FR_Event
{
    public abstract class EventActionAbstract
    {
        protected void EventEnd()
        {
            FR_Event.EventManager.eventContainer.StartEvent();
        }

        public abstract void Handle();
    }
}
```

抽象类只要有一个 `Handle`（处理动作的方法），以及一个 `EventEnd` 事件结束方法。
`Handle` 方法要在子类进行重写，因为每个事件都不一样。

接着创建一个对话事件：

```
namespace FR_Event
{
    namespace EventAction
    {
        public class DialogAction : EventActionAbstract
        {
            private string textPath;

            public DialogAction(string textPath)
            {
                this.textPath = textPath;
            }

            public override void Handle()
            {
                WindowManager.CallDialog(textPath, EventEnd);
            }
        }
    }
}
```

这里又添加了一层命名空间 `EventAction`，比较规范的是每一层文件夹就创建一个。

![文件结构](https://files.catbox.moe/bm3a6u.jpg)

接着创建两个测试对话文本：

![测试文本](https://files.catbox.moe/wfy3jw.jpg)

然后开始“优雅”的实现剧情对话：

```
 FR_Event.EventManager
           .StartNewQueue()
           .Append(new FR_Event.EventAction.DialogAction("00_test/01_text"))
           .Append(new FR_Event.EventAction.DialogAction("00_test/02_text"))
           .StartEvent();
```

看起来好多了！进游戏测试一下。

![对话测试](https://files.catbox.moe/jc0hvw.gif)

接下来只要再编写等待事件、移动事件、增减道具事件、淡入淡出事件……

这样完全可以用链式调用实现整个剧情事件！

参考的完整版代码：

```
  FR_Event.EventManager
           .StartNewQueue()
           .Append(new FR_Event.EventAction.MoveAction(target, new Vector3(1f, 1f), 1f) // 在1秒的时间移动到(1,1)位置
           .Append(new FR_Event.EventAction.WaitAction(1f)) // 等待1秒
           .Append(new FR_Event.EventAction.DialogAction("00_test/02_text")) // 执行对话
           .Append(new FR_Event.EventAction.FadeOutAction(2f)) // 在2秒内淡出屏幕
           .Append(new FR_Event.EventAction.GoMapAction("map_name")) // 移动到某个地图
           .StartEvent();
```

只要编写好以下事件动作：

- MoveAction：移动事件
- WaitAction：等待事件
- DialogAction：对话事件
- FadeOutAction：淡出事件
- GoMapAction：场景移动事件
- ……更多事件动作

无论想要实现什么事件，只要继承 `EventActionAbstract` 将其用 `Append` 方法加入到事件容器中就可以优雅的执行了！

最后，如果觉得命名空间有点碍事，可以在顶部用 `using` 关键词引入。

```

using FR_Event.EventAction;

FR_Event.EventManager
          .StartNewQueue()
          .Append(new DialogAction("00_test/01_text"))
          .Append(new DialogAction("00_test/02_text"))
          .StartEvent();

```

如此一来，代码就更加简洁了！

> 有了这个剧情事件处理系统，终于解放双手了！

接下来就可以开开心心的写代码了 ♪(^∇^*)
跟乱糟糟的代码 say bye bye ~

## 后言
关于游戏剧情的编辑还有很多种方法，上面还是脱离不了代码层面。

也有大佬用 Unity 设计了一套剧情编辑器：

![剧情编辑器](https://files.catbox.moe/t9znpv.jpg)

还有这种：

![剧情编辑器2](https://files.catbox.moe/c8yrrs.jpg)

看起来密密麻麻，密集恐惧症可能都犯了。
对于这种用非代码实现的方式，其实才是最好的方法。
因为人为手写代码很容易出错，但如果设计好程序，用程序自动生成就可以避免人为写出的 BUG 了。

除此之外，用文本来写特殊指令的方法也是可以的，总的来说就是把数据转化为指令，可视化的界面目前对我来说难度太高了，我们游戏的体量也没必要专门设计一个制作剧情的工具。

一个程序员写太多代码不是什么值得夸耀的事，反而是能不写代码就实现功能才顶呱噶。

> 程序员的最高境界就是“无码”