# 如何做到 KVO 手动通知？
`显式的调用 didChangeValueForKey:`
如果想实现手动通知，我们需要借助一个额外的方法
`+ (BOOL)automaticallyNotifiesObserversForKey:(NSString *)key`
这个方法默认返回YES,用来标记 Key 指定的属性是否支持 KVO，如果返回值为 NO，则需要我们手动更新。


