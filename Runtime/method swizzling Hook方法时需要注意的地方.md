# method swizzling Hook方法时需要注意的地方
Runtime运行时Hook方法需要注意以下几点：
### 1、避免交换父类方法
如果当前类未实现被交换的方法而父类实现了的情况下，此时父类的实现会被交换。如果此时父类的多个继承者都在进行交换方法，会导致方法被交换多次而混乱。同时调用父类的方法时会因为找不到而发生奔溃。
所以在交换前都应该先尝试为当前类添加被交换的函数的新的实现IMP，如果添加成功则说明类没有实现被交换的方法，则只需要替代分类交换方法的实现为原方法的实现，如果添加失败，则原类中实现了被交换的方法，可以直接进行交换。
这个过程主要涉及以下三个函数：

1、给类cls的SEL添加一个IMP实现，返回YES则表明类cls并未实现此方法，返回NO则表明类已实现了此方法。**注意**：添加成功与否，完全由该类本身来决定，与父类有无该方法无关。

```
BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types)
```

2、替换类cls的SEL的函数实现为IMP

```
class_replaceMethod(Class _Nullable cls, SEL _Nonnull name, IMP _Nonnull imp, 
                    const char * _Nullable types)
```
3、交换两个方法的实现

```
method_exchangeImplementations(Method _Nonnull m1, Method _Nonnull m2)
```
完整代码如下：

```
Method originalMethod = class_getInstanceMethod(class, originalSelector);
    Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
    BOOL didAddMethod =
    class_addMethod(class,
                    originalSelector,
                    method_getImplementation(swizzledMethod),
                    method_getTypeEncoding(swizzledMethod));
    if (didAddMethod) {//添加成功：则表明没有实现IMP，将源SEL的IMP替换到交换SEL的IMP
        class_replaceMethod(class,
                            swizzledSelector,
                            method_getImplementation(originalMethod),
                            method_getTypeEncoding(originalMethod));
    } else {//添加失败：表明已实现，则可以直接交换实现IMP
     method_exchangeImplementations(originalMethod, swizzledMethod);
    }
```
### 2、交换方法应在+load方法
方法交换应该在调用前交换完成，+load方法发生在运行时初始化过程中类被加载的时候调用，且每个类在加载的时候只会调用一次+load方法，调用的顺序是父类-类-分类，且他们之间是相互独立的，不会存在覆盖的关系，所以放在+load方法中可以确保在使用时已经交换完成。
### 3. 交换方法应该放到dispatch_once中执行
在上面已经写到+load方法在类被加载的时候调用，并且只会调用一次，那么为什么还要dispatch_once呢？这是为了防止手动调用+load方法而导致反复的被交换，因为这是存在可能的。
### 4. 交换的分类方法应该添加自定义前缀，避免冲突
因为分类的方法会覆盖类中同名的方法，这样会导致无法预估的后果。分类的方法有个调用原则。如果方法名称相同的话，会调用最后一个方法。
### 5. 交换的分类方法应调用原实现
很多情况下我们不清楚被交换的方法具体做了什么内部逻辑，而且很多倍交换的方法都是系统方法，所以为了保证其原有的逻辑性都应该在分类的交换方法中去调用原来被交换的方法，注意：调用时方法已经交换完成，在分类方法中应该调用分类本身才正确。

```
- (void)aop_viewDidAppear:(BOOL)animated {
    [self aop_viewDidAppear:animated];
}

-(void)aop_viewWillAppear:(BOOL)animated {
    [self aop_viewWillAppear:animated];
}
```


