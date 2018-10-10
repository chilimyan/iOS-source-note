# Text Kit基本使用
Text Kit主要是有以下三个对象：`NSTextStorage`、`NSLayoutManager`、`NSTextContainer`。
#### NSTextStorage
`NSTextStorage`是`NSMutableAttributedString`的子类，存储用于显示的文本。`NSTextStorage`对象也管理一组`NSLayoutManager`对象，对其字符发生的任何变化，对这些`manager`进行通知，以对文本进行及时的`relay`和`redisplay`。
```
[self.textStorage addLayoutManager:self.layoutManager];
```
#### NSLayoutManager
`NSLayoutManager`调解将`NSTextStorage`中的数据到view显示区域的文本的所有操作过程。它将`Unicode`字符转换成`glyph`，并监督`glyph`在`NSTextContainer`对象所定义的区域中布局的过程。（**需要注意的是**，layout manager,text storage,text container可以从子线程访问，只要app guarantees the access from a single thread
#### NSTextContainer
通常`NSTextContainter`定义文本显示的区域，通常是矩形区域，但可以通过继续它从而指定圆形，五边形等非矩形。`NSTextContainter`不仅定义了文本显示区域的轮廓，也维护了一个贝塞尔路径的数组以定义不可布局文本的区域。因此在布局的时候，文本流会围绕不可布局的路径，以此引入`graphics`及非文本布局的因素。
### 文本属性
Text Kit处理三种文本属性：`字符属性`，`paragraph属性`和`文档属性`。字符属性包括font，颜色和下标等特性，即单个字符或者一组字符的相关特性。`paragraph`是比如缩进，制表符和line spacing等属性。文档属性包括纸张大小，页边距，和视角放大百分比等文档属性。
#### 字符属性
`Attributed`字符串使用`NSDictionary`存储键值对形式的字符属性，`key`是字符串常量表示的属性名，比如`NSFontAttributeName`。
![](https://upload-images.jianshu.io/upload_images/1840444-d1afc198dd01e7e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
概念上，`Attributed string`中的每个字符都对应一个属性`dictionary`，但这组属性通常是针对一连串的字符串的，可能通过方法获取某字符或者某串字符对应的属性。

可以对`Attributed string`应用任意自定义的`key-value`对，可以对`NSTextStorage`对象中的文本应用`NSMutableAttributedString`的`addAttribute:value:range:`方法以添加属性，也可以通过`addAttributes:range:`添加一组自定义属性。要支持自定义的属性，需要继续`NSLayoutManager`，重载`drawGlyphsForGlyphRange:atPoint:`，可以在此方法中先调用super将各字符先绘制出来然后将自己的属性在其上绘制出来，也可以完全自己绘制glyphs。
##### 段落属性
`paragraph`属性影响段落中各行的排列，使用`NSParagraphStyle`类对象封装段落属性，`NSParagraphStyleAttributeName`这个字符串属性指定段落属性对象，`attribute fixing`这一机制会确保在编辑过程中，每个段落只对应一个`NSParagraphStyle`对象。
#### 文档属性
虽然文本系统没有内建机制存储文档属性，但可以通过`NSAttributedString`的类似于`initWithRTF:documentAttributes:`的方法指定从一段RTF或者HTML数据流中提取的文档属性，相反地可以通过比如`RTFFromRange:documentAttributes:`的方法将RTF数据写入。
##### attribute fixing
为了处理编辑过程中产生的不一致，`NSMutableAttributedString`的UIKit扩展定义了`fixAttributesInRange:`方法来修复attachment，字符，段落属性之间的不一致。确保attachments在对应的attachment字符串删除之后不再存在，字符属性只应用于对应font可作用的字符上，且段落属性在整个段落中一致。
### 操作NSTextStorage
要编辑`NSTextStorage`，需要3个步骤，先用`beginEditing`声明更改开始，然后用`replaceCharactersInRange:withString:` 和 `setAttributes:range:`类似的方法添加修改，每次调用这样的方法，`NSTextStorage`都会调用`edited:range:changeInLength:`以跟踪受其影响的字符。完成修改时，调用endEditing，这会导致其delegate调用到`textStorage:willProcessEditing:range:changeInLength:`，并调用其自己的processEditing方法，完成attribute fixing。

修复完attributes后，会调用delegate的t`extStorage:didProcessEditing:range:changeInLength:`方法以给予delegate验证及修改attribute的机会（虽然delegate可以改变字符属性，但它会引起attribute不一致）。最终`NSTextStorage`对所有相关的`layout manager`发送`processEditingForTextStorage:edited:range:changeInLength:invalidatedRange:`，通知这些range属性的变化，便于重新布局。
### 布局文本NSLayoutManager
#### layout 过程
`NSLayoutManager`通过两个步骤控制文本的布局：`glyph生成`和`glyph布局`，这两个步骤都是lazily进行的，在生成了glyph并计算出其位置信息之后，会存起来以备以后使用，并监听`glyph range`的`invalidated`。character range可以有两种方式自动失效：需要glyphs生成时和需要glyphs布局时。当然也可以手动失效glyph或者布局信息，当成layout manager收到获取失效区域glyph或者布局信息的时候，会重新生成glyph或者重新布局。
#### 生成行fragment rectangle
`NSTextContainer`中各行是由其形状及不可布局的路径所决定的，一旦行fragment 矩形与不可布局的路径相交，这些部分的行必须被缩短或者fragmented。如果区域中有gap，则重叠于其上的行必须进行偏移。

`NSLayoutManager`提出一个矩形给某行，并请求text container调整这个矩形，这个矩形可以全部或者部分地超出text container的区域，但其可以变宽也可以变窄，请求调整的方法是`lineFragmentRectForProposedRect:atIndex:writingDirection:remainingRect:`，其返回值是所请求区域中基于文本方向的最大的可用区域，它同时也会返回一个包含所有剩余区域的矩形，比如hole或者gap另一边的空间。

在将文本适配进矩形之后，会做最后一个调整，称为`line fragment padding`，定义每行尾在行fragment矩形中的空白比例，可以通过`lineFragmentPadding`属性更改`padding`，但这并不是一个设置页边距的合适方法，可以设置textview在其父view中的位置，而对于text margin，可以设置`textContainerInset`属性，当然也可以设置段落的indent。

除了`line fragment`矩形本身，`NSLayoutManager`还会返回一个称为`used rectangle`，这是line fragment矩形中实际显示glyph和mark的区域。通常两个矩形都包括`line fragment padding`，和行间空间（比如font的line height和段落的行spacing参数)。然而段落spacing及任何text周围的空白，及center-spaced文本引起的空白，只在行fragment矩形中出现，并不出现在used rectangle中。

