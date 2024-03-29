# xLua

## 个人理解

这里简单概括一下自己对 `xLua` 实现的理解。

应用通过替换代码来实现更新的效果，然而原生C#编译后的文件，应用自己不能主动将自身文件进行替换，只能通过安装包的形式 (apk、ipa) 更新应用。

因此大部分方案采用了解释性语言（Lua、JavaScript、Python等）来解决这些问题，这些代码可以不需要运行前编译 (AOT)，而是一边编译一边运行，这称之为即时编译 (JIT)。

直接将脚本文件通过即时编译，再将生成的指令写入内存交由操作系统执行是一种热更新的解决方法，然而 **IOS不支持** 这种方案，应用主动申请写入内存的内容会被标记为 **不可执行** ，只能简单的读写，主要还是出于安全的考虑吧。

既然不能让操作系统去直接执行，那么只能由程序本身去执行了！因此，开发人员们直接在程序里塞了一个脚本解释器。

简单的来说，这就好比你在 `Minecraft` 里用红石电路造了个CPU（不知道大家有没有看过类似的视频），再拿这个CPU来算1+1。

显而易见，这种方案会带来性能损耗，不过游戏性能损耗的大头还是引擎、渲染这些计算密集的部分，游戏逻辑一般不会存在太大的计算压力，有些固定比较损耗性能的算法也可以不写入热更新部分，而是原生代码编译再通过脚本调用。

当然，框架开发人员也通过一些手段尽可能的去保持Native运行，比如下面这段[^1]：

```c#
public class TestXLua
{
    public int Add(int a, int b)
    {
        return a - b;
    }
}
```

通过 `xLua` 会 **生成** 类似于下面的代码：

```c#
public class TestXLua
{
    static Func<object, int, int, int> hotfix_Add = null;
    int Add(int a, int b)
    {
        if (hotfix_Add != null) return hotfix_Add(this, a, b);
        return a - b;
    }
}
```

当客户端发现需要热更新这段函数时，直接往 `hotfix_Add` 里面塞委托，而这个函数也只是多了一次 `null` 的判断，不会带来过重的开销。 

总而言之，这套方案已经非常成熟，大伙儿都用了好多年了，可以说是不错的解决方案。当然到了现在，也慢慢出现更多优秀的解决方法了。

[^1]: [深入理解xLua基于IL代码注入的热更新原理 - iwiniwin](https://www.cnblogs.com/iwiniwin/p/15474919.html)