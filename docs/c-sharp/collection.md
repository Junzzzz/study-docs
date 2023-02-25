# 常用集合类

## List<T\>

`List` 类是一个泛型集合类，事实上就是一个可变数组，非线程安全。

当添加的元素超过数组容量时会进行扩容，重新分配一个新的内存空间，并将原有的数据复制到新的内存里，这时候添加操作的时间复杂度 $O(1)$ 会退化成 $O(n)$。

### 注意点

1. 相较于 `ArrayList` ， `List<T>` 在大多数情况表现更好，同时由于泛型的类型约束，不容易出错。此外，可以避免值类型和引用类型之间的转换（拆、装箱）。
2. `T` 类型需要实现 `IEquatable<T>` 接口，否则像 `Contains` 方法需要将元素装箱然后调用 `Objects.Equals(Object)` 方法才能进行比较。
3. List类是无序集合，在使用 `BinarySearch` 方法前需要调用 `Sort` 方法进行排序或者在二分查找参数里传入 `IComparer<T>` 比较器。

## ArrayList

相当于 `List<Object>` ，是非泛型集合类，历史遗留产物，官方不推荐使用[^1]

## Dictionary

主要用于存放键值对，通过键(Key)来查找值(Value)。速度很快，一般情况下时间复杂度很接近 $O(1)$

### 注意点

前面也写到了是 **一般情况** 查找速度才会很快，当然也会出现极端情况。由于存放数据是通过计算键的Hash散列值来得到存放位置的，难免会出现重复的情况（碰撞）。出现碰撞的概率取决于定义的 `HashCode` 函数以及字典的容量大小。

1. `GetHashCode` 函数产生的整数需要尽可能分散，当然大多数情况不需要考虑这个问题。
2. 在可以得知大致存放数据容量大小的情况下，在实例化 `Dictionary` 对象时最好填写初始容量，避免因为不断添加数据使得存放数组扩容而导致的性能损耗。
3. 存放在其中的键对象属性不能改变（主要是会影响哈希值的属性），否则可能会无法检索到对应的值。


!!! tip "HashCode"

    哈希值代表了一个对象的特征值，两个不同的对象可能会产生相同的特征值。

### 源码浅析

这里只取部分代码作简要分析

[Dictionary 源代码 - Insert](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/dictionary.cs,fd1acf96113fbda9)

```c#
private struct Entry {
   public int hashCode; // Lower 31 bits of hash code, -1 if unused
   public int next;     // Index of next entry, -1 if last
   public TKey key;     // Key of entry
   public TValue value; // Value of entry
}

private int[] buckets;
private Entry[] entries;

private void Insert(TKey key, TValue value, bool add) {

    if( key == null ) {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
    }

    // 初始化数组buckets和entries 会取素数作为size 这里最小为3
    if (buckets == null) Initialize(0);

    // 取Hash值 符号位 置0
    int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;

    // 取余数得到桶位置
    int targetBucket = hashCode % buckets.Length;

#if FEATURE_RANDOMIZED_STRING_HASHING
    // 记录碰撞次数
    int collisionCount = 0;
#endif

    // 检查是否已经存在键值对，有的话直接写入
    for (int i = buckets[targetBucket]; i >= 0; i = entries[i].next) {
        if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) {
            if (add) { 
                ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_AddingDuplicate);
            }
            entries[i].value = value;
            version++;
            return;
        }

#if FEATURE_RANDOMIZED_STRING_HASHING
        // 发生碰撞
        collisionCount++;
#endif
    }
    int index;

    // 是否存在空闲位置（执行删除操作后的空闲位置）
    if (freeCount > 0) {
        // 取空闲位置index
        index = freeList;
        freeList = entries[index].next;
        freeCount--;
    }
    else {
        if (count == entries.Length)
        {
            // 数组满，扩容
            Resize();
            targetBucket = hashCode % buckets.Length;
        }
        index = count;
        count++;
    }

    // 插入数据
    entries[index].hashCode = hashCode;
    // 存在冲突的话会指向冲突位置
    entries[index].next = buckets[targetBucket];
    entries[index].key = key;
    entries[index].value = value;
    // 桶中记录位置
    buckets[targetBucket] = index;
    version++;

    // 碰撞次数过多会触发扩容 HashHelpers.HashCollisionThreshold = 100
#if FEATURE_RANDOMIZED_STRING_HASHING

#if FEATURE_CORECLR
    // In case we hit the collision threshold we'll need to switch to the comparer which is using randomized string hashing
    // in this case will be EqualityComparer<string>.Default.
    // Note, randomized string hashing is turned on by default on coreclr so EqualityComparer<string>.Default will 
    // be using randomized string hashing

    if (collisionCount > HashHelpers.HashCollisionThreshold && comparer == NonRandomizedStringEqualityComparer.Default) 
    {
        comparer = (IEqualityComparer<TKey>) EqualityComparer<string>.Default;
        Resize(entries.Length, true);
    }
#else
    if(collisionCount > HashHelpers.HashCollisionThreshold && HashHelpers.IsWellKnownEqualityComparer(comparer)) 
    {
        comparer = (IEqualityComparer<TKey>) HashHelpers.GetRandomizedEqualityComparer(comparer);
        Resize(entries.Length, true);
    }
#endif // FEATURE_CORECLR

#endif

}
```
#### 个人评价

