# 在什么情况下会触发 KVO?
#### 1.使用了KVC
使用了 KVC，如果有访问器方法，则运行时会在访问器方法中调用 `will/didChangeValueForKey:` 方法； 没用访问器方法，运行时会在 `setValue:forKey` 方法中调用 `will/didChangeValueForKey:`方法。
#### 2.有访问器方法
运行时会重写访问器方法调用 `will/didChangeValueForKey: `方法。 因此，直接调用访问器方法改变属性值时，KVO 也能监听到。
#### 3.直接调用
显式调用 `will/didChangeValueForKey:` 方法。


