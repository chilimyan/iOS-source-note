# 利用 runloop 解释一下页面的渲染的过程
当我们调用 `[UIView setNeedsDisplay]` 时，这时会调用当前 `View.layer` 的 `[view.layer setNeedsDisplay]`方法。

这等于给当前的 `layer` 打上了一个脏标记，而此时并没有直接进行绘制工作。而是会到当前的 `Runloop` 即将休眠，也就是 `beforeWaiting` 时才会进行绘制工作。

紧接着会调用` [CALayer display]`，进入到真正绘制的工作。`CALayer` 层会判断自己的 `delegate` 有没有实现异步绘制的代理方法 `displayer:`，这个代理方法是异步绘制的入口，如果没有实现这个方法，那么会继续进行系统绘制的流程，然后绘制结束。
![](https://upload-images.jianshu.io/upload_images/1840444-9f539c7322375c02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`CALayer` 内部会创建一个 `Backing Store`，用来获取图形上下文。接下来会判断这个 `layer` 是否有 `delegate`如果有的话，会调用 `[layer.delegate drawLayer:inContext:]`，并且会返回给我们 `[UIView DrawRect:]` 的回调，让我们在系统绘制的基础之上再做一些事情。如果没有 `delegate`，那么会调用 `[CALayer drawInContext:]`。以上两个分支，最终 `CALayer` 都会将位图提交到`Backing Store`，最后提交给 `GPU`。
![](https://upload-images.jianshu.io/upload_images/1840444-c4464110fb8610bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


