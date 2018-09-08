# Runtime-分类和协议
`Objective-C`中的分类允许我们通过给一个类添加方法来扩充它（但是通过`category`不能添加新的实例变量）
`Objective-C`中的协议是普遍存在的接口定义方式，即在一个类中通过`@protocol`定义接口，在另外类中实现接口，这种接口定义方式也成为“`delegation`”模式，`@protocol`声明了可以呗其他任何方法类实现的方法，协议仅仅是定义一个接口，而由其他的类去负责实现。
### 基础数据类型
`Category`是表示一个指向分类的结构体的指针，其定义如下

```
typedef struct objc_category *Category; 
struct objc_category { 
char *category_name ; // 分类名 
char *class_name ; // 分类所属的类名 
struct objc_method_list *instance_methods ; // 实例方法列表 
struct objc_method_list *class_methods ; // 类方法列表 
struct objc_protocol_list *protocols ; // 分类所实现的协议列表 
}
```
这个结构体主要包含了分类定义的实例方法与类方法，其中`instance_methods`列表是`objc_class`中方法列表的一个子集，而`class_methods`列表是元类方法列表的一个子集。
`Protocol`的定义如下

```
typedef struct objc_object Protocol;
```

```
// 返回指定的协议 
Protocol * objc_getProtocol ( const char *name ); //需要注意的是如果仅仅是声明了一个协议，而未在任何类中实现这个协议，则该函数返回的是nil。

// 获取运行时所知道的所有协议的数组 
Protocol ** objc_copyProtocolList ( unsigned int *outCount ); //获取到的数组需要使用free来释放

// 创建新的协议实例 
Protocol * objc_allocateProtocol ( const char *name ); //如果同名的协议已经存在，则返回nil

// 在运行时中注册新创建的协议 
void objc_registerProtocol ( Protocol *proto ); //创建一个新的协议后，必须调用该函数以在运行时中注册新的协议。协议注册后便可以使用，但不能再做修改，即注册完后不能再向协议添加方法或协议

// 为协议添加方法 
void protocol_addMethodDescription ( Protocol *proto, SEL name, const char *types, BOOL isRequiredMethod, BOOL isInstanceMethod ); 

// 添加一个已注册的协议到协议中 
void protocol_addProtocol ( Protocol *proto, Protocol *addition ); 

// 为协议添加属性 
void protocol_addProperty ( Protocol *proto, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount, BOOL isRequiredProperty, BOOL isInstanceProperty ); 

// 返回协议名 
const char * protocol_getName ( Protocol *p ); 

// 测试两个协议是否相等 
BOOL protocol_isEqual ( Protocol *proto, Protocol *other ); 

// 获取协议中指定条件的方法的方法描述数组 
struct objc_method_description * protocol_copyMethodDescriptionList ( Protocol *p, BOOL isRequiredMethod, BOOL isInstanceMethod, unsigned int *outCount );

 // 获取协议中指定方法的方法描述 
struct objc_method_description protocol_getMethodDescription ( Protocol *p, SEL aSel, BOOL isRequiredMethod, BOOL isInstanceMethod ); 

// 获取协议中的属性列表 
objc_property_t * protocol_copyPropertyList ( Protocol *proto, unsigned int *outCount ); 

// 获取协议的指定属性 
objc_property_t protocol_getProperty ( Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty ); 

// 获取协议采用的协议 
Protocol ** protocol_copyProtocolList ( Protocol *proto, unsigned int *outCount );

 // 查看协议是否采用了另一个协议 
BOOL protocol_conformsToProtocol ( Protocol *proto, Protocol *other );

需要强调的是，协议一旦注册后就不可再修改，即无法再通过调用protocol_addMethodDescription、protocol_addProtocol和protocol_addProperty往协议中添加方法等。
对于协议，我们可以动态地创建协议，并向其添加方法、属性及继承的协议，并在运行时动态地获取这些信息
```

