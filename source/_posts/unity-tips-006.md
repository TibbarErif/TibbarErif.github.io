---
title: 【Unity小技巧】全局事件监听——观察者/订阅发布模式
date: 2021-09-24 12:31:21
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
这里的「事件」指的是宏观的事件，而不是具体的事件，例如：按钮点击事件、碰撞事件等等，这些都属于事件；这一篇文章要解决的问题就是怎么处理这些事件的关系，比如你得到 100 个胡萝卜可以获得「萝卜收集者」的称号，如果你得到 1000 个胡萝卜，就可以得到「胡萝卜大师」的称号，那么应该怎么实现呢？可能你第一个想法就是在获得萝卜的时候，用 `if-else` 结构来判断，这样确实很简单，但并不是最优解。

本篇文章会基于一个比较合理的通知机制来实现这种功能，而不是用大量的代码堆砌面向过程的开发，我们要实现的是面向对象的开发，而非把业务逻辑全部写在核心代码中，并且要求“低耦合”，以便后期维护和移植（可以适用于所有游戏项目，而非只能用在这个游戏上面）。

## 观察者模式与订阅发布模式
观察者模式与订阅发布模式虽然类似，它们都是在监听某个对象的变化，但也存在一些区别，主要是对象主体的不同。

### 观察者模式
观察者模式通常有一个或者多个观察者在同时“观察”着某个对象的变化。

