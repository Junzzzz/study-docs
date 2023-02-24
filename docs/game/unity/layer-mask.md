# LayerMask

## 前言

开始并不了解 `LayerMask` 的用法，看到 `Physics2D` 里面方法参数就是一个 `int` 类型，还以为是第几个图层就填数字几，况且还有 `LayerMask.NameToLayer` 这个方法，看方法名和返回值类型挺像那么回事，就用这个函数的返回值来填参数了。后来因为没有生效才兜兜转转发现并没有理解 `LayerMask` 。

!!! note "官方说明"

    LayerMask is a bitmask. Use LayerMask.GetMask and LayerMask.LayerToName to generate the bitmask. [^1]

## 介绍

其实回到 `Physics2D` 里的方法参数就可以知道那样的理解是很有问题的，因为参数只有一个 `int` 类型，如果需要检测多个图层该怎么办呢？

所以说，这个 `int` 参数是可以表示多个类型的，而实现就是类似于 `Bitmap`。

一个 `int` 类型变量有4字节，也就是32位，从低到高，第几位为1就代表包含第几个图层。

例如：

LayerMask = 0000 0000 0000 0000 0000 0000 0000 1001

表示这个 `LayerMask` 包含第0层以及第3层 `Layer`

## 使用方法

常用的有两种方法 `LayerMask.GetMask` 和 `LayerMask.NameToLayer`，两者并不等价

简单的来说 `LayerMask.GetMask("Default")` 等价于 `1 << LayerMask.NameToLayer("Default")`

### LayerMask.GetMask(string...)

获取的是一个 `Mask` 表示 `Layer` 集合，这个方法可以填写多个图层名

### LayerMask.NameToLayer(string)

通过图层名来获取 `Layer` 在第几层


[^1]: [官方文档 - LayerMask](https://docs.unity3d.com/ScriptReference/LayerMask.html)
