# @synthesize 和 @dynamic 分别有什么作用
* `@synthesize`是自动合成` Setter，Getter `方法。
* `@dynamic`是动态添加` Setter，Getter `方法。
经常我们会重写类的set方法和get方法。
比如我们在.h文件中声明一个这样的属性
```
@property (nonatomic, copy) NSString *titleStr;
```
我们在.m里面重写它的set方法和get方法。发现当我们只重写一个set方法或者get方法时是没问题的。当我们同时重写set方法和get方法时编译器就报错了。
主要是因为当你同时重写了get和set方法之后`@property`就不会起作用了。

用`@property`声明的成员变量，相当于自动生成了`setter`和`getter`方法，重写set和get之后与`@property`声明的不是一个属性了，需要重新在.m中使用
```
@synthesize  titleStr＝_titleStr
```

