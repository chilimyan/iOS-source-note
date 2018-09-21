# loadView方法
* `loadView`是定义`UIViewControoler`的`self.view`的值。如果自己重写了`loadView`方法，则一定要给`self.view`赋值。

* 每当调用完`loadView`方法后系统会自动调用`viewDidLoad()`方法。如果自己重写了`loadView`方法，但是在`loadView`方法里面没有给`self.view`赋值那么系统会反复的调用`loadView`方法和`viewDidLoad()`方法。

* 我们可以在`loadView`方法里面对`View`做一些设置

