# 说一下 Runtime 的方法缓存？存储的形式、数据结构以及查找的过程
`cache_t`增量扩展的哈希表结构。哈希表内部存储的 `bucket_t`。
`bucket_t` 中存储的是 `SEL` 和 `IMP`的键值对。

* 如果是有序方法列表，采用二分查找
* 如果是无序方法列表，直接遍历查找


