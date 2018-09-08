# Runtime-运行时的概念
### 什么是运行时
首先我们要先知道编程语言有静态和动态之分。

#### 静态语言
就是在程序运行前决定了所有的类型判断，类的所有成员、方法在编译阶段就确定好了内存地址。也就意味着所有类对象只能访问属于自己的成员变量和方法，否则编译器直接报错。比较常见的静态的语言如：`java，c++，c`等等
#### 动态语言
类型的判断、类的成员变量、方法的内存地址都是在程序的运行阶段才最终确定，并且还能动态的添加成员变量和方法。也就意味着你调用一个不存在的方法时，编译也能通过，甚至一个对象它是什么类型并不是表面我们所看到的那样，只有运行之后才能决定其真正的类型。相比于静态语言，动态语言具有较高的灵活性和可订阅性。而oc，正是一门动态语言。
如下代码：

```
@interface Person : NSObject
@property (nonatomic ,copy) NSString *presonName;
- (void)getA;
@end
@implementation Person
- (void)doSomeThing{
    NSLog(@"Person");
}
@end
Person *p = [[Person alloc] init];
 [p getA];
```
这里只是声明了`getA`方法，并没有实现。但是编译的时候并不会报错。依然能通过。只有在运行的时候才发现原来`getA`并没有实现。从而报错`reason: '-[Person getA]: unrecognized selector sent to instance 0x60400000c260'`，如果是`java，c++，c`等等出现上面这种情况，首先编译的时候就会报错，不通过。



