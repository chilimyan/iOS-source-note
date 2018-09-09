# 如何给 Category 添加属性？关联对象以什么形式进行存储？
关联对象 以哈希表的格式，存储在一个全局的单例中。

```
@interface NSObject (Extension)

@property (nonatomic,copy  ) NSString *name;

@end


@implementation NSObject (Extension)

- (void)setName:(NSString *)name {
    
    objc_setAssociatedObject(self, @selector(name), name, OBJC_ASSOCIATION_COPY_NONATOMIC);
}


- (NSString *)name {
    
    return objc_getAssociatedObject(self,@selector(name));
}

@end
```


