# Objective-C中Block精解
## C语言函数指针
`int func(int count)`
正常调用:`int result = func(10)`
函数指针调用:

```
int (*funcPtr)(int) = &func;
int result = (*funcPtr)(10);
```
看到这里是不是觉得函数指针调用有点像Block的味道了
## typedef给类型取一个别名
`typedef int (^blk)(int)`
给这个`block`类型取一个`blk`的名称
## Block的实质
其实就是结构体`__main_block_impl_0`指针

```
void (^blk)(void) = ^{printf("Block");};
```
`Block`结构体

```
struct __block_impl {
      void *isa;
      int  Flags;
      int  Reserved;
      void *FuncPtr;//函数指针
}
struct __main_block_impl_0 {
      struct __block_impl  impl;
      struct __main_block_desc_0 * Desc;
      //结构体初始化
      __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0){
              impl.isa = &_NSConcreteStackBlock;
              impl.Flags = flags;
              impl.FuncPtr = fp;
              Desc = desc;
     }
}
```
注意这里也有个`isa`指针，我们知道`OC`的对象也有`isa`指针，说明`Block`其实也是一个对象

```
{printf("Block");};
实际上是下面这个函数
static void __main_block_func_0(struct __main_block_impl_0 *__cself){
    printf("Block");
}
```
`Block`类型变量

```
blk= &__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA);
```

```
static struct __main_block_desc_0  __main_block_desc_0_DATA = {
      0,
      sizeof(struct __main_block_impl_0)
};
__main_block_desc_0_DATA即是结构体实例__main_block_impl_0的大小
```
## Block的调用实质就是调用这个结构体指针所指向的函数blk();

```
（*blk->impl.FuncPtr）(blk)
```
## 截获变量
### 自动变量（局部变量）【在block函数中不能改变其值】
被截获的自动变量被保存到`__main_block_impl_0`结构体中，作为它的成员变量，如：

```
struct __main_block_impl_0 {
      struct __block_impl  impl;
      struct __main_block_desc_0 * Desc;
      const char *fmt;
      int val;//被截获的变量
      //结构体初始化
      __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0){
              impl.isa = &_NSConcreteStackBlock;
              impl.Flags = flags;
              impl.FuncPtr = fp;
              Desc = desc;
              //截获的值在这里初始化
              fmt = "block";
              val = 10;
     }
}
```
`block`函数转换为以下函数：

```
{printf(fmt, val);};
```
实际上是一个函数

```
static void __main_block_func_0(struct __main_block_impl_0 *__cself){
    const char *fmt = __cself->fmt;
    int val = __cself->val;
    printf(fmt, val);
}
```
### 静态变量【由于截获的是该变量的地址，所以在block函数中可以做任何操作】
与自动变量不同，被截获的静态变量的地址被保存到`__main_block_impl_0`结构体中，作为它的成员变量，如：

```
static int static_val = 3;
struct __main_block_impl_0 {
      struct __block_impl  impl;
      struct __main_block_desc_0 * Desc;
      int *static_val;//静态变量的地址
      //结构体初始化
      __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0, int *static_val){
              impl.isa = &_NSConcreteStackBlock;
              impl.Flags = flags;
              impl.FuncPtr = fp;
              Desc = desc;
              //截获的值在这里初始化
              static_val = static_val;
     }
}
```
`block`函数转换为以下函数：

```
static int static_val = 3;
{ static_val = 10;};
```
实际上是一个函数

```
static void __main_block_func_0(struct __main_block_impl_0 *__cself){
    int *static_val = __cself->static_val;
    *static_val = 10;
}
```
### 静态全局变量
变量存储在全局区，不归`block`管不会被保存到`block`结构体中。在`block`函数中做任何操作都行
### 全局变量
变量存储在全局区，不归`block`管不会被保存到`block`结构体中。在`block`函数中做任何操作都行
## 截获对象
截获对象时`Block`结构体会多出一个`__strong`修饰符的对象如：

```
struct __main_block_impl_0 {
   struct __block_impl impl;
   struct __main_block_desc_0 *Desc;
   id __strong array;//被截获的对象
   //结构体初始化
      __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, id __strong _array,int flags=0,){
              impl.isa = &_NSConcreteStackBlock;
              impl.Flags = flags;
              impl.FuncPtr = fp;
              Desc = desc;
     }
}
```
此外还会多两个函数：
当`Block`从栈复制到堆时调用

```
static void __main_block_func_0(struct __main_block_impl_0 *__cself, id obj);
```
当`Block`废弃时调用

```
static void __main_block_dispose_0(struct __main_block_impl_0 *src);
```
## __block说明符
自动变量在`block`函数中是不能改变其值的，那如果要改变其值该怎么办？
在变量前加个`__block`修饰符就行了
`__block`类似于`static`，用于指定该变量设置到哪个存储域中，如：

```
__block int val = 10;
```
那么加了`__block`修饰符的自动变量将转换为一个结构体：

```
__Block_byref_val_0 val = {
      0,
      &val,
      0,
      sizeof(__Block_byref_val_0),
      10
}
struct __Block_byref_val_0 {
      void *__isa;
      __Block_byref_val_0 *__forwarding;//指向结构体自身的指针
      int flags;
      int __size;
      int val;//变量的值
};
```
这样一来就跟静态变量一样，只要把`__Block_byref_val_0`的地址保存到`__main_block_impl_0`结构体中，作为它的成员变量，如：

```
struct __main_block_impl_0 {
      struct __block_impl  impl;
      struct __main_block_desc_0 * Desc;
      __Block_byref_val_0 *val;
      //结构体初始化
      __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0, __Block_byref_val_0 *val){
              impl.isa = &_NSConcreteStackBlock;
              impl.Flags = flags;
              impl.FuncPtr = fp;
              Desc = desc;
              //截获的值在这里初始化
              val = val;
     }
}
```
`block`函数转换为以下函数：

```
__block int val = 10;
{ val = 10;};
```
实际上是一个函数

```
static void __main_block_func_0(struct __main_block_impl_0 *__cself){
    __Block_byref_val_0 *val = __cself->val;
    val ->__forwarding->val = 10;
}
```
## Block存储域
### __NSConcreteGlobalBlock
全局`block`，存放在`data`区，以下为全局`block`：
1. 记述全局变量的地方有`Block`语法时
2. `Block`语法的表达式中不使用应截获的自动变量时
### __NSConcreteStackBlock
栈`block`，存放在栈区，作用域结束时，系统自动回收，除了以上全局变量的情况之外，其余的`block`都是栈`block`
### __NSConcreteMallocBlock
堆`block`，存放在堆区，由程序猿手动释放内存。这是为了解决`block`夸作用域使用，将栈中的`block`拷贝到堆上，大部分情况，编译器会自动拷贝。有时也要自己手动拷贝
#### 什么情况下Block会从栈复制到堆
* 调用`Block`的`copy`方法
* `Block`作为函数返回值返回时
* 将`Block`赋值给附有`__strong`修饰符`id`类型的类或`Block`类型成员变量时
* 在方法名中含有`usingBlock`的`cocoa`框架方法或`Grand Central Dispatch`的`API`中传递`Block`时
## __block类型变量的存储域
使用`__block`变量，当`block`从栈复制到堆时，`__block`变量也会一并复制到堆上，并被`Block`持有，当`Block`被废弃时`__block`变量也废弃


