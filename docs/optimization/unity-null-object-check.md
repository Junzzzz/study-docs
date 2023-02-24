# Unity 空对象检测

## 前言

在使用 `Rider` 写null检查时 `gameObject == null` 中的等号换了一种颜色显示，点开后才知道Unity引擎重载了 `==` 的方法。

当时没有在意，直到在 `Update()` 方法内写这段语句时出现了提示： `Comparison to 'null' is expensive` 

仔细查了查资料还是发现有些门道的。

## Unity对象检查逻辑

事实上，我们所拥有的 `GameObject` 对象并没有真正存在于我们可以直接交互的对象池中，而是存储在由底层 `C++` 所编写Unity引擎中。

我们所交互的对象可以说仅仅是一个接口、一个包装对象，而这个包装对象由 `C#` 的垃圾回收器接管，在使用 `Object.Destroy(gameObject)` 后，并不会使包装对象立刻被回收。因此在编写逻辑时往往会存在一些误区。

```C# hl_lines="21"
public static bool operator ==(Object x, Object y) => Object.CompareBaseObjects(x, y);

public static bool operator !=(Object x, Object y) => !Object.CompareBaseObjects(x, y);

private static bool CompareBaseObjects(Object lhs, Object rhs)
{
    bool flag1 = (object) lhs == null;
    bool flag2 = (object) rhs == null;
    if (flag2 & flag1)
        return true;
    if (flag2)
        return !Object.IsNativeObjectAlive(lhs);
    return flag1 ? !Object.IsNativeObjectAlive(rhs) : lhs.m_InstanceID == rhs.m_InstanceID;
}

private static bool IsNativeObjectAlive(Object o)
{
    if (o.GetCachedPtr() != IntPtr.Zero)
        return true;
    return !(o is MonoBehaviour) && !(o is ScriptableObject) 
        && Object.DoesObjectWithInstanceIDExist(o.GetInstanceID());
}

private IntPtr GetCachedPtr() => this.m_CachedPtr;
```
从上述反编译的代码可以看出在进行 `==` 检查时，当左右都为 `null` 时，检查结果可以立即得到，但当其中一项存在非空时，则需要进行额外的检查。

最终，检查会落在 `Object.DoesObjectWithInstanceIDExist(id)` 这个方法上。

虽然对于底层生命周期检查的具体内容我并不清楚，但是通过 `int` 类型来查找一个对象是否存活，显然是需要消耗不少性能。

最好的结果也是类似于在 `Dictionary` 这样的数据结构中进行查找，但由于 **哈希碰撞** 的问题，也容易出现较坏的结果，甚至可能会 **扫描全部** 的 `GameObject`。

因此在 **高频率** 调用中，使用 `gameObject == null` 这样进行生命周期检查是 **及其不推荐** 的！

## 优化方案

简单的来说无非是分为两种情况

1. 检查这个对象 **是否被销毁**
2. 检查这个对象 **是否初始化** 对原生对象的引用

### 第一种情况

需要得到这种效果就没有办法避免去调用Unity自定义的重载方法了，但是一般情况下也不会在高频环境中去检查一个对象是否被销毁。

我们碰到的往往是第二种情况。

### 第二种情况

对于规避生命周期的检查办法还是挺多的。比如：

 - C# 9.0 is null / is not null
 - object.ReferenceEquals(gameObject, null)

此外要注意使用

 - 空合并运算符 ??
 - 空条件运算符 ?

```C#
mono = mono ?? Instantiate();

mono?.DoSomething();
```

这两个操作也会绕过生命周期的检查，有时候并不确定需要检查的是销毁还是是否初始化，这样在快速编写代码逻辑时，往往会忽略从而造成不必要的BUG，所以 **并不推荐** 这样书写。

因此这种情况或许使用更清晰的检查（开头两种）会更好，`==` 这种检查则固定为对Unity对象生命周期的检查。

## 总结

!!! quote "引用"

    Turns out two of our own engineers missed that the null check was more expensive than expected, and was the cause of not seeing any speed benefit from the caching. This led to the "well if even we missed it, how many of our users will miss it?", which results in this blogpost :)[^1]

Unity的工程师自己也发现这样自定义的null检查带来的性能损耗远超预期。早在2014年就考虑是否删除这种特性，虽然现在2023年还在就是了。

在自定义类似于 `==` 这样的操作符方法时，还是需要小心，虽然可以让编程语言更接近自然语言，简化代码，但不能忽略在原有编程语言中设计时的初衷。

[^1]: [UnityBlog - Custom == operator, should we keep it?](https://blog.unity.com/technology/custom-operator-should-we-keep-it)