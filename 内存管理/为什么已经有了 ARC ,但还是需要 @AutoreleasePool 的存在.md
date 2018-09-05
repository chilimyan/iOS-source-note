# 为什么已经有了 ARC ,但还是需要 @AutoreleasePool 的存在
为了避免内存峰值，及时释放不需要的内存空间。
比如：当我们在代码中要临时创建很多个对象的时候，这时候就会短时的内存增加。这个时候我们就要使用@AutoreleasePool了。当使用完临时对象的时候及时释放掉他。比如：

```
NSMutableArray *tempArr = [NSMutableArray array];
    for (int i = 0; i < 99999999; i ++) {
        @autoreleasepool {
        NSString *tempStr = [NSString stringWithFormat:@"%d",i];
        [tempArr addObject:tempStr];
        }
    }
```
MRC时代：

```
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
    NSMutableArray *tempArr = [NSMutableArray array];
    for (int i = 0; i < 99999999; i ++) {
        NSString *tempStr = [NSString stringWithFormat:@"%d",i];
        [tempArr addObject:tempStr];
    }
    [pool drain];
```


