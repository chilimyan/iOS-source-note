# UIView和CALayer
* `UIView`主要负责显示显示视图，`CALayer`主要负责绘图，渲染。有了`CALayer`，`UIView`才能将视图显示出来。
* `UIView`类似于一个`CALayer`的管理器，访问它的跟绘图和坐标的属性，如frame、bounds等，实际上内部都是在访问它所包含的CALayer相关的属性。
* `UIView`有个`layer`属性，可以返回它的`CALayer`实例：`CALayer *layer = myView.layer`。所有从`UIView`继承来的对象都继承了这个属性。这意味着你可以转换、缩放、旋转，甚至可以在`Navigation bars，Tables，Text boxes`等其它的View类上增加动画。每个`UIView`都有一个层，控制着各自的内容最终被显示在屏幕上的方式。
* `UIView`的`layerClass`方法，可以返回主layer所使用的类，`UIView`的子类可以通过重载这个方法，来让`UIView`使用不同的`CALayer`来显示。代码示例：
```
- (class)layerClass {
 return ([CAEAGLLayer class]);
 }
 ```
* `UIView`的`CALayer`类似`UIView`的子View树形结构，也可以向它的layer上添加子layer，来完成某些特殊的表示。即CALayer层是可以嵌套的。示例代码：

 ```
  grayCover = [[CALayer alloc] init];
  grayCover.backgroundColor = [[UIColor blackColor] colorWithAlphaComponent:0.2] CGColor];
  [self.layer addSubLayer:grayCover]; 
  ``` 
  上述代码会在目标View上敷上一层黑色透明薄膜的效果。 
* `UIView`的layer树形在系统内部，被维护着三份copy。`逻辑树：`这里是代码可以操纵的；`动画树：`是一个中间层，系统就在这一层上更改属性，进行各种渲染操作；`显示树：`其内容就是当前正被显示在屏幕上得内容。
* 动画的运作：对`UIView`的`subLayer`（非主Layer）属性进行更改，系统将自动进行动画生成，动画持续时间的缺省值似乎是0.5秒。
* 坐标系统：`CALayer`的坐标系统比`UIView`多了一个`anchorPoint`属性，使用`CGPoint`结构表示，值域是0~1，是个比例值。这个点是各种图形变换的坐标原点，同时会更改layer的`position`的位置，它的缺省值是{0.5,0.5}，即在layer的中央。
某`layer.anchorPoint = CGPointMake(0.f,0.f);`
如果这么设置，只会将layer的左上角被挪到原来的中间位置，必须加上这一句：
某`layer.position = CGPointMake(0.f,0.f);`



