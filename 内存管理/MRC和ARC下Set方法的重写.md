# MRC和ARC下Set方法的重写

```
@property (nonatomic, assign/weak) NSInteger age;
相当于：
在ARC下:
- (void)setAge:(NSInteger)age{
    _age = age;
}
在MRC下：
- (void)setAge:(NSInteger)age{
    _age = age;
}
```
基本数据类型和weak都不会使对象新增引用。所以都是一样的。

```
@property (nonatomic, strong) NSString *name;
在ARC下:
- (void)setName:(NSString *)name{
    _name = name;
    //如果是@property (nonatomic, copy) NSString *name;
    //则应该要对name进行一次copy：如_name = [name copy];
}
在MRC下：每次赋值要把之前的值release掉然后在retain或者copy新值
- (void)setName:(NSString *)name{
    if(_name != name){
       [_name release];
       _name = [name retain/copy];
    }
}
```


