# 协程

## 前言

当时以为协程是什么类似于多线程一样的麻烦玩意儿，事实上可以直接理解成监听事件，执行过程仍然是发生在主线程上的

## 介绍

协程主要是通过C#提供的 `yield` 关键字来完成的，有关这个关键字的内容可以参考[这篇](../../c-sharp/yield.md)

下面给一张[官网](https://docs.unity3d.com/Manual/ExecutionOrder.html)提供的生命周期图

??? note "脚本生命周期"

    ![脚本生命周期流程图](../../assets/images/monobehaviour_flowchart.svg)

可以看到中间有 `yield WaitForFixedUpdate` 之类的内容，这就是事件完成的位置，同时会继续执行你定义的 `Enumerator` 迭代器，这样就可以执行你在事件发生后想要完成的内容了

## 用法

这里贴一下各种返回值的作用[^1]:

 - `yield return null` 暂停协程等待下一帧继续执行
 - `yield return <int>` 暂停协程等待下一帧继续执行
 - `yield return new WairForSeconds(time)` 等待规定时间后继续执行
 - `yield return StartCoroutine("method")` 开启一个协程（嵌套协程)
 - `yield return GameObject;` 当游戏对象被获取到之后执行
 - `yield return new WaitForFixedUpdate()` 等到下一个固定帧数更新
 - `yield return new WaitForEndOfFrame()` 等到所有相机画面被渲染完毕后更新
 - `yield break` 跳出协程对应方法，其后面的代码不会被执行

再贴一下示例:

```c#
private void Start()
{
    // 推荐
    StartCoroutine(YourCoroutine());
    StartCoroutine(nameof(YourCoroutine));
}

// 使用字符串方式调用不能让 IEnumerator 带泛型（虽然一般也用不到）
private IEnumerator YourCoroutine()
{
    yield return new WaitForSeconds(1);
    
    // ... 
}
```

作用就是每次 `FixedUpdate` 后1秒执行...的部分，这里建议使用上面的方法，相比使用方法名字符串调用方法开销要更低一点，并且可以带更多参数（方法名调用只能带一个参数）。

用方法名的字符串调用的话唯一一点好处大概就是可以用 `StopCoroutine("YourCoroutine")` 这种用字符串停止的方式，否则需要保存协程信息，才能停止协程。

```c#
private void Start()
{
    // 第一种关闭方式
    Coroutine c = StartCoroutine(YourCoroutine());
    StopCoroutine(c);

    // 第二种关闭方式
    StartCoroutine(nameof(YourCoroutine));
    StopCoroutine(nameof(YourCoroutine));
}
```

!!! summary "后记"

    这篇引用的博客帮了我很大的忙，特别是中间这张生命周期图，可以说是让我突然 **悟了** ，说实话我并没有习惯去翻官方文档，但是官方文档真的是个好东西

    更详细的内容可以参考这篇博客，这里的学习笔记只是简单总结一下

[^1]: [@xinzhilinger](https://github.com/xinzhilinger)分享的[Unity 协程(Coroutine)原理与用法详解](https://blog.csdn.net/xinzhilinger/article/details/116240688)