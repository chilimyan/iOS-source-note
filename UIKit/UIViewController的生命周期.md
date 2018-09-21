# UIViewController的生命周期

* `[ViewController initWithCoder:]`或`[ViewController initWithNibName:Bundle]`:首先从归档文件中加载`UIViewController`对象。即使是纯代码，也会把`nil`作为参数传给后者。
* `[UIView awakeFromNib]:` 作为第一个方法的助手，方法处理一些额外的设置。
* `[ViewController loadView]:` 创建或加载一个view并把它赋值给UIViewController的view属性。
* `-[ViewController viewDidLoad]:`此时整个视图层次(view hierarchy)已经放到内存中，可以移除一些视图，修改约束，加载数据等。
* `[ViewController viewWillAppear:]:` 视图加载完成，并即将显示在屏幕上。还没设置动画，可以改变当前屏幕方向或状态栏的风格等。
* `[ViewController viewWillLayoutSubviews]：`即将开始子视图位置布局
* [ViewController viewDidLayoutSubviews]：用于通知视图的位置布局已经完成
* [ViewController viewDidAppear:]：视图已经展示在屏幕上，可以对视图做一些关于展示效果方面的修改。
* [ViewController viewWillDisappear:]：视图即将消失
* [ViewController viewDidDisappear:]：视图已经消失

![](https://upload-images.jianshu.io/upload_images/1840444-200d08b14ce6480b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

详细版：

1. `initWithNibName:bundle:`初始化UIViewController，执行关键数据初始化操作，非StoryBoard创建UIViewController都会调用这个方法。
2. 注意: 不要在这里做View相关操作，View在loadView方法中才初始化。

3. `initWithCoder:`如果使用StoryBoard进行视图管理，程序不会直接初始化一个UIViewController，StoryBoard会自动初始化或在segue被触发时自动初始化，因此方法initWithNibName:bundle不会被调用，但是initWithCoder会被调用。

4. `awakeFromNib:`当awakeFromNib方法被调用时，所有视图的outlet和action已经连接，但还没有被确定，这个方法可以算作适合视图控制器的实例化配合一起使用的，因为有些需要根据用户洗好来进行设置的内容，无法存在storyBoard或xib中，所以可以在awakeFromNib方法中被加载进来。

5. `loadView:`当执行到loadView方法时，如果视图控制器是通过nib创建，那么视图控制器已经从nib文件中被解档并创建好了，接下来任务就是对view进行初始化。
6. loadView方法在UIViewController对象的view被访问且为空的时候调用。这是它与awakeFromNib方法的一个区别。
7. 假设我们在处理内存警告时释放view属性:self.view = nil。因此loadView方法在视图控制器的生命周期内可能被调用多次。
8. loadView方法不应该直接被调用，而是由系统调用。它会加载或创建一个view并把它赋值给UIViewController的view属性。
9. 在创建view的过程中，首先会根据nibName去找对应的nib文件然后加载。如果nibName为空或找不到对应的nib文件，则会创建一个空视图(这种情况一般是纯代码)
1. 注意:在重写loadView方法的时候，不要调用父类的方法。

2. `viewDidLoad:`当loadView将view载入内存中，会进一步调用viewDidLoad方法来进行进一步设置。此时，视图层次已经放到内存中，通常，我们对于各种初始化数据的载入，初始设定、修改约束、移除视图等很多操作都可以这个方法中实现。
3. 视图层次(view hierachy):因为每个视图都有自己的子视图，这个视图层次其实也可以理解为一颗树状的数据结构。而树的根节点，也就是根视图(root view),在UIViewController中以view属性。它可以看做是其他所有子视图的容器，也就是根节点。

4. `viewWillAppear:`系统在载入所有的数据后，将会在屏幕上显示视图，这时会先调用这个方法，通常我们会在这个方法对即将显示的视图做进一步的设置。比如，设置设备不同方向时该如何显示；设置状态栏方向、设置视图显示样式等。
5. 另一方面，当APP有多个视图时，上下级视图切换是也会调用这个方法，如果在调入视图时，需要对数据做更新，就只能在这个方法内实现。

6. `viewWillLayoutSubviews:`view 即将布局其Subviews。 比如view的bounds改变了(例如:状态栏从不显示到显示,视图方向变化)，要调整Subviews的位置，在调整之前要做的工作可以放在该方法中实现。

7. `viewDidLayoutSubviews:`view已经布局其Subviews，这里可以放置调整完成之后需要做的工作。

8. `viewDidAppear:`在view被添加到视图层级中以及多视图，上下级视图切换时调用这个方法，在这里可以对正在显示的视图做进一步的设置。

9. `viewWillDisappear:`在视图切换时，当前视图在即将被移除、或被覆盖是，会调用该方法，此时还没有调用removeFromSuperview。

1. `viewDidDisappear:`view已经消失或被覆盖，此时已经调用removeFromSuperView

2. `dealloc:`视图被销毁，此次需要对你在init和viewDidLoad中创建的对象进行释放

3. `didReceiveMemoryWarning:`在内存足够的情况下，app的视图通常会一直保存在内存中，但是如果内存不够，一些没有正在显示的viewController就会收到内存不足的警告，然后就会释放自己拥有的视图，以达到释放内存的目的。但是系统只会释放内存，并不会释放对象的所有权，所以通常我们需要在这里将不需要显示在内存中保留的对象释放它的所有权，将其指针置nil。

