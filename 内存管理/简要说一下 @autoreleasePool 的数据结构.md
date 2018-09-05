# 简要说一下 @autoreleasePool 的数据结构
简单说是双向链表，每张链表头尾相接，有 `parent`、`child`指针

每创建一个池子，会在首部创建一个哨兵对象,作为标记

最外层池子的顶端会有一个`next`指针。当链表容量满了，就会在链表的顶端，并指向下一张表。
![](https://upload-images.jianshu.io/upload_images/1840444-adae9b18a44cdc49.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

