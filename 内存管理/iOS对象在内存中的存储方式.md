# iOS对象在内存中的存储方式
由以下三种方式相结合：
### TaggedPointer指针
我们知道以前在32位CPU下，一个对象所占用的内存是4个字节，如今在64位CPU中，一个对象所占用的内存是8个字节。那么像`NSNumber、NSDate、NSString`等一些小对象明明可以用4个字节存储的，那么在64位CPU下用8个字节存储就显得有些烂费资源了。
![](https://upload-images.jianshu.io/upload_images/1840444-50b6c5f50f1b1f73.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
所以苹果引入`TaggedPointer`的概念。直接指针指向的内容直接放在了指针变量的内存地址。
![](https://upload-images.jianshu.io/upload_images/1840444-96380a9da548b81d.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
例如：

```
NSMutableString *str = [NSMutableString stringWithString:@"1"];
    for (int i = 2; i < 20; i ++) {
        NSNumber *number = @([str longLongValue]);
        NSLog(@"%@, %p",[number class],number);
        [str appendString:[NSString stringWithFormat:@"%d",i]];
    }
```
输出：

```
__NSCFNumber, 0xb000000000000013
__NSCFNumber, 0xb0000000000000c3
__NSCFNumber, 0xb0000000000007b3
......
__NSCFNumber, 0xb007048860f3a383
__NSCFNumber, 0xb2bdc545df2bded3
__NSCFNumber, 0x600000039f00
__NSCFNumber, 0x600000039f00
```
数值按1，12，123...依次递增，看对象内存地址可以知道

| 位 | 说明 |
| --- | --- |
| 最低4位是3 | 这个用于标记是long（float则为4，Int为2，double为5） |
| 最高4位是b | 表示是NSNumber类型 |
| 中间56位 | 用来存储数值本身内容 |


当存储用的数值超过56位存储上限的时候，那么`NSNumber`才会用真正的64位内存地址存储数值，然后用指针指向该内存地址。（如果数值长度超过64位，那么就`crash`）。因为`Tagged Pointed`不是一个真正的对象，所以其没有`isa`，所以在代码中应该要避免直接访问`isa`指针。
再来看看常见的`Tagged Pointer String`

```
NSMutableString *str = [NSMutableString stringWithString:@"1"];
    for (int i = 2; i < 17; i ++) {
        NSString *str1 = [[str mutableCopy] copy];
        NSLog(@"%@, %p",[str1 class],str1);
        [str appendString:[NSString stringWithFormat:@"%d",i]];
    }
```
输出：   

```
NSTaggedPointerString, 0xa000000000000311
NSTaggedPointerString, 0xa000000000032312
NSTaggedPointerString, 0xa000000003332313
...... 
NSTaggedPointerString, 0xa007a87dcaecc2a8
NSTaggedPointerString, 0xa1ea1f72bb30ab19
__NSCFString, 0x60400022e100
__NSCFString, 0x60400022de20
__NSCFString, 0x604000253050
```

| 位 | 说明 |
| --- | --- |
| 最低4位 | 表示字符串长度 |
| 最高4位 | 表示类型 |
| 中间56位 | 用来存储字符的ASCII码值 |

英文字符串用`ASCII`码存储，当字符串内存长度超过了56位的时候，`Tagged Pointer`并没有立即用指针转向，而是用了一种[算法编码](http://www.cocoachina.com/ios/20150918/13449.html)，把字符串长度进行压缩存储，当这个算法压缩的数据长度超过56位了才使用指针指向。**当字符串中有中文或者特殊字符时只采用指针存储**。

### NONPOINTER_ISA（64位系统）

这个是对象的`isa`指针。它是指向对象本身的。我们可以通过`isa`指针找到这个对象


| 位 | 名称 | 说明 |
| --- | --- | --- |
| 第1位 | indexed | 0 或 1 ，0代表是纯地址型 isa 指针，1是 NONPOINTER_ISA 指针 |
| 第2位 | has_assoc | 是否有关联对象 |
| 第3位 | has_cxx_dtor | 代表是否有C++代码和ARC析构函数 |
| 中间30位 | shiftcls | 代表指向的内存地址 |
| 接下来9位 | magic | 值为0xd2，调试器使用它来区分实际对象和未初始化的垃圾 |
| 接下来1位 | weakly_referenced | 是否有弱引用的标记 |
| 接下来1位 | deallocating | 是否有delloc的标记 |
| 接下来1位 | has_sidetable_rc | 是否对象的引用计数太大，无法内联存储 |
| 最后19位 | extra_rc | 对象的retain count计数大于1。(例如，如果extra_rc是5，那么对象的实际retain count就是6) |
### 散列表（引用计数表、weak表）
1. `SideTables`表在非嵌入式的64位系统中，有64张`SideTable`表
2. 每一张`SideTable`主要是由三部分组成。自旋锁、引用计数表、弱引用表。
3. 全局的引用计数之所以不存在同一张表中，是为了避免资源竞争，解决效率的问题。
4. 引用计数表 中引入了分离锁的概念将一张表分拆成多个部分，对他们分别加锁可以实现并发操作，提升执行效率

更详细内容查看[这里](https://www.jianshu.com/p/8577286af88e)


