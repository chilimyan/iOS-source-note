# Runtime-消息转发机制原理
`SEL`又叫选择器，是表示一个方法的`selector`的指针，其定义如下：

```
typedef struct objc_selector *SEL;
```
`objc_selector`结构体的详细定义没有在头文件中找到。方法的`selector`用于表示运行时方 法的名字。`Objective-C`在编译时，会依据每一个方法的名字、参数序列，生成一个唯一的整型标识(Int类型的地址)，这个标识就是SEL。如下 代码所示：

```
SEL sel1 = @selector(method1);
```
两个类之间，不管它们是父类与子类的关系，还是之间没有这种关系，只要方法名相同，那么方法的SEL就是一样的。每一个方法都对应着一个SEL。所以在 Objective-C同一个类(及类的继承体系)中，不能存在2个同名的方法，即使参数类型不同也不行。相同的方法只能对应一个SEL。如在某个类中定义以下两个方法将报错：

```
- (void)setWidth:(int)width;
- (void)setWidth:(double)width;
```
工程中的所有的SEL组成一个`Set集合`，`Set`的特点就是唯一，因此`SEL`是唯一的。因此，如果我们想到这个方法集合中查找某个方法时，只需要去 找到这个方法对应的`SEL`就行了，`SEL`实际上就是根据方法名`hash`化了的一个字符串，而对于字符串的比较仅仅需要比较他们的地址就可以了，可以说速度 上无语伦比！！但是，有一个问题，就是数量增多会增大`hash`冲突而导致的性能下降（或是没有冲突，因为也可能用的是`perfect hash`）。但是不管使用什么样的方法加速，如果能够将总量减少（多个方法可能对应同一个`SEL`），那将是最犀利的方法。那么，我们就不难理解，为什么 `SEL`仅仅是函数名了。
本质上，`SEL`只是一个指向方法的指针（准确的说，只是一个根据方法名`hash`化了的KEY值，能唯一代表一个方法），它的存在只是为了加快方法的查询速度。这个查找过程我们将在下面讨论。
我们可以在运行时添加新的`selector`，也可以在运行时获取已存在的`selector`，我们可以通过下面三种方法来获取`SEL`
`sel_registerName`函数
`Objective-C`编译器提供的`@selector()`
`NSSelectorFromString()`方法
IMP实际上是一个函数指针，指向方法实现的首地址。其定义如下
`id (*IMP)(id, SEL, ...)`
这个函数使用当前CPU架构实现的标准的C调用约定。第一个参数是指向`self`的指针(如果是实例方法，则是类实例的内存地址；如果是类方法，则是指向元类的指针)，第二个参数是方法选择器(`selector`)，接下来是方法的实际参数列表。
前面介绍过的`SEL`就是为了查找方法的最终实现IMP的。由于每个方法对应唯一的SEL，因此我们可以通过SEL方便快速准确地获得它所对应的 IMP，查找过程将在下面讨论。取得IMP后，我们就获得了执行这个方法代码的入口点，此时，我们就可以像调用普通的C语言函数一样来使用这个函数指针 了。
通过取得IMP，我们可以跳过Runtime的消息传递机制，直接执行IMP指向的函数实现，这样省去了`Runtime`消息传递过程中所做的一系列查找操作，会比直接向对象发送消息高效一些。
Method用于表示类定义中的方法，则定义如下：
```
typedef struct objc_method *Method; 
struct objc_method { 
SEL method_name ; // 方法名 
char *method_types ;
 IMP method_imp ; // 方法实现
 }
```
`objc_method_description`定义了一个`Objective-C`方法，其定义如下：

```
struct objc_method_description { 
SEL name; 
char *types; }
```
方法列表

