# id和instanceType的区别
#### 相同点：
instanceType和id都是万能指针，指向对象
#### 不同点：
* `id`在编译的时候不能判断对象的真实类型，`instancetype` 在编译的时候可以判断对象的真实类型。
* `id` 可以用来定义变量，可以作为返回值类型，可以作为形参类型；`instancetype` 只能作为返回值类型。

