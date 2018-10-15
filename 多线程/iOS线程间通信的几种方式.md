# iOS线程间通信的几种方式
在1个线程中执行完特定任务后，转到另1个线程继续执行任务。线程间通信的常用方法有：
#### 1、利用perform的方式

```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait;
```
2、NSPort端口
配置`NSMachPort`对象－－－本地线程间通信，通过传递端口对象变量进行端口间通讯
#### 通信机制
A线程（父线程）创建NSMachPort对象，并加入A线程的run loop。当创建B线程（辅助线程）时，将创建的NSMachPort对象传递到主体入口点，B线程（辅助线程）就可以使用相同的端口对象将消息传回A线程（父线程）。NSPort是通过代理模式传送消息。

```
- (void)handleMachMessage:(void *)machMessage
- (void)handlePortMessage:(NSPortMessage *)portMessage
```
[NSPort与NSRunloop的关系是流与消息调度的关系](https://www.cnblogs.com/feng9exe/p/8867475.html)
### 3、GCD

```
dispatch_async(
dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    // 执行耗时的异步操作...
      dispatch_async(dispatch_get_main_queue(), ^{
        // 回到主线程，执行UI刷新操作
        });
});
```

