# @dynamic 关键字
@dynamic意味着编译期不会帮助我们自动合成`setter` 和 `getter` 方法。我们需要手动实现。
如果`@property`里面是`readonly`的话那么只需要提供`get`方法就行了。

