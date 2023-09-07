---
title: 【Unity小技巧】资源的加载：Resources 和 AssetBundles
date: 2021-12-22 18:58:27
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
游戏中有许多资源，例如：音频、图片，这些资源的加载还是有一些门道的。
Unity 自带了一个 Resources 用来加载资源，可以说是十分好用了。
但是最近几天刷知乎看到不少人都在说不推荐用这种方式，然后我就去仔细搜了些相关资料。
下午也花了一两个小时看了一些视频教程，有了一些小收获，就记下来吧。

## 特殊目录
在 Unity 中有一些特殊的【文件目录】，这些目录有特别的用处。

### Resources
官方文档：[https://docs.unity3d.com/ScriptReference/Resources.html](https://docs.unity3d.com/ScriptReference/Resources.html)

摘录官方文档原话：

> All assets that are in a folder named "Resources" anywhere in the Assets folder can be accessed via the Resources.Load functions. Multiple "Resources" folders may exist and when loading objects each will be examined. 

意思是说，在我们开发环境的根目录 Assets 下，你可以在任意的位置，建立任意个名字为“Resources”的文件夹，这些文件夹下的资源都可以通过 `Resources.Load` 函数加载。

Resources 文件夹下的资源，会在游戏构建的时候被打包。

### Plugins
在 Assets 下新建一个 Plugins 目录，可以用来保存第三方的插件，比如 JSON 相关的 DLL 扩展。

所有的扩展都不需要手动加载，只要导入到 Assets 目录就会自动加载。

### Scenes
新建工程 Unity 默认创建的一个目录。
其实这个文件夹没有特别的意义，可以随意删除和修改。
你也可以按照 Unity 的意思，把游戏的场景全部放在这个目录下面。

### StreamingAssets
官方文档：[https://docs.unity3d.com/Manual/StreamingAssets.html](https://docs.unity3d.com/Manual/StreamingAssets.html)

在 Assets 下创建一个新的文件夹，命名为：StreamingAssets。
这个目录在不同的平台是不一样的，而且在安卓系统还有读取权限问题。
推荐使用 `Application.streamingAssetsPath` 来获取该目录。

它的作用与 Resources 类似，用来保存一些游戏的资源，需要注意的是，这个文件夹下的所有文件在构建发布的时候会原封不动的复制到构建后的游戏中。

读取 StreamingAsset 目录下的文件，示例代码：

```
/**
* 读取StreamingAsset目录下的文件
*/
public static string LoadStreamingAsset(string filename, string suffix = "json")
{
    string path = Application.streamingAssetsPath + "/" + filename + "." + suffix;
    StreamReader streamReader = new StreamReader(path);

    return streamReader.ReadToEnd();
}
```

可以在该目录下保存一些配置文件。
（这个目录十分重要，下文还会提到）

## 资源加载
Unity 的资源通常是放在 Resources 或者 StreamingAssets 这两个目录。
放在 Resources 下的资源文件，可以直接用 Unity 内置的 API 加载，非常方便。

对于一般小型的游戏可以直接用这种方法快速构建出游戏 demo。
而对于一些大型游戏和对性能要求比较高的，就不推荐用这种方法了。

### 粗暴的拖动
拖动是加载资源的最快方法，毕竟你只要声明一个 `public` 变量，然后把脚本挂在物体上面，就可以把资源拖到这个 public 的变量上面了。

这种方法虽然是最简单的，但也是最粗暴的。
当一个脚本需要几十个资源的时候……手可能都要软了。

而且也不美观，一个脚本点开一个……好家伙，一个脚本挂了几十个 public 变量？？？
跟你一起开发的小伙伴可能当场就散伙了……

这种做法不仅粗暴，而且也不规范。
万一哪天替换了新的资源，你又得重新挂一遍。

这里的拖动指的是【动态加载的资源】，而对于一些依赖脚本，使用拖动完全是没问题的，在某种情况甚至可以节约性能的开销。

假设有一个刚体组件，我们要获取它可以用 `gameObject.GetComponent<刚体组件>` 来实现。物体不多的时候还好，省去了拖动的烦恼，但如果这样的物体在游戏场景存在很多个，并且还在被源源不断的创造出来，那么性能的开销就会很大。与其这样还不如声明一个 public 的刚体变量，直接把物体的刚体组件拖进去。

还有一种情况是子物体也挂着某个脚本，而父物体需要用到子物体的脚本，也可以用拖动的方式，不然寻找子物体也是一件费力的事情。

总之，资源的动态加载不推荐用拖进去的，但如果是脚本依赖某些组件或其他脚本，拖进去也可以，一个简单的操作能节省大量性能开销还是很划算的，根据应用场景选择最佳方案吧。

### Resources 的缺点
`Resources.Load` 可以直接加载 Resources 目录下的资源，十分方便，小白上手也简单。

但是查了很多网上的资料，大都不推荐用这种方法。

因为 Resources 文件夹是可以多个的，因此在构建的时候，Unity 会将它们合并在一起，然后打包到游戏中。

对于单机游戏，其实差别不大，即使你放到别的地方，文件依然也要打包到游戏里面，这里的打包缺陷其实主要指的是一些手游或者网游。

单机游戏并不需要【热更新】，资源固定写死对单机游戏来说没有差别。手游和网游就不一样了，通常更新频率非常高，比如王者农药时不时就得更新一下，但是我们并不需要直接到应用商店重新下载，而是直接在启动游戏后就能动态下载更新补丁，这其实就是【热更新】了；单机游戏一般都是直接替换本地文件，比如我们直接下载补丁，然后手动把文件复制粘贴到旧版本的目录，把旧版的文件用新的补丁覆盖掉，实现手动更新。

Resources 目录下的资源，无法实现动态更新，意味着你在打包时把哪些资源放到这个目录，打包出来的游戏就只能加载这些资源，如果你后期想动态的添加或者修改资源，就得重新构建一次。

好像有人说 Resources 最大只能保存 2GB 的资源，目前来说我还没完整的发布过游戏，不清楚是否真的。

因为我做的是 2D 游戏，整体应该不会很大，如果是大型 3D 游戏的话，可能就要考虑一下性能问题了，Resources 目录需要合并成一个，也就是说这样存在读取的时间，如果 Resources 目录下的文件非常多的话，那不用想都知道加载的速度会变得很慢了，但具体慢多少就不清楚了，毕竟……本兔连一个游戏作品都没发布过。

### StreamingAssets
除了上面介绍的用 `StreamReader` 读取文件之外，还有一种特殊的动态加载资源的方法。
叫做【分包加载】，见名知意。
意思是把资源分成几个不同的包，分别加载。
听起来就知道可以解决大文件的读取问题。

下文介绍。

## AssetBundles-Browser
AssetBundles 既然叫做【分包】，那自然就涉及到打包的概念了。
Unity 没有内置的方法来实现 AssetBundles 包的管理，需要依赖第三方工具包。

我们需要到 Github 去下载包管理工具【Asset Bundle Browser】。
下载地址：[https://github.com/Unity-Technologies/AssetBundles-Browser](https://github.com/Unity-Technologies/AssetBundles-Browser)

点击 Tag，找到自己想要的版本下载即可，最新版为 v1.7.0，一般下载这个就行了。

这里再介绍一下包的安装方法，比如我们要安装的是 1.7.0 最新版的。
直接从 Github 下载包文件：[https://github.com/Unity-Technologies/AssetBundles-Browser/archive/refs/tags/1.7.0.zip](https://github.com/Unity-Technologies/AssetBundles-Browser/archive/refs/tags/1.7.0.zip)

将下载后的压缩包解压，得到如下文件：

![AssetBundles-Browser](https://pic.imgdb.cn/item/61c400022ab3f51d91776a01.jpg)

这里我们只需要 Editor 这个文件夹，直接把这个文件夹复制到工程目录下面。

![Editor](https://pic.imgdb.cn/item/61c400422ab3f51d91777ca3.jpg)

OK，这样就手动导入了这个包。
然后点开顶部的菜单，即可发现增加了一个 AssetBundles Browser 选项：

![AssetBundles Browser](https://pic.imgdb.cn/item/61c4006d2ab3f51d91778925.jpg)

看到这个就代表导入成功了。Editor 文件夹也是 Unity 中比较特殊的一个文件夹，这个文件夹是与编辑器有关的，不过这是属于大神的领域了，本兔连一个游戏都没做出来，就不再仔细研究了。

## AssetBundles
具体教程可以参考：[https://www.bilibili.com/video/BV1LD4y1m7kF?p=1](https://www.bilibili.com/video/BV1LD4y1m7kF?p=1)

这个视频讲的很好，虽然我也是刚学……大概花一个小时就能现学现卖了。
也没什么特别的难点，按部就班就行了，这边就不详细介绍了。

## 关于 Resources 的补充
后面又查了一下关于 Resources 加载的资料，发现一个比较详细的，可以参考：[https://answer.uwa4d.com/question/598b1a22fb389f61434c5d4c](https://answer.uwa4d.com/question/598b1a22fb389f61434c5d4c)

其中比较关键的一个是：Resources 合并后会生成一个序列化的文件，而在游戏加载的时候会读取这个序列化的文件，如果 Resources 目录下的文件太多，就会导致游戏在首次启动的时候变得很慢。

序列化的文件我这边大概猜下：应该是把文件路径生成一个随机的 ID，一般是哈希值之类的，根据这个 ID 就可以找到对应的资源文件。这么猜测的理由其实很简单，因为 Resources 既然可以存在多个，并且在游戏构建后会合并成一个，那肯定有存在【重名】并且【同类型】文件的可能性，比如有两张 `huotu.jpg` 的图片，为了保证加载的资源唯一性，就会用一个随机的 ID 进行区分；人可以存在同名，但身份证号码是唯一的。

所以还是尽可能不要用 `Resources.Load` 来加载资源了，即使是单机游戏，除非……你想让玩家打开游戏，黑屏几分钟（真的有这样的游戏）。
## AssetBundles 示例
学有所用，现学现卖！

### 单例模式
一直都看到别人用单例模式，其实在 Unity 中好像没什么必要。
业界达成了某种共识？好吧，那我也看看。
当做熟悉 C# 代码了，然后还顺便学了下泛型相关的知识：

第一种单例，纯粹的工具类，不挂在 Unity 的 GameObject 上面的：

```
public class Singleton<T> where T : new()
{
    static T instance;

    public static T GetInstance()
    {
        if (instance == null)
        {
            instance = new T();
        }

        return instance;
    }
}
```

没想到泛型居然可以这么用！？
这里有一个小问题，如果要实例化泛型，那么必须要加上：`new()`，原理是什么我也不懂，总之它能实现就好了。

C# 是强类型语言，如果不使用泛型，而是直接返回 Singleton，那这样只是得到 Singleton 类，里面除了一个 GetInstance 之外没有任何方法，这样即使能得到单例，但没什么用。

没什么卵用的示例：

```
public class Singleton
{
    static Singleton instance;

    public static Singleton GetInstance()
    {
        if (instance == null)
        {
            instance = new Singleton();
        }

        return instance;
    }
}
```

我们用单例模式，肯定是要拿到子类，然后用子类的方法，拿到这个 Singleton 类根本没用。
因此这里必须要用泛型来延迟设定变量的类型，这样我们才能延迟设定子类。
类的名字：`Singleton<T>` 也就是说这里的 T 是可以在继承的时候才进行设定的。

```
public class AssetBundleLoader : Singleton<AssetBundleLoader>
{
    // ... 这里写业务逻辑
}
```

如此一来，GetInstance 方法得到的就是 AssetBundleLoader 这个类了。
这种单例模式适用于不需要继承 MonoBehaviour 的工具类。

MonoBehaviour 是所有需要挂在 GameObject 组件的基类。
并且如果一个类继承了 MonoBehaviour，那么它就不能再用 `new` 来实例化了。

假如我们要让一个继承 MonoBehaviour 的类，也是一个单例——就是自相矛盾了。
但是看了上面视频，居然真的可以实现？！

下面是第二种单例模式，该模式可以继承 MonoBehaviour：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MonoSingleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T instance;

    public static T GetInstance()
    {
        if(instance == null)
        {
            var obj = new GameObject();
            obj.name = typeof(T).ToString();
            DontDestroyOnLoad(obj);

            instance = obj.AddComponent<T>();
        }

        return instance;
    }
}
```

上面的代码是直接照抄视频里面的，并非我原创。
看它具体的实现：

```
var obj = new GameObject();
obj.name = typeof(T).ToString();
DontDestroyOnLoad(obj);

// 最终得到的是泛型指定的脚本组件
instance = obj.AddComponent<T>();
```

这里可以发现它是实例化了一个 GameObject（游戏物体）而不是它自身（毕竟继承了 MonoBehaviour 就不能用 new 实例化了）。

`DontDestroyOnLoad();` 为了不会因场景的销毁而被销毁，也就是常驻内存中，要不然切换个场景，单例就被销毁什么的……那也太奇怪了。

最后是 `obj.AddComponent<T>();`，这里才用到泛型，也就是加上泛型指定的类型作为组件。

那么原理也就清楚了，继承了 MonoBehaviour 的单例模式，并不是真正的实例化得到本身的对象，而是先实例化创建一个 GameObject，然后再给 GameObject 添加上泛型指定的脚本组件（也就是自身这个类），其实是用特殊的方法实现“单例”的效果。

具体的使用如下：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public class AssetBundleLoader : MonoSingleton<AssetBundleLoader>
{
    // ... 这里写业务代码
}
```

这样就得到一个可以挂在 GameObject 上面的单例组件了。
但是为什么要继承 `MonoBehaviour` 呢？
在通常情况下，能不继承这个父类就不要继承，因为这个父类可是存在 Update 这个方法的……
让一个工具类继承了 MonoBehaviour，那就会白白损耗一些性能。

工具类指的是跟游戏对象没有直接关系，而是作为辅助工具使用的。
例如：XML 读取类，本地文件读取类。
这些工具没必要挂在 GameObject 上面，因此不需要继承 MonoBehaviour。

之所以要继承 MonoBehaviour，是因为 MonoBehaviour 提供了【协程】的处理。
如果不通过继承 MonoBehaviour，实现协程是比较困难的。

下面来看看我自己写的一个 AssetBundle 加载类。

### AssetBundle 加载
现学现卖的 AssetBundle 加载方法。
同步加载简单测试了下好像没问题，异步加载还没测试过，仅供参考：

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public class AssetBundleLoader : MonoSingleton<AssetBundleLoader>
{
    // 主包
    private AssetBundle mainPackage = null;

    // 依赖包配置文件
    private AssetBundleManifest manifest = null;

    // 所有包
    private Dictionary<string, AssetBundle> assetBundles = new Dictionary<string, AssetBundle>();

    // 资源路径
    private string resPath
    {
        get
        {
            return Application.streamingAssetsPath + "/";
        }
    }

    // 主包名，即asset bundle browser 打包出来的名字
    private string mainPackageName
    {
        get
        {
#if UNITY_UNITY_EDITOR_WIN
            return "WIN";
#elif UNITY_EDITOR_OSX
            return "MAC";
#else
            return "OTHER";
#endif
        }
    }

    /**
    * 加载包资源(异步)
    */
    public void LoadResAsync(string packageName, string resName, UnityAction<Object> onCompleted)
    {
        StartCoroutine(LoadResAsyncAction(packageName, resName, onCompleted));
    }

    private IEnumerator LoadResAsyncAction(string packageName, string resName, UnityAction<Object> onCompleted)
    {
        InitLoadAssets(packageName);

        // 加载目标包
        if (assetBundles.ContainsKey(packageName) == false)
        {
            var target = AssetBundle.LoadFromFileAsync(resPath + packageName);
            yield return target;

            assetBundles.Add(packageName, target.assetBundle);
        }

        var asset = assetBundles[packageName].LoadAssetAsync<Object>(resName);
        yield return asset;

        onCompleted(asset.asset);
    }

    /**
     * 泛型返回值加载资源（同步）
     */
    public T LoadRes<T>(string packageName, string resName) where T : Object
    {
        LoadTargetPackage(packageName);

        // 加载资源
        return assetBundles[packageName].LoadAsset<T>(resName);
    }

    /**
     * 加载包资源(同步)
     */
    public Object LoadRes(string packageName, string resName)
    {
        LoadTargetPackage(packageName);

        return assetBundles[packageName].LoadAsset(resName);
    }

    /**
     * 加载目标包
     */
    private void LoadTargetPackage(string packageName)
    {
        InitLoadAssets(packageName);

        // 加载目标包
        if (assetBundles.ContainsKey(packageName) == false)
        {
            var target = AssetBundle.LoadFromFile(resPath + packageName);
            assetBundles.Add(packageName, target);
        }
    }

    /**
     * 加载主包和配置，以及目标包的依赖包
     */
    private void InitLoadAssets(string packageName)
    {
        string mainPath = resPath + mainPackageName;

        // 加载主包及配置
        if (mainPackage == null)
        {
            Debug.Log("主包路径：" + mainPath);

            mainPackage = AssetBundle.LoadFromFile(mainPath);
            manifest = mainPackage.LoadAsset<AssetBundleManifest>("AssetBundleManifest");
        }

        // 加载依赖包
        string[] dependencies = manifest.GetAllDependencies(packageName);
        foreach (var item in dependencies)
        {
            if (!assetBundles.ContainsKey(item))
            {
                var bundle = AssetBundle.LoadFromFile(resPath + item);
                assetBundles.Add(item, bundle);
            }
        }
    }

    public void Unload(string packageName, bool unloadAllObjects = false)
    {
        if (assetBundles.ContainsKey(packageName))
        {
            assetBundles[packageName].Unload(unloadAllObjects);
            assetBundles.Remove(packageName);
        }
    }

    public void ClearAll(bool unloadAllObjects = false)
    {
        AssetBundle.UnloadAllAssetBundles(unloadAllObjects);
    }
}
```

异步加载其实不算是完全的异步，这里为了简化直接封装了 `InitLoadAssets`，在这个方法里面，加载主包和依赖包是同步的，其实也可以用 yield 改写，使它完全变成协程化的处理，不过缺点是会存在冗余的代码。

>  UnityAction 原来也可以与 delegate 一样使用，原本我一直都是用 System.Action 当做回调方法的类型使用的，又学到了一个小知识。

其实这里还可以结合我前篇写的：[构建舒适的开发环境](https://huotuyouxi.com/2021/12/22/unity-tips-012/)
AssetBundle 需要配置资源所属的包，而且还需要在 Browser 面板上面点 Build（构建）。
对于开发环境来说是比较麻烦的，毕竟我们总不能每次导入一个新资源，就来一遍上面的操作吧？

这里可以根据【构建舒适的开发环境】中写的方法进行一些改动。
当游戏的环境处于【编辑器环境】下，就不使用 AssetBundle 的加载方式，而是直接 Resources 或者其他方法直接读取本地资源，而在【非编辑器环境】下，就按照正常的 AssetBundle 来加载，并且可以结合 Git 的 `.gitignore` 来忽略掉调试用的资源，最后，在游戏制作完成，只需要在 Asset Bundle Browser 构建一次，也就是说资源仅需要打包一次就行了，开发过程即使添加了一些新资源，整个过程也是无感知的，这样就可以实现【无痛】开发了。

## 结尾
游戏的资源管理是一个比较难的地方，因为资源都是加载进内存的，如果没有释放就会一直占据内存，即【内存泄露】，如果泄露的内存越来越多，那么手机游戏就会出现闪退的情况，电脑也同样，只不过手机内存要小很多，比较容易被注意到。

总之，我决定把资源的加载改成 AssetBundles 了。
学到新的东西，手痒想试试(～￣▽￣)～