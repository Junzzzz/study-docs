# 热更新

~~内容主要收集自互联网，并掺杂个人主观意见，不保熟。~~

最完美的热更新，自然是能够自动分析更新前后的差别，自动生成补丁包。

然而，对编译后生成的汇编、机器指令做差量更新难度很高，或者说几乎不可能实现，所以只能整体替换。

整体替换的代价很高，因此出现了各种方法去减小这个代价。

目前市面上的方案主要还是将核心代码保留，对需要热更新的代码进行注入。

主流似乎仍然使用的是嵌入 `Lua` 的方法（毕竟方案成体系了），部分公司开始使用原生 `C#` 更新方案。

!!! example "IOS打包"

    按程序集打包DLL然后替换并非完全不可以，若是细分下来每个文件尺寸也不会太大，主要还是因为要跨平台支持IOS系统。
    
    Unity通过IL2Cpp将IL转换为Cpp代码再编译成机器指令，使得可以跨平台支持IOS。

    然而AppStore上发布应用需要审核，不能做到及时更新，因此也只能通过特殊方法来达成 `热更新` 的效果。

## Lua

主要为C#环境提供Lua脚本执行能力，使Lua代码可以方便的和C#相互调用。

### [xLua](https://github.com/Tencent/xLua)

<figure markdown>
  ![xLua](https://github.com/Tencent/xLua/blob/master/Assets/XLua/Doc/xLua.png?raw=true)
  <figcaption>长得有点像 Edge</figcaption>
</figure>


xLua在功能、性能、易用性都有不少突破，这几方面分别最具代表性的是：

 - 可以运行时把C#实现（方法，操作符，属性，事件等等）替换成 Lua 实现；
 - 出色的 GC 优化，自定义 struct，枚举在Lua和C#间传递无C# GC Alloc；
 - 编辑器下无需生成代码，开发更轻量；

### [toLua](https://github.com/topameng/tolua)

toLua 是一个 Unity Lua 静态绑定方案。 它是第一个通过反射分析代码并生成包装类的解决方案。

它是一个 Unity 插件，大大简化了 C# 代码与 Lua 的集成。它可以自动生成从 Lua 访问 Unity 的绑定代码，并将 C# 常量、变量、函数、属性、类和枚举映射到 Lua。

### [sLua](https://github.com/pangweiwei/slua)

主要特性：

 - 静态代码生成，没有反射，没有额外的 GC 分配，非常快
 - 远程调试器
 - 全面支持iOS/iOS64，支持il2cpp
 - 90% 以上的 UnityEngine 接口导出（移除 flash，平台相关接口）
 - 100% UnityEngine.UI界面（需要Unity4.6+）
 - 在没有 Unity3D 的情况下支持 .net framework/mono 中的独立模式
 - 支持UnityEvent/UnityAction通过lua函数进行事件回调
 - 支持通过lua函数委托（包括iOS）
 - 支持 yield call
 - 支持自定义类导出
 - 支持扩展方法
 - 将枚举导出为整数
 - 返回数组作为 lua 表
 - 使用原始luajit，可以替换为lua5.3/lua5.1

## C\#

相较于 Lua 的解决方案，统一使用同一种语言，可以降低开发、维护成本和难度。

### [ILRuntime](https://github.com/Ourpalm/ILRuntime)

ILRuntime项目为基于C#的平台（例如Unity）提供了一个纯C#实现，快速、方便且可靠的IL运行时，使得能够在不支持JIT的硬件环境（如iOS）能够实现代码的热更新。

同市面上的其他热更方案相比，ILRuntime主要有以下优点：

 - 无缝访问C#工程的现成代码，无需额外抽象脚本API
 - 直接使用VS2015进行开发，ILRuntime的解译引擎支持.Net 4.6编译的DLL
 - 执行效率是L#的10-20倍
 - 选择性的CLR绑定使跨域调用更快速，绑定后跨域调用的性能能达到slua的2倍左右（从脚本调用GameObject之类的接口）
 - 支持跨域继承
 - 完整的泛型支持
 - 拥有Visual Studio的调试插件，可以实现真机源码级调试。支持Visual Studio 2015 Update3、Visual Studio 2017、Visual Studio 2019和Visual Studio 2022
 - 支持VS Code源码级调试，支持Mac OSX
 - 最新的2.0版引入的寄存器模式将数学运算性能进行了大幅优化

### [HybridCLR](https://github.com/focus-creative-games/hybridclr) (huatuo)

![HybridCLR](https://github.com/focus-creative-games/hybridclr/blob/main/docs/images/logo.jpg?raw=true)

HybridCLR 扩充了 IL2Cpp 的代码，使它由纯 [AOT](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) Runtime 变成 AOT + Interpreter 混合 Runtime，进而原生支持动态加载 Assembly ，使得基于 IL2Cpp Backend 打包的游戏不仅能在 Android 平台，也能在 IOS、Consoles 等限制了 JIT 的平台上高效地以 **AOT + interpreter** 混合模式执行，从底层彻底支持了热更新。