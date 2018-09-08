# Runtime-类和对象
### Class的定义

```
typedef struct objc_class *Class;

struct objc_class { 
Class isa ;                                                         //每个Class都有一个isa指针 
#if !__OBJC2__ 
Class super_class ;                                          //父类 
const char *name ;                                          //类名 
long version ;                                                   //类版本
 long info ;                                                        //!*!供运行期使用的一些位标识。如：CLS_CLASS (0x1L)表示该类为普   class;CLS_META(0x2L)表示该类为metaclass等(runtime.h中有详细列出) 
long instance_size ;                                        //实例大小 
struct objc_ivar_list *ivars ;                            //存储每个实例变量的内存地址 
struct objc_method_list **methodLists ;      //!*!根据info的信息确定是类还是实例，运行什么函数方法等 struct objc_cache *cache ;                            //缓存 
struct objc_protocol_list *protocols ;            //协议 
#endif  
} ;
```
##### isa指针
* 对象的`isa`指针指向所属的类
* 类的`isa`指针指向所属的元类
* 所有的元类的 `isa`指针都会指向一个根元类 (`root metaclass`)
* 根元类的 `isa`指针指向自己，行成了一个闭环

#### super_class
指向该类的父类，如果该类已经是最顶层的根类(如`NSObject`或`NSProxy`)，则`super_class`为`NULL`。

#### cache
用于缓存最近使用的方法。一个接收者对象接收到一个消息时，它会根据`isa`指针去查找能够响应这个消息的对象。在实际使用中，这个对象只有一部分方法是常用的，很多方法其实很少用或者根本用不上。这种情况下，如果每次消息来时，我们都是`methodLists`中遍历一遍，性能势必很差。这时，`cache`就派上用场了。在我们每次调用过一个方法后，这个方法就会被缓存到`cache`列表中，下次调用的时候`runtime`就会优先去`cache`中查找，如果`cache`没有，才去`methodLists`中查找方法。这样，对于那些经常用到的方法的调用，但提高了调用的效率。

#### version
我们可以使用这个字段来提供类的版本信息。这对于对象的序列化非常有用，它可是让我们识别出不同类定义版本中实例变量布局的改变。
#### objc_cache
上面提到了`objc_class`结构体中的`cache`字段，它用于缓存调用过的方法。这个字段是一个指向`objc_cache`结构体的指针，其定义如下：

```
struct objc_cache {
    unsigned int mask /* total = mask + 1 */            // 一个整数，指定分配的缓存bucket的总数;
    unsigned int occupied                                         // 一个整数，指定实际占用的缓存bucket的总数   ;
    Method buckets[1]                                               //指向Method数据结构指针的数组 ;
};
```
> 在`objc_class`结构体中：
> `ivars`是`objc_ivar_list`指针`methodLists`是指向`objc_method_list`指针的指针。也就是说可以动态修改 `*methodLists` 的值来添加成员方法，这也是`Category`实现的原理。

### id类型
`id` 是一个参数类型，它是指向某个类的实例的指针。定义如下：

```
typedef struct objc_object *id;
struct objc_object { Class isa; };
```
以上定义，看到 `objc_object` 结构体包含一个 `isa` 指针，根据 `isa` 指针就可以找到对象所属的类。
#### 注意：
`isa` 指针在代码运行时并不总指向实例对象所属的类型，所以不能依靠它来确定类型，要想确定类型还是需要用对象的 `-class `方法。
> KVO 的实现机理就是将被观察对象的 isa 指针指向一个中间类而不是真实类型。
### 元类（meta class）

```
NSArray *array = [NSArray array];
```
`+array`消息发送给了`NSArray`类，而这个`NSArray`也是一个对象。既然是对象，那么它也是一个`objc_object`指针，它包含一个指向其类的一个`isa`指针。那么这些就有一个问题了，这个`isa`指针指向什么呢？为了调用`+array`方法，这个类的`isa`指针必须指向一个包含这些类方法的一个`objc_class`结构体。这就引出了`meta-class`的概念：`meta-class是一个类对象的类`。

当我们向一个对象发送消息时，`runtime`会在这个对象所属的这个类的方法列表中查找方法；而向一个类发送消息时，会在这个类的`meta-class`的方法列表中查找。
`meta-class`之所以重要，是因为它存储着一个类的所有类方法。每个类都会有一个单独的`meta-class`，因为每个类的类方法基本不可能完全相同。
再深入一下，`meta-class`也是一个类，也可以向它发送一个消息，那么它的`isa`又是指向什么呢？为了不让这种结构无限延伸下去，`Objective-C`的设计者让所有的`meta-class`的`isa`指向基类的`meta-class`，以此作为它们的所属类。即，任何`NSObject`继承体系下的`meta-class`都使用`NSObject`的`meta-class`作为自己的所属类，而基类的`meta-class`的`isa`指针是指向它自己。这样就形成了一个完美的闭环。
通过上面的描述，再加上对`objc_class`结构体中`super_class`指针的分析，我们就可以描绘出类及相应`meta-class`类的一个继承体系了，如下图所示：
![](https://upload-images.jianshu.io/upload_images/1840444-8781a1b16bc474af.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 类与对象操作函数
`runtime`提供了大量的函数来操作类与对象。类的操作方法大部分是以`class`为前缀的，而对象的操作方法大部分是以`objc`或`object_`为前缀

```
//获取类的名称
const char * class_getName ( Class cls );  
//获取类的父类
Class class_getSuperclass ( Class cls );    //class_getSuperclass函数，当cls为Nil或者cls为根类时，返回Nil。不过通常我们可以使用NSObject类的superclass方法来达到同样的目的。
//判断给定的Class是否是一个元类
BOOL class_isMetaClass ( Class cls );   //class_isMetaClass函数，如果是cls是元类，则返回YES；如果否或者传入的cls为Nil，则返回NO
//获取实例变量的大小
size_t class_getInstanceSize ( Class cls );
```
#### 成员变量(ivars)及属性
在`objc_class`中，所有的成员变量、属性的信息是放在链表`ivars`中的。`ivars`是一个数组，数组中每个元素是指向`Ivar`(变量信息)的指针。

```
// 获取类中指定名称实例成员变量的信息 
Ivar class_getInstanceVariable ( Class cls, const char *name ); //class_getInstanceVariable函数，它返回一个指向包含name指定的成员变量信息的objc_ivar结构体的指针(Ivar)。
// 获取类成员变量的信息 
Ivar class_getClassVariable ( Class cls, const char *name ); //class_getClassVariable函数，目前没有找到关于Objective-C中类变量的信息，一般认为Objective-C不支持类变量。注意，返回的列表不包含父类的成员变量和属性
// 添加成员变量 
BOOL class_addIvar ( Class cls, const char *name, size_t size, uint8_t alignment, const char *types ); //Objective-C不支持往已存在的类中添加实例变量，因此不管是系统库提供的提供的类，还是我们自定义的类，都无法动态添加成员变量。但如果我们通过运行时来创建一个类的话，又应该如何给它添加成员变量呢？这时我们就可以使用class_addIvar函数了。不过需要注意的是，这个方法只能在objc_allocateClassPair函数与objc_registerClassPair之间调用。另外，这个类也不能是元类。成员变量的按字节最小对齐量是1<<alignment。这取决于ivar的类型和机器的架构。如果变量的类型是指针类型，则传递log2(sizeof(pointer_type))

// 获取整个成员变量列表 
Ivar * class_copyIvarList ( Class cls, unsigned int *outCount );//class_copyIvarList函数，它返回一个指向成员变量信息的数组，数组中每个元素是指向该成员变量信息的objc_ivar结构体的指针。这个数组不包含在父类中声明的变量。outCount指针返回数组的大小。需要注意的是，我们必须使用free()来释放这个数组。

// 获取指定的属性 
objc_property_t class_getProperty ( Class cls, const char *name ); 
// 获取属性列表 
objc_property_t * class_copyPropertyList ( Class cls, unsigned int *outCount ); 
// 为类添加属性 
BOOL class_addProperty ( Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount ); 
// 替换类的属性 
void class_replaceProperty ( Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount );
```
#### 方法(methodLists)

```
// 添加方法 
BOOL class_addMethod ( Class cls, SEL name, IMP imp, const char *types ); //class_addMethod的实现会覆盖父类的方法实现，但不会取代本类中已存在的实现，如果本类中包含一个同名的实现，则函数会返回NO。如果要修改已存在实现，可以使用method_setImplementation。一个Objective-C方法是一个简单的C函数，它至少包含两个参数—self和_cmd。所以，我们的实现函数(IMP参数指向的函数)至少需要两个参数，与成员变量不同的是，我们可以为类动态添加方法，不管这个类是否已存在。另外，参数types是一个描述传递给方法的参数类型的字符数组，这就涉及到类型编码。
// 获取实例方法 
Method class_getInstanceMethod ( Class cls, SEL name ); 

// 获取类方法 
Method class_getClassMethod ( Class cls, SEL name ); 
// 获取所有方法的数组 
Method * class_copyMethodList ( Class cls, unsigned int *outCount ); //返回包含所有实例方法的数组，如果需要获取类方法，则可以使用class_copyMethodList(object_getClass(cls), &count)(一个类的实例方法是定义在元类里面)。该列表不包含父类实现的方法。outCount参数返回方法的个数。在获取到列表后，我们需要使用free()方法来释放它
// 替代方法的实现 
IMP class_replaceMethod ( Class cls, SEL name, IMP imp, const char *types );//该函数的行为可以分为两种：如果类中不存在name指定的方法，则类似于class_addMethod函数一样会添加方法；如果类中已存在name指定的方法，则类似于method_setImplementation一样替代原方法的实现
 // 返回方法的具体实现 
IMP class_getMethodImplementation ( Class cls, SEL name ); //该函数在向类实例发送消息时会被调用，并返回一个指向方法实现函数的指针。这个函数会比method_getImplementation(class_getInstanceMethod(cls, name))更快。返回的函数指针可能是一个指向runtime内部的函数，而不一定是方法的实际实现。例如，如果类实例无法响应selector，则返回的函数指针将是运行时消息转发机制的一部分
IMP class_getMethodImplementation_stret ( Class cls, SEL name ); 
// 类实例是否响应指定的函数 
selector BOOL class_respondsToSelector ( Class cls, SEL sel );//我们通常使用NSObject类的respondsToSelector:或instancesRespondToSelector:方法来达到相同目的
```
#### 协议(objc_protocol_list)

```
// 添加协议 
BOOL class_addProtocol ( Class cls, Protocol *protocol );

 // 返回类是否实现指定的协议 
BOOL class_conformsToProtocol ( Class cls, Protocol *protocol ); //函数可以使用NSObject类的conformsToProtocol:方法来替代
// 返回类实现的协议列表
Protocol * class_copyProtocolList ( Class cls, unsigned int *outCount );//函数返回的是一个数组，在使用后我们需要使用free()手动释放。
```
实例

```
//----------------------------------------------------------- // 
MyClass.h 
@interface MyClass : NSObject <NSCopying, NSCoding>
 @property (nonatomic, strong) NSArray *array;
 @property (nonatomic, copy) NSString *string; 
- (void)method1; 
- (void)method2;
 + (void)classMethod1; 
@end 
//----------------------------------------------------------- // 
MyClass.m 
#import "MyClass.h" 
@interface MyClass () 
{ NSInteger _instance1; NSString * _instance2; } 
@property (nonatomic, assign) NSUInteger integer; 
- (void)method3WithArg1:(NSInteger)arg1 arg2:(NSString *)arg2;
 @end 
@implementation MyClass
 + (void)classMethod1 { } 
- (void)method1 { NSLog(@"call method method1"); 
} 
- (void)method2 { } 
- (void)method3WithArg1:(NSInteger)arg1 arg2:(NSString *)arg2 { 
NSLog(@"arg1 : %ld, arg2 : %@", arg1, arg2); 
} 
@end 
//----------------------------------------------------------- // 
main.h
 #import "MyClass.h" 
#import "MySubClass.h"
 #import <objc/runtime.h> 
int main(int argc, const char * argv[]) { 
@autoreleasepool { 
MyClass *myClass = [[MyClass alloc] init]; 
unsigned int outCount = 0; 
Class cls = myClass.class; 
// 类名 
NSLog(@"class name: %s", class_getName(cls)); NSLog(@"=========================================================="); // 父类 NSLog(@"super class name: %s", class_getName(class_getSuperclass(cls))); NSLog(@"=========================================================="); // 是否是元类 NSLog(@"MyClass is %@ a meta-class", (class_isMetaClass(cls) ? @"" : @"not")); NSLog(@"=========================================================="); Class meta_class = objc_getMetaClass(class_getName(cls)); NSLog(@"%s's meta-class is %s", class_getName(cls), class_getName(meta_class)); NSLog(@"=========================================================="); // 变量实例大小 NSLog(@"instance size: %zu", class_getInstanceSize(cls)); NSLog(@"=========================================================="); // 成员变量
 Ivar *ivars = class_copyIvarList(cls, &outCount);
 for (int i = 0; i < outCount; i++) { 
Ivar ivar = ivars[i]; 
NSLog(@"instance variable's name: %s at index: %d", ivar_getName(ivar), i); 
} 
free(ivars);
Ivar string = class_getInstanceVariable(cls, "_string");
 if (string != NULL) {
 NSLog(@"instace variable %s", ivar_getName(string));
 } 
NSLog(@"=========================================================="); 
// 属性操作
 objc_property_t * properties = class_copyPropertyList(cls, &outCount); 
for (int i = 0; i < outCount; i++) {
 objc_property_t property = properties[i]; 
NSLog(@"property's name: %s", property_getName(property)); 
} 
free(properties);
 objc_property_t array = class_getProperty(cls, "array"); 
if (array != NULL) {
 NSLog(@"property %s", property_getName(array));
 }
 NSLog(@"=========================================================="); 
// 方法操作 
Method *methods = class_copyMethodList(cls, &outCount); 
for (int i = 0; i < outCount; i++) { 
Method method = methods[i]; 
NSLog(@"method's signature: %s", method_getName(method));
 } 
free(methods); 
Method method1 = class_getInstanceMethod(cls, @selector(method1)); 
if (method1 != NULL) { 
NSLog(@"method %s", method_getName(method1)); 
} 
Method classMethod = class_getClassMethod(cls, @selector(classMethod1)); 
if (classMethod != NULL) { 
NSLog(@"class method : %s", method_getName(classMethod)); 
} 
NSLog(@"MyClass is%@ responsd to selector: method3WithArg1:arg2:", class_respondsToSelector(cls, @selector(method3WithArg1:arg2:)) ? @"" : @" not"); 
IMP imp = class_getMethodImplementation(cls, @selector(method1)); imp(); NSLog(@"=========================================================="); 
// 协议
 Protocol * __unsafe_unretained * protocols = class_copyProtocolList(cls, &outCount); 
Protocol * protocol; for (int i = 0; i < outCount; i++) { 
protocol = protocols[i]; 
NSLog(@"protocol name: %s", protocol_getName(protocol)); 
} 
NSLog(@"MyClass is%@ responsed to protocol %s", class_conformsToProtocol(cls, protocol) ? @"" : @" not", protocol_getName(protocol)); NSLog(@"==========================================================");
 } 
return 0;
 }
```
### 动态创建类和对象

```
// 创建一个新类和元类 
Class objc_allocateClassPair ( Class superclass, const char *name, size_t extraBytes ); //如果我们要创建一个根类，则superclass指定为Nil。extraBytes通常指定为0，该参数是分配给类和元类对象尾部的索引ivars的字节数。为了创建一个新类，我们需要调用objc_allocateClassPair。然后使用诸如class_addMethod，class_addIvar等函数来为新创建的类添加方法、实例变量和属性等。完成这些后，我们需要调用objc_registerClassPair函数来注册类，之后这个新类就可以在程序中使用了。实例方法和实例变量应该添加到类自身上，而类方法应该添加到类的元类上

// 销毁一个类及其相关联的类 
void objc_disposeClassPair ( Class cls ); //用于销毁一个类，不过需要注意的是，如果程序运行中还存在类或其子类的实例，则不能调用针对类调用该方法
// 在应用中注册由objc_allocateClassPair创建的类 
void objc_registerClassPair ( Class cls );
// 创建类实例 
id class_createInstance ( Class cls, size_t extraBytes ); //创建实例时，会在默认的内存区域为类分配内存。extraBytes参数表示分配的额外字节数。这些额外的字节可用于存储在类定义中所定义的实例变量之外的实例变量。该函数在ARC环境下无法使用。调用class_createInstance的效果与+alloc方法类似。不过在使用class_createInstance时，我们需要确切的知道我们要用它来做什么
id theObject = class_createInstance(NSString.class, sizeof(unsigned));
 id str1 = [theObject init]; 
NSLog(@"%@", [str1 class]); 
id str2 = [[NSString alloc] initWithString:@"test"]; 
NSLog(@"%@", [str2 class]);
// 在指定位置创建类实例
 id objc_constructInstance ( Class cls, void *bytes ); //在指定的位置(bytes)创建类实例

// 销毁类实例 
void * objc_destructInstance ( id obj );//销毁一个类的实例，但不会释放并移除任何与其相关的引用。

// 返回指定对象的一份拷贝
id object_copy ( id obj, size_t size );

// 释放指定对象占用的内存
id object_dispose ( id obj );
有这样一种场景，假设我们有类A和类B，且类B是类A的子类。类B通过添加一些额外的属性来扩展类A。现在我们创建了一个A类的实例对象，并希望在运行时将这个对象转换为B类的实例对象，这样可以添加数据到B类的属性中。这种情况下，我们没有办法直接转换，因为B类的实例会比A类的实例更大，没有足够的空间来放置对象。此时，我们就要以使用以上几个函数来处理这种情况，如下代码所示
NSObject *a = [[NSObject alloc] init];
 id newB = object_copy(a, class_getInstanceSize(MyClass.class)); 
object_setClass(newB, MyClass.class); 
object_dispose(a);
// 修改类实例的实例变量的值 
Ivar object_setInstanceVariable ( id obj, const char *name, void *value );

 // 获取对象实例变量的值 
Ivar object_getInstanceVariable ( id obj, const char *name, void **outValue ); 

// 返回指向给定对象分配的任何额外字节的指针 
void * object_getIndexedIvars ( id obj );

 // 返回对象中实例变量的值
 id object_getIvar ( id obj, Ivar ivar );

 // 设置对象中实例变量的值
 void object_setIvar ( id obj, Ivar ivar, id value );
如果实例变量的Ivar已经知道，那么调用object_getIvar会比object_getInstanceVariable函数快，相同情况下，object_setIvar也比object_setInstanceVariable快
// 返回给定对象的类名 
const char * object_getClassName ( id obj ); 

// 返回对象的类 
Class object_getClass ( id obj );

 // 设置对象的类 
Class object_setClass ( id obj, Class cls );
```
`Objective-C`动态运行库会自动注册我们代码中定义的所有的类。我们也可以在运行时创建类定义并使用`objc_addClass`函数来注册它们。

```
// 获取已注册的类定义的列表
 int objc_getClassList ( Class *buffer, int bufferCount ); 

// 创建并返回一个指向所有已注册类的指针列表
 Class * objc_copyClassList ( unsigned int *outCount );//获取已注册的类定义的列表。我们不能假设从该函数中获取的类对象是继承自NSObject体系的，所以在这些类上调用方法是，都应该先检测一下这个方法是否在这个类中实现

 // 返回指定类的类定义 
Class objc_lookUpClass ( const char *name ); //如果类在运行时未注册，则objc_lookUpClass会返回nil，而objc_getClass会调用类处理回调，并再次确认类是否注册，如果确认未注册，再返回nil。而objc_getRequiredClass函数的操作与objc_getClass相同，只不过如果没有找到类，则会杀死进程
Class objc_getClass ( const char *name ); 
Class objc_getRequiredClass ( const char *name ); 

// 返回指定类的元类 
Class objc_getMetaClass ( const char *name );//如果指定的类没有注册，则该函数会调用类处理回调，并再次确认类是否注册，如果确认未注册，再返回nil。不过，每个类定义都必须有一个有效的元类定义，所以这个函数总是会返回一个元类定义，不管它是否有效
```


