# 有效降低 APP 包的大小
降低包大小需要从两方面着手
### 可执行文件
#### 编译器优化
* `Strip Linked Product、Make Strings Read-Only、Symbols Hidden by Default` 设置为 `YES`
* 去掉异常支持，`Enable C++ Exceptions、Enable Objective-C Exceptions` 设置为 `NO`， `Other C Flags` 添加 `-fno-exceptions`

#### 利用 AppCode 检测未使用的代码
菜单栏 -> Code -> Inspect Code
#### 编写LLVM插件检测出重复代码、未被调用的代码

### 资源
> 资源包括 图片、音频、视频 等

* 优化的方式可以对资源进行无损的压缩
* 去除没有用到的资源： https://github.com/tinymind/LSUnusedResources


