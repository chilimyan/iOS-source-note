# Category 的实现原理
`Category` 实际上是 `Category_t` 的结构体，在运行时，新添加的方法，都被以倒序插入到原有方法列表的最前面，所以不同的`Category`，添加了同一个方法，执行的实际上是最后一个。比如Nsobject+Tools的分类：

```
static struct _catrgory_t _OBJC_$_CATEGORY_NSObject_$_Tools __attribute__ ((used,section),("__DATA,__objc__const"))
{
    char *category_name                          OBJC2_UNAVAILABLE; // 分类名
  char *class_name                             OBJC2_UNAVAILABLE; // 分类所属的类名
  struct objc_method_list *instance_methods    OBJC2_UNAVAILABLE; // 实例方法列表
  struct objc_method_list *class_methods       OBJC2_UNAVAILABLE; // 类方法列表
  struct objc_protocol_list *protocols         OBJC2_UNAVAILABLE; // 分类所实现的协议列表
}
```
以上看到没有属性列表，所以分类不能添加属性！

`Category` 在刚刚编译完的时候，和原来的类是分开的，只有在程序运行起来后，通过 `Runtime` ，`Category` 和原来的类才会合并到一起。



