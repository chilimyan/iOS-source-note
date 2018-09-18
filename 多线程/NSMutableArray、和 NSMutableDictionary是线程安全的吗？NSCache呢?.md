# NSMutableArray、和 NSMutableDictionary是线程安全的吗？NSCache呢?
只有`NSCache`是线程安全的
`NSCache` 优点如下：

1. 系统资源将要耗尽时，它可以自动删减缓存，LRU算法优化缓存。
2. 可以设置最大缓存数量。
3. 可以设置最大占用内存值。
4. NSCache 线程是安全的。

