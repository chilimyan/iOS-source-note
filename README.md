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

### 2、多线程

### 3、数据存储

### 4、性能优化

### 5、数据结构与算法

### 6、网络

### 7、音视频处理

### 8、项目架构

### 9、设计模式

### 10、源代码阅读

### 11、蓝牙

### 12、Foundation

### 13、WebView

### 14、Block

[Block精解](https://github.com/chilimyan/iOS-source-note/blob/master/Block/Objective-C中Block精解.md)

### 15、Runtime-[链接]()

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

