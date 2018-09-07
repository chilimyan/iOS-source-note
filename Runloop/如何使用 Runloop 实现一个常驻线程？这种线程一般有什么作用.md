# 如何使用 Runloop 实现一个常驻线程？这种线程一般有什么作用
一般做法是向 `Runloop` 中放一个 `port`。
子线程启动后，启动`runloop`
```
NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
//如果注释了下面这一行，子线程中的任务并不能正常执行
[runLoop addPort:[NSMachPort port] forMode:NSRunLoopCommonModes];
[runLoop run];
```
实际上是添加了一个`source0`事件源


