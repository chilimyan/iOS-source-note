# 访问 __weak 修饰的变量，是否已经被注册在了 @autoreleasePool 中？为什么？
因为__weak修饰符表示弱引用，不能持有对象实例。比如这句代码`id __weak obj = [[NSObject alloc] init];`
`obj`会随时销毁，所以要把对象注册到自动释放池中。这样才能安全的访问`obj`。所以`__weak` 修饰的变量肯定是已经注册到@autoreleasePool中。

