# SDWebImage加载图片引发的内存大量消耗卡顿的问题
当我们使用`SDWebImage`加载比较大的高清图时,会出现内存占用急剧上升,页面出现卡顿导致程序崩溃的问题。
##### 解决方法:
我们全局搜索`decodedImageWithImage`,发现在`SDWebImage`中有几处调用了这个方法,这个方法的用处是减压缩图片,并将图片存到cache使得之后的加载更加快,效果更加好。但是问题就在于去压缩这个操作,如果传进的图片分辨率特别的高,它的减压缩会消耗大量的内存。当我们把这些地方注释掉后重新运行,内存的增长就恢复正常了。

