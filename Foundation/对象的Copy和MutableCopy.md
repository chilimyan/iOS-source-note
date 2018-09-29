# 对象的Copy和MutableCopy
![](https://upload-images.jianshu.io/upload_images/1840444-9757403cddf44f74.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
所谓的`copy`是浅复制，`mutableCopy`是深复制。这种说法是**错误**的

从上面这张图中我们可以知道

1. 对于可变对象不管是`Copy`还是`MutableCopy`其实都是产生一个新的对象，只不过是如果是`Copy`则出来的副本对象是不可变的，`MutableCopy`出来的副本对象还是可变的。所以当我们在定义属性的时候对于`NSMutablexxx`不能使用`@property(copy,nonatmoic)`。
2. 对于不可变对象，`copy`出来的副本还是不可变对象，且并不是一个新对象，只是指针复制。`MutableCopy`出来的副本对象就变成了一个可变对象，并且是一个新对象。

#### 自定义对象的复制
对于自定义对象要想实现复制则要实现以下方法：

1. 让类实现NSCopying/NSMutableCopying协议。
2. 让类实现copyWithZone:/mutableCopyWithZone:方法

```
@interface CLPerson : NSObject<NSCopying>
@property (nonatomic, assign) NSInteger age;
@property (nonatomic, copy) NSString *name;
@end
#import "CLPerson.h"
@implementation CLPerson
- (id)copyWithZone:(NSZone *)zone {
    CLPerson *person = [[[self class] allocWithZone:zone] init];
    person.age = self.age;
    person.name = self.name;
    return person;
}
@end

CLPerson *person = [[CLPerson alloc] init];
    person.name = @"张三";
    person.age = 18;
    
    CLPerson *person1 = [person copy];
    NSLog(@"person:%p",person,person);
    NSLog(@"person:%@---%p",person.name,person.name);
    NSLog(@"person:%d---%p",person.age,person.age);
    NSLog(@"person1:%p",person1);
    NSLog(@"person1:%@---%p",person1.name,person1.name);
    NSLog(@"person1:%d---%p",person1.age,person1.age);
```
打印结果如下：

```
2018-09-29 18:34:47.120957+0800 Demo[46464:344037] person:0x604000236320
2018-09-29 18:34:47.123234+0800 Demo[46464:344037] person:张三---0x1037f5bc8
2018-09-29 18:34:47.123877+0800 Demo[46464:344037] person:18---0x12
2018-09-29 18:34:47.124841+0800 Demo[46464:344037] person1:0x604000236620
2018-09-29 18:34:47.125007+0800 Demo[46464:344037] person1:张三---0x1037f5bc8
2018-09-29 18:34:47.125178+0800 Demo[46464:344037] person1:18---0x12
```
可以看到，这里是一个深拷贝。