区别于 `Java` 中的 `HashMap`，解决冲突虽然用了类似拉链法的操作，但这里的存储形式是全数组形式的，并没有使用链表。

相比之下 `Entry` 是 `struct` 值类型，纯数组形式内存空间一次性分配，因此对于内存管理会更友好一些，垃圾回收器回收起来嘎嘎爽。

但是缺点也很明显，这样的结构很容易产生冲突，虽然有冲突扩容机制，但 **个人感觉** 不太适合存储大量数据，查找性能不如Java的HashMap（有对链表进行红黑树转换），观点如下：

1. HashCode与素数取余（需要除法运算）不如HashMap中位运算取余快

    （素数取余虽然可以让数据分布更分散一些，但是Java中使用异或 `^` 叠加高位数据和低位数据来处理哈希值也能得到类似的效果）

2. 冲突时查找慢，Java有红黑树转换，最坏情况也是O(logN)
3. 内存空间浪费，一次性分配内存虽然爽，但是同样也很浪费

!!! tip 

    Java中使用 `HashCode & (Length - 1)` 进行取余，不过只有在 `Length` 是2的倍数情况下才能使用，因此扩容也是按照2的倍数进行扩容的

对于 `Dictionary` 来讲，大部分的使用场景更偏向于查找，因此像Java中对查找做了更多优化（虽然有代价），我认为这样更合理些

## Hashtable

相当于没有泛型的 `Dictionary` 类型，同样官方不推荐使用[^1]

## Stack

堆栈，经典类型。数据就像物品一样堆叠存放，只能在一个堆的顶端进行存取，最后放上来的东西会被优先取出。

这种数据结构有多种实现方式，比如数组或者链表，C#中采用数组来存放数据，因此会存在数组扩容问题，超出预设容量会导致添加数据的时间复杂度增加到 $O(n)$ ，因此使用时建议最好预设初始容量。

## Queue

队列，经典类型。顾名思义，排在队伍前面的优先，即先存放的数据先被取出。

在C#中采用循环数组的方式实现，同样存在扩容问题，注意预设容量。

!!! tip "注意"

    1. 使用 `Stack` 和 `Queue` 时注意选择有泛型的类，即 `System.Collections.Generic` 命名域下的类，因为 **存在同名** 没有泛型的历史遗留产物。
    2. 可以使用 `TrimExcess()` 方法裁剪多余容量，减小内存消耗。（只在小于90%的情况下有效）

[^1]: 官方说明：[DE0006: Non-generic collections shouldn't be used](https://github.com/dotnet/platform-compat/blob/master/docs/DE0006.md)
