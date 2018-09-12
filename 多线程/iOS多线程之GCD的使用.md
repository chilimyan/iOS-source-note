# iOS多线程之GCD的使用
### 栅栏函数（控制任务的执行顺序）

```
dispatch_barrier_async(queue, ^{
    
        NSLog(@"barrier");
});
```
### 延迟执行（延迟·控制在哪个线程执行）

```
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"---%@",[NSThread currentThread]);
    });
```
### 一次性代码
 
```
static dispatch_once_t onceToken;
   dispatch_once(&onceToken, ^{

       NSLog(@"-----");
   });
```
### 快速迭代（开多个线程并发完成迭代操作）

```
dispatch_apply(subpaths.count, queue, ^(size_t index) {
});
```
### 队列组（同栅栏函数）

```
dispatch_group_t group = dispatch_group_create();
    // 队列组中的任务执行完毕之后，执行该函数
    dispatch_group_notify(dispatch_group_t group,dispatch_queue_t queue,dispatch_block_t block);

    // 进入群组和离开群组
    dispatch_group_enter(group);//执行该函数后，后面异步执行的block会被gruop监听
    dispatch_group_leave(group);//异步block中，所有的任务都执行完毕，最后离开群组
    //注意：dispatch_group_enter|dispatch_group_leave必须成对使用
```

