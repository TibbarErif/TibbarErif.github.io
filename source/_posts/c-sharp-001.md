---
title: C# 的研究（一）利用内置函数简化数值判断与 ref、out、in 的用法
date: 2021-10-21 09:34:03
tags:
 - C Sharp
 - 开发技术

categories:
 - 日常学习
---
## 前言
记录学习 C# 过程的小知识点。

## 简化数值判断的方法
在进行数值计算的时候，经常会要判断一个数值“越界”的问题。例如商城购物，用户可以用优惠券下单抵免部分金额，但是商品肯定存在一个最低价，而不是叠加了优惠券之后，出现负数倒贴的情况。

比如点餐有“无门槛红包 10 元”这样的优惠券，假设用户只点了一份 8 元的餐，然后用了 10 元的红包，实际支付金额应该是 0，而不是 -2。

这是一个很简单的逻辑，不用想都能写出来：

```
public float TotalMoney(float originalPrice, float discount) {
    float totalMoney = originalPrice - discount;
    if(totalMoney < 0) {
        totalMoney = 0;
    }
    return totalMoney;
}
```

用一个 `if-else` 结构即可防止数值越界，但是这样写其实很麻烦而且不雅观。
实际上可以用 C# 内置的 `Mathf` 函数来简化这种结构：

```
public float TotalMoney(float originalPrice, float discount) {
    return Mathf.Max(originalPrice - discount, 0);
}
```

`Mathf.Max` 方法从两个值中选择一个较大的数返回，无需再手动写 `if-else` 结构判断，心智负担大大降低。
同理，还有 `Mathf.Min` 方法可以从两个值中选择一个较小的数返回。

## ref、out 和 in
microsoft 官方文档：[ref](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/ref)、[out](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/out-parameter-modifier)。

### ref
ref 可以用来修饰变量、函数返回值。

#### 引用传参
我们知道将变量传递给函数作为参数是不会影响原来的值，这种方式的参数叫做「形参」，意思是形式上的参数。

示例：

```
 private void Start()
{
    int a = 0, b = 0;

    TestMethod(a, b);

    Debug.Log(a);
    Debug.Log(b);
}

public void TestMethod(int a, int b)
{
    a -= 1;
    b -= 1;
}
```

虽然函数体对 a 和 b 进行了减法运算，但实际上只是改变了形参的值，原参数并不会改变，即最终仍输出两个 0，而不是 -1。
如果想要改变传进来参数的值就要用到 `ref` 关键字了，当 `ref` 关键字修饰了一个函数的参数时，这个参数就叫「引用参数」，引用参数的值在函数内发生改变时，原参数的值也将被改变。

示例：

```
private void Start()
{
    int a = 0, b = 0;

    // 在传入变量时也需要加上 ref 关键词
    TestMethod(a,ref b);

    Debug.Log(a);
    Debug.Log(b);
}

// 将变量 b 用 ref 修饰为引用参数
public void TestMethod(int a, ref int b)
{
    a -= 1;
    b -= 1;
}
```

最终输出结果为：0 和 1。

#### 修饰返回值
如果 `ref` 修饰一个函数的返回值，那么代表返回的结果是一个引用。

先看一个简单的例子：

```
// 这是定义在类的一个 int 型数组
int[] numbers = new int[] { 1, 2, 3 };

private void Start()
{
    // 通过 TestMethod 方法取出下标为 0 的元素的值，保存在 temp 变量
    int temp = TestMethod(0);

    // 打印出来，毫无疑问是：1
    Debug.Log(temp);

    // 把 temp 变量乘以 10
    temp *= 10;

    // 毫无疑问现在是：10
    Debug.Log(temp);

    // 那这个呢？原来的数组第一个元素也翻了 10 倍吗？并没有！
    Debug.Log(numbers[0]);
}

public int TestMethod(int index)
{
    return numbers[index];
}
```

`numbers` 是一个 int 型的数组，`TestMethod` 方法返回其中一个索引的数据，然后用一个 `temp` 变量保存下表为 0 的值，再把这个值乘以 10，最终 `temp` 变量肯定是翻了 10 倍，但是数组下标为 0 的值不会改变。

如果要想改变数组下标为 0  的值，必须直接引用数组变量进行操作，用一个临时变量保存是不行的，像下面这样：

```
// 这是定义在类的一个 int 型数组
int[] numbers = new int[] { 1, 2, 3 };

private void Start()
{
    // 正确答案：要直接对数组下标进行操作才可以改变值
    numbers[0] *= 10;

    // 错误答案：用一个临时变量来保存是不行的！
    int temp = numbers[0];
    temp *= 10;

    Debug.Log(numbers[0]);
    Debug.Log(temp);
}
```

上面我们已经知道了有「形式参数」和「引用参数」两种，那么同理，也存在「引用值」的概念。

当 `ref` 修饰的是函数的返回值，它就是一个引用返回值：

```
int[] numbers = new int[] { 1, 2, 3 };

private void Start()
{
    // 用 temp 保存返回值的引用
    ref int temp = ref TestMethod(0);

    // 这个时候它还是：1
    Debug.Log(temp);

    // 乘以10，因为保存的是引用值，等同于numbers[0]
    temp *= 10;

    // 两个都输出：10
    Debug.Log(temp);
    Debug.Log(numbers[0]);
}

public ref int TestMethod(int index)
{
    // 返回的是该数组某个下标的引用，意味着可以被改变
    return ref numbers[index];
}
```

输出结果分别是：1、10、10。
其实就是与「引用参数」相同概念的「引用返回值」而已。

### out
“函数只能有一个返回值！”（Go 语言除外）这恐怕是程序员入门一定会学到的知识点。

那么能不能在一个函数里，返回两个值？
严格意义来说 C# 并不能返回两个值，但是可以用 `out` 方法来实现从一个函数获取多个值。

示例：

```
private void Start()
{
    TestMethod(out int a);
    Debug.Log(a);
}

public void TestMethod(out int a)
{
    a = 100;
}
```

上面的 `TestMethod` 方法并没有返回值，通过 `out` 关键字修饰了形式参数后，再对其进行赋值，在 `Start` 里却意外的获取到 `a=100` 的结果，这就是 `out` 关键词的作用，它可以不需要通过函数的返回值而直接拿到变量计算的结果。

假如一个函数内的变量有多个是需要返回的，就可以用 `out` 关键词来进行修饰。

这里存在一个“歧义”，如果对参数多次操作，那么返回的是初始化的值还是计算结果的值呢？
答案是：得到的值是函数体运行结束后最后一次的结果：

```
public void TestMethod(out int a)
{
    a = 100;
    a -= 10;
}
```

比如上面先赋值 100，然后减去 10，最终 a 获取到的结果是 90，即函数整个都运行完成后最终的值。

值得一提的是，在外部调用包含 `out` 关键词参数的方法时，不需要声明变量。

示例：

```
 private void Start()
{
    // 多此一举的方法，不需要特意声明
    int a;
    TestMethod(out a);

    // 简化方法：TestMethod(out int a)

    Debug.Log(a);
}

public void TestMethod(out int a)
{
    a = 100;
}
```

### in
关键字 `in` 比较简单，它修饰的函数参数将视为只读状态，无法在函数内修改：

```
private void Start()
{
    TestMethod(100);
}

public void TestMethod(in int a)
{
    // 报错，in声明的变量无法在函数内修改
    a += 1;
}
```

用 `in` 关键字修饰的 a 变量是只读的，因此无法在函数内部进行修改。
除了修饰函数参数之外，在 `foreach` 结构也可以使用。

这个修饰词主要是为了安全，避免改动了不应该改变的值。