# UITableView的优化
[iOS保持界面流畅的技巧-ibireme大神](https://blog.ibireme.com/2015/11/12/smooth_user_interfaces_for_ios/)

本质上是降低 `CPU`、`GPU` 的工作，从这两个大的方面去提升性能。

* CPU：对象的创建和销毁、对象属性的调整、布局计算、文本的计算和排版、图片的格式转换和解码、图像的绘制
* GPU：纹理的渲染

#### CPU层面的卡顿优化
* 尽量提前计算好布局，在有需要时一次性调整对应的属性，不要多次修改属性
* 尽量把耗时的操作放到子线程比如：文本处理（尺寸计算、绘制）、图片处理（解码、绘制）
* 我们滑动`UITableView`时候 `RunLoop`的运行模式就会变为`UITrackingRunLoopMode`   所以我们把给`ImageView`加载图片的方法用`PerformSelector`设置当前线程的`RunLoop`的运行模式`kCFRunLoopDefaultMode`  这样滑动时候就不会执行加载图片的方法了，也就能避免因为加载图片导致`UITableView`滚动时卡顿的问题，代码`[cell performSelector:@selector(setImage) withObject:nil afterDelay:0 inModes:@[NSDefaultRunLoopMode]];  `
* 尽量用轻量级的对象，比如用不到事件处理的地方，可以考虑使用 `CALayer` 取代 `UIView`
* 不要频繁地调用 `UIView` 的相关属性，比如 `frame`、`bounds`、`transform` 等属性，尽量减少不必要的修改。
* `Autolayout` 会比直接设置 `frame` 消耗更多的 `CPU` 资源
* 图片的 `size` 最好刚好跟 `UIImageView` 的 `size` 保持一致
* 控制一下线程的最大并发数量

#### GPU层面的卡顿优化
* GPU能处理的最大纹理尺寸是 `4096x4096`，一旦超过这个尺寸，就会占用 `CPU` 资源进行处理，所以纹理尽量不要超过这个尺寸
* 尽量减少视图数量和层次
* 减少透明的视图（`alpha<1`），不透明的就设置 `opaque` 为 `YES`
* 尽量避免出现离屏渲染`CALayer` 的 `Border`、`corner`、`shadow`、`mask` 等技术，这些都会触发离屏渲染
* 尽量避免短时间内大量图片的显示，尽可能将多张图片合成一张进行显示

