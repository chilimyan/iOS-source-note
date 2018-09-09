# 一个 NSObject 对象占用多少内存空间

```
NSObject *obj = [[NSObject alloc]init];
```
一个`NSobject`对象其实就是一个`Nsobject_Impl`结构体。将其转换为C++代码可以看到具体实现

```
struct Nsobject_Impl {
  Class isa;
}
```
![](https://upload-images.jianshu.io/upload_images/1840444-57a1ec3f7278b67b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
#import <objc/runtime.h>
#import <Foundation/Foundation.h>
#import <malloc/malloc.h>
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        NSObject *obj = [[NSObject alloc]init];
     //获得NSObject 类的实例对象的大小
  NSLog(@"%zd",class_getInstanceSize([NSObject class])  );
        //获取obj对象指针获取的大小
        NSLog(@"%zd",malloc_size((__bridge const void *)obj));
    }
    return 0;
}
```
输出结果分别是8 和 16 
系统分配了16个字节给`NSObject`对象（通过`malloc_size`获得）
但`NSObject`对象内部只使用了8个字节的空间（64bit环境下通过`class_getInstanceSize`获得）


