# Type Encoding类型编码
作为对`Runtime`的补充，编译器将每个方法的返回值和参数类型编码为一个字符串，并将其与方法的`selector`关联在一起。这种编码方案在其它情况下也是非常有用的，因此我们可以使用`@encode`编译器指令来获取它。当给定一个类型时，`@encode`返回这个类型的字符串编码。这些类型可以是诸如`int`、指针这样的基本类型，也可以是结构体、类等类型。事实上，任何可以作为`sizeof()`操作参数的类型都可以用于`@encode()`。
在`Objective-C Runtime Programming Guide`中的`Type Encoding`一节中，列出了`Objective-C`中所有的类型编码。需要注意的是这些类型很多是与我们用于存档和分发的编码类型是相同的。但有一些不能在存档时使用。
注：`Objective-C`不支持`long double`类型。`@encode(long double)`返回`d`，与`double`是一样的。
一个数组的类型编码位于方括号中；其中包含数组元素的个数及元素类型。如以下示例：
`float a[] = {1.0, 2.0, 3.0};
NSLog(@"array encoding type: %s", @encode(typeof(a)));`
另外，还有些编码类型，`@encode`虽然不会直接返回它们，但它们可以作为协议中声明的方法的类型限定符。可以参考`Type Encoding`。
对于属性而言，还会有一些特殊的类型编码，以表明属性是只读、拷贝、`retain`等等，详情可以参考`Property Type String`。

[苹果官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)

