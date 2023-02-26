# Unity 序列化字段置空

对于在 `MonoBehaviour` 中公开的序列化字段置空时，只会对内部字段设置为空

```c#
[Serializable]
public class Test
{
    public string name;
}

public class TestMono : MonoBehaviour
{
    public Test test1;
    [SerializeField] private Test test2;

    public void Test() {
        test1 = null;
        test2 = null;

        // test1 != null
        // test1.name == null
        // test2 同理
    }
}
```

没有深入研究，应该是被Unity引擎接管了，也有可能是在编辑器Inspector中自动检测为空就初始化了。