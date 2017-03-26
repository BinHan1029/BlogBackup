title: UIBezierPath 配合 CAShapeLayer 画一些有趣的图形
date: 2015-05-17 22:37:14
tags:
- 移动开发
- iOS
---

CAShapeLayer是CALayer的子类，但是比CALayer更灵活，配合一个神奇的属性path用这个属性配合上 UIBezierPath 这个类就可以达到超神的效果。

## 玩一下 UIBezierPath
UIBezierPath 顾名思义，这是用贝塞尔曲线的方式来构建一段弧线，你可以用任意条弧线来组成你想要的形状，它包含起始点、终点、及控制点三个参数。如下图红色矩形范围内的白色背景，最上面就是一条有弧度的曲线。

<!-- more -->

![Alt text](/assets/blogImg/bezier_1.png)

它可以用 CAShapeLayer+UIBezierPath 来做，思路大概是这样，先移动到左上方的位置，然后向下划线，然后往右划线，然后往上划线，还剩一个盖子，这个盖子就用一个控制点控制曲率，非常简单，图中的红色点即为控制点，代码如下：

```objc
-(void)initLayer
{
    CAShapeLayer *layer =  [[CAShapeLayer alloc] init];
    UIBezierPath *path = [[UIBezierPath alloc] init];
    [path moveToPoint:CGPointMake(0, SCREEN_HEIGHT - MENU_HEIGHT)];
    [path addLineToPoint:CGPointMake(0, SCREEN_HEIGHT)];
    [path addLineToPoint:CGPointMake(SCREEN_WIDTH, SCREEN_HEIGHT)];
    [path addLineToPoint:CGPointMake(SCREEN_WIDTH, SCREEN_HEIGHT - MENU_HEIGHT)];
    [path addQuadCurveToPoint:CGPointMake(0, SCREEN_HEIGHT - MENU_HEIGHT) controlPoint:CGPointMake(SCREEN_WIDTH/2.f, SCREEN_HEIGHT - MENU_HEIGHT * 2.0f)];
    layer.path = path.CGPath;
    //设置填充色及画笔颜色
    layer.fillColor = [UIColor purpleColor].CGColor;
    layer.strokeColor = [UIColor blueColor].CGColor;
    layer.lineWidth = 1.f;
    [self.view.layer addSublayer:layer];
}
```

效果如图：

![Alt text](/assets/blogImg/bezier_2.png)

## 使用 UIBezierPath 画圆及添加加载动画



```objc
-(void)initCircle
{
    CAShapeLayer *layer = [[CAShapeLayer alloc] init];
    UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:CGRectMake(0, 0, 200, 200)];
    layer.path = path.CGPath;
    layer.fillColor = [UIColor clearColor].CGColor;
    layer.strokeColor = [UIColor blueColor].CGColor;
    layer.lineWidth = 1.f;
    //设置stroke起始点
    layer.strokeStart = 0;
    layer.strokeEnd = 0.75;
    layer.frame = CGRectMake(0, 0, 200, 200);
    layer.position = self.view.center;
    [self.view.layer addSublayer:layer];
    //动画显示了从1到0之间每一个值对这条曲线的影响
    CABasicAnimation *animation = [[CABasicAnimation alloc] init];
    animation.fromValue = [NSNumber numberWithFloat:1.0];
    animation.toValue = [NSNumber numberWithFloat:0.0];
    animation.duration = 5.f;
    [layer addAnimation:animation forKey:@"strokeStart"];
}
```

## 利用 layer 的 mask 属性为图片添加遮罩

```objc
-(void)showImage
{
    UIImage *image = BHIMG(@"sunyizhen.jpg");
    UIImageView *imageView = [[UIImageView alloc] initWithImage:image];
    imageView.contentMode = UIViewContentModeCenter;
    imageView.mj_y = kNavBarHeight;
    [self.view addSubview:imageView];
    
    UIBezierPath *originPath = [UIBezierPath bezierPathWithRect:CGRectInset(imageView.bounds, 67.f, 91.5f)];
    UIBezierPath *finalPath = [UIBezierPath bezierPathWithRect:imageView.bounds];
    CAShapeLayer *layer = [CAShapeLayer layer];
    layer.path = finalPath.CGPath;
    //    layer.fillColor = [UIColor clearColor].CGColor;
    //    layer.strokeColor = [UIColor blueColor].CGColor;
    //    layer.lineWidth = 2.f;
    imageView.layer.mask = layer;
    CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"path"];
    animation.fromValue = (__bridge id _Nullable)(originPath.CGPath);
    animation.toValue = (__bridge id _Nullable)(finalPath.CGPath);
    animation.duration = 2.f;
    [layer addAnimation:animation forKey:@"path"];
}
```

效果如下

![Alt text](/assets/blogImg/bezier_4.gif)

## 最后
总之使用 UIbezierPath 和 CAShapeLayer 可以画出你想要的任何形状，没有它做不到，只有你想不到，当然有时候可能会比较复杂而显得比较笨拙，但最终实现后确实会显得很有趣。

代码可以下载GITHUB中[BlogDemo](https://github.com/binhandev/BlogDemo)进行查看。