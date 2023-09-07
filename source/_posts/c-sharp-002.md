---
title: C# 的研究（二）统一标准参数接口——函数参数优化
date: 2021-11-06 09:59:57
tags:
 - C Sharp
 - 开发技术

categories:
 - 日常学习
---
## 前言
今天尝试把前面发布过的 [【Unity小技巧】全局事件监听——观察者/订阅发布模式
](https://huotuyouxi.com/2021/09/24/unity-tips-006/) 实装到游戏项目中，结果发现了其中一个不够完善的地方。

就是在接收通知的参数上面：

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

第一个参数是事件类型，第二个参数是一个字符串（json）。
原来的想法是接收一个 json 字符串，然后再在实际调用的时候转为对象，但是这样的局限性太大了。

比如：我想要一个自定义的类当做参数呢？
很明显接收一个字符串类型无法满足复杂的需求。

**函数既要统一参数，同时又要求参数有各种不同类型，这看上去是很矛盾的。**
「类的重载」可以解决这个问题，例如下面这样：

```
public function Method(string a) {
    // 操作内容
}

public function Method(int a) {
    // 操作内容
}
```

上面就实现了同名的 Method 方法，可以接收 `int` 类型和 `string` 类型两种参数，满足了上面的需求。
但这样却要写好多的重载方法，尤其是参数还很多的情况。

**我们如何保证同一个方法，使它能够接收不同类型的参数而且只需要写一次呢？**
只要利用类的「多态性」就能实现！

## 类的多态
多态就是字面意思：“多种形态”。
意思是说类在某种情况下可以转变成不同的形态。

例如：猫和狗都是动物，一只猫也可以说是一只动物，猫就转化成了「动物」。
用代码来实现就是这样：

```
public class Animal {
    public virtual void Sound(){}
}

public class Cat : Animal {
    public override void Sound() {
        // 猫的叫声
    }

    public void OtherMethod() {
        // 猫特有的方法        
    }
}

public class Dog : Animal {
    public override void Sound() {
        // 狗的叫声
    }
}

// 如果有这样两个方法，接收的是 Animal 类型
public void Test(Animal animal) {
    animal.Sound();
}

// 把 Animal 转换成 Cat 试试？
public void TestCat(Animal animal) {
    var cat = animal as Cat;
    cat.OtherMethod();
}

// 如果我们这样做
var cat = new Cat();
var dog = new Dog();

// 这肯定是没有问题的
Test(cat);
Test(dog);

// 但如果要把 Animal 转换成 Cat 并调用 Cat 特有方法呢？
TestCat(cat);
```

答案是：也没有问题！
这就是类的多态特性，当类之间是继承关系的时候，就可以实现转换。

但如果要把猫转换成狗就不行了，多态只能是父类和子类之间进行转换。
多态其实就是把父类转化成特定子类的能力。

我们利用这个特性，就可以实现只要传一个参数，却能多样化的定义参数。

## 统一参数接口
我们希望能够限定参数的类型，同时还要求参数可以多样化的进行自定义。
限定参数类型的目的是统一接口的标准，避免在调用的时候传进来各种奇奇怪怪的参数。

首先定义一个父类：

```
public class BattleNotifyData { }
```

父类什么都不需要写，只是空的类。

接着修改函数的参数：

```
// 将 string 类型替换为 BattleNotifyData
public static void SendNotify(string eventType, BattleNotifyData data = null)
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

OK，这样就完成了！
接着来写几个类，让它们继承 `BattleNotifyData` ：

```
// 第一个类：用于战斗结束的通知参数
public class BattleNotifyData_BattleEnd : BattleNotifyData
{
    // 获胜的阵营
    public string camp;
}

// 第二个类：用于角色死亡的通知
public class BattleNotifyData_OnDead : BattleNotifyData
{
    // 死者(是一个 BattleCharacter 类型)
    public BattleCharacter victim;
}
```

最后我们只需要这样：

```
 public override void TakeNotify(string eventType, BattleNotifyData data)
    {
        switch (eventType)
        {
            case Constant.BATTLE_EVENT_BATTLE_END:
                var newData = data as BattleNotifyData_BattleEnd;
                // 现在可以拿到camp变量了
                Debug.Log(newData.camp);
                break;
            case Constant.BATTLE_EVENT_ON_DEAD:
                var newData = data as BattleNotifyData_On_Dead;
                // 可以拿到死者对象
                Debug.Log(newData.victim);
                break;
        }
    }
```

通过 `as` 参数将父类转化为对应的子类即可！
发送通知的时候，可以实例化真实的类：

```
// 当角色死亡的时候，实例化 BattleNotifyData_OnDead 作为参数传递
BattleNotifyPublisher.SendNotify(Constant.BATTLE_EVENT_ON_DEAD, new BattleNotifyData_OnDead
{
    victim = this
});
```

## 总结
当我们要让一个函数只接受一个参数，还要求这个参数是「可变」的，那就利用类的多态来实现。

## 特别篇
有时候一个方法有超多的参数：

```
public void Test(string a, string b, int c, floa d) {
    // ...
}
```

我以前在上班的时候还真的遇到过这么写的人，而且最多的时候一个函数居然有 16 个参数！
这是人干的事情吗？就不能把参数封装成一个类吗？

```
public class Data {
    public string a;
    public string b;
    public int c;
    public float d;
}
```

实际调用方法的时候，实例化类作为参数：

```
Test(new Data {
    a="aaa",
    b="bbb",
    c=100,
    d=1f,
});
```

这样不比写十多个参数强？
结构体（Struct）作为参数也是可以的。