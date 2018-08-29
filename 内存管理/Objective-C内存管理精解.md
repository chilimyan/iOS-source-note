# Objective-C内存管理精解
## 引用计数
### 引用计数表
每个对象都有一个对应的引用计数表，苹果使用引用计数表来管理各对象的引用计数，表中记录有各对象内存块的地址，从而根据地址找到各对象的内存块。
这样做的话有几个好处：
1. 对象的内存块的分配无需考虑内存块的头部（如果没有引用计数表，则每个对象的内存块头部将用来存放引用计数）
2. 即使出现故障导致对象占用的内存块损坏，但只要引用计数表没有被破坏，就能够确认各内存块的位置。
3. 利用工具检测内存泄漏时，引用计数表的记录也有助于检测各对象的持有者是否存在。
生成一个对象或者持有一个对象都会使引用计数加1，释放对象时引用计数减1，当引用计数为0时，对象被废弃。
## MRC规则
### 内存管理的思考方式
#### 自己生成的对象自己持有
比如使用`alloc`、`new`、`copy`、`mutableCopy`等方法，引用计数自动加1

```
id obj = [[NSObject alloc] init];
```
#### 非自己生成的对象，自己也能持有
除了上述方法之外生成的对象，并且使用`retain`方法持有对象。

```
id obj = [NSMutableArray array];//生成对象
[obj  retain];//持有对象
```
#### 不再需要自己持有的对象时释放
如：`release`方法
#### 非自己持有的对象无法释放
如：`dealloc`方法
### retain、retainCount、release的实现
调用同一个函数`__CFDoExternRefOperation`，根据对象的内存地址和引用计数表去修改对象的引用计数
### autorelease
对象调用`autorelease`后被加入`NSAutoreleasePool`自动释放池中，当自动释放池废弃时，对象会自动调用`release`方法

在大量产生`autorelease`的对象时，有可能会出现内存不足的情况，这时应该手动生成一个`NSAutoreleasePool`，把这些对象放进这个池子里，然后用完之后再手动`[pool drain]`释放池子
### autorelease的实现
调用`autorelease`方法实际上是通过add方法将对象放入一个`NSAutoreleasePool`自动释放池中。先是用`push`方法获得一个`NSAutorelease`自动释放池，然后用`pop`方法废弃一个`NSAutorelease`自动释放池

```
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
//等同于objc_autoreleasePoolPush()生成或持有NSAutoreleasePool对象

id obj= [[NSObject alloc] init];
[obj autorelease];
//等同于objc_autorelease(obj)将对象追加到自动释放池中
[pool drain];
```
### autorelease一个NSAutoreleasePool对象会发生异常
因为无论调用哪个对象的`autorelease`方法，实现上都是调用`NSObject`类的`autorelease`实例方法，但是对于`NSAutorelease`类，`autorelease`实例方法已被该类重载。因此运行时就会出错
## ARC规则
ARC只是自动的帮助我们处理引用计数的相关部分
### ARC自动追加所有权修饰符，附有这些修饰符的自动变量初始化为nil
id __strong obj0等价于id __strong obj0 = nil
#### __strong
所有对象默认的所有权修饰符,表示对对象的强引用，持有强引用的变量超出其作用域时，随着强引用的失效，引用对象随之释放

```
id __strong obj = [[NSObject alloc] init]
在ARC无效时等价于
id obj = [[NSObject alloc] init];
[obj release];
```
`__strong`变量互相赋值也能正确管理对象内存

```
id __strong obj0 = [[NSObject alloc] init];//obj0持有对象A的强引用
id __strong obj1 = [[NSObject alloc] init];//obj1持有对象B的强引用
obj0 = obj1；
引用obj0被赋值，所以原先持有的对对象A的强引用失效，对象A的所有者不存在因此废弃对象A
id __strong obj2 = nil;//obj2不持有任何对象
```
#### __weak
`__weak`修饰符表示弱引用，不能持有对象实例，解决`__strong`循环引用的问题

```
id __weak obj = [[NSObject alloc] init];
这句代码会有警告。因为obj并不持有对象，这样生成的对象会被立即释放，obj=nil。
但是如果这样先赋值给强引用，再赋值给弱引用就没问题：
id __strong obj0 = [[NSObject alloc] init];
id __weak obj1 = obj0; 
```
有`__weak`修饰某对象的弱引用时，如果对象被废弃，该弱引用将失效自动赋值为`nil`

在访问附有`__weak`修饰符的变量时必须访问注册到自动释放池中的对象，因为__weak修饰符只持有对象的弱引用，在访问引用对象的过程中，该对象有可能被废弃，如果把要访问的对象注册到自动释放池中，则在自动释放池废弃前对象都存在。
#### __unsafe_unretained

```
id __unsafe_unretained obj1 = nil;
id __strong obj0 = [[NSObject alloc] init];
 obj1 = obj0;
虽然obj0变量赋值给obj1，但是obj1变量既不持有对象的强引用也不持有弱引用
```
同`__weak`一样,不能持有对象，生成的对象会立即释放
与`__weak`不同，当对象被废弃时，并不指向`nil`而是产生一个野指针。
#### __autoreleasing
在`ARC`中用`autoreleasing`修饰的对象等价于`MRC`中对象调用`autorelease`方法

同`__strong`一样，没有显式指定`__autoreleasing`的对象，也会自动注册到`autoreleasePool`
### ARC规则
* 不能使用`retain/releaase/retainCount/autorelease`
* 不能使用`NSAllocateObject/NSDeallocateObject`
* 不要显示调用`dealloc`
* 使用`@autoreleasepool`块代替`NSAutoreleasePool`
* 不能使用区域`NSZone`
* 对象型变量不能作为C语言结构体的成员
* 显示转换`“id”`和`“voic *”`（通过`__bridge`转换，`id`和`void *`就能互相转换）

```
id obj = [[NSObject alloc] init]
void *p = (__bridge void *)obj
但是__bridge跟__unsafe_unretained一样都是不安全的。可以使用__bridge_retained，这个相当于retain。而__bridge_transfer相当于release
```

无论是ARC无效还是有效用于对象生成持有的方法必须遵守以下命名规则：
`alloc`、`new`、`copy`、`mutableCopy`
但是在`ARC`有效时新增`init`命名规则，该方法必须是实例方法，并且必须要返回对象。返回的对象类型应为`id`类型或者该方法声明类的对象类型，或者是该类的超类或者子类。返回的对象并不注册到`autoreleasepool`上，基本上只是对`alloc`方法返回值的对象进行初始化并返回该对象
### 属性

| 声明的属性 | 对应的所有权修饰符 |
| --- | --- |
| assign | __unsafe_unretained修饰符 |
| copy | __strong修饰符 |
| retain | __strong修饰符 |
| strong | __strong修饰符 |
| unsafe_unretained | __unsafe_unretained修饰符 |
| weak | __weak修饰符 |
`copy`属性不是简单的赋值，它赋值的是通过`NSCopying`接口的`copyWithZone：`方法复制赋值源所生成的对象
### 数组
使用`calloc`函数可以使分配给附有`__strong`修饰符变量的数组初始化为`nil`，比如：

```
array = (id __strong *)calloc(entries, sizeof(id))
```
如果使用C函数的`malloc`则分配后还需要用`memset`函数将内存填充为0。
比如这样写是不对的:

```
array = (id __strong *)malloc(sizeof(id) *entries);
for (NSUInteger i = 0; i< entries; i++)
      array[i] = nil;
```
这里使用malloc函数分配的内存区域没有被初始化为0，因此nil会被赋值给附有__strong修饰符的并被赋值了随机地址的变量中，从而释放一个不存在的对象。

动态数组中操作附有`__strong`修饰符的变量时需要手动释放内存。如：`free(array)`;因为动态数组编译器不能确定数组的生存周期，无从处理。
而静态数组则不需要，因为在静态数组中，编译器能够根据变量的作用域自动插入释放赋值对象的代码。
## ARC实现
### __strong修饰符
`alloc、new、copy、mutableCopy`生成并持有的对象内部实现如下：

```
id obj = objc_msgSend(NSObject, @selector(alloc));
objc_msgSend(obj,@selector(init));
objc_release(obj);//ARC有效时自动插入了release方法
```

