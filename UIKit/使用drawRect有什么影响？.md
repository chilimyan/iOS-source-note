# 使用drawRect有什么影响？
* **缺点：**它处理touch事件时每次按钮被点击后，都会用`setNeddsDisplay`进行强制重绘；而且不止一次，每次单点事件触发两次执行。这样的话从性能的角度来说，对CPU和内存来说都是欠佳的。特别是如果在我们的界面上有多个这样的`UIButton`实例，那就会很糟糕了

* 当你调用 `setNeedsDisplay` 方法时, `UIKit` 将会把当前图层标记为dirty,但还是会显示原来的内容,直到下一次的视图渲染周期,才会将标记为 dirty 的图层重新建立`Core Graphics`上下文,然后将内存中的数据恢复出来, 再使用 `CGContextRef` 进行绘制


