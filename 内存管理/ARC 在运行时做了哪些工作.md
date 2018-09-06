# ARC 在运行时做了哪些工作
主要是指 `weak` 关键字。`weak` 修饰的变量能够在引用计数为0 时被自动设置成 `nil`，显然是有运行时逻辑在工作的。

为了保证向后兼容性，ARC 在运行时检测到类函数中的 `autorelease` 后紧跟其后 `retain`，此时不直接调用对象的 `autorelease` 方法，而是改为调用 `objc_autoreleaseReturnValue`。 `objc_autoreleaseReturnValue` 会检视当前方法返回之后即将要执行的那段代码，若那段代码要在返回对象上执行 `retain` 操作，则设置全局数据结构中的一个标志位，而不执行 `autorelease` 操作，与之相似，如果方法返回了一个自动释放的对象，而调用方法的代码要保留此对象，那么此时不直接执行 `retain` ，而是改为执行 `objc_retainAoutoreleasedReturnValue`函数。此函数要检测刚才提到的标志位，若已经置位，则不执行 `retain` 操作，设置并检测标志位，要比调用 `autorelease` 和`retain`更快。