```
// 调用指定方法的实现 
id method_invoke ( id receiver, Method m, ... ); 返回的是实际实现的返回值。参数receiver不能为空。这个方法的效率会比method_getImplementation和method_getName更快

// 调用返回一个数据结构的方法的实现 
void method_invoke_stret ( id receiver, Method m, ... ); 

// 获取方法名 
SEL method_getName ( Method m ); 返回的是一个SEL。如果想获取方法名的C字符串，可以使用sel_getName(method_getName(method))

// 返回方法的实现
 IMP method_getImplementation ( Method m ); 

// 获取描述方法参数和返回值类型的字符串 
const char * method_getTypeEncoding ( Method m ); 

// 获取方法的返回值类型的字符串 
char * method_copyReturnType ( Method m ); 类型字符串会被拷贝到dst中

// 获取方法的指定位置参数的类型字符串 
char * method_copyArgumentType ( Method m, unsigned int index ); 

// 通过引用返回方法的返回值类型字符串 
void method_getReturnType ( Method m, char *dst, size_t dst_len );

 // 返回方法的参数的个数 
unsigned int method_getNumberOfArguments ( Method m ); 

// 通过引用返回方法指定位置参数的类型字符串 
void method_getArgumentType ( Method m, unsigned int index, char *dst, size_t dst_len ); 

// 返回指定方法的方法描述结构体 
struct objc_method_description * method_getDescription ( Method m ); 

// 设置方法的实现 
IMP method_setImplementation ( Method m, IMP imp );注意该函数返回值是方法之前的实现

 // 交换两个方法的实现 
void method_exchangeImplementations ( Method m1, Method m2 );

选择器相关的操作函数包括：
// 返回给定选择器指定的方法的名称 
const char * sel_getName ( SEL sel ); 

// 在Objective-C Runtime系统中注册一个方法，将方法名映射到一个选择器，并返回这个选择器 
SEL sel_registerName ( const char *str ); //在我们将一个方法添加到类定义时，我们必须在Objective-C Runtime系统中注册一个方法名以获取方法的选择器

// 在Objective-C Runtime系统中注册一个方法 
SEL sel_getUid ( const char *str ); 

// 比较两个选择器 
BOOL sel_isEqual ( SEL lhs, SEL rhs );
```
### 方法调用流程
在`Objective-C`中，消息直到运行时才绑定到方法实现上。编译器会将消息表达式`[receiver message]`转化为一个消息函数的调用，即`objc_msgSend`。这个函数将消息接收者和方法名作为其基础参数，如以下所示：

```
objc_msgSend(receiver, selector)
```
如果消息中还有其它参数，则该方法的形式如下所示

```
objc_msgSend(receiver, selector, arg1, arg2, ...)
```
这个函数完成了动态绑定的所有事情：
首先它找到`selector`对应的方法实现。因为同一个方法可能在不同的类中有不同的实现，所以我们需要依赖于接收者的类来找到的确切的实现。
它调用方法实现，并将接收者对象及方法的所有参数传给它。
最后，它将实现返回的值作为它自己的返回值。

`objc_msgSend`的发送流程：
> 先在Class中的缓存查找`imp`（没缓存则初始化缓存），如果没找到，则向父类的`Class`查找。如果一直查找到根类仍旧没有实现，就走消息转发(`_objc_msgForward`)了。

给`nil`发送消息不会有什么作用，但是返回值有些区别，具体如下：

* 如果方法返回值是 对象，返回`nil`
* 如果方法返回值是 指针类型，其指针大小为小于或者等于sizeof(void*)，float，double，long double 或者 long long 的整型标量
* 如果方法返回值是 结构体，发送给 nil 的消息将返回0。结构体中各个字段的值将都是0。
* 如果方法返回值不是 上述提到的几种情况，那么发送给 nil 的消息的返回值将是未定义的。
消息的关键在于objc_class结构体,这个结构体有两个字段是我们在分发消息的关注的：
指向父类的指针
一个类的方法分发表，即`methodLists`。

当我们创建一个新对象时，先为其分配内存，并初始化其成员变量。其中`isa`指针也会被初始化，让对象可以访问类及类的继承体系。

