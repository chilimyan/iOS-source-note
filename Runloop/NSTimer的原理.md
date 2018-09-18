# NSTimer的原理
NSTimer 其实就是 `CFRunLoopTimerRef`，他们之间是 toll-free bridged 的。一个 `NSTimer` 注册到 `RunLoop` 后，`RunLoop` 会为其重复的时间点注册好事件。例如 10:00, 10:10, 10:20 这几个时间点。`RunLoop` 为了节省资源，并不会在非常准确的时间点回调这个`Timer`。`Timer` 有个属性叫做 `Tolerance` (宽容度)，标示了当时间点到后，容许有多少最大误差。

如果某个时间点被错过了，例如执行了一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行。就比如等公交，如果 10:10 时我忙着玩手机错过了那个点的公交，那我只能等 10:20 这一趟了。

`CADisplayLink` 是一个和屏幕刷新率一致的定时器（但实际实现原理更复杂，和 `NSTimer` 并不一样，其内部实际是操作了一个 `Source`）。如果在两次屏幕刷新之间执行了一个长任务，那其中就会有一帧被跳过去（和 NSTimer 相似），造成界面卡顿的感觉。在快速滑动 `TableView` 时，即使一帧的卡顿也会让用户有所察觉。`Facebook` 开源的 `AsyncDisplayLink` 就是为了解决界面卡顿的问题，其内部也用到了 `RunLoop`。

#### NSTimer循环引用的解决方案
我们使用NSTimer时一直都是这样：

``` 
_timer = [NSTimer timerWithTimeInterval:inTimeInterval target:self selector:@selector(executeTimerBlock:) userInfo:block repeats:1.0];
 [[NSRunLoop currentRunLoop] addTimer:_timer forMode:NSRunLoopCommonModes];
 - (void)dealloc{
    if (_timer) {
        [_timer invalidate];
        _timer = nil;
    }
}
```
这样的话`dealloc`其实是不会执行的，因为`NSTimer`内部会对`self`进行强引用。这样`self`和`NSTimer`就形成了一个循环引用。所以`dealloc`是永远不会执行的。
解决方案就是写一个NSTimer的分类。把执行事件写在一个block里面target指向NSTimer自己。例如：

```
+ (NSTimer *)cl_scheduledTimerWithTimeInterval:(NSTimeInterval)inTimeInterval block:(void (^)())inBlock repeats:(BOOL)inRepeats
{
    void (^block)() = [inBlock copy];
    NSTimer * timer = [self scheduledTimerWithTimeInterval:inTimeInterval target:self selector:@selector(__executeTimerBlock:) userInfo:block repeats:inRepeats];
    return timer;
}
```
这样`block`只是当成一个参数进行传递。然后让`__executeTimerBlock`方法去执行这个`block`。这样`NSTimer`就避免强引用`self`了。