其他生成对象如：`id __strong obj = [NSMutableArray array];`的内部实现如下：

```
id obj = objc_msgSend(NSMutableArray, @selector(array));
objc_retainAutoreleasedReturnValue(obj);
objc_release(obj);
```
### __weak修饰符

```
id __weak obj1 = obj;内部实现如下：
id obj1;
objc_initWeak(&obj1,obj);//objc_initWeak函数将变量初始化为0即obj1 = 0；然后再调用objc_storeWeak(&obj1,obj);将修饰符__weak的变量的地址注册到weak表中。
objc_destroyWeak(&obj1);//相当于调用objc_storeWeak(&obj1,0);把变量的地址从weak表中删除。
```
若附有`__weak`修饰符的变量所引用的对象被废弃，则将`nil`赋值给该变量

使用附有`__weak`修饰符的变量，即是使用注册到`autoreleasepool`中的对象：

```
id __weak obj1 = obj;
以上代码的实现如下：
id obj1;
objc_initWeak(&obj1, obj);//生成一个Weak对象
id tmp = objc_loadWeakRetained(&obj1);//写入weak表新增一个引用
objc_autorelease(tmp);//将对象注册到自动释放池
objc_destroyWeak(&obj1);
```
每使用一次附有`__weak`修饰符都会注册一次对象到自动释放池，例如：

```
id __weak o = obj;
NSLog(@"1 %@",o);
NSLog(@"2 %@",o);
NSLog(@"3 %@",o);
```
这里注册了3次自动释放池。但是将`__weak`修饰符的变量赋给`__strong`修饰的变量就可以避免这种情况如：

```
id __weak o = obj;
id tmp = o;
NSLog(@"1 %@",tmp);
NSLog(@"2 %@",tmp);
NSLog(@"3 %@",tmp);
```
这里只注册了一次自动释放池
因此在`@autoreleasePool`块结束之前都可以放心使用`__weak`修饰符的变量
#### 对象释放过程：objc_release执行步骤
1. 调用`objc_release`
2. 因为引用计数为0所以执行`dealloc`
3. 调用`_objc_rootDealloc`
4. 调用`object_dispose`
5. 调用`objc_destructInstance`
6. 调用`objc_clear_deallocating`

>1.从`weak`表中获取废弃对象的地址为键值的记录
>2.将包含在记录中的所有附有`__weak`修饰符变量的地址，赋值为`nil`
>3.从`weak`表中删除该记录
>4.从引用计数表中删除废弃对象的地址为键值的记录

#### 使用__weak修饰符时，以下源代码会引起编译警告
 
```
id __weak obj = [[NSObject alloc] init];
```
上述代码，内部实现如下：
内部实现如下：

```
id obj ;
id tmp = objc_msgSend(NSObject, @selector(alloc));
objc_msgSend(tmp,@selector(init));
objc_initWeak(&obj, tmp);
objc_release(tmp);
objc_destoryWeak(&object);
```
这段代码会引发警告，因为编译器判断生成并持有的对象并不能继续持有，`__weak`是不持有对象的.所以对象会被立即通过`objc_release`函数释放和废弃。`obj`则会为`nil`
使用`_unsafe_unretained`也是一样的效果，只是不会把对象置为`nil`，而是成为一个野指针
### __autoreleasing修饰符
将对象赋值给附有`__autoreleasing`修饰符的变量等同于`ARC`无效时调用对象的`autorelease`方法

```
@autoreleasepool{
         id __autoreleasing obj = [[NSObject alloc] init];
}
以上实现如下：
id pool = objc_autoreleasePoolPush();
id obj = objc_msgSend(NSObject, @selector(alloc));
objc_msgSend(obj, @selector(init));
objc_autorelease(obj);//如果使用alloc、new、copy、mutableCopy方法之外的方法中使用注册到自动释放池的对象则这里执行objc_retainAutoreleasedReturnValue(obj);方法，其他不变
objc_autoreleasePoolPop(obj);
```
### 引用计数
* 使用`__strong`修饰符的对象引用计数+1
* 使用`__autoreleasing`修饰符的对象引用计数+1
* 使用`__weak`修饰符的对象，由于弱引用并不持有对象本身，所以引用计数不变

