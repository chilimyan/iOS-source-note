# Runloop 和线程的关系
1. 一个线程对应一个 `Runloop`
2. 主线程的默认创建并开启了`Runloop`，子线程的`Runloop`要自己手动创建并开启
3. 子线程的 `Runloop` 以懒加载的形式创建通过调用`currentRunloop`方法创建，如果当前子线程已经有了`runloop`则返回，否则就创建一个返回。然后自己调用`run`方法开启。
4. `Runloop` 存储在一个全局的可变字典里，线程是 `key` ，`Runloop` 是 `value`

