# 使用自动引用计（ARC）数应该遵循的原则?
1. 不能使用retain/releaase/retainCount/autorelease
2. 不能使用NSAllocateObject/NSDeallocateObject
3. 不要显示调用dealloc
4. 使用@autoreleasepool块代替NSAutoreleasePool
5. 不能使用区域NSZone
6. 对象型变量不能作为C语言结构体的成员
7. 显示转换“id”和“voic *”（通过__bridge转换，id和void *就能互相转换）

```
id obj = [[NSObject alloc] init]
void *p = (__bridge void *)obj
但是__bridge跟__unsafe_unretained一样都是不安全的。可以使用__bridge_retained，这个相当于retain。而__bridge_transfer相当于release
```

