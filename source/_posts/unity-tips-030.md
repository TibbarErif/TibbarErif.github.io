---
title: 【Unity小技巧】C# 拷贝对象问题
date: 2022-02-01 19:10:27
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
对象即是引用是很多程序语言的共通特性，在 C# 也是如此。因此在使用对象类型的时候就要十分小心因为修改了对象的属性而导致结果发生变化的情况，今天的制作游戏战斗状态的时候又遇到新的问题：拷贝出来的对象的某些属性依然是引用？！
## 数据拷贝
只要让一个类实现 `ICloneable` 接口即可令其变为可以克隆的对象。
现在要实现一个技能【破刃斩】攻击敌人同时降低目标一定百分百的护甲，降低的护甲与角色当前的”剑意“层数有关。

创建一个战斗状态类用来保存状态的各种属性：
```
public class FormatterBattleState : FormatterData, ICloneable
{
    public string icon;
    public string name;

    // 持续回合数,-1代表无限回合
    public int remain_time;

    // 状态的最大层数，达到该值后则无法继续叠加，只会刷新回合
    public int max_count = 1;

    /**
     * 附加属性，基础属性后缀为_percent则为百分比，负值代表减少
     * atk: 物理攻击
     * def: 物理防御
     * mat: 魔法攻击
     * def: 魔法防御
     * mov: 速度
     * skill_power: 技能威力
     * damage_decrement: 伤害减免
     */
    public Dictionary<string, decimal> attributes = new Dictionary<string, decimal>();

    public object Clone()
    {
        return this as object;
    }
}
```
上面的战斗状态类已经可以被克隆了，接着还有一个读取状态的方法：

```
public FormatterBattleState GetState(string id)
{
    var states = TableLoader.GetInstance().GetStateTable();
    var state = states.ContainsKey(id) ? states[id] : null;

    if (state == null) return null;

    return state.Clone() as FormatterBattleState;
}
```

然后完成技能附加状态的效果：

```
protected override void OnSkillHit()
{
    // 这里是一些业务逻辑，不用管
    int maxCount = 10;
    int currentCount = battleSkillData.user.info.GetStateCount("jianyi");
    int spentCount = Mathf.Min(currentCount, maxCount);

    // 这是计算破防状态具体减少多少护甲的公式
    decimal percent = (20 + spentCount * 1) * -1;

    // 得到上面的那个类的实例
    FormatterBattleState state = BattleStateLoader.GetInstance().GetState("porenzhan");

    // 赋值状态的破甲属性
    state.attributes.Add("def_percent", percent);

    battleSkillData.target.info.TakeState(state);
}
```
这样看起来没什么问题了，虽然 state 是引用，但是在获取 state 的时候使用了克隆，按理说 `attributes` 属性也应该会被克隆一份，可实际上如果连续使用破刃斩两次，就会提示 `def_percent` 这个 Key 已经存在了。

也就是说虽然克隆了 state 出来，但是 state 里面的 `attributes` 属性却依然还是引用。

为了验证自己的观点，将获取状态的方法改为：

```
public FormatterBattleState GetState(string id)
{
    var states = TableLoader.GetInstance().GetStateTable();
    var state = states.ContainsKey(id) ? states[id] : null;

    if (state == null) return null;

    // 注释掉，看看是不是因为克隆的问题
    //return state.Clone() as FormatterBattleState;

    // 创建一个新的对象，然后手动赋值
    var temp = new FormatterBattleState
    {
        id = state.id,
        icon = state.icon,
        attributes = new Dictionary<string, decimal>(state.attributes),
    };

    return temp;
}
```

再进入游戏测试，连续使用两次破刃斩不会报错了。
果然是克隆不完全的关系，然后我就去查了一下克隆的更细致说明，得到下面的结论。
## 浅拷贝
浅拷贝只是将对象的值逐一赋值到一个新的对象，值类型的变量没问题，因为拷贝的是副本；但是引用类型的变量，拷贝到新的变量却依旧是引用类型，所以对于含有引用类型数据的类不能直接用浅拷贝，而应该用深拷贝。
## 深拷贝
深层拷贝有许多方法，比如上面的：

```
var temp = new FormatterBattleState
{
    id = state.id,
    icon = state.icon,
    attributes = new Dictionary<string, decimal>(state.attributes),
};
```

直接赋值给一个新创建的实例，但这种方法每次都要手写，十分繁琐。
## 深拷贝的方法
### 二进制序列化
第二种就是将一个对象序列化，再反序列化，就变成一个新的实例了。

```
using System;
using UnityEngine;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using System.Runtime.Serialization;
using System.Runtime.Serialization.Formatters.Binary;

// 序列化注解
[Serializable]
public class FormatterBattleState : FormatterData, ICloneable
{
    public string icon;
    public string name;

    // 持续回合数,-1代表无限回合
    public int remain_time;

    // 状态的最大层数，达到该值后则无法继续叠加，只会刷新回合
    public int max_count = 1;

    /**
     * 附加属性，基础属性后缀为_percent则为百分比，负值代表减少
     * atk: 物理攻击
     * def: 物理防御
     * mat: 魔法攻击
     * def: 魔法防御
     * mov: 速度
     * skill_power: 技能威力
     * damage_decrement: 伤害减免
     */
    public Dictionary<string, decimal> attributes = new Dictionary<string, decimal>();

    public object Clone()
    {
        return this as object;
    }

    // 二进制序列化
    public FormatterBattleState DeepClone()
    {
        using (Stream objectStream = new MemoryStream())
        {
            IFormatter formatter = new BinaryFormatter();
            formatter.Serialize(objectStream, this);
            objectStream.Seek(0, SeekOrigin.Begin);
            return formatter.Deserialize(objectStream) as FormatterBattleState;
        }
    }
}
```

给类加上一个注解：`[Serializable]`，注意不要打错成：`[SerializeField]`，后者是给类的字段添加的注解，然后这个类就可以被序列化了。

### JSON 序列化
第三种是把类转化成 JSON 字符串，然后再把 JSON 字符串转回对象，得到的也是一个新的实例，可以使用 LitJson 插件。
### 反射机制
第四种是利用反射原理实现自动赋值变量：

```
private static TOut TransReflection<TIn, TOut>(TIn tIn)
{
    TOut tOut = Activator.CreateInstance<TOut>();
    var tInType = tIn.GetType();
    foreach (var itemOut in tOut.GetType().GetProperties())
    {
        var itemIn = tInType.GetProperty(itemOut.Name); ;
        if (itemIn != null)
        {
            itemOut.SetValue(tOut, itemIn.GetValue(tIn));
        }
    }
    return tOut;
}
```

调用方法：

```
NewUserInfo newInfo = TransReflection<UserInfo, NewUserInfo>(info);
```

### AutoMapper
这个对于初学 C# 的我来说比较陌生，其实跟反射差不多，官方文档：[https://docs.automapper.org/en/latest/](https://docs.automapper.org/en/latest/)

示例代码：

```
public class Foo
{
    public int ID { get; set; }

    public string Name { get; set; }
}

public class FooDto
{
    public int ID { get; set; }

    public string Name { get; set; }
}

public void Map()
{
    var config = new MapperConfiguration(cfg => cfg.CreateMap<Foo, FooDto>());

    var mapper = config.CreateMapper();

    Foo foo = new Foo { ID = 1, Name = "Tom" };

    FooDto dto = mapper.Map<FooDto>(foo);
}
```

教程介绍：[https://www.cnblogs.com/gl1573/p/13098031.html](https://www.cnblogs.com/gl1573/p/13098031.html)