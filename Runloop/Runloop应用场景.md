# Runloop应用场景
`Runloop`经典应用场景有以下几种：
1. NSTimer
2. ImageView显示：控制方法在特定的模式下可用
3. PerformSelector
4. 常驻线程：在子线程中开启一个runloop
5. AutoreleasePool 自动释放池
6. UI更新

### 常驻线程
线程创建出来就处于等待状态(有或无任务)，想用它的时候就用它执行任务，不想用的时候就处于等待状态。如：
1.聊天发送语音消息,可能会专门开一个子线程来处理；
2.在后台记录用户的停留时间或某个按钮点击次数,这些用主线程做可能不太方便,可能会开启一个子线程后台默默收集；
开启 Runloop循环，目的是让线程持续存在。
### AutoreleasePool 自动释放池
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
### UI更新
如果打印App启动之后的主线程RunLoop可以发现另外一个`callout`为`_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv `的 `Observer`，这个监听专门负责UI变化后的更新，比如修改了`frame`、调整了UI层级（`UIView/CALayer`）或者手动设置了`setNeedsDisplay`/`setNeedsLayout` 之后就会将这些操作提交到全局容器。而这个`Observer`监听了主线程`RunLoop`的即将进入休眠和退出状态，一旦进入这两种状态则会遍历所有的UI更新并提交进行实际绘制更新。
通常情况下这种方式是完美的，因为除了系统的更新，还可以利用 `setNeedsDisplay` 等方法手动触发下一次 RunLoop 运行的更新。但是如果当前正在执行大量的逻辑运算可能UI的更新就会比较卡，因此facebook 推出了 `AsyncDisplayKit` 来解决这个问题。`AsyncDisplayKit` 其实是将UI排版和绘制运算尽可能放到后台，将UI的最终更新操作放到主线程（这一步也必须在主线程完成），同时提供一套类 `UIView` 或 `CALayer` 的相关属性，尽可能保证开发者的开发习惯。这个过程中 AsyncDisplayKit 在主线程 `RunLoop` 中增加了一个`Observer` 监听即将进入休眠和退出 RunLoop 两种状态,收到回调时遍历队列中的待处理任务一一执行。
### UIImageView 延迟加载图片
利用`PerformSelector`设置当前线程的`RunLoop`的运行模式
`kCFRunLoopDefaultMode`：App的默认运行模式，通常主线程是在这个运行模式下运行
`UITrackingRunLoopMode`：跟踪用户交互事件（用于 `ScrollView` 追踪触摸滑动，保证界面滑动时不受其他Mode影响）
然后 我们滑动`UITableView`时候 `RunLoop`的运行模式就会变为`UITrackingRunLoopMode`   
所以我们把给`ImageView`加载图片的方法用`PerformSelector`设置当前线程的`RunLoop`的运行模式`kCFRunLoopDefaultMode`  这样滑动时候就不会执行加载图片的方法了
也就能避免因为加载图片导致`UITableView`滚动时卡顿的问题
代码
`[cell performSelector:@selector(setImage) withObject:nil afterDelay:0 inModes:@[NSDefaultRunLoopMode]];  `
### UITableView 与 NSTimer 冲突
由于 `UItabelView` 在滑动的时候，会从当前的 `RunLoop` 默认的模式 `kCFRunLoopDefaultMode (NSDefaultRunLoopMode) `自动切换到 `UITrackingRunLoopMode`界面追踪模式。这个时候，处于 `NSDefaultRunLoopMode` 里面的 `NSTimer` 由于切换了模式造成计时器无法继续运行。
解决：
1、更改RunLoop运行Mode（NSRunLoopCommonModes）
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
2、将NSTimer放到新的线程中

```
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(newThread) object:nil];
    [thread start];
- (void)newThread{
    @autoreleasepool{
        //在当前Run Loop中添加timer，模式是默认的NSDefaultRunLoopMode
        timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(incrementCounter:) userInfo: nil repeats:YES];
        //开始执行新线程的Run Loop，如果不启动run loop，timer的事件是不会响应的
        [[NSRunLoop currentRunLoop] run];
    }  
}

```
### NSTimer
`NSTimer`定时器的触发正是基于`RunLoop`运行的，所以使用`NSTimer`之前必须注册到`RunLoop`，但是`RunLoop`为了节省资源并不会在非常准确的时间点调用定时器，如果一个任务执行时间较长，那么当错过一个时间点后只能等到下一个时间点执行，并不会延后执行（`NSTimer`提供了一个`tolerance`属性用于设置宽容度，如果确实想要使用`NSTimer`并且希望尽可能的准确，则可以设置此属性）。
`NSTimer`的创建通常有两种方式，尽管都是类方法，一种是timerWithXXX，另一种scheduedTimerWithXXX。

```
 + (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;
    + (NSTimer *)timerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo
    + (NSTimer *)timerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block ;
    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti invocation:(NSInvocation *)invocation repeats:(BOOL)yesOrNo;
    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)interval repeats:(BOOL)repeats block:(void (^)(NSTimer *timer))block ;
    + (NSTimer *)scheduledTimerWithTimeInterval:(NSTimeInterval)ti target:(id)aTarget selector:(SEL)aSelector userInfo:(nullable id)userInfo repeats:(BOOL)yesOrNo
```
二者最大的区别就是后者除了创建一个定时器外会自动以`NSDefaultRunLoopModeMode`添加到当前线程`RunLoop`中，不添加到`RunLoop`中的`NSTimer`是无法正常工作的。例如下面的代码中如果timer2不加入到`RunLoop`中是无法正常工作的。同时注意如果滚动`UIScrollView（UITableView、UICollectionview是类似的`）二者是无法正常工作的，但是如果将`NSDefaultRunLoopMode`改为`NSRunLoopCommonModes`则可以正常工作，这也解释了前面介绍的Mode内容。
`NSTimer`不是一种实时机制，官方文档明确说明在一个循环中如果`RunLoop`没有被识别（这个时间大概在50-100ms）或者说当前`RunLoop`在执行一个长的call out（例如执行某个循环操作）则`NSTimer`可能就会存在误差，`RunLoop`在下一次循环中继续检查并根据情况确定是否执行（NSTimer的执行时间总是固定在一定的时间间隔，例如1:00:00、1:00:01、1:00:02、1:00:05则跳过了第4、5次运行循环）。

`NSTimer`会对`Target`进行强引用直到任务结束或exit之后才会释放

`CADisplayLink`是一个执行频率（fps）和屏幕刷新相同（可以修改`preferredFramesPerSecond`改变刷新频率）的定时器，它也需要加入到`RunLoop`才能执行。与`NSTimer`类似，`CADisplayLink`同样是基于`CFRunloopTimerRef`实现，底层使用`mk_timer`（可以比较加入到`RunLoop`前后`RunLoop`中timer的变化）。和NSTimer相比它精度更高（尽管`NSTimer`也可以修改精度），不过和`NStimer`类似的是如果遇到大任务它仍然存在丢帧现象。通常情况下`CADisaplayLink`用于构建帧动画，看起来相对更加流畅，而`NSTimer`则有更广泛的用处。

### GCD和RunLoop的关系
在`RunLoop`的源代码中可以看到用到了GCD的相关内容，但是`RunLoop`本身和GCD并没有直接的关系。当调用了`dispatch_async(dispatch_get_main_queue(), <#^(void)block#>)`时`libDispatch`会向主线程`RunLoop`发送消息唤醒`RunLoop`，`RunLoop`从消息中获取`block`，并且在`__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__`回调里执行这个block。不过这个操作仅限于主线程，其他线程`dispatch`操作是全部由`libDispatch`驱动的。

### NSURLConnection
一旦启动NSURLConnection以后就会不断调用delegate方法接收数据，这样一个连续的的动作正是基于RunLoop来运行。