#### 消息发送流程
当消息发送给一个对象时：

1. `objc_msgSend`通过对象的isa指针获取到类的结构体
2. 然后在结构体中方法分发表里面查找方法的`selector`。
3. 如果没有找到`selector`，则通过`objc_msgSend`结构体中的`指向父类的指针`找到其父类，并在父类的分发表里面查找方法的selector。
4. 依此，会一直沿着类的继承体系到达`NSObject`类。一旦定位到`selector`，函数会就获取到了实现的入口点，并传入相应的参数来执行方法的具体实现。
5. 如果最后没有定位到selector，则会走消息转发流程

为了加速消息的处理，运行时系统缓存使用过的selector及对应的方法的地址

`objc_msgSend` 有两个隐藏参数：

* 消息接收对象
* 方法的`selector`

这两个参数为方法的实现提供了调用者的信息。之所以说是隐藏的，是因为它们在定义方法的源代码中没有声明。它们是在编译期被插入实现代码的。

虽然这些参数没有显示声明，但在代码中仍然可以引用它们。我们可以使用`self`来引用接收者对象，使用`_cmd`来引用选择器。如下代码所示：

```
- strange {
 id target = getTheReceiver(); 
SEL method = getTheMethod(); 
if ( target == self || method == _cmd ) 
      return nil;
 return [target performSelector:method]; 
}
```
这两个参数我们用得比较多的是`self`，`_cmd`在实际中用得比较少。

`Runtime`中方法的动态绑定让我们写代码时更具灵活性，如我们可以把消息转发给我们想要的对象，或者随意交换一个方法的实现等。不过灵活性的提 升也带来了性能上的一些损耗。毕竟我们需要去查找方法的实现，而不像函数调用来得那么直接。当然，方法的缓存一定程度上解决了这一问题。

我们上面提到过，如果想要避开这种动态绑定方式，我们可以获取方法实现的地址，然后像调用函数一样来直接调用它。特别是当我们需要在一个循环内频繁地调用一个特定的方法时，通过这种方式可以提高程序的性能。

`NSObject`类提供了`methodForSelector:`方法，让我们可以获取到方法的指针，然后通过这个指针来调用实现代码。我们需要将`methodForSelector:`返回的指针转换为合适的函数类型，函数参数和返回值都需要匹配上。

### 消息转发
当一个对象能接收一个消息时，就会走正常的方法调用流程。但如果一个对象无法接收指定消息时，又会发生什么事呢？默认情况下，如果是以 `[object message]`的方式调用方法，如果`object`无法响应`message`消息时，编译器会报错。但如果是以`perform`…的形式来调用，则需要等到运 行时才能确定`object`是否能接收`message`消息。如果不能，则程序崩溃。
通常，当我们不能确定一个对象是否能接收某个消息时，会先调用`respondsToSelector:`来判断一下。如下代码所示：

```
if ([self respondsToSelector:@selector(method)]) {
    [self performSelector:@selector(method)];
}
```
当一个对象无法接收某一消息时，就会启动所谓”消息转发`(message forwarding)`“机制，通过这一机制，我们可以告诉对象如何处理未知的消息。默认情况下，对象接收到未知的消息，会导致程序崩溃，通过控制台，我们可以看到以下异常信息：

```
-[SUTRuntimeMethod method]: unrecognized selector sent to instance 0x100111940 *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[SUTRuntimeMethod method]: unrecognized selector sent to instance 0x100111940'
```
这段异常信息实际上是由`NSObject`的”`doesNotRecognizeSelector`”方法抛出的。不过，我们可以采取一些措施，让我们的程序执行特定的逻辑，而避免程序的崩溃

消息转发机制基本上分为三个步骤：
> 1. 动态方法解析
> 2. 备用接收者
> 3. 完整转发

