# 简述 KVO 的实现机制
`KVO`是基于`runtime`机制实现的

1. 利用 `Runtime API` 动态生成这个被监听对象的子类，并且让 `instance` 对象的 `isa` 指向这个全新的子类。

2. 这个子类会重写基类中任何被观察属性的`setter` 方法。子类在被重写的`setter`方法内实现真正的通知机制。

3. 键值观察通知依赖于NSObject 的两个方法: `willChangeValueForKey:` 和 `didChangevlueForKey:`；在一个被观察属性发生改变之前， `willChangeValueForKey:`一定会被调用，这就 会记录旧的值。而当改变发生后，`didChangeValueForKey:`会被调用，继而 `observeValueForKey:ofObject:change:context: `也会被调用。

