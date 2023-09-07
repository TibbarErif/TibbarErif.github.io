---
title: 【Unity小技巧】对象即是“引用”
date: 2021-12-21 23:07:12
tags:
 - unity
 - 开发技术

categories:
 - 小技巧
---
## 前言
在一些开发语言中，**对象即是引用**。
例如 C# 和 JavaScript 都是这样的，而从其他语言转过来的，很容易忽视这个小知识点。

## 奇怪的“BUG”
例如有一个 List 类型的变量，先给它初始值。
接着我们再创建一个新的变量，让它等于刚才创建的 List。
再把新创建的变量移除掉 a 元素，输出剩下的元素。

```
var items = new List<string> { "a", "b", "c" };
var test = items;
test.Remove("a");

foreach(var item in test)
{
    Debug.Log(item);
}
```

结果将会输出：

```
b
c
```

结果应该没有任何异议，我们给 `test` 赋值为 `items`，然后再把 `test` 的 a 移除，结果不就剩下 b 和 c 了吗？
但……如果我们输出的是 `items` 呢？

```
var items = new List<string> { "a", "b", "c" };
var test = items;
test.Remove("a");

// 这里输出的是 items，而不是 test
foreach (var item in items)
{
    Debug.Log(item);
}
```

咋看之下，你修改 `test` 变量，关我 `items` 什么事？
如果你以为输出的结果是：a、b 和 c，那就错了。

**因为在 C# 中，对象即引用。**
当执行了语句 `var test = items;` 意味着 test 变量得到的是 `items` 的引用。
而修改了 test 等于修改了 items，最终输出的还是：b 和 c。

大部分的开发语言可能都是这样的，但可怜的 PHP 并不是……（踩坑了）

## 基础类型不是引用
需要注意只有对象类型才是引用，变量的基础类型都不属于引用。

```
int a = 1;
int b = a;
b++;

Debug.Log(b);
```

int 是基础类型，因此结果输出：2
所有基础类型都属于【值类型】，值类型的数据是可以直接使用等号来实现拷贝的。

## 不想被引用！
既然我们都创建一个新变量来保存对象了，为什么还要被当做引用呢？
这样我们创建一个新变量有什么意义？
其实这种设定超级麻烦……每次赋值对象类型都得特殊处理。

### 方案一：重新赋值
这种方法比较无脑，就是直接创建一个空的对象，然后给空对象添加值。

```
var items = new List<string> { "a", "b", "c" };
var test = new List<string>();

foreach (var item in items)
{
    test.Add(item);
}

test.Remove("a");

foreach (var item in items)
{
    Debug.Log(item);
}
```
### 方案二：转圈圈
以 JavaScript 为例：

```
let data = {
        a: "a",
        b: "b",
    };

let test =data;
test.a ="bbb";

console.log(data);
```

输出结果：`{a: 'bbb', b: 'b'}`。
因为在 JavaScript 里面对象也同样属于引用。

如果不希望得到的是引用，可以这样……
先把 JSON 对象转化成 JSON 字符串，再把 JSON 字符串转回 JSON 对象。

如下所示：

```
let data = {
        a: "a",
        b: "b",
    };

// 将json对象转化为字符串，再转化为json对象
let test = JSON.parse(JSON.stringify(data));

test.a ="bbb";
console.log(data);
```

因为 test 得到的是新实例化出来的 JSON 对象，所以不是 data 的引用，也就不会改变它的值了。

### 方案三：拷贝对象
**如果使用方案一和方案二，不出意外你已经被扫地出门了。**

接下来才是真正的解决方法。
在 JavaScript 中可以直接使用 `Object.assign()` 关键词来拷贝一个对象。
当然，并不是直接赋值就行了，下面是**错误的**：

```
let data = {
            a: "a",
            b: "b",
        };

// 错误的示例！
let test = Object.assign(data);

test.a ="bbb";
console.log(data);
```

如果直接赋值一个参数，这样得到的 `test` 变量依然是 `data` 的引用。
**正确** 的使用方法如下：

```
let data = {
        a: "a",
        b: "b",
    };

// 正确的！需要传一个空对象
let test = Object.assign({}, data);

test.a ="bbb";
console.log(data);
```

如此一来 `test` 变量就不再是 `data` 的引用了。
`Object.assign(target, source)` 方法用于将所有可枚举属性的值从一个或多个源对象 source 复制到目标对象，如果存在相同的键，则 source 的键会覆盖掉 target 的键，所以当你把第一个参数写成要拷贝对象，其实还是得到了引用，这里写了个 `{}` 空对象，这是一个新建出来的对象，因此不再是原来的引用了，两个变量的位置不能搞混了。

```
let data = {
        a: "a",
        b: "b",
    };

let test = Object.assign({a:"fff", c:"ccc"}, data);
console.log(test.a, test.c);
```

上面的例子中，data 对象中有 a，因此会覆盖左侧的 `a:"fff"`，最终得到的 a 为 `"a"`，而 c 是 data 没有的，则原样保留，最终输出结果为：`a ccc`。

详细的用法可参考手册：[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

### 方案四：我也要拷贝！
那……C# 能这样直接拷贝吗？
答案是……不行！
并没有提供一个方法可以直接拷贝对象。

具体的操作如下，先定义一个 `TestData` 类用来测试，它需要继承 `ICloneable`：

```
using System;

public class TestData : ICloneable
{
    public string a, b, c;

    public object Clone()
    {
        return MemberwiseClone();
    }
}

```

好了，这样它就是一个可以被拷贝的对象。

```
 var data = new TestData
    {
        a = "aaa",
        b = "bbb",
        c = "ccc"
    };

var test = data.Clone() as TestData;
test.a = "fff";

Debug.Log(data.a);
```

通过上述的操作，终于不再是引用了！
在调用 `Clone` 方法后，还需要将变量转化为 `TestData` 类型，这样才算完成。

好麻烦啊 （╯‵□′）╯︵┴─┴
不过，回到开头 List 类型的变量，则可以使用比较简单的方式直接拷贝：

```
List<string> data = new List<string> { "a", "b", "c" };
var test = new List<string>(data);

test.Remove("a");

foreach (var item in data)
{
    Debug.Log("data=" + item);
}

foreach (var item in test)
{
    Debug.Log("test=" + item);
}
```

对于简单的值类型 List，通过 `new` 一个新的对象，得到的就不再是引用了。
不过，如果 List 是引用类型的，例如上面示例的 `TestData` 类；
`List<TestData>` 用这种方法复制出来的却依然是引用。
对于这种【引用类型】的数据就要用到上面继承 `ICloneable` 的方法来解决了。

总之，使用对象的时候还是得小心一点，不然很容易出现“离奇的 BUG”。

## 浅拷贝和深拷贝
参考文章：[https://www.cnblogs.com/dotnet261010/p/12329220.html](https://www.cnblogs.com/dotnet261010/p/12329220.html)
有兴趣的自行了解吧……

## 结尾
到底是谁想出来的……为什么对象要是引用啊？
此时应当高呼：**壮哉我大 PHP 神教！**