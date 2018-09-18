# GCD 与 NSOperationQueue 有哪些异同

1. `GCD` 是纯 `C` 语言的 `API` 。`NSOperationQueue` 是基于 `GCD` 的 `OC` 的封装。
2. `GCD` 只支持 `FIFO` 队列，`NSOperationQueue` 可以方便设置执行顺序，设置最大的并发数量。
3. `NSOperationQueue` 可以方便的设置 `operation` 之间的依赖关系，`GCD` 则需要很多代码。
4. `NSOperationQueue` 支持 `KVO`，可以检测 `operation` 是否正在执行（`isExecuted`），是否结束（`isFinished`），是否取消（`isCanceled`）
5. `GCD`的执行速度比`NSOperationQueue`快。
6. 任务之间不太相互依赖使用GCD
7. 任务之间有依赖或要监听任务的执行情况使用NSOperationQueue