#### 1、动态方法解析
对象在接收到未知的消息时，首先会调用所属类的类方法`+resolveInstanceMethod:`(实例方法)或 者`+resolveClassMethod:`(类方法)。在这个方法中，我们有机会为该未知消息新增一个”处理方法”“。不过使用该方法的前提是我们已经 实现了该”处理方法”，只需要在运行时通过`class_addMethod`函数动态添加到类里面就可以了。如下代码所示

```
void functionForMethod1(id self, SEL _cmd) { 
NSLog(@"%@, %p", self, _cmd);
 } 
+ (BOOL)resolveInstanceMethod:(SEL)sel { 
NSString *selectorString = NSStringFromSelector(sel); 
if ([selectorString isEqualToString:@"method1"]) { class_addMethod(self.class, @selector(method1), (IMP)functionForMethod1, "@:"); 
} 
return [super resolveInstanceMethod:sel]; 
}
```
#### 2、备用接收者
如果在上一步无法处理消息，则Runtime会继续调以下方法

```
- (id)forwardingTargetForSelector:(SEL)aSelector
```
如果一个对象实现了这个方法，并返回一个非`nil`的结果，则这个对象会作为消息的新接收者，且消息会被分发到这个对象。当然这个对象不能是`self`自身，否则就是出现无限循环。当然，如果我们没有指定相应的对象来处理`aSelector`，则应该调用父类的实现来返回结果。
使用这个方法通常是在对象内部，可能还有一系列其它对象能处理该消息，我们便可借这些对象来处理消息并返回，这样在对象外部看来，还是由该对象亲自处理了这一消息。如下代码所示：

```
@interface SUTRuntimeMethodHelper : NSObject
- (void)method2; 
@end 
@implementation SUTRuntimeMethodHelper 
- (void)method2 { NSLog(@"%@, %p", self, _cmd); 
} 
@end 
#pragma mark - 
@interface SUTRuntimeMethod () { 
SUTRuntimeMethodHelper *_helper; 
} 
@end
 @implementation SUTRuntimeMethod 
+ (instancetype)object { 
return [[self alloc] init]; 
} 
- (instancetype)init {
 self = [super init];
 if (self != nil) { 
_helper = [[SUTRuntimeMethodHelper alloc] init];
 } 
return self; 
} 
- (void)test {
 [self performSelector:@selector(method2)]; 
}
 - (id)forwardingTargetForSelector:(SEL)aSelector { NSLog(@"forwardingTargetForSelector"); 
NSString *selectorString = NSStringFromSelector(aSelector);
 // 将消息转发给_helper来处理 
if ([selectorString isEqualToString:@"method2"]) {
 return _helper; 
} 
return [super forwardingTargetForSelector:aSelector];
 }
 @end
```
这一步合适于我们只想将消息转发到另一个能处理该消息的对象上。但这一步无法对消息进行处理，如操作消息的参数和返回值。

