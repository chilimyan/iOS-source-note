# @synthesize关键字的理解

`@property`有两个对应的词，一个是 `@synthesize`，一个是 `@dynamic`。如果两个都没写则默认就是`@syntheszie var = _var`；
`@synthesize`的语义是如果你没有手动实现 `setter` 方法和 `getter` 方法，那么编译器会自动为你加上这两个方法。


