# autoreleasePool 在何时被释放
`AutoreleasePool` 与 `RunLoop` 并没有直接的关系，之所以将两个话题放到一起讨论最主要的原因是因为在iOS应用启动后会注册两个 `Observer` 管理和维护 `AutoreleasePool`。应用程序刚刚启动时打印 `currentRunLoop`可以看到系统默认注册了很多个`Observer`，其中有两个`Observer`的 `callout` 都是 `_ wrapRunLoopWithAutoreleasePoolHandler`，这两个是和自动释放池相关的两个监听。

``` 
<CFRunLoopObserver 0x6080001246a0 [0x101f81df0]>{valid = 
Yes, activities = 0x1, repeats = Yes, order = -2147483647, callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), 
context = <CFArray 0x60800004cae0 [0x101f81df0]>{type = mutable-small, count = 0, values = ()}}
 <CFRunLoopObserver 0x608000124420 [0x101f81df0]>{valid = Yes, activities = 0xa0, repeats = Yes, order = 2147483647, 
callout = _wrapRunLoopWithAutoreleasePoolHandler (0x1020e07ce), context = <CFArray 0x60800004cae0 [0x101f81df0]>
{type = mutable-small, count = 0, values = ()}}

```
第一个 `Observer` 会监听 `RunLoop` 的进入，它会回调`objc_autoreleasePoolPush()` 向当前的 `AutoreleasePoolPage` 增加一个哨兵对象标志创建自动释放池。这个 `Observer` 的 `order` 是 -2147483647 优先级最高，确保发生在所有回调操作之前。
第二个 `Observer` 会监听 `RunLoop` 的进入休眠和即将退出 `RunLoop` 两种状态，在即将进入休眠时会调用 `objc_autoreleasePoolPop()` 和 `objc_autoreleasePoolPush()` 根据情况从最新加入的对象一直往前清理直到遇到哨兵对象。而在即将退出 `RunLoop` 时会调用`objc_autoreleasePoolPop()` 释放自动自动释放池内对象。这个`Observer` 的 `order` 是 2147483647 ，优先级最低，确保发生在所有回调操作之后。

主线程的其他操作通常均在这个 `AutoreleasePool` 之内（main函数中），以尽可能减少内存维护操作(当然你如果需要显式释放【例如循环】时可以自己创建 `AutoreleasePool` 否则一般不需要自己创建)。

