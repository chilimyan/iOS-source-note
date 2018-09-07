# GCD 在 Runloop 中的使用

GCD由子线程返回到主线程,只有在这种情况下才会触发 `RunLoop`。

当调用了`dispatch_async(dispatch_get_main_queue(), <#^(void)block#>)`时`libDispatch`会向主线程`RunLoop`发送消息唤醒`RunLoop`，`RunLoop`从消息中获取`block`，并且在`__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__`回调里执行这个block。不过这个操作仅限于主线程，其他线程`dispatch`操作是全部由`libDispatch`驱动的。

