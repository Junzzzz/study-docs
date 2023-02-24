# 集合类使用优化

## 场景1

先前看别人代码时发现了一个非常离谱的使用方法，场景是在 `List` 中取出最小的一个值并删除，一直循环到全部取出，看到这里可以简单思考一下解决方案。

??? failure "请勿模仿"

    ```c#
    // 排序
    list.Sort();

    // 然后删除第一项
    list.RemoveAt(0);
    ```

    时间复杂度 O(NlogN)

??? info "解决思路"

    类似于选择排序，用临时变量记录最小值和位置，再删除该项就行，时间复杂度为O(N)

### List.RemoveAt(index) 使用技巧

这里可以先看下官方的源代码[^1]

```c# hl_lines="8"
public void RemoveAt(int index) {
    if ((uint)index >= (uint)_size) {
        ThrowHelper.ThrowArgumentOutOfRangeException();
    }
    Contract.EndContractBlock();
    _size--;
    if (index < _size) {
        Array.Copy(_items, index + 1, _items, index, _size - index);
    }
    _items[_size] = default(T);
    _version++;
}
```

当使用 `List.RemoveAt` 时，从中间项直接删除会多一次 `Array.Copy()`，将数组后半部分拷贝到前面，填充掉空位。

这样操作重复多了会造成很大的浪费，但可以发现，如果删除位置发生在末尾，这个拷贝的操作就可以避免。

所以只需要删除位置与末尾位置进行交换再删除即可。（只适用于当前场景）

```c# title="正确方案"
// 与末尾进行交换
(list[^1], list[minIndex]) = (list[minIndex], list[^1]);

list.RemoveAt(list.Count - 1);
```

---

有新案例再补充...

[^1]: [C# - List 源代码](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/list.cs,3d46113cc199059a)