# yield

先贴一段官方的描述，可以跳过直接看总结的部分。

## 官方描述

在语句中使用 `yield` 上下文关键字时，表明它出现的方法、运算符或 `get` 访问器是迭代器。当您为自定义集合类型实现 `IEnumerable` 和 `IEnumerator` 模式时，使用 yield 定义迭代器无需显式的额外类。

下面给出了`yield`语句使用的两个格式：

```c#
yield return <expression>;
yield break;
```

!!! note "备注"

    `yield return`语句一次只能返回一个元素

    可以使用 `foreach` 语句或 `LINQ` 查询来使用从迭代器方法返回的序列。`foreach` 循环的每次迭代都会调用迭代器方法。 迭代器方法运行到 `yield return` 语句时，会返回一个 `expression`，并保留当前在代码中的位置。 下次调用迭代器函数时，将从该位置重新开始执行。

    当迭代器返回 `System.Collections.Generic.IAsyncEnumerable<T> `时，可以使用 `await foreach` 语句异步使用该序列。 循环的迭代类似于 `foreach` 语句。 不同之处在于，在返回下一个元素的表达式之前，每次迭代都可能因异步操作而挂起。

    可以使用 `yield break` 语句来终止迭代。

### 迭代器方法和 get 访问器

迭代器的声明必须满足以下要求：

 - 返回类型必须为下列类型之一：

    * IAsyncEnumerable<T>
    * IEnumerable<T>
    * IEnumerable
    * IEnumerator<T>
    * IEnumerator

 - 声明不能有任何 in、ref 或 out 参数。

### 异常处理

不能将 `yield return` 语句置于 try-catch 块中。 可将 `yield return` 语句置于 try-finally 语句的 try 块中。

可将 `yield break` 语句置于 try 块或 catch 块中，但不能将其置于 finally 块中。

如果 `foreach` 或 `await foreach` 主体（在迭代器方法之外）引发异常，则将执行迭代器方法中的 finally 块。

## 总结

`yield return` 主要用来返回迭代器中的元素，简单的说，可以写一个复杂的方法来生成一个集合用于迭代，但并不是直接生成完整的一个集合，而是每次取下一个元素时，接着从上一次执行的位置接着执行。

```c#
void Run()
{
    IEnumerable<int> elements = MyIteratorMethod();
    foreach (int element in elements)
    {
        // ...
    }
}

IEnumerable<int> MyIteratorMethod()
{
    yield return 1;

    // ... 做一些别的操作

    yield return 2;
}
```

普通创建迭代器一般会有类似于`hasNext`和`moveNext`这两个方法，然后每次迭代都会完整的去执行这两个方法。相比之下，这里提供的 `yield` 关键字创建迭代器会更灵活一点，不需要模板式的提取下一个元素，甚至可以直接自己写一些常量值进去。

??? summary "后记"

    还在写本篇前，是因为想要理解协程才搜了好多 `yield` 的资料，后来才知道这个只是C#的关键字而已，当时还以为Unity还整了个关键字出来，一直把协程和 `yield` 这个关键字混在一起理解，当时还纳闷为什么这么麻烦，还有什么 `IEnumerator` 这样的中间类型，以为还有好多高端用法。

    现在明白了之后把两个东西分开看，确实是一下就理解了……
    
    有关协程的内容可以参考[这篇](../game/unity/coroutine.md)