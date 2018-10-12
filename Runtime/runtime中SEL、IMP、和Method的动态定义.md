# runtime中SEL、IMP、和Method的动态定义
他们的存储结构如下：

```
typedef struct objc_method *Method;
typedef struct objc_selector *SEL;
typedef void (*IMP)(void /* id, SEL, ... */ ); 
//方法描述
struct objc_method_description {
	SEL name;               //方法名称
	char *types;            //参数类型字符串
};
//以下代码是 ObjC2.0 之前method的定义
struct objc_method {
    SEL method_name;
    char *method_types;
    IMP method_imp;
}
```
* SEL selector的简写,俗称方法选择器,实质存储的是方法的名称
* IMP implement的简写,俗称方法实现,看源码得知它就是一个函数指针
* Method 对上述两者的一个包装结构.

实践，动态创建一个类并为其添加方法。

```
//创建继承自NSObject类的People类
Class People = objc_allocateClassPair([NSObject class], "People", 0);
//将People类注册到runtime中
objc_registerClassPair(People);
//注册test: 方法选择器
SEL sel = sel_registerName("test:");
//函数实现
IMP imp = imp_implementationWithBlock(^(id this,id args,...){
    NSLog(@"方法的调用者为 %@",this);
    NSLog(@"参数为 %@",args);
    return @"返回值测试";
});
//向People类中添加 test:方法;函数签名为@@:@,
//    第一个@表示返回值类型为id,
//    第二个@表示的是函数的调用者类型,
//    第三个:表示 SEL
//    第四个@表示需要一个id类型的参数
class_addMethod(People, sel, imp, "@@:@");
//替换People从NSObject类中继承而来的description方法
class_replaceMethod(People,
                    @selector(description),
                    imp_implementationWithBlock(^NSString*(id this,...){
  return @"我是Person类的对象";}),
                    "@@:");
//完成 [[People alloc]init];
id p1 = objc_msgSend(objc_msgSend(People, @selector(alloc)),@selector(init));
//调用p1的sel选择器的方法,并传递@"???"作为参数
id result = objc_msgSend(p1, sel,@"???");
//输出sel方法的返回值
NSLog(@"sel 方法的返回值为 ： %@",result);
 
//获取People类中实现的方法列表
NSLog(@"输出People类中实现的方法列表");
```

