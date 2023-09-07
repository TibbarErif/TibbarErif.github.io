---
title: 【Unity小技巧】解决 AssetsBundle Fatal Error！闪退问题
date: 2021-12-29 20:12:54
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
在使用了 AssetsBundle 之后，从前天开始莫名其妙会报错然后闪退退出 Unity，而且还是偶发性的。
只要 Build 重建资源就有可能会出现这种情况。

报错如下：
![AssetsBundle Fatal Error！](https://pic.imgdb.cn/item/61cc51ef2ab3f51d91fa7941.png)
关键报错信息：

```
The file 'archive:/CAB-8bc5a956f7efa6356fcd1d00c8005f99/CAB-8bc5a956f7efa6356fcd1d00c8005f99' is corrupted! Remove it and launch unity again!
[Position out of bounds!]
```

里面其实已经提到了问题的原因：

```
The file 'archive:/xxxxx' is corrupted!Remove it and launch unity again!
```

意思是说这个文件已经损坏了，只要将它删掉再重启即可。

## 原因
想起来我正在制作菜单的 UI，中间有一次把旧的 prefab 删掉了，后面又创建了一个新的菜单，名字相同的，并且放在同一个目录，然后重新打包了一次。所以很可能是因为名称修改之后，本地的 meta 文件没有更新，导致 AssetsBundle 读取到的还是被删除的那个菜单 UI 的 meta 信息。

还有一种可能是将不同类型的文件（如 .png，.jpg）却取了相同的名字，打包时又放在同一个包里面，导致文件名称冲突无法正常识别，不过我遇到的不是这种情况，如果是这种情况只要把同名的文件改成别的，再打包一次就能解决。

## 方案一：修改 manifest
AssetsBundle 打包保存的是预制体的 guid 信息。

![meta 的 guid](https://pic.imgdb.cn/item/61cc59a32ab3f51d910031cb.jpg)

找到 AssetsBundle 的 `prefab.manifest` 文件（prefab 是我打的包名），因为已经定位到问题是菜单预制体了，所以找到这个包即可，然后搜索到报错的那个 guid 修改一下就可以。

## 方案二：恢复 Meta 文件
只要把 meta 的 id 修改为被删除的那个预制体的 id 即可。
第一步，找到预制体的 meta 文件：

![预制体的 meta 文件](https://pic.imgdb.cn/item/61cc53312ab3f51d91fb69d4.jpg)

第二步，用文本编辑器打开这个文件，修改其中的 guid 字段：

![guid](https://pic.imgdb.cn/item/61cc53312ab3f51d91fb69da.jpg)

这个 guid 字段可以在你的版本管理工具查找，我用的是 Git，直接在提交记录就可以看到修改记录，找到变化的 ID 进行对应的修改即可（所以说，代码托管十分重要，遇到这种情况就可以找到之前修改的记录了）。

## 方案三：删除 Library
直接删除项目下的 `Library` 目录。
![Library](https://pic.imgdb.cn/item/61cc51622ab3f51d91fa15ce.jpg)

然后重新启动 Unity，就会自动识别资源的 meta 信息了。

## 方案四：删掉 meta 文件
当你知道哪个文件出问题的时候也可以这么做。
找到 `Menu.prefab.meta` 删掉，然后重启 unity 就会生成新的 meta 文件。
删除 meta 之后，需要重新配置 Menu 为 prefab 包，再用 AssetsBundle Browser 重新构建。