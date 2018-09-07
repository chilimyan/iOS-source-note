# 利用 runloop 解释一下页面的渲染的过程
如果打印App启动之后的主线程RunLoop可以发现另外一个`callout`为`_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv `的 `Observer`，这个监听专门负责UI变化后的更新，比如修改了`frame`、调整了UI层级（`UIView/CALayer`）或者手动设置了`setNeedsDisplay`/`setNeedsLayout` 之后就会将这些操作提交到全局容器。而这个`Observer`监听了主线程`RunLoop`的即将进入休眠和退出状态，一旦进入这两种状态则会遍历所有的UI更新并提交进行实际绘制更新。
通常情况下这种方式是完美的，因为除了系统的更新，还可以利用 `setNeedsDisplay` 等方法手动触发下一次 RunLoop 运行的更新。但是如果当前正在执行大量的逻辑运算可能UI的更新就会比较卡，因此facebook 推出了 `AsyncDisplayKit` 来解决这个问题。`AsyncDisplayKit` 其实是将UI排版和绘制运算尽可能放到后台，将UI的最终更新操作放到主线程（这一步也必须在主线程完成），同时提供一套类 `UIView` 或 `CALayer` 的相关属性，尽可能保证开发者的开发习惯。这个过程中 AsyncDisplayKit 在主线程 `RunLoop` 中增加了一个`Observer` 监听即将进入休眠和退出 RunLoop 两种状态,收到回调时遍历队列中的待处理任务一一执行。

**以下是页面渲染过程：**
当我们调用 `[UIView setNeedsDisplay]` 时，这时会调用当前 `View.layer` 的 `[view.layer setNeedsDisplay]`方法。

这等于给当前的 `layer` 打上了一个脏标记，而此时并没有直接进行绘制工作。而是会到当前的 `Runloop` 即将休眠，也就是 `beforeWaiting` 时才会进行绘制工作。

紧接着会调用` [CALayer display]`，进入到真正绘制的工作。`CALayer` 层会判断自己的 `delegate` 有没有实现异步绘制的代理方法 `displayer:`，这个代理方法是异步绘制的入口，如果没有实现这个方法，那么会继续进行系统绘制的流程，然后绘制结束。
![](https://upload-images.jianshu.io/upload_images/1840444-9f539c7322375c02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`CALayer` 内部会创建一个 `Backing Store`，用来获取图形上下文。接下来会判断这个 `layer` 是否有 `delegate`如果有的话，会调用 `[layer.delegate drawLayer:inContext:]`，并且会返回给我们 `[UIView DrawRect:]` 的回调，让我们在系统绘制的基础之上再做一些事情。如果没有 `delegate`，那么会调用 `[CALayer drawInContext:]`。以上两个分支，最终 `CALayer` 都会将位图提交到`Backing Store`，最后提交给 `GPU`。
![](https://upload-images.jianshu.io/upload_images/1840444-c4464110fb8610bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


