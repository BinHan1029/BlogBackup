	---
title: 链式创建UILabel和UIButton
date: 2015-08-05 18:56:53
tags:
- 移动开发
- iOS
---


其实最早接触链式编程是使用 [Masonry](https://github.com/SnapKit/Masonry) 框架的时候，对于之前大部分写集中式编码的人而言会觉得眼前一亮，当所有的操作通过点号(.)符号链接起来时候之后，代码的可读性大大的增强了。所以在之前就使用这种方式对 [AFNetworking](https://github.com/AFNetworking/AFNetworking) 进行了二次封装，[链接地址](http://binhan1029.github.io/2015/07/11/%E5%AF%B9AFNetworking%E7%9A%84%E9%93%BE%E5%BC%8F%E4%BA%8C%E6%AC%A1%E5%B0%81%E8%A3%85/)。

<!-- more -->

### 为什么想到把 UILabel/UIButton 也使用这种方式
其实之前在项目开发中会在多个地方用到手动创建添加 UILabel/UIButton 的视图，写了多了就将其封装在了一个 tools 文件内了，一直觉得并不是很好，因为这种方式一旦有某一个地方对 UILable/UIButton 有针对性的属性修改时，一种解决方式是当真正获取到 View 时再修改，另一种方式就是在原有的方法上添加参数进行修改，这样就很不灵活，而且需要给默认参数使用 nil 进行占位，代码看起来就很丑陋；而使用链式编程恰恰可以解决这个问题，另外使用 Category 的方式也不会浸入项目原有代码。



### UILable 式例
其实我们通常创建一个 UILable 时常用的属性会有 text/textColor/font/textAlignment，这里我主要封装的也是这几个属性，其实可以根据项目动态的进行添加其他属性。

``` objc
#import "UILabel+BHLabel.h"

@implementation UILabel (BHLabel)

/**
 *  文本内容
 */
-(UILabel *(^)(NSString *text))bh_text
{
    return ^UILabel *(NSString *text) {
        self.text = text;
        return self;
    };
}

/**
 *  文本颜色
 */
-(UILabel *(^)(UIColor *color))bh_textColor
{
    return ^UILabel *(UIColor *color) {
        self.textColor = color;
        return self;
    };
}

/**
 *  文本字体大小
 */
-(UILabel *(^)(CGFloat fontSize))bh_textFont
{
    return ^UILabel *(CGFloat fontSize) {
        self.font = [UIFont systemFontOfSize:fontSize];
        return self;
    };
}

/**
 *  文本居中方式
 */
-(UILabel *(^)(NSTextAlignment textAlignment))bh_textAlignment
{
    return ^UILabel *(NSTextAlignment textAlignment) {
        self.textAlignment = textAlignment;
        return self;
    };
}

@end
``` 

### 最后关于链式编程的一些理解
链式编程的代码结构上，通常我们的方法返回结果会是 void/NSString */NSInteger….这种空值或者指针对象，但是链式编程里所有的函数方法返回的都是一个 block，当这个 block 真正执行完成后将把对象自身返回进行下一步操作。
我觉得链式编程从某种角度讲大大提高了代码的可读性，这样后期代码就减少了维护成本，而在学习 [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) 时也是使用了这种函数式编程思想。
代码可以下载GITHUB中 [BlogDemo](https://github.com/BinHan1029/BlogDemo)进行查看。

