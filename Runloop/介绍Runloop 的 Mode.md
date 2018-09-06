# 介绍Runloop 的 Mode
一共有5个 Mode：
**kCFRunLoopDefaultMode (NSDefaultRunLoopMode)**: App的`默认Mode`，通常主线程是在这个`Mode`下运行。
**UITrackingRunLoopMode**: 界面跟踪 `Mode`，用于 `ScrollView` 追踪触摸滑动，保证界面滑动时不受其他 `Mode` 影响。
**UIInitializationRunLoopMode**: 在刚启动 `App` 时第进入的第一个 `Mode`，启动完成后就不再使用。
**GSEventReceiveRunLoopMode**: 接受系统事件的内部 `Mode`，通常用不到。
**kCFRunLoopCommonModes (NSRunLoopCommonModes)**: 这个并不是某种具体的 `Mode`, 可以说是一个占位用的`Mode`（一种模式组合）。

`Mode` 里面有一个或者多个 `Source、timer、 Observer`。

* `Source`事件源，分 0 和 1；本质就是函数回调，0是用户触发的事件，1是系统内部的事件
* `timer` 就是计时器
* `Observer` 观察者

