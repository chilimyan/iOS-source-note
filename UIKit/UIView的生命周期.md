# UIView的生命周期

```
- (instancetype)init{
    if (self = [super init]) {
        NSLog(@"%s",__func__);
    }
    return self;
}

- (instancetype)initWithFrame:(CGRect)frame{
    if (self = [super initWithFrame:frame]) {
        NSLog(@"%s",__func__);
    }
    return self;
}

- (instancetype)initWithCoder:(NSCoder *)aDecoder{
    if (self = [super initWithCoder:aDecoder]) {
        NSLog(@"%s",__func__);
    }
    return self;
}

- (void)awakeFromNib{
    [super awakeFromNib];
    NSLog(@"%s",__func__);
}

- (void)layoutSubviews{
    [super layoutSubviews];
    NSLog(@"%s",__func__);
}

- (void)didAddSubview:(UIView *)subview{
    [super didAddSubview:subview];
    NSLog(@"%s",__func__);
}

- (void)willRemoveSubview:(UIView *)subview{
    [super willRemoveSubview:subview];
    NSLog(@"%s",__func__);
}

- (void)willMoveToSuperview:(UIView *)newSuperview{
    [super willMoveToSuperview:newSuperview];
    NSLog(@"%s",__func__);
}

- (void)didMoveToSuperview{
    [super didMoveToSuperview];
    NSLog(@"%s",__func__);
}

- (void)willMoveToWindow:(UIWindow *)newWindow{
    [super willMoveToWindow:newWindow];
    NSLog(@"%s",__func__);
}

- (void)didMoveToWindow{
    [super didMoveToWindow];
    NSLog(@"%s",__func__);
}

- (void)removeFromSuperview{
    [super removeFromSuperview];
    NSLog(@"%s",__func__);
}

- (void)dealloc{
    NSLog(@"%s",__func__);
}
```
运行后打印如下：

```
2018-09-21 16:39:35.664287+0800 CLWebViewDemo[7046:274872] -[CLTestView initWithFrame:]
2018-09-21 16:39:35.664475+0800 CLWebViewDemo[7046:274872] -[CLTestView willMoveToSuperview:]
2018-09-21 16:39:35.664648+0800 CLWebViewDemo[7046:274872] -[CLTestView didMoveToSuperview]
2018-09-21 16:39:35.674194+0800 CLWebViewDemo[7046:274872] -[CLTestView willMoveToWindow:]
2018-09-21 16:39:35.674381+0800 CLWebViewDemo[7046:274872] -[CLTestView didMoveToWindow]
2018-09-21 16:39:35.698137+0800 CLWebViewDemo[7046:274872] -[CLTestView layoutSubviews]
2018-09-21 16:39:35.698459+0800 CLWebViewDemo[7046:274872] -[CLTestView layoutSubviews]
2018-09-21 16:39:40.991378+0800 CLWebViewDemo[7046:274872] -[CLTestView willMoveToWindow:]
2018-09-21 16:39:40.991612+0800 CLWebViewDemo[7046:274872] -[CLTestView didMoveToWindow]
2018-09-21 16:39:40.992742+0800 CLWebViewDemo[7046:274872] -[CLTestView willMoveToSuperview:]
2018-09-21 16:39:40.993035+0800 CLWebViewDemo[7046:274872] -[CLTestView didMoveToSuperview]
2018-09-21 16:39:40.993137+0800 CLWebViewDemo[7046:274872] -[CLTestView removeFromSuperview]
2018-09-21 16:39:40.993389+0800 CLWebViewDemo[7046:274872] -[CLTestView dealloc]
```