![观察者模式](https://pic.imgdb.cn/item/614d1c4a2ab3f51d9155bbbc.jpg)

如果上面的例子比较抽象，那么换个例子。
一个女神通常有多条“舔狗”，舔狗无时无刻不在刷新微信朋友圈，观察女神有没有发动态，然后第一时间给她点赞。

在这个例子中，女神就是被观察对象，而舔狗们就是观察者。

### 订阅发布模式
观察者模式中，观察者是直接对被观察者进行观察的，也就是存在着耦合性。
订阅发布模式则是把这种耦合性拆分出来，用一个中间件来给观察者发布通知。

![订阅发布模式](https://pic.imgdb.cn/item/614d1ef82ab3f51d9158518a.jpg)

在这个模式中，观察者不再直接监听被观察者，而是通过一个中间件来获得消息。

后来，舔狗们学聪明了，只盯着一个女神获得青睐的几率很低，那不如“广撒网”，如果能同时盯着多个女神，那么被青睐的几率不是大大提高了吗？但是自己的精力有限，不可能同时盯着很多个女神，所以舔狗们陷入了烦恼。其中有一个脑袋比较灵光的舔狗突然发现了商机！于是他建立了一个「女神朋友圈动态共享群」，入群者收费 ￥10 元/月，并且打出了诱人的广告标语：“只要加了这个群，女神动态第一时间知道！”，于是舔狗们纷纷入群，而发现商机的舔狗则悄悄的联系到女神，说如果你要发朋友圈的时候，就提前告诉我一下，我给你￥10 块钱，女神心想发个朋友圈告诉你一下就行，白赚￥10 块，那好，于是答应了。与商人舔狗合作的女神越来越多，而且商人舔狗也不需要盯着朋友圈，而是女神主动告诉他要发朋友圈了，所以商人舔狗根本不需要花什么精力，只要当一个消息传递筒就够了，女神通知他要发朋友圈了，商人舔狗只要在群里发一句：“xxx 女神发了朋友圈，速去点赞！”。最后商人舔狗成了富一代，再加上他手里多达数百个女神的联系方式，自然而然的获得了女神的青睐，钱财与美色兼得，而那些入群的舔狗们，仍然保持着给女神点赞的好习惯。

### 差异点
观察者模式与订阅发布模式的区别就在于是否直接监听被观察对象，观察者模式是直接监听被观察者的变化，而订阅发布模式则是通过一个中间件来将消息通知给订阅者，只要想一下上面的舔狗例子就可以明白区别在于哪了。

订阅发布模式可以解除观察者与被观察者之间的耦合，因此本文将采用这种方法。

## 场景示例
假如学校要召开家长会，而你的班主任在班会课上要求你回家的时候通知父母这周五来学校开家长会，放学后你回到家，父母正在客厅看电视，于是你就跟他们说周五要开家长会，记得去学校。

上面这个就是订阅发布模式，订阅就如字面意思那样，比如订阅一个周刊，那么你每周都能收的到一份读物；你在 B 站关注了一个 UP 主，那么 UP 主如果有动态，你就会收到一条通知。

在上面的场景，你的父母扮演的就是订阅者，父母订阅了学校的动态，而你是充当学校和父母之间传递消息的桥梁，学校作为发布者宣布要开家长会，因此你需要负责将学校的通知转达给父母。至于父母周五去不去学校，那就是他们的事情了，他们可能因为工作忙去不了，也可能按时参加，总之，你把周五要去学校开家长会的消息传达给你父母了，他们要怎么做与你无关，你尽到通知的义务就足够了。

## 抽象示例
订阅发布模式可以是一对多也可以是多对多的关系，一个订阅者可以订阅多个消息，一个发布者可以被多个订阅者订阅。

发布者是用来下发通知的，上面的例子根据胡萝卜的数量获得称号，这就是一个需要下发的通知，发布者告诉所有的订阅者：玩家得到胡萝卜啦！订阅者可能是称号系统，也可能是任务系统；称号系统会根据萝卜的总数判断玩家是否能获得某个称号，而任务系统则监听玩家是否领取了上交胡萝卜的任务，如果达到任务要求数量就显示任务完成标志。

![消息的通知](https://pic.imgdb.cn/item/614c98ed2ab3f51d91c7e2a5.jpg)

中间部分是一个消息筛选器，发布者可以发布很多种事件，不局限于发布“获得胡萝卜”的通知，需要筛选这个消息应该告诉哪些人，不告诉哪些人，并不是把所有的消息都告诉订阅者，如果订阅者没有订阅这个类型的消息就不给它们发送。

## 具体实现
这个设计模式里包含了三个角色：被观察者、发布者与订阅者。

被观察者在进行某个动作的时候，就告诉发布者，发布者再根据订阅了此类消息的订阅者进行通知。

### 订阅者
为了比较规范的开发以及方便以后的移植，我给所有的类都加上命名空间。
新建一个 Observer 文件夹来存放相关类，创建一个订阅者 `Subscriber`：

```
using UnityEngine;
using System.Collections;

namespace Core.Observer
{
    public abstract class Subscriber : MonoBehaviour
    {
        /**
         * 接收发布者消息的方法
         */
        public abstract void TakeNotify(string eventType, string data);
    }
}
```

让它继承 `MonoBehaviour`，这样就可以挂在场景中的物体上面。这里有一个 `TakeNotify` 方法用来接收消息通知，并且还附加了一个 `string` 类型的 `data` 字段，可以接收一个 json 字符串用来传递某些数据，这样这套系统就可以胜任大部分的使用场景了。

如果需要一个场景来监听某个事件，就让那个场景的脚本继承这个类。

### 发布者
再创建一个发布者 `Publisher`：

```
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

namespace Core.Observer
{
    public class Publisher
    {
        static Dictionary<string, List<Subscriber>> subscribers = new Dictionary<string, List<Subscriber>>();

        public static void Register(string eventType, Subscriber subscriber)
        {
            // 如果不包含这个键就初始化
            if (!subscribers.ContainsKey(eventType))
            {
                subscribers.Add(eventType, new List<Subscriber>());
            }

            // 判断该订阅者是否已经订阅过了
            if (subscribers[eventType].Exists(x => x == subscriber))
            {
                Debug.Log(subscriber.name + "已经订阅过" + eventType);
                return;
            }

            Debug.Log("成功注册事件监听，类型：" + eventType + "，监听者：" + subscriber.name);

            subscribers[eventType].Add(subscriber);
        }

        public static void SendNotify(string eventType, string data = null)
        {
            // 如果是空的就没必要通知
            if (!subscribers.ContainsKey(eventType))
            {
                Debug.Log(eventType + "是空的");
                return;
            }

            foreach (Subscriber subscriber in subscribers[eventType])
            {
                subscriber.TakeNotify(eventType, data);
            }
        }
    }
}
```

发布者的作用是向订阅者发布消息通知，只提供两个方法：

- `Register`：注册一个监听者（订阅者）
- `SendNotify`：向所有订阅该事件类型的人发送通知

好了，就是这么简单，「发布-订阅模式」完成了！

## 实际使用
光是写好代码，却没有实际使用，还是难以搞懂这东西到底怎么用？
那么接下来我用正在开发中的推理文字游戏来进行解释。

由于是推理向的文字游戏，自然就少不了调查场景。
在调查场景中，玩家可以用鼠标点击屏幕中的物体进行调查（类似逆转裁判）。

![游戏场景](https://pic.imgdb.cn/item/614d3f552ab3f51d91828152.jpg)

上面的场景有三个调查物体：①烟头，②面包屑，③脚印。
当玩家调查了三个物体之后就会自动进入下一段剧情。

### 实现思路

整理一下思绪，这里需要一个主场景脚本，以及 3 个调查物体的脚本。
我们先来看一下如果不使用设计模式来开发，而是面向过程的开发会是怎样的。

**第一步**
给你所有调查物体添加 `Button` 组件，然后添加一个 `ClickEvent`（点击事件）。

**第二步**
玩家点击物体之后，触发一段对话，然后把某个开关打开，一共需要三个开关：①调查过烟头的开关，②调查过面包屑的卡关，③调查过脚印的开关。

**第三步**
在对话执行结束后，判断三个开关是否已经全部打开，如果是的话就执行下一段剧情。

### 不用设计模式
编写场景中调查事件的脚本父类 `SurveyEvent`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

namespace Core.Survey
{
    public abstract class SurveyEvent : MonoBehaviour
    {
        private void Awake()
        {
            Button btn = gameObject.AddComponent<Button>();
            btn.onClick.AddListener(ClickEvent);

            InitAction();
        }

        protected void ClickEvent()
        {
            ClickHandle();
        }

        protected virtual void InitAction() { }
        protected abstract void ClickHandle();
    }
}
```

为了保管所有的开关，编写一个专门存放常量的脚本 `Constant`：

```
namespace Core
{
    public class Constant
    {
        /**
         * 调查事件相关的开关
         */
        public const string SURVEY_EVENT_0001_BREADCRUMBS = "SURVEY_EVENT_0001_BREADCRUMBS";
        public const string SURVEY_EVENT_0001_BUTTS = "SURVEY_EVENT_0001_BUTTS";
        public const string SURVEY_EVENT_0001_FOOTPRINT = "SURVEY_EVENT_0001_FOOTPRINT";

        /**
         * 监听事件类型
         */
        public const string SUBSCRIBER_SURVEY = "SUBSCRIBER_SURVEY";
    }
}
```

这个脚本保存了所有事件开关的名字，方便维护。

接着，以烟头脚本为例，编写脚本 `Event_0001_Butts`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace Core.Survey
{
    public class Event_0001_Butts : SurveyEvent
    {
        protected override void ClickHandle()
        {
            GlobalManager.CreateDialog("00015", delegate
            {
                // 打开某个开关
                GlobalManager.SetSwitch(Constant.SURVEY_EVENT_0001_BUTTS, true);
                
                // 判断3个开关是否全部开启
                if(GlobalManager.GetSwitch(Constant.SURVEY_EVENT_0001_BUTTS) && GlobalManager.GetSwitch(Constant.SURVEY_EVENT_0001_BUTTS) && GlobalManager.GetSwitch(Constant.SURVEY_EVENT_0001_FOOTPRINT)) {
                    // 全部调查完毕执行下一步的逻辑
                }
            });
        }
    }
}
```

然后是面包屑的脚本 `Event_0001_BreadCrumbs`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace Core.Survey
{
    public class Event_0001_BreadCrumbs : SurveyEvent
    {
        protected override void ClickHandle()
        {
            GlobalManager.CreateDialog("00014", delegate
            {
                // 打开某个开关
                GlobalManager.SetSwitch(Constant.SURVEY_EVENT_0001_BREADCRUMBS, true);
                
                // 判断3个开关是否全部开启
                if(GlobalManager.GetSwitch(Constant.SURVEY_EVENT_0001_BUTTS) && GlobalManager.GetSwitch(Constant.SURVEY_EVENT_0001_BUTTS) && GlobalManager.GetSwitch(Constant.SURVEY_EVENT_0001_FOOTPRINT)) {
                    // 全部调查完毕执行下一步的逻辑
                }
            });
        }
    }
}
```

写到这里应该可以发现不妥之处了，就是判断全部调查完成之后的逻辑被写了两次，如果再加上脚印的脚本，就得写三次。

以下这部分代码存在重复：

```
// 判断3个开关是否全部开启
if(GlobalManager.GetSwitch(Constant.SURVEY_EVENT_0001_BUTTS) && GlobalManager.GetSwitch(Constant.SURVEY_EVENT_0001_BUTTS) && GlobalManager.GetSwitch(Constant.SURVEY_EVENT_0001_FOOTPRINT)) {
    // 全部调查完毕执行下一步的逻辑
}
```

进一步的优化，把判断多个开关的脚本封装一下：

```
/**
 * 检查键是否全部满足
 */
public static bool CheckSavefleSwitch(Dictionary<string, bool> keyValuePairs)
{
    foreach (string key in keyValuePairs.Keys)
    {
        if (GetSwitch(key) != keyValuePairs[key])
        {
            return false;
        }
    }

    return true;
}
```

现在不需要用一长串的 `if` 来判断开关的开启状态，而只需要传入一个 `Dictionary` 即可判断多个开关的状态是否满足要求。不过这样还是治标不治本，处理下一步剧情的逻辑不可避免的要重复写好几次。

### 发布订阅模式
控制调查场景整个流程的应该是场景，而不是调查物体，所以判断下一步剧情的逻辑代码不应该写在点击事件上，如果玩家点击了物体，只需要告诉发布者，让发布者将消息通知给场景，由场景来判断是否应该执行下一步的剧情。

编写场景的父类 `SurveyScene`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Core.Observer;

namespace Core.Survey
{
    public abstract class SurveyScene : Subscriber
    {
        private void Awake()
        {
            Publisher.Register(Constant.SUBSCRIBER_SURVEY, this);
        }

        /**
         * 场景调查完毕，执行下一步的剧情
         */
        protected abstract void OnCompleted();
    }
}
```

场景的父类在唤醒时就向发布者注册了监听 `SURVEY_EVENT` 类型的事件的通知，这样一旦有调查的事件，发布者就知道要告诉场景这件事，场景再自行进行处理。

然后编写实际的场景类 `Event_0001`：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Core.Observer;
using Core;

namespace Core.Survey
{
    public class Event_0001 : SurveyScene
    {
        public override void TakeNotify(string eventType, string data)
        {
            bool res = GlobalManager.CheckSavefleSwitch(new Dictionary<string, bool>
            {
                { Constant.SURVEY_EVENT_0001_BREADCRUMBS,true },
                { Constant.SURVEY_EVENT_0001_BUTTS,true },
                { Constant.SURVEY_EVENT_0001_FOOTPRINT,true },
            });

            if (res)
            {
                OnCompleted();
            }
        }

        protected override void OnCompleted()
        {
            Debug.Log("全部调查完了");
        }
    }
}
```

在接收订阅消息的方法 `TakeNotify` 里编写剧情处理逻辑，当三个开关全部打开时就执行 `OnCompleted` 方法，这里只简单的打印出来。

接下来改写场景中调查物体的方法：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using Core.Observer;

namespace Core.Survey
{
    public abstract class SurveyEvent : MonoBehaviour
    {
        private void Awake()
        {
            Button btn = gameObject.AddComponent<Button>();
            btn.onClick.AddListener(ClickEvent);

            InitAction();
        }

        protected void ClickEvent()
        {
            ClickHandle();
        }

        protected virtual void InitAction() { }
        protected virtual void HandleNotfy(string eventType, string data) { }

        protected virtual void SendNotify()
        {
            Publisher.SendNotify(Constant.SUBSCRIBER_SURVEY);
        }

        protected abstract void ClickHandle();
    }
}
```

`SendNotify` 方法向发布者发布一个 `SURVEY_EVENT` 类型的事件通知，告诉发布者“我被点击了”。

然后改写烟头脚本：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace Core.Survey
{
    public class Event_0001_Butts : SurveyEvent
    {
        protected override void ClickHandle()
        {
            GlobalManager.CreateDialog("00015", delegate
            {
                GlobalManager.SetSwitch(Constant.SURVEY_EVENT_0001_BUTTS, true);
                SendNotify();
            });
        }
    }
}
```

再改写面包屑脚本：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace Core.Survey
{
    public class Event_0001_BreadCrumbs : SurveyEvent
    {
        protected override void ClickHandle()
        {
            GlobalManager.CreateDialog("00014", delegate
            {
                GlobalManager.SetSwitch(Constant.SURVEY_EVENT_0001_BREADCRUMBS, true);
                SendNotify();
            });
        }
    }
}
```

脚印脚本同理，可以看到与剧情处理有关的逻辑已经被拆出去了！
当玩家点击了物体就触发了点击事件，然后 `ClickHandle` 方法执行一段对话，在对话结束后打开开关，同时通知发布者“我被点了”。

发布者将消息逐一通知给订阅者：

```
public static void SendNotify(string eventType, string data = null)
{
    // 如果是空的就没必要通知
    if (!subscribers.ContainsKey(eventType))
    {
        Debug.Log(eventType + "是空的");
        return;
    }

    foreach (Subscriber subscriber in subscribers[eventType])
    {
        subscriber.TakeNotify(eventType, data);
    }
}
```

其中的订阅者就是 `Event_0001` 场景脚本，因为它在 `Awake` 方法里注册了监听 `SURVEY_EVENT` 事件：

```
private void Awake()
{
    Publisher.Register(Constant.SUBSCRIBER_SURVEY, this);
}
```

接收消息并且进行处理的方法是 `TakeNotify`：

```
public override void TakeNotify(string eventType, string data)
{
    // 这里为什么不需要判断eventType的类型呢？因为它只注册了SURVEY_EVENT的通知，不会有别的通知进来
    bool res = GlobalManager.CheckSavefleSwitch(new Dictionary<string, bool>
    {
        { Constant.SURVEY_EVENT_0001_BREADCRUMBS,true },
        { Constant.SURVEY_EVENT_0001_BUTTS,true },
        { Constant.SURVEY_EVENT_0001_FOOTPRINT,true },
    });

    if (res)
    {
        OnCompleted();
    }
}
```

第一次点击烟头，打开了烟头的开关，然后通知场景，场景对比发现还有两个开关没打开，就不进行处理；接着玩家点击了面包屑，场景又收到通知，但一比对，发现还少了一个脚印的开关，最后玩家点击脚印，最后缺失的脚印开关也被打开了，因此条件全部满足，执行 `OnCompleted` 开始下一步的处理。

整个过程逻辑非常清晰，而且不存在耦合，作为开发者书写这样的代码没有心智负担，非常轻松。

### 取消订阅
上面例子还缺少了一个取消订阅的功能，当调查事件完成之后，就要把场景销毁，但是订阅者还留在发布者存储的变量里，当场景被销毁的时候，还需要把订阅者移除才行。

给发布者 `Publisher` 添加方法：

```
/**
 * 将一个订阅者移除
 */
public static void Remove(string eventType, Subscriber subscriber)
{
    // 如果不包含这个键就不处理
    if (!subscribers.ContainsKey(eventType))
    {
        Debug.Log("不存在订阅者，类型：" + eventType + "，脚本名：" + subscriber.name);
        return;
    }

    Debug.Log("订阅者被移除，类型：" + eventType + "，脚本名：" + subscriber.name);

    // 获取订阅者的索引
    subscribers[eventType].Remove(subscriber);
}
```

移除订阅者只需要在 unity 提供的生命周期函数执行即可：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Core.Observer;

namespace Core.Survey
{
    public abstract class SurveyScene : Subscriber
    {
        private void Awake()
        {
            Publisher.Register(Constant.SUBSCRIBER_SURVEY, this);
        }

        /**
         * 场景调查完毕，执行下一步的剧情
         */
        protected abstract void OnCompleted();

        private void OnDestroy()
        {
            Publisher.Remove(Constant.SUBSCRIBER_SURVEY, this);
        }
    }
}
```

只需要在 `OnDestroy` 将订阅移除即可。

## 小测验
这里做了一个测试 List 下标变化的实验，这个实验的目的是尝试删除 List 元素之后，观察下标是否会被重置。

```
List<string> data = new List<string> { "aa", "bb", "cc" };

foreach (string item in data)
{
    Debug.Log(item + "=" + data.IndexOf(item));
}

data.RemoveAt(1);

foreach (string item in data)
{
    Debug.Log(item + "=" + data.IndexOf(item));
}
```

输出结果：

![输出结果](https://pic.imgdb.cn/item/614d28d82ab3f51d9162f85c.jpg)

结论是在删掉 List 其中一个元素之后，下标就会被重置。

## 总结
发布订阅模式与观察者模式相比在于解耦，发布订阅模式是完全没有任何耦合的。

你在朋友圈发布一条动态，微信就充当发布者的角色，它会向你的好友广播这条动态消息。你可能并不关注谁看了这条消息，你只负责把动态发出去，那些订阅者（你的好友）就会收到消息，至于他们看完点不点赞，就是他们的事。如果其中一个好友把你的朋友圈屏蔽了，那么 Ta 就取消订阅你的消息，即使你发了朋友圈，Ta 也收不到。

在游戏中，你吃到一个金币就对全世界广播：“啊，我吃到一个金币了！”
场景中的 UI 听到这个消息后：你吃到了金币啊！那我得把界面中的金币数量更新一下。
场景中的史莱姆（敌人）听到这个消息：你吃到金币关我屁事。（史莱姆鸟都不鸟你）
场景中的山贼（敌人）听到这个消息：啊？你有钱？那我得去抢劫了。（山贼准备抢你的钱）

总之，发布者订阅模式就是这样，执行了一个事件就进行广播通知，那些想要知道这些消息的人就会对消息进行处理，那些不需要这个消息的人就不会管你发了什么。
