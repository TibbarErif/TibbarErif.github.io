---
title: 【Unity小技巧】随机系统
date: 2021-12-27 12:33:08
tags: 
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
随机数在游戏中无处不在。
例如扭蛋抽奖、怪物掉落物、怪物 AI 等等。

那么，如何实现如何按照设定好的几率，在几件奖励中选择其中一件呢？
## 实现思路
假定我们现在要实现手游的「扭蛋池」。
扭蛋池中有 5 个不同的角色，玩家每次扭蛋可以从里面抽出其中一个。
每个角色的抽取几率，都可以通过配置参数的方式进行控制。

示例如下：
- 皮卡丘：10%
- 火球鼠：10%
- 小锯鳄：10%
- 杰尼龟：20%
- 妙蛙种子：50%

总计几率为 100%，即一定可以从中抽取到一个角色。
当然，也可以设计成抽到空的情景，不过这里就不演示了。

这种范围性的概率判断，不能用通常的方法。
而是要使用「线段法」来实现：

![线段法](https://pic.imgdb.cn/item/61c945cb2ab3f51d911a15e5.jpg)

假设存在一条长度为 100 的线段，根据概率，每个角色占据这条线段的比例分别为：10、10、10、20、50。只要我们随机从线段上面取出一个点，这个点落在哪个范围，那就抽到哪个角色。

## 处理随机数
使用 Unity 自带的方法 `UnityEngine.Random.Range(min,max)` 来实现随机数即可。
需要注意的是这个方法本身只能取到 `[min, max)` 的值，即不包括 max，我们需要手动加上 1。
这是一个通用的方法，将处理几率的方法封装为 `Calculator` 工具类以便后续使用：

```
/**
* 取范围[min,max]
*/
public static int GetRandomInt(int min, int max)
{
    return UnityEngine.Random.Range(min, max + 1);
}
```

## 奖品基类
在实现抽奖之前，我们需要先实现奖品。

```
public class PrizeBase 
{
    // 抽取几率
    public float rate;

    // 实际获得的奖品
    public string prize;
}
```

奖品暂时用一个字符串演示，可以改成任何你需要的东西。
之所以要命名为 `Base` 类，是因为它不仅能用来抽奖，还可以实现很多东西，需要用到抽奖机的时候，只要让子类继承它就可以了。

## 抽奖机
计算出每个奖品在这条线段上所在的点的范围（其实只要一个点就够）。
![奖品分布图](https://pic.imgdb.cn/item/61c94a342ab3f51d911b8e1e.jpg)

根据上面设定的角色几率，我们可以获得每个角色所在的区间范围：
- 皮卡丘：`[1, 10]`
- 火球鼠：`[11, 20]`
- 小锯鳄：`[21, 30]`
- 杰尼龟：`[31, 50]`
- 妙蛙种子：`[51, 100]`

生成一个 `[1, 100]` 的随机数 x，我们只需要依次判断：

- `x <= 10` 皮卡丘
- `x <= 20` 火球鼠
- `x <= 30` 小锯鳄
- `x <= 50` 杰尼龟
- `x <= 100` 妙蛙种子 

原理已经很清楚了，那么开始实现：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PrizeMachine
{
    public static PrizeBase GetRandomPrize(List<PrizeBase> prizes)
    {
        // 判断几率是否超过100%
        float total = 0;
        foreach (var item in prizes)
        {
            total += item.rate;
        }

        if (total > 100)
        {
            Debug.LogError("奖品几率设置不正确（超过100%）");
        }

        int random = Calculator.GetRandomInt(1, 10000);
        int index = 0;
        foreach (var item in prizes)
        {
            index += (int)(item.rate * 100);
            if (index >= random)
            {
                return item;
            }
        }

        // 当设定的奖品总计几率不为100%时，即没抽到任何东西，返回空
        return null;
    }
}
```

这里之所以要使用 `[1, 10000]` 是因为可以支持两位小数点，例如：0.25% 这样的几率。

## 测验结果
编写测试代码：

```
Dictionary<string, int> res = new Dictionary<string, int>();

var prizes = new List<PrizeBase>
{
    new PrizeBase{ prize ="pikaqiu", rate = 10 },
    new PrizeBase{ prize ="huoqiushu", rate = 10 },
    new PrizeBase{ prize ="xiaojue", rate = 10 },
    new PrizeBase{ prize ="jienigui", rate = 20 },
    new PrizeBase{ prize ="miaowazhognzi", rate = 50 },
};

for (int i = 0; i < 100000; i++)
{
    var prize = PrizeMachine.GetRandomPrize(prizes);

    if (prize == null) continue;

    if (res.ContainsKey(prize.prize))
    {
        res[prize.prize]++;
    }
    else
    {
        res[prize.prize] = 1;
    }
}

int total = 0;
foreach (var item in res.Keys)
{
    total += res[item];
    Debug.Log(item + "=" + res[item]);
}

Debug.Log("total=" + total);
```

得到的结果与设定的几率差不多，虽然存在一些误差，不过这样就足够了。

![10万次模拟测试](https://pic.imgdb.cn/item/61c94fc42ab3f51d911d6ebe.jpg)
## 适用场景
对于任何需要从给定奖池中抽取其中一件的随机性事件，均可使用此脚本实现。
示例：随机宝箱、怪物掉落物、怪物 AI 随机选择技能释放等等。