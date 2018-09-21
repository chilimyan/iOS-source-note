# iOS-source-note
## iOS开发笔记总结
![](https://upload-images.jianshu.io/upload_images/1840444-c7dd5e3ab30ba9e1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个仓库主要是梳理自己做iOS开发以来收集整理的iOS相关的偏原理性的知识点，以及自己的一些总结和体会。供以后复习及面试。

### 1、内存管理
[内存管理精解](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/Objective-C内存管理精解.md)
#### 问题
1. [请讲一下对iOS内存管理的理解](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/iOS对象在内存中的存储方式.md)
2. [使用自动引用计（ARC）数应该遵循的原则](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/使用自动引用计（ARC）数应该遵循的原则%3F.md)
3. [ARC 自动内存管理的原则](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/ARC%20自动内存管理的原则.md)
4. [访问 __weak 修饰的变量，是否已经被注册在了 @autoreleasePool 中？为什么？](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/访问%20__weak%20修饰的变量，是否已经被注册在了%20%40autoreleasePool%20中？为什么？.md)
5. [ARC 的 retainCount 怎么存储的](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/ARC%20的%20retainCount%20怎么存储的？.md)
6. [简要说一下 @autoreleasePool 的数据结构](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/简要说一下%20%40autoreleasePool%20的数据结构.md)
7. [为什么已经有了 ARC ,但还是需要 @AutoreleasePool 的存在](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/为什么已经有了%20ARC%20%2C但还是需要%20%40AutoreleasePool%20的存在.md)
8. [__weak 属性修饰的变量，如何实现在变量没有强引用后自动置为 nil](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/__weak%20属性修饰的变量，如何实现在变量没有强引用后自动置为%20nil.md)
9. [@dynamic 关键字](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/%40dynamic%20关键字.md)
10. [Dealloc 的实现机制](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/Dealloc%20的实现机制.md)
11. [autoReleasePool 什么时候释放](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/autoReleasePool%20什么时候释放.md)
12. [retain、release 的实现机制](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/retain、release%20的实现机制.md)
13. [ARC 在编译时做了哪些工作](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/ARC%20在编译时做了哪些工作.md)
14. [ARC 在运行时做了哪些工作](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/ARC%20在运行时做了哪些工作.md)
15. [MRC和ARC下Set方法的重写](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/MRC和ARC下Set方法的重写.md)
16. [@synthesize关键字的理解](https://github.com/chilimyan/iOS-source-note/blob/master/内存管理/%40synthesize关键字的理解.md)

### 2、多线程
1. [GCD 与 NSOperationQueue 有哪些异同](https://github.com/chilimyan/iOS-source-note/blob/master/多线程/GCD%20与%20NSOperationQueue%20有哪些异同.md)
2. [GCD 并发队列实现机制](https://github.com/chilimyan/iOS-source-note/blob/master/多线程/GCD%20并发队列实现机制.md)
3. [GCD的各种任务和队列](https://github.com/chilimyan/iOS-source-note/blob/master/多线程/GCD的各种任务和队列.md)
4. [NSMutableArray、和 NSMutableDictionary是线程安全的吗？NSCache呢?](https://github.com/chilimyan/iOS-source-note/blob/master/多线程/NSMutableArray、和%20NSMutableDictionary是线程安全的吗？NSCache呢%3F.md)
5. [iOS多线程之GCD的使用](https://github.com/chilimyan/iOS-source-note/blob/master/多线程/iOS多线程之GCD的使用.md)
6. [同步与异步、串行队列与并发队列、并发与并行](https://github.com/chilimyan/iOS-source-note/blob/master/多线程/同步与异步、串行队列与并发队列、并发与并行.md)
7. [多线程的各种锁](https://github.com/chilimyan/iOS-source-note/blob/master/多线程/多线程的各种锁.md)
8. [如何确保线程安全](https://github.com/chilimyan/iOS-source-note/blob/master/多线程/如何确保线程安全.md)
9. [解释一下多线程中的死锁？](https://github.com/chilimyan/iOS-source-note/blob/master/多线程/解释一下多线程中的死锁？.md)

### 3、数据存储

### 4、性能优化
1. [UITableView的优化](https://github.com/chilimyan/iOS-source-note/blob/master/性能优化/UITableView的优化.md)
2. [iOSApp编译过程及签名](https://github.com/chilimyan/iOS-source-note/blob/master/性能优化/iOSApp编译过程及签名.md)
3. [什么是离屏渲染？什么情况下会触发？该如何应对？](https://github.com/chilimyan/iOS-source-note/blob/master/性能优化/什么是离屏渲染？什么情况下会触发？该如何应对？.md)
4. [日常检查内存泄露](https://github.com/chilimyan/iOS-source-note/blob/master/性能优化/日常检查内存泄露.md)
5. [有效降低 APP 包的大小](https://github.com/chilimyan/iOS-source-note/blob/master/性能优化/有效降低%20APP%20包的大小.md)

### 5、数据结构与算法

### 6、网络
1. [HTTP 请求报文 和 响应报文的结构？](https://github.com/chilimyan/iOS-source-note/blob/master/网络/HTTP%20请求报文%20和%20响应报文的结构？.md)
2. [GET与POST的区别](https://github.com/chilimyan/iOS-source-note/blob/master/网络/GET与POST的区别.md)
3. [Http 和 Https 的区别？为什么更加安全？](https://github.com/chilimyan/iOS-source-note/blob/master/网络/Http%20和%20Https%20的区别？为什么更加安全？.md)
4. [OSI 七层模型和TCP/IP五层模型的协议](https://github.com/chilimyan/iOS-source-note/blob/master/网络/OSI%20七层模型和TCP-IP五层模型的协议.md)

### 7、音视频处理

### 8、项目架构

### 9、设计模式

### 10、源代码阅读
* [源代码带中文注释](https://github.com/chilimyan/source-code-read)

### 11、蓝牙

### 12、Foundation
1. [简述 KVO 的实现机制](https://github.com/chilimyan/iOS-source-note/blob/master/Foundation/简述%20KVO%20的实现机制.md)

### 13、WebView

### 14、Block

[Block精解](https://github.com/chilimyan/iOS-source-note/blob/master/Block/Objective-C中Block精解.md)

### 15、Runtime
[Runtime-运行时的概念](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Runtime-运行时的概念.md)

[Runtime-类和对象](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Runtime-类和对象.md)

[Runtime-成员变量和属性](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Runtime-成员变量和属性.md)

[Runtime-分类和协议](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Runtime-分类和协议.md)

[Runtime-Method Swizzling](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Runtime-Method%20Swizzling.md)

[Runtime-消息转发机制原理](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Runtime-消息转发机制原理.md)

#### 问题

1. [Category 和 Extension 有什么区别](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Category%20和%20Extension%20有什么区别.md)
2. [Category 有哪些用途](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Category%20有哪些用途.md)
3. [Category 的实现原理](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Category%20的实现原理.md)
4. [如何给 Category 添加属性？关联对象以什么形式进行存储？](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/如何给%20Category%20添加属性？关联对象以什么形式进行存储？.md)
5. [Obj-C 中的类信息存放在哪里](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Obj-C%20中的类信息存放在哪里.md)
6. [Objective-C 如何实现多重继承](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Objective-C%20如何实现多重继承.md)
7. [一个 NSObject 对象占用多少内存空间](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/一个%20NSObject%20对象占用多少内存空间.md)
8. [在 Obj-C 中为什么叫发消息而不叫函数调用](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/在%20Obj-C%20中为什么叫发消息而不叫函数调用.md)
9. [如何实现动态添加方法和属性](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/如何实现动态添加方法和属性.md)
1. [如何运用 Runtime 字典转模型](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/如何运用%20Runtime%20字典转模型.md)
2. [如何运用 Runtime 进行模型的归解档](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/如何运用%20Runtime%20进行模型的归解档.md)
3. [实例对象、类对象、元类对象的数据结构](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/实例对象、类对象、元类对象的数据结构.md)
4. [对 isa 指针的理解， 对象的isa 指针指向哪里？isa 指针有哪两种类型](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/对%20isa%20指针的理解，%20对象的isa%20指针指向哪里？isa%20指针有哪两种类型.md)
5. [消息解析和转发](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/消息解析和转发.md)
6. [说一下 Runtime 的方法缓存？存储的形式、数据结构以及查找的过程](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/说一下%20Runtime%20的方法缓存？存储的形式、数据结构以及查找的过程.md)
7. [说一下对 class_ro_t 的理解](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/说一下对%20class_ro_t%20的理解.md)
8. [说一下对 class_rw_t 的理解？](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/说一下对%20class_rw_t%20的理解？.md)
9. [Type Encoding类型编码](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/Type%20Encoding类型编码.md)
10. [method swizzling Hook方法时需要注意的地方](https://github.com/chilimyan/iOS-source-note/blob/master/Runtime/method%20swizzling%20Hook方法时需要注意的地方.md)

### 16、Animation-[链接]()

### 17、Runloop
[Runloop精解](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/Runloop精解.md)

[Runloop应用场景](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/Runloop应用场景.md)
#### 问题
1. [GCD 在 Runloop 中的使用](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/GCD%20在%20Runloop%20中的使用.md)
2. [NSTimer的原理](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/NSTimer的原理.md)
3. [PerformSelector 的实现原理](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/PerformSelector%20的实现原理.md)
4. [PerformSelector:afterDelay:这个方法在子线程中是否起作用？为什么？怎么解决](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/PerformSelector:afterDelay:这个方法在子线程中是否起作用？为什么？怎么解决.md)
5. [Runloop 和线程的关系](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/Runloop%20和线程的关系.md)
6. [Runloop的Observer](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/Runloop的Observer.md)
7. [autoreleasePool 在何时被释放](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/autoreleasePool%20在何时被释放.md)
8. [为什么 NSTimer 有时候不好使](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/为什么%20NSTimer%20有时候不好使.md)
9. [介绍Runloop 的 Mode](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/介绍Runloop%20的%20Mode.md)
1. [利用 runloop 解释一下页面的渲染的过程](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/利用%20runloop%20解释一下页面的渲染的过程.md)
2. [如何使用 Runloop 实现一个常驻线程？这种线程一般有什么作用](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/如何使用%20Runloop%20实现一个常驻线程？这种线程一般有什么作用.md)
3. [异步绘制](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/异步绘制.md)
4. [解释一下事件响应的过程](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/解释一下事件响应的过程.md)
5. [解释一下手势识别的过程](https://github.com/chilimyan/iOS-source-note/blob/master/Runloop/解释一下手势识别的过程.md)

### 18、UIKit-[链接]()