#### 3、完整消息转发
![消息转发流程](https://upload-images.jianshu.io/upload_images/1840444-4a4f29f7430be66f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果在上一步还不能处理未知消息，则唯一能做的就是启用完整的消息转发机制了。此时会调用以下方法：

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
```
运行时系统会在这一步给消息接收者最后一次机会将消息转发给其它对象。对象会创建一个表示消息的`NSInvocation`对象，把与尚未处理的消息 有关的全部细节都封装在`anInvocation`中，包括`selector`，目标`(target)`和参数。我们可以在`forwardInvocation` 方法中选择将消息转发给其它对象。

`forwardInvocation:`方法的实现有两个任务：
定位可以响应封装在`anInvocation`中的消息的对象。这个对象不需要能处理所有未知消息。
使用`anInvocation`作为参数，将消息发送到选中的对象。`anInvocation`将会保留调用结果，运行时系统会提取这一结果并将其发送到消息的原始发送者。

不过，在这个方法中我们可以实现一些更复杂的功能，我们可以对消息的内容进行修改，比如追回一个参数等，然后再去触发消息。另外，若发现某个消息不应由本类处理，则应调用父类的同名方法，以便继承体系中的每个类都有机会处理此调用请求。
还有一个很重要的问题，我们必须重写以下方法：

```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
```
消息转发机制使用从这个方法中获取的信息来创建`NSInvocation`对象。因此我们必须重写这个方法，为给定的`selector`提供一个合适的方法签名。

```
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector { NSMethodSignature *signature = [super methodSignatureForSelector:aSelector];
 if (!signature) { 
if ([SUTRuntimeMethodHelper instancesRespondToSelector:aSelector]) { signature = [SUTRuntimeMethodHelper instanceMethodSignatureForSelector:aSelector]; 
} 
} 
return signature; 
} 
- (void)forwardInvocation:(NSInvocation *)anInvocation {
 if ([SUTRuntimeMethodHelper instancesRespondToSelector:anInvocation.selector]) {
 [anInvocation invokeWithTarget:_helper];
 }
 }
```
`NSObject`的`forwardInvocation:`方法实现只是简单调用了`doesNotRecognizeSelector:`方法，它不会转发任何消息。这样，如果不在以上所述的三个步骤中处理未知消息，则会引发一个异常。

从某种意义上来讲，`forwardInvocation:`就像一个未知消息的分发中心，将这些未知的消息转发给其它对象。或者也可以像一个运输站一样将所有未知消息都发送给同一个接收对象。这取决于具体的实现。

#### 4、消息转发与多重继承
前面第二和第三步，通过这两个方法我们可以允许一个对象与其它对象建立关系，以处理某些未知消息，而表面上看仍然是该对象在处理消息。
通过这 种关系，我们可以模拟“多重继承”的某些特性，让对象可以“继承”其它对象的特性来处理一些事情。不过，这两者间有一个重要的区别：多重继承将不同的功能 集成到一个对象中，它会让对象变得过大，涉及的东西过多；而消息转发将功能分解到独立的小的对象中，并通过某种方式将这些对象连接起来，并做相应的消息转 发。不过消息转发虽然类似于继承，但`NSObject`的一些方法还是能区分两者。如`respondsToSelector:`和`isKindOfClass:`只能用于继承体系，而不能用于转发链。便如果我们想让这种消息转发看起来像是继承，则可以重写这些方法，如以下代码所示：

```
- (BOOL)respondsToSelector:(SEL)aSelector { 
if ( [super respondsToSelector:aSelector] ) 
return YES; 
else { /* Here, test whether the aSelector message can * * be forwarded to another object and whether that * * object can respond to it. Return YES if it can. */ 
} 
return NO; 
}
```
#### 5、避免消息转发的办法
在消息转发三个过程中，未知消息的处理过程越往后，代价越大；一般我们可以这么做 尽可能避免消息转发，可以这么做：
调用`delegate` 方法前检查方法是否实现`(respondsToSelector:)`, 只有实现了`(respondsToSelector:返回YES)` ，才去真正调用delegate 方法。

```
if([self.delegate respondsToSelector: @selector(sayHello)]) {
    [self.delegate sayHello];
}
```
直接调用方法，少用`performSelector:`；因为在直接调用方法时，编译自动校验，如果方法不存在，编译器会直接报错；而使用`performSelector:`的话一定是在运行时候才能发现，如果此方法不存在就会崩溃。

```
//直接使用方法调用,少使用performSelector
[dog sayHello];
// [dog performSelector:@selector(sayHello) withObject:nil];
```
使用`performSelector:`，最好先判断方法是否实现`(respondsToSelector:)`，只有实现了`(respondsToSelector:返回YES)` ，才去调用`performSelector：`方法。

```
//respondsToSelector:和performSelector:组合使用
    if ([dog respondsToSelector:@selector(sayHello)])         {
    [dog performSelector:@selector(sayHello)];
 }

强制类型转换，先判断对象是否属于强制转换后的类
if([data isKindOfClass:[NSDictionary class]]){
  //
}
```

