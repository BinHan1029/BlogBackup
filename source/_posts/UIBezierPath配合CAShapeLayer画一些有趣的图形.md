title: UIBezierPath配合CAShapeLayer画一些有趣的图形
date: 2016-02-17 22:37:14
tags:
- 移动开发
- IOS
---

## 前言
CAShapeLayer是CALayer的子类，但是比CALayer更灵活，配合一个神奇的属性path用这个属性配合上 UIBezierPath 这个类就可以达到超神的效果。

## 玩一下UIBezierPath
UIBezierPath顾名思义，这是用贝塞尔曲线的方式来构建一段弧线，你可以用任意条弧线来组成你想要的形状，它包含起始点、终点、及控制点三个参数。如下图红色矩形范围内的白色背景，最上面就是一条有弧度的曲线。

![Alt text](/assets/blogImg/bezier_1.png)

它可以用CAShapeLayer+UIBezierPath来做，思路大概是这样，先移动到左上方的位置，然后向下划线，然后往右划线，然后往上划线，还剩一个盖子，这个盖子就用一个控制点控制曲率，非常简单，图中的红色点即为控制点，代码如下：

``` objc
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
``` 

<!-- more -->

效果如图：

![Alt text](/assets/blogImg/bezier_2.png)

## 使用UIBezierPath画圆及添加加载动画

![Alt text](/assets/blogImg/bezier_3.gif)

最终下过如下：

``` objc
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
``` 

## 微信中下拉拍摄眼睛的加载动画

![Alt text](/assets/blogImg/bezier_4.gif)

首先你画出这只眼睛，这是眼睛包括5个部分组成

``` objc
@interface WXPullView ()

@property (strong, nonatomic) CAShapeLayer *eyeFirstLightLayer;
@property (strong, nonatomic) CAShapeLayer *eyeSecondLightLayer;
@property (strong, nonatomic) CAShapeLayer *eyeballLayer;
@property (strong, nonatomic) CAShapeLayer *topEyesocketLayer;
@property (strong, nonatomic) CAShapeLayer *bottomEyesocketLayer;

@end
``` 

然后，还是通过 UIBezierPath 和 CAShapeLayer 这样的老套路来画，代码较多:

