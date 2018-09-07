# PerformSelector:afterDelay:这个方法在子线程中是否起作用？为什么？怎么解决
不起作用，子线程默认没有 `Runloop`，也就没有 `Timer`。
解决的办法是可以使用 `GCD` 来实现：`Dispatch_after`

