# retain、release 的实现机制
1. Retain的实现机制。
    
    ```
    SideTable& table = SideTables()[This];
    
    size_t& refcntStorage = table.refcnts[This];
    
    refcntStorage += SIZE_TABLE_RC_ONE;
    ```

2. Release的实现机制。

```
SideTable& table = SideTables()[This];

size_t& refcntStorage = table.refcnts[This];

refcntStorage -= SIZE_TABLE_RC_ONE;
```

二者的实现机制类似，概括讲就是通过第一层 `hash` 算法，找到 指针变量 所对应的 `sideTable`。然后再通过一层 `hash` 算法，找到存储 引用计数 的 `size_t`，然后对其进行增减操作。`retainCount` 不是固定的 1，`SIZE_TABLE_RC_ONE` 是一个宏定义，实际上是一个值为 4 的偏移量。


