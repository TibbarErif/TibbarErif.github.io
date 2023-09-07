---
title: 【Unity小技巧】AssetBundles 加载图集
date: 2021-12-23 20:01:59
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
前面已经实现了 AssetBundles 加载资源的方法。
但是今天又遇到了一个难题……那就是如何读取图集资源？

## 图集
图集一共有两种，一种是通过第三方软件打包的图集，另一种是 Unity 内部切分的图集。

Unity 切割的图集如下所示：

![Unity 切割的图集](https://pic.imgdb.cn/item/61c4669c2ab3f51d91a61cba.jpg)

上面的角色战斗图是由许多张序列帧动画图片合成的。
而前文介绍的读取 Assets Bundle 方法，只能读取到一张 Sprite 图片，无法读取切割后的小图。

## 读取图集
参考网站：[https://tsubakit1.hateblo.jp/entry/2015/11/14/000000](https://tsubakit1.hateblo.jp/entry/2015/11/14/000000)
读取 Assets Bundle 图集的方法可以参考上述网站的方法。

Unity 的方法 `AssetBundle.LoadAsset<Sprite>` 只能读取单张的图片，并非我们的需求。
在 Resources 加载资源中，有一个 `LoadAll` 的方法，那么是不是在 AssetsBundle 里面也可以用这个方法来加载图集？
查看文档 API，发现恰巧也有一个方法： `AssetBundle.LoadAllAssets`，用这个方法就可以读取图集了吗？

其实可以猜到结局，因为 AssetBundle 是分包加载，用这个方法只不过是加载某个包底下的所有资源而已，并不是加载图集的方法。

![AssetBundle.LoadAllAssets](https://pic.imgdb.cn/item/61c46c9e2ab3f51d91a811fe.jpg)

不过还是眼见为实，实际测试一下吧：

```
// 此方法在前文有介绍，这边不再赘述，主要是加载主包和依赖包
InitLoadAssets(packageName);

var res = AssetBundle.LoadFromFile(resPath + packageName);
var items = res.LoadAllAssets(typeof(Sprite));

foreach(var item in items)
{
    Debug.Log(item);
}
```

在战斗图的包里面放两个角色的战斗图：

![战斗图文件夹](https://pic.imgdb.cn/item/61c46ee02ab3f51d91a8d716.jpg)

最后启动游戏测试：

![输出结果](https://pic.imgdb.cn/item/61c46e6d2ab3f51d91a8aa82.jpg)

咦？原本以为它应该像 `LoadAsset` 一样不能加载切割的图片，结果竟然能把小图也加载出来？
但是……两张图片全部被加载了。
虽然结果有些意外，但依然不是我们想要的。
如果只为拿一张图片，就得加载整个包，那得多浪费内存啊。

**我们的需求是：加载 AssetsBundle 包中一张图集的所有小图。**

接着继续测试，发现 API 里面还有一个 `LoadAssetWithSubAssets` 方法。

![LoadAssetWithSubAssets](https://pic.imgdb.cn/item/61c470312ab3f51d91a94027.jpg)

那这个方法会不会是我们想要的呢？
查看一下文档的说明：

> Loads asset and sub assets with name from the bundle.

意思是说：从包中加载带有名称的资产和子资产。
此方法的返回值还是 `Object[]`，应该八九不离十了。
测试一下就清楚了！

```
InitLoadAssets(packageName);

var res = AssetBundle.LoadFromFile(resPath + packageName);
var items = res.LoadAssetWithSubAssets(resName);

foreach(var item in items)
{
    Debug.Log(item);
}
```

测试结果：

![LoadAssetWithSubAssets 测试结果](https://pic.imgdb.cn/item/61c471312ab3f51d91a992b8.jpg)

好耶！确实是我们想要的结果。

## 封装方法
接着，我们要封装一个读取图集的方法。
在之前对泛型的学习中，我涨了一些姿势……

先来看一个错误的示例：

```
public T[] LoadAll<T>(string packageName, string resName)
{
    InitLoadAssets(packageName);

    var res = AssetBundle.LoadFromFile(resPath + packageName);
    var items = res.LoadAssetWithSubAssets<T>(resName);

    return items;
}
```

报错信息如下：

![泛型返回值的问题](https://pic.imgdb.cn/item/61c472592ab3f51d91a9f3df.jpg)

这是由于**没有加上约束条件导致的。**
当泛型作为返回值时，需要与方法中有相同的约束条件。
`LoadAssetWithSubAssets` 方法里有一个返回泛型的，注意右边的 `where` 即是约束条件。

![约束条件](https://pic.imgdb.cn/item/61c471cf2ab3f51d91a9c40f.jpg)

如果我们也想封装一个返回泛型的方法，那么约束条件必须与它保持一致。
如下所示：

```
// 加上约束条件  where T : Object
public T[] LoadAll<T>(string packageName, string resName) where T : Object
{
    InitLoadAssets(packageName);

    var res = AssetBundle.LoadFromFile(resPath + packageName);
    var items = res.LoadAssetWithSubAssets<T>(resName);

    return items;
}
```

如此一来，程序就不再报错了。
这样，AssetsBundle 加载图集的方法也完成了(～￣▽￣)～

## 战斗图对应表
最后再附上战斗图对应动作表，RPG Maker 系列生成的纸娃娃模型可用：

![战斗图对应动作表](https://pic.imgdb.cn/item/61c475972ab3f51d91ab1cf5.png)

如果每一个战斗都要制作一个 Unity 动画，那就太麻烦了。
配置动画状态机也很容易出错，所以这里我采用读取图集，然后自己原创了一个动画系统的方法。

毕竟这些战斗图的格式都是统一的，只要根据这个战斗动作对应表加载固定位置的动画即可。
（可惜的是……这套动作图没有行走动画）