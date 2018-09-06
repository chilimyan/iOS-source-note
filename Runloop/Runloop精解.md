# Runloop精解
### Runloop的概念
`CFRunloopRef` 是纯C的函数，而 `NSRunloop` 仅仅是 `CFRunloopRef` 的一层OC封装，并未提供额外的其他功能，因此要了解 `RunLoop` 内部结构，需要多研究 `CFRunLoopRef API（Core Foundation \ 更底层）`。`CFRunloopRef` 其实就是 `__CFRunloop` 这个结构体指针（按照OC的思路我们可以将RunLoop看成一个对象），这个对象的运行才是我们通常意义上说的运行循环，核心方法是 `__CFRunloopRun() `

### Runloop 作用
1. 保持程序的持续运行（如：程序一启动就会开启一个主线程（中的 `runloop` 是自动创建并运行），`runloop` 保证主线程不会被销毁，也就保证了程序的持续运行）。
2. 处理App中的各种事件（如：`touches` 触摸事件、`NSTimer` 定时器事件、`Selector`事件（选择器 `performSelector`））。
3. 节省CPU资源，提高程序性能（有事情就做事情，没事情就休息 (其资源释放)）。
4. 负责渲染屏幕上的所有UI。

#### CFRunLoop.c源码

```
#【用DefaultMode启动，具体实现查看 CFRunLoopRunSpecific Line2704】
#【RunLoop的主函数，是一个死循环 dowhile】
void CFRunLoopRun(void) {   /* DOES CALLOUT */
    int32_t result;
    do {
        /*
         参数一：CFRunLoopRunSpecific   具体处理runloop的运行情况
         参数二：CFRunLoopGetCurrent()  当前runloop对象
         参数三：kCFRunLoopDefaultMode  runloop的运行模式的名称
         参数四：1.0e10                 runloop默认的运行时间，即超时为10的九次方
         参数五：returnAfterSourceHandled 回调处理
         */
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
        
        //【判断】：如果runloop没有停止 且 没有结束则继续循环，相反侧退出。
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}
```
简单说`RunLoop` 其实内部就是`do-while`循环，在这个循环内部不断地处理各种任务（`比如Source、Timer、Observer`），
通过判断`result`的值实现的。所以 可以看成是一个死循环。
如果没有`RunLoop`，`UIApplicationMain` 函数执行完毕之后将直接返回，就是说程序一启动然后就结束；
### Runloop 开启&退出
#### Runloop 的开启
我们来验证 `Runloop` 是在那开启的？答案：`UIApplicationMain` 中开启；

```
# int 类型返回值
UIKIT_EXTERN int UIApplicationMain(int argc, char *argv[], NSString * __nullable principalClassName, NSString * __nullable delegateClassName);

int main(int argc, char * argv[]) {
    @autoreleasepool {
        NSLog(@"开始");
        int number = UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
        NSLog(@"结束");
        return number;
    }
}

#【验证结果】：只会打印开始，并不会打印结束。

```
#### Runloop 的退出条件
App退出；线程关闭；设置最大时间到期；
说明在`UIApplicationMain`函数内部开启了一个和主线程相关的`RunLoop` (保证主线程不会被销毁)，导致 `UIApplicationMain` 不会返回，一直在运行中，也就保证了程序的持续运行。

### Runloop和线程关系
1. 每条线程都有唯一的一个与之对应的`RunLoop`对象。
2. 主线程的`RunLoop`已经自动创建，子线程的`RunLoop`需要主动创建。
3. `RunLoop`在第一次获取时创建，在线程结束时销毁。
#### CFRunLoop.c 源码

```
# NOTE: 获得runloop实现 (创建runloop)
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
    if (pthread_equal(t, kNilPthreadT)) {//【主线程相关联的RunLoop创建】,如果为空，默认是主线程
        t = pthread_main_thread_np();
    }
    __CFLock(&loopsLock);
    if (!__CFRunLoops) { // 如果 RunLoop 不存在
        __CFUnlock(&loopsLock);
        // 创建字典
        CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
        // 创建主线程对应的runloop
        CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
        // 使用字典保存（KEY:线程 -- Value:线程对应的runloop）, 以保证一一对应关系。
        CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
        if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
            CFRelease(dict);
        }
        CFRelease(mainLoop);
        __CFLock(&loopsLock);
    }
    
    //【创建与子线程相关联的RunLoop】,从字典中获取 子线程的runloop
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFUnlock(&loopsLock);
    if (!loop) {
        // 如果子线程的runloop不存在,那么就为该线程创建一个对应的runloop
        CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFLock(&loopsLock);
        loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
        // 把当前子线程和对应的runloop保存到字典中
        if (!loop) {
            CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
            loop = newLoop;
        }
        // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFUnlock(&loopsLock);
        CFRelease(newLoop);
    }
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    return loop;
}
```
以上可知`Runloop` 对象是利用字典来进行存储，而且 Key:线程 -- Value:线程对应的 `runloop`。
iOS开发过程中对于开发者而言更多的使用的是`NSRunloop`,它默认提供了三个常用的`run`方法：

```
//开始运行，run方法对应上面CFRunloopRef中的CFRunloop并不会退出，除非调用CFRunLoopStop(),通常如果想要永远不会退出RunLoop才会使用此方法，否则可以使用runUntilDate。
- (void)run;

//到某个时间点运行,则对应CFRunLoopRunInModel(model，limiteDate，true)方法，只执行一次，执行晚就退出；通常用于手动控制RunLoop（例如在while循环中）
- (void)runUntilDate:(NSdate *)limitDate;

//在某个期限前运行，方法其实是CFRunLoopRunInModel(kCFRunLoopDefaultModel,limiteDate,true),执行完并不会退出，继续下一次RunLoop直到timeout
- (void)runMode:(NSRunLoopMode)mode beforeDate:(NSdate)limiteDate;
```
如何创建子线程对应的 `Runloop` ?
【解决】：开一个子线程创建 `runloop` ，不是通过`[alloc init]`方法创建，而是直接通过调用`currentRunLoop` 方法来创建。
【原因】：`currentRunLoop` 本身是懒加载的，当第一次调用`currentRunLoop` 方法获得该子线程对应的 `Runloop` 的时候,它会先去判断(去字典中查找)这个线程的`Runloop` 是否存在,如果不存在就会自己创建并且返回,如果存在直接返回。

### Runloop 获取

``` 
// Foundation框架
    NSRunLoop *mainRunloop = [NSRunLoop mainRunLoop]; // 获得主线程对应的 runloop对象
    NSRunLoop *currentRunloop = [NSRunLoop currentRunLoop]; // 获得当前线程对应的runloop对象
    
    // Core Foundation框架
    CFRunLoopRef maiRunloop = CFRunLoopGetMain(); // 获得主线程对应的 runloop对象
    CFRunLoopRef maiRunloop = CFRunLoopGetCurrent(); // 获得当前线程对应的runloop对象

    // NSRunLoop <--> CFRunLoopRef 相互转化
    NSLog(@"NSRunLoop <--> CFRunloop == %p--%p",CFRunLoopGetMain() , [NSRunLoop mainRunLoop].getCFRunLoop);

#【打印结果】：内存地址相同
0000-00-13 00:30:16.527 MultiThreading[57703:1217113] NSRunLoop <--> CFRunloop == 0x60000016a680--0x60000016a680
```
### Runloop 源码&运行流畅
[源码](https://github.com/opensource-apple/CF/blob/master/CFRunLoop.c#L2021)
```
int32_t __CFRunLoopRun()
{
    // 通知即将进入runloop
    __CFRunLoopDoObservers(KCFRunLoopEntry);
    
    do
    {
        // 通知将要处理timer和source
        __CFRunLoopDoObservers(kCFRunLoopBeforeTimers);
        __CFRunLoopDoObservers(kCFRunLoopBeforeSources);
        
        // 执行被加入的Block（处理非延迟的主线程调用）
        __CFRunLoopDoBlocks();
        // 处理Source0事件
        __CFRunLoopDoSource0();
        
        if (sourceHandledThisLoop) {
            __CFRunLoopDoBlocks();
         }
        // 如果有 Source1 (基于port) 处于 ready 状态，直接处理这个 Source1 然后跳转去处理消息。
        if (__Source0DidDispatchPortLastTime) {
            Boolean hasMsg = __CFRunLoopServiceMachPort();
            if (hasMsg) goto handle_msg;
        }
            
        // 通知 Observers: RunLoop 的线程即将进入休眠(sleep)。
        if (!sourceHandledThisLoop) {
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeWaiting);
        }
            
        // GCD dispatch main queue
        CheckIfExistMessagesInMainDispatchQueue();
        
        // 即将进入休眠
        __CFRunLoopDoObservers(kCFRunLoopBeforeWaiting);
        
        // 等待内核mach_msg事件
        mach_port_t wakeUpPort = SleepAndWaitForWakingUpPorts();
        
        // 等待。。。
        
        // 从等待中醒来
        __CFRunLoopDoObservers(kCFRunLoopAfterWaiting);
        
        // 处理因timer的唤醒
        if (wakeUpPort == timerPort)
            __CFRunLoopDoTimers();
        
        // 处理异步方法唤醒,如dispatch_async
        else if (wakeUpPort == mainDispatchQueuePort)
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__()
            
        // 处理Source1
        else
            __CFRunLoopDoSource1();
        
        // 再次确保是否有同步的方法需要调用
        __CFRunLoopDoBlocks();
        
    } while (!stop && !timeout);
    
    // 通知即将退出runloop
    __CFRunLoopDoObservers(CFRunLoopExit);
}
```
程执行了这个函数 (`__CFRunLoopRun`) 后，就会一直处于这个函数内部 "接受消息->等待->处理" 的循环中，直到这个循环结束函数才返回，当然`Runloop`精华在于在休眠时几乎不会占用系统资源（系统内核负责）。
![](https://upload-images.jianshu.io/upload_images/1840444-9386ec44d8dbc03d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
尽管 `CFRunLoopPerformBlock` 在上图中作为唤醒机制有所体现，但事实上执行 `CFRunLoopPerformBlock` 只是入队，下次 `RunLoop` 运行才会执行，而如果需要立即执行则必须调用 `CFRunLoopWakeUp` 。
### Runloop 相关类
`Core Foundation` 中关于 `RunLoop` 的5个类
1. `CFRunloopRef`【`RunLoop`本身】
2. `CFRunloopModeRef`【`Runloop`的运行模式】
3. `CFRunloopSourceRef`【`Runloop`要处理的事件源】`Source0`:处理的是App内部的事件、App自己负责管理，如按钮点击事件等。`Source1`:由`RunLoop`和内核管理，`Mach Port`驱动，如`CFMachPort`、`CFMessagePort`
4. `CFRunloopTimerRef`【`Timer`事件】
5. `CFRunloopObserverRef`【`Runloop`的观察者（监听者）】
![](https://upload-images.jianshu.io/upload_images/1840444-7776c2c2d66e7922.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由上图可知
一条线程 对应一个 `Runloop`，`Runloop` 总是运行在某种特定的`CFRunLoopModeRef`（运行模式）下。
每个 `Runloop` 都可以包含若干个 `Mode` ，每个 `Mode` 又包含`Source`源 / `Timer`事件 / `Observer`观察者。
在 `Runloop` 中有多个运行模式，每次调用 `RunLoop` 的主函数【`__CFRunloopRun()`】时，只能指定其中一个 `Mode`（称 `CurrentMode`）运行， 如果需要切换 `Mode`，只能是退出 `CurrentMode` 切换到指定的 `Mode` 进入，目的以保证不同 `Mode` 下的` Source / Timer / Observer` 互不影响。
`Runloop` 有效，`mode` 里面 至少 要有一个`timer`(定时器事件) 或者是`source`(源)；

```
struct __CFRunLoop {
        CFRuntimeBase _base;
        pthread_mutex_t _lock;          /* locked for accessing mode list */
        __CFPort _wakeUpPort;           // used for CFRunLoopWakeUp 
        Boolean _unused;
        volatile _per_run_data *_perRunData;              // reset for runs of the run loop
        pthread_t _pthread;
        uint32_t _winthread;
        CFMutableSetRef _commonModes;
        CFMutableSetRef _commonModeItems;
        CFRunLoopModeRef _currentMode;
        CFMutableSetRef _modes;
        struct _block_item *_blocks_head;
        struct _block_item *_blocks_tail;
        CFAbsoluteTime _runTime;
        CFAbsoluteTime _sleepTime;
        CFTypeRef _counterpart;
    };

    struct __CFRunLoopMode {
        CFRuntimeBase _base;
        pthread_mutex_t _lock;  /* must have the run loop locked before locking this */
        CFStringRef _name; // mode名
        Boolean _stopped;
        char _padding[3];
        CFMutableSetRef _sources0; // source0 源
        CFMutableSetRef _sources1; // source1 源
        CFMutableArrayRef _observers; // observer 源
        CFMutableArrayRef _timers; // timer 源
        CFMutableDictionaryRef _portToV1SourceMap;// mach port 到 mode的映射,为了在runloop主逻辑中过滤runloop自己的port消息。
        __CFPortSet _portSet;// 记录了所有当前mode中需要监听的port，作为调用监听消息函数的参数。
        CFIndex _observerMask;
    #if USE_DISPATCH_SOURCE_FOR_TIMERS
        dispatch_source_t _timerSource;
        dispatch_queue_t _queue;
        Boolean _timerFired; // set to true by the source when a timer has fired
        Boolean _dispatchTimerArmed;
    #endif
    #if USE_MK_TIMER_TOO
        mach_port_t _timerPort;// 使用 mk timer， 用到的mach port，和source1类似，都依赖于mach port
        Boolean _mkTimerArmed;
    #endif
    #if DEPLOYMENT_TARGET_WINDOWS
        DWORD _msgQMask;
        void (*_msgPump)(void);
    #endif
        uint64_t _timerSoftDeadline; /* TSR timer触发的理想时间*/
        uint64_t _timerHardDeadline; /* TSR timer触发的实际时间，理想时间加上tolerance（偏差*/
    };
```
### Runloop 相关类（Mode）
`CFRunLoopModeRef` 代表 `RunLoop` 的运行模式；系统默认提供了5个 `Mode` 。
**kCFRunLoopDefaultMode (NSDefaultRunLoopMode)**: App的`默认Mode`，通常主线程是在这个`Mode`下运行。
**UITrackingRunLoopMode**: 界面跟踪 `Mode`，用于 `ScrollView` 追踪触摸滑动，保证界面滑动时不受其他 `Mode` 影响。
**UIInitializationRunLoopMode**: 在刚启动 `App` 时第进入的第一个 `Mode`，启动完成后就不再使用。
**GSEventReceiveRunLoopMode**: 接受系统事件的内部 `Mode`，通常用不到。
**kCFRunLoopCommonModes (NSRunLoopCommonModes)**: 这个并不是某种具体的 `Mode`, 可以说是一个占位用的`Mode`（一种模式组合）。

### Runloop 相关类（Source）
按照函数调用栈的分类 `source0` 和 `source1`
Source0：非基于端口Port的事件；`（用于用户主动触发的事件，如：点击按钮 或点击屏幕）`。
Source1：基于端口Port的事件；`（通过内核和其他线程相互发送消息，与内核相关）`。
补充：Source1 事件在处理时会分发一些操作给 Source0 去处理。

### Runloop 相关类（Timer）
`CFRunLoopTimerRef`是基于时间的触发器。
基本上说的就是`NSTimer`(`CADisplayLink`也是加到`RunLoop`),它受`RunLoop`的`Mode`影响。
而与`NSTimer`相比，`GCD定时器不会受Runloop影响`。
### Runloop 相关类（Observer）
`CFRunloopObserverRef`相当于消息循环中的一个监听器，随时通知外部当前`RunLoop`的运行状态（它包含一个函数指针`_callout_`将当前状态及时告诉观察者）。
给`RunLoop`添加监听者，监听其运行状态

```
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event { 
//创建监听者 /* 
第一个参数 CFAllocatorRef allocator：分配存储空间 CFAllocatorGetDefault()默认分配 
第二个参数 CFOptionFlags activities：要监听的状态 kCFRunLoopAllActivities 监听所有状态 
第三个参数 Boolean repeats：YES:持续监听 NO:不持续 
第四个参数 CFIndex order：优先级，一般填0即可 
第五个参数 ：回调 两个参数observer:监听者 activity:监听的事件 
*/ /* 所有事件 
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
 kCFRunLoopEntry = (1UL << 0), // 即将进入RunLoop 
kCFRunLoopBeforeTimers = (1UL << 1), // 即将处理Timer 
kCFRunLoopBeforeSources = (1UL << 2), // 即将处理Source 
kCFRunLoopBeforeWaiting = (1UL << 5), //即将进入休眠 
kCFRunLoopAfterWaiting = (1UL << 6),// 刚从休眠中唤醒 
kCFRunLoopExit = (1UL << 7),// 即将退出RunLoop 
kCFRunLoopAllActivities = 0x0FFFFFFFU }; 
*/ 
CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(), kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) { 
switch (activity) { 
case kCFRunLoopEntry: 
NSLog(@"RunLoop进入"); 
break; 
case kCFRunLoopBeforeTimers: 
NSLog(@"RunLoop要处理Timers了"); 
break; 
case kCFRunLoopBeforeSources: 
NSLog(@"RunLoop要处理Sources了"); 
break; 
case kCFRunLoopBeforeWaiting: 
NSLog(@"RunLoop要休息了"); 
break; 
case kCFRunLoopAfterWaiting: 
NSLog(@"RunLoop醒来了"); 
break; 
case kCFRunLoopExit: 
NSLog(@"RunLoop退出了"); 
break; 
default: 
break;
 } });
 // 给RunLoop添加监听者
 /*
第一个参数 CFRunLoopRef rl：要监听哪个RunLoop,这里监听的是主线程的RunLoop 
第二个参数 CFRunLoopObserverRef observer 监听者 
第三个参数 CFStringRef mode 要监听RunLoop在哪种运行模式下的状态 
*/ 
CFRunLoopAddObserver(CFRunLoopGetCurrent(), observer, kCFRunLoopDefaultMode); 
/* CF的内存管理（Core Foundation） 凡是带有Create、Copy、Retain等字眼的函数，创建出来的对象，都需要在最后做一次release GCD本来在iOS6.0之前也是需要我们释放的，6.0之后GCD已经纳入到了ARC中，所以我们不需要管了 */ 
CFRelease(observer);
 }
```
### Runloop 休眠
其实对于 `Event Loop` 而言 `RunLoop` 最核心的事情就是保证线程在没有消息时休眠以避免占用系统资源，有消息时能够及时唤醒。 `RunLoop` 的这个机制完全依靠系统内核来完成，具体来说是苹果操作系统核心组件 `Darwin` 中的 Mach 来完成的。
`mach_msg() `的本质是一个调用` mach_msg_trap()`，这相当于一个系统调用，会触发内核状态切换。当程序静止时，`RunLoop`停留在`__CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort, poll ? 0 : TIMEOUT_INFINITY, &voucherState, &voucherCopy)`，而这个函数内部就是调用了`mach_msg` 让程序处于休眠状态。

