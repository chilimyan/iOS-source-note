# UIView的布局方法和重绘
### 布局
#### layoutSubViews
UIView在这个方法中布局子view。当我们需要改变子视图的样式时需要重写这个方法。

在以下情况下系统会自动调用这个方法：

1. `init`初始化不会触发`layoutSubviews`，但是使用`initWithFrame`进行初始化时，且`frame`不为空时会自动调用此方法。
2. `addSubview`会触发`layoutSubviews`方法
3. 设置`view`的`Frame`，且`Frame`的值前后发生变化会触发`layoutSubviews`。
4. 滚动一个`UIScrollView`会触发`layoutSubviews`
5. 旋转`Screen`会触发父`UIView`上的`layoutSubviews`。
6. 改变一个`UIView`的大小的时候也会触发父`UIView`上的`layoutSubviews`

#### setNeedLayout
调用这个方法的时候表示设置了需要布局子视图的标记，但是不会立马调用layoutSubviews，而是异步调用，如果需要马上实现重新布局，那么应该调用下面这个layoutIfNeeds。

#### layoutIfNeeds

这个方法在设置流需要布局标记后，会自动调用layoutSubviews。一般会和setNeedLayout一起用。

### 重绘
#### drawRect
调用这个方法会重绘视图，包括子视图，当需要重绘添加其他东西或者自定义绘画，则需要重写这个方法。
这个方法一般是系统自动调用。不能手动调用。以下情况会调用`drawRect`方法：

1. 如果在`UIView`初始化时没有设置rect大小，将直接导致`drawRect`不被自动调用。`drawRect` 掉用是在`Controller->loadView, Controller->viewDidLoad` 两方法之后掉用的.所以不用担心在 控制器中,这些View的`drawRect`就开始画了.这样可以在控制器中设置一些值给View(如果这些View draw的时候需要用到某些变量 值)。
2. 该方法在调用`sizeToFit`后被调用，所以可以先调用`sizeToFit`计算出size。然后系统自动调用`drawRect`:方法。
3. 通过设置`contentMode`属性值为`UIViewContentModeRedraw`。那么将在每次设置或更改frame的时候自动调用`drawRect:`。

drawRect方法使用应注意的点：

1.  若使用`UIView`绘图，只能在`drawRect：`方法中获取相应的`contextRef`并绘图。如果在其他方法中获取将获取到一个`invalidate` 的ref并且不能用于画图。`drawRect：`方法不能手动显示调用，必须通过调用`setNeedsDisplay` 或 者 `setNeedsDisplayInRect`，让系统自动调该方法。
2. 若使用`calayer`绘图，只能在`drawInContext: `中（类似鱼drawRect）绘制，或者在`delegate`中的相应方法绘制。同样也是调用`setNeedDisplay`等间接调用以上方法
3. 若要实时画图，不能使用`gestureRecognizer`，只能使用`touchbegan`等方法来掉用`setNeedsDisplay`实时刷新屏幕

#### setNeedsDisplay

标记需要重绘，异步调用`drawRect`方法，在下一个draw周期（1/60s）调用`drawRect`进行重绘

#### setNeedsDisplayInRect
标记在某个Rect范围需要重绘

