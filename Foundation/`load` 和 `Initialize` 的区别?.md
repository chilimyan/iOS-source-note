# `load` 和 `Initialize` 的区别?
#### load方法
`load` 方法会在加载类的时候就被调用，也就是应用启动的时候，在调用 `main` 函数之前会加载所有的类，也会调用每个类的 `load` 方法。
#### Initialize方法
* 这个方法会在 第一次初始化这个类之前 被调用，我们用它来初始化静态变量。
* `initialize` 方法类似一个懒加载，如果没有使用这个类，那么系统默认不会去调用这个方法，且默认只加载一次
* `initialize` 的调用发生在 `+init` 方法之前，创建子类的时候，子类会去调用父类的 `initialize` 方法


