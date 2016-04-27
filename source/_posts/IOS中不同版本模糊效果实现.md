title: IOS中不同版本模糊效果实现
date: 2016-01-03 21:44:13
tags:
- 移动开发
- IOS
---
## 前言
苹果在IOS7中界面扁平化设计之后，加入了大量的毛玻璃效果，如控制中心，而我们在开发应用的时候通常也不会浪费navigationBar及tabBar这两块系统的毛玻璃效果。

如下面的效果

![Alt text](/assets/blogImg/blureffect_1.png)

<!-- more -->

### IOS7中我们需要自己实现毛玻璃效果
虽然IOS7中有大量的毛玻璃效果，但是系统却并没有提供直接的api供我们使用，但好在苹果为我们提供了[Blurring and Tinting an Image Demo](https://developer.apple.com/library/ios/samplecode/UIImageEffects/Introduction/Intro.html)，里面的UIImageEffects我们可以直接拿来使用。

``` objc
+ (UIImage*)imageByApplyingLightEffectToImage:(UIImage*)inputImage;
+ (UIImage*)imageByApplyingExtraLightEffectToImage:(UIImage*)inputImage;
+ (UIImage*)imageByApplyingDarkEffectToImage:(UIImage*)inputImage;
+ (UIImage*)imageByApplyingTintEffectWithColor:(UIColor *)tintColor toImage:(UIImage*)inputImage;
+ (UIImage*)imageByApplyingBlurToImage:(UIImage*)inputImage withRadius:(CGFloat)blurRadius tintColor:(UIColor *)tintColor saturationDeltaFactor:(CGFloat)saturationDeltaFactor maskImage:(UIImage *)maskImage;
```
上面三个接口分别对应的三种不同的模糊效果，分别对应着IOS8中的UIBlurEffectStyleExtraLight，UIBlurEffectStyleLight，UIBlurEffectStyleDark，省去参数的配置直接进行使用，其中第一个即对比正常的UIBlurEffectStyleLight要明亮一些，第三种效果要比正常的灰暗一些。而最后一个涉及颜色的饱和度及maskImage特定区域的模糊效果。

``` objc
//UIImage *effectImage = [UIImageEffects imageByApplyingLightEffectToImage:_bgImageView.image];
UIImage *effectImage = [UIImageEffects imageByApplyingTintEffectWithColor:[UIColor blueColor] toImage:_bgImageView.image];
_bgImageView.image = effectImage;
```

### IOS8中可以直接使用系统提供的API
IOS8中系统为我们提供了UIVisualEffectView类进行模糊，使用起来更快捷方便。

``` objc
UIVisualEffectView *visualEffectView = [[UIVisualEffectView alloc] initWithEffect:[UIBlurEffect effectWithStyle:UIBlurEffectStyleLight]];
visualEffectView.frame = self.view.bounds;
//低于1.0会导致离屏渲染，
visualEffectView.alpha = 1.0;
[self.view addSubview:visualEffectView];        
```

效果如下：

![Alt text](/assets/blogImg/blureffect_2.png)

这两种方案看起来效果相同，但是UIVisualEffectView是直接创建了一块毛玻璃盖在了UIImageView上，而UIImageEffects是将UIImageView的image从新进行box blur计算而使图像接近高斯模糊的效果。这点可以通过Xcode的Capture View Hierarchy来进行查看。

![Alt text](/assets/blogImg/blureffect_3.png)

![Alt text](/assets/blogImg/blureffect_4.png)

代码可以下载GITHUB中[BlogDemo](https://github.com/octopusline/BlogDemo.git)进行查看。