``` objc
@implementation WXPullView

- (instancetype)initWithFrame:(CGRect)frame {
    self = [super initWithFrame:frame];
    if (self) {
        [self.layer addSublayer:self.eyeFirstLightLayer];
        [self.layer addSublayer:self.eyeSecondLightLayer];
        [self.layer addSublayer:self.eyeballLayer];
        [self.layer addSublayer:self.topEyesocketLayer];
        [self.layer addSublayer:self.bottomEyesocketLayer];
        [self setupAnimation];
    }
    return self;
}

- (CAShapeLayer *)eyeFirstLightLayer {
    if (!_eyeFirstLightLayer) {
        _eyeFirstLightLayer = [CAShapeLayer layer];
        CGPoint center = CGPointMake(CGRectGetWidth(self.frame) / 2, CGRectGetHeight(self.frame) / 2);
        UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center
                                                            radius:CGRectGetWidth(self.frame) * 0.2
                                                        startAngle:(230.f / 180.f) * M_PI
                                                          endAngle:(265.f / 180.f) * M_PI
                                                         clockwise:YES];
        _eyeFirstLightLayer.borderColor = [UIColor blackColor].CGColor;
        _eyeFirstLightLayer.lineWidth = 5.f;
        _eyeFirstLightLayer.path = path.CGPath;
        _eyeFirstLightLayer.fillColor = [UIColor clearColor].CGColor;
        _eyeFirstLightLayer.strokeColor = [UIColor whiteColor].CGColor;
    }
    return _eyeFirstLightLayer;
}

- (CAShapeLayer *)eyeSecondLightLayer {
    if (!_eyeSecondLightLayer) {
        _eyeSecondLightLayer = [CAShapeLayer layer];
        CGPoint center = CGPointMake(CGRectGetWidth(self.frame) / 2, CGRectGetHeight(self.frame) / 2);
        UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center
                                                            radius:CGRectGetWidth(self.frame) * 0.2
                                                        startAngle:(211.f / 180.f) * M_PI
                                                          endAngle:(220.f / 180.f) * M_PI
                                                         clockwise:YES];
        _eyeSecondLightLayer.borderColor = [UIColor blackColor].CGColor;
        _eyeSecondLightLayer.lineWidth = 5.f;
        _eyeSecondLightLayer.path = path.CGPath;
        _eyeSecondLightLayer.fillColor = [UIColor clearColor].CGColor;
        _eyeSecondLightLayer.strokeColor = [UIColor whiteColor].CGColor;

    }
    return _eyeSecondLightLayer;
}

- (CAShapeLayer *)eyeballLayer {
    if (!_eyeballLayer) {
        _eyeballLayer = [CAShapeLayer layer];
        CGPoint center = CGPointMake(CGRectGetWidth(self.frame) / 2, CGRectGetHeight(self.frame) / 2);
        UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center
                                                            radius:CGRectGetWidth(self.frame) * 0.3
                                                        startAngle:(0.f / 180.f) * M_PI
                                                          endAngle:(360.f / 180.f) * M_PI
                                                         clockwise:YES];
        _eyeballLayer.borderColor = [UIColor blackColor].CGColor;
        _eyeballLayer.lineWidth = 1.f;
        _eyeballLayer.path = path.CGPath;
        _eyeballLayer.fillColor = [UIColor clearColor].CGColor;
        _eyeballLayer.strokeColor = [UIColor whiteColor].CGColor;
        _eyeballLayer.anchorPoint = CGPointMake(0.5, 0.5);
        
    }
    return _eyeballLayer;
}

- (CAShapeLayer *)topEyesocketLayer {
    if (!_topEyesocketLayer) {
        _topEyesocketLayer = [CAShapeLayer layer];
        CGPoint center = CGPointMake(CGRectGetWidth(self.frame) / 2, CGRectGetHeight(self.frame) / 2);
        UIBezierPath *path = [UIBezierPath bezierPath];
        [path moveToPoint:CGPointMake(0, CGRectGetHeight(self.frame) / 2)];
        [path addQuadCurveToPoint:CGPointMake(CGRectGetWidth(self.frame), CGRectGetHeight(self.frame) / 2)
                     controlPoint:CGPointMake(CGRectGetWidth(self.frame) / 2, center.y - center.y - 20)];
        _topEyesocketLayer.borderColor = [UIColor blackColor].CGColor;
        _topEyesocketLayer.lineWidth = 1.f;
        _topEyesocketLayer.path = path.CGPath;
        _topEyesocketLayer.fillColor = [UIColor clearColor].CGColor;
        _topEyesocketLayer.strokeColor = [UIColor whiteColor].CGColor;
    }
    return _topEyesocketLayer;
}

- (CAShapeLayer *)bottomEyesocketLayer {
    if (!_bottomEyesocketLayer) {
        _bottomEyesocketLayer = [CAShapeLayer layer];
        CGPoint center = CGPointMake(CGRectGetWidth(self.frame) / 2, CGRectGetHeight(self.frame) / 2);
        UIBezierPath *path = [UIBezierPath bezierPath];
        [path moveToPoint:CGPointMake(0, CGRectGetHeight(self.frame) / 2)];
        [path addQuadCurveToPoint:CGPointMake(CGRectGetWidth(self.frame), CGRectGetHeight(self.frame) / 2)
                     controlPoint:CGPointMake(CGRectGetWidth(self.frame) / 2, center.y + center.y + 20)];
        _bottomEyesocketLayer.borderColor = [UIColor blackColor].CGColor;
        _bottomEyesocketLayer.lineWidth = 1.f;
        _bottomEyesocketLayer.path = path.CGPath;
        _bottomEyesocketLayer.fillColor = [UIColor clearColor].CGColor;
        _bottomEyesocketLayer.strokeColor = [UIColor whiteColor].CGColor;

    }
    return _bottomEyesocketLayer;
}

- (void)setupAnimation {
    self.eyeFirstLightLayer.lineWidth = 0.f;
    self.eyeSecondLightLayer.lineWidth = 0.f;
    self.eyeballLayer.opacity = 0.f;
    _bottomEyesocketLayer.strokeStart = 0.5f;
    _bottomEyesocketLayer.strokeEnd = 0.5f;
    _topEyesocketLayer.strokeStart = 0.5f;
    _topEyesocketLayer.strokeEnd = 0.5f;
}

- (void)animationWith:(CGFloat)y {
    CGFloat flag = self.frame.origin.y * 2.f - 20.f;
    if (y < flag) {
        if (self.eyeFirstLightLayer.lineWidth < 5.f) {
            self.eyeFirstLightLayer.lineWidth += 1.f;
            self.eyeSecondLightLayer.lineWidth += 1.f;
        }
    }
    
    if(y < flag - 20) {
        if (self.eyeballLayer.opacity <= 1.0f) {
            self.eyeballLayer.opacity += 0.1f;
        }
        
    }
    
    if (y < flag - 40) {
        if (self.topEyesocketLayer.strokeEnd < 1.f && self.topEyesocketLayer.strokeStart > 0.f) {
            self.topEyesocketLayer.strokeEnd += 0.1f;
            self.topEyesocketLayer.strokeStart -= 0.1f;
            self.bottomEyesocketLayer.strokeEnd += 0.1f;
            self.bottomEyesocketLayer.strokeStart -= 0.1f;
        }
    }
    
    if (y > flag - 40) {
        if (self.topEyesocketLayer.strokeEnd > 0.5f && self.topEyesocketLayer.strokeStart < 0.5f) {
            self.topEyesocketLayer.strokeEnd -= 0.1f;
            self.topEyesocketLayer.strokeStart += 0.1f;
            self.bottomEyesocketLayer.strokeEnd -= 0.1f;
            self.bottomEyesocketLayer.strokeStart += 0.1f;
        }
    }
    
    if (y > flag - 20) {
        if (self.eyeballLayer.opacity >= 0.0f) {
            self.eyeballLayer.opacity -= 0.1f;
        }
    }
    
    if (y > flag) {
        if (self.eyeFirstLightLayer.lineWidth > 0.f) {
            self.eyeFirstLightLayer.lineWidth -= 1.f;
            self.eyeSecondLightLayer.lineWidth -= 1.f;
        }
    }
}

``` 

通过监听UIScrollView的contentOffset的偏移量来控制各种属性

* (void)animationWith:(CGFloat)y;

## 最后
总之使用 UIbezierPath 和 CAShapeLayer 可以画出你想要的任何形状，没有它做不到，只有你想不到，当然有时候可能会比较复杂而显得比较笨拙，但最终实现后确实会显得很有趣。

代码可以下载GITHUB中[BlogDemo](https://github.com/binhan666/BlogDemo)进行查看。