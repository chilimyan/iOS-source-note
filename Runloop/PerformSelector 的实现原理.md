# PerformSelector 的实现原理

当调用 `NSObject` 的 `performSelecter:afterDelay:` 后，实际上其内部会创建一个 `Timer` 并添加到当前线程的 `RunLoop` 中。所以如果当前线程没有 `RunLoop`，则这个方法会失效。

当调用 `performSelector:onThread:` 时，实际上其会创建一个 `Timer` 加到对应的线程去，同样的，如果对应线程没有 `RunLoop` 该方法也会失效。