#### 指定不可布局的路径
`NSTextContainer`维护着一个贝塞尔路径对象数组代表bounding区域中不可布局的区域。当`lineFragmentRectForProposedRect:atIndex:writingDirection:remainingRect:`所请求的区域与这些路径包围的区域重叠的时候，会返回调整过的区域。
![](https://upload-images.jianshu.io/upload_images/1840444-bbf1d6010b025dbf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 指定多页和多列布局
最简单的情况下，text kit对象都是单个的，即一个text storage object,一个text container，一个layout manager
![](https://upload-images.jianshu.io/upload_images/1840444-46964a039b3170a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
但只有一个container和storage，这种安排下，文本流在textcontainer定义的空间中连续。但page  breaks,多列布局，和更复杂的布局是无法由这种安排满足的。

通过使用多text container，每个联系一个text view，可以实现更复杂的文本布局，比如可以通过这样实现page break
![](https://upload-images.jianshu.io/upload_images/1840444-12d08c0ce177697a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
多列文档的对象结构可以是这样
![](https://upload-images.jianshu.io/upload_images/1840444-728d3249b8c12646.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
与其每页单独对应一个text container，现在对应两个text container，页中每列一个。每个container控制文档的一部分，文本先是在左上的container中，满了之后delegate会收到通知，如果还有文本需要排布会在下一个text container中布局，并在完成的时候通知delegate，依此类推。
不仅可以有多container，还可以有多个layout manager 访问同一个text storage，目的是提供同一段文本的多个view，如果用户更改了上面view中的文本，下面的view会直接反映出来
![](https://upload-images.jianshu.io/upload_images/1840444-720173483dd58c87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

