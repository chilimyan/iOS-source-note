# iOSApp编译过程及签名
#### App编译
`OC`和`Swift`一样都是采用`Clang`作为编译器前端，`LLVM`作为编译器后端。
![](https://upload-images.jianshu.io/upload_images/1840444-de4ade8fbaae5e24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`Clang`编译器前端主要是进行语法分析，语义分析，生成中间代码，在这一过程中会检查出一些错误或者警告。

`LVVM`编译器后端主要把代码生成机器语言，针对不同的架构，比如`arm64`等生成不同的机器语言。

当通过`xcodebuild`命令编译时，会创建编译后的文件`name.app`，`.m`文件编译成`.o`文件。生成`dsYM`文件，这个文件存储了16进制函数地址映射。打完包后应该保留这个文件。因为`App`使用时`crash`，会抛出奔溃信息，可以使用工具收集这些奔溃信息，通过这些奔溃的调用栈结合`dsym`文件，就可以通过地址映射到具体函数位置，从而知道哪一行代码闪退。

### 代码签名
编译完之后就需要进行代码签名了。签名是使用`codesign`命令完成。一旦进行签名，那么编译后的那个`.app`包里面的所有文件都会进行签名。为了达到给所有文件签名的目的，签名的过程会在包中新建一个`_CodeSignatue/CodeResources`文件，这个文件存储了包中被签名的所有文件的签名。

#### 授权机制Entitlements
它决定系统哪些资源允许被该应用使用，它是一个`entitlements.plist`文件。沙盒机制也是由于这个授权机制建立起来的。在签名的时候需要用到这个文件。
#### 配置文件provisioning profiles
在`.app`包里面可以看到一个`embedded.mobileprovision`文件，这个配置文件将签名，授权和沙盒联系起来。这个配置文件是一个信息的集合。

签名完成之后压缩导出ipa包
### 应用重签名
上面说到已经完整打出一个`ipa`包了。那么我们要对这个包重新签名该怎么做呢。首先解压`ipa`包，得到`.app`文件。上面说到`.app`里面的所有文件都已经被签名了。那么我们要重新签名首先要有`entitlements.plist`授权机制文件，因为签名的时候要把这个文件传过去。而授权机制信息又在`embedded.mobileprovision`文件中。因为刚说到`embedded.mobileprovision`配置文件是一个信息集合，将签名授权沙盒联系在一起。那么我们首先要从`embedded.mobileprovision`配置文件中拿到授权信息，然后生成一个`entitlements.plist`授权机制文件。然后我们只要使用`codesign -f -s`命令把`entitlements.plist`文件和证书信息传过去生成新的签名，替换掉原来的`_CodeSignatue/CodeResources`文件即可。
### 编译优化
#### 编译器指令 __attribute__
```
语法格式：__attribute__((attribute-list))是一个高级的编译器指令。它允许开发者指定更多的编译检查和编译优化。//如果没有使用返回值，编译的时候给出警告。比如常用的有以下几种：
//弃用API
#define __deprecated  __attribute__((deprecated))
//带描述信息的弃用
#define __deprecated  __msg(_msg)__attribute__((deprecated(_msg)))
//遇到__unavailable的变量/方法，编译器直接抛出Error 
#define __unavailable   __attribute__((unavailable)) 
//告诉编译器，即使这个变量/方法 没被使用，也不要抛出警告
#define __unused    __attribute__((unused)) 
//和__unused相反
#define __used      __attribute__((used)) 
//Clang警告处理，忽略警告
#pragma clang diagnostic push 
#pragma clang diagnostic ignored "-Wundeclared-selector" 
///代码 
#pragma clang diagnostic pop  
```
#### 预处理
就是在编译之前的处理。预处理能够让你定义编译器变量，实现条件编译。比如

```
#ifdef DEBUG 
//... 
#else 
//... 
#endif 
```
#### 插入编译期脚本
比如`cocoaPods`，其原理就是通过修改`xcodeproject`，然后配置编译期脚本，来保证三方库能够正确编译链接。
#### 通过前向声明forward declaration
当要引入一个类时在.h文件中使用`@class`然后在`.m`文件中`#import`。可以加快编译速度。
#### 打包工具类
将常用工具类进行打包成`.framework`或者`.a`。这样在编译的时候就不需要重新编译这部分代码。
将常用的头文件放到`.pch`预编译文件中。

