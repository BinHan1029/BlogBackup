---
title: ReactiveCocoa 系列(一) 基本概念
date: 2016-04-26 14:39:33
tags:
- 移动开发
- iOS
---

[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) 是一个 iOS 中的函数式响应式编程框架，现在已经发展到了 v4.x，开始支持 Swift 了。如果我们的项目还是 Objective-C 开发的话建议使用 v2.5，也是一个很稳定的版本。当然每当我们学习一个新的框架前，除了了解简单的 api 接口使用外，另外很重要的一点就是要思考作者所表现的编程思想。

首先 [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) 是一个响应式编程范式，既 FRP(Functional Reactive Programming)，简单讲在程序开发中：a ＝ b ＋ c，传统的命令式编程中赋值之后 b 或者 c 的值变化后，a 的值不会跟着变化，而在响应式编程中，a 的值会随着 b 或者 c 的值变化而变化。

另一方面 [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) 作者大量了使用了链式编程和函数式编程，使用过 [Masonry](https://github.com/SnapKit/Masonry) 框架的应该都会有所体会，这样会使代码的可读性大大增强，代码更紧凑。

<!-- more -->

### 开发中常见的几个场景

* 比如我们给一个按钮添加点击事件，我们会怎么做呢？代码如下：

``` objc
-(void)addEvent
{
	[self.btn addTarget:self action:@selector(chick:) forControlEvents:UIControlEventTouchUpInside];
}

-(void)chick:(UIButton *)btn
{
	// 处理按钮的点击事件
}
```

其实这里我们可以发现一个问题，按钮的点击事件和回调函数 @selector(chick:) 已经被分离开了，我们如果事先不知道按钮和回调函数是通过 addTarget 方法建立关系的话，我们是很难知道他俩之间的联系的，另外一方面我们都知道方法的调用最终是给一个对象发送消息，这里我们也很难看到到底是哪个对象最终执行了回调函数 @selector(chick:)。而如果改写成 RAC 的写法理解可能就变得容易的多：

``` objc
-(void)addEvent
{
	[[self.btn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(UIButton *avatarButton) 
	{
		// 处理按钮的点击事件
	}
}
```

* 第二个场景，在一个 table 顶部放 banner 广告应该是一个很常见的需求，但是通常 table 列表接口与 banner 广告接口服务器端会分拆成两个不同的接口进行返回，当 banner 的广告数量为 0 时，我们需要隐藏 banner 视图。这里我们就需要对着两个接口请求做同步。当然这里比较简单粗暴的，我们可以当 banner 接口返回结果后再进行 table 列表接口，这样做当然显得不那么优雅也不建议使用。不过在 iOS 中有一个专门处理多线程执行回调同步的框架 [PromiseKit](https://github.com/mxcl/PromiseKit)。这里最后还是说下 GCD 的方式，这可能是大部分情况下采用的方式。通过 dispatch _group _t 线程同步，伪代码：

``` objc
-(void)getData
{
	// 创建
	dispatch_group_t group = dispatch_group_create();
	//  banner 接口
	dispatch_group_enter(group);
	[[BHNetReqManager sharedManager].bh_requestUrl(@"banner") startRequestWithCompleteHandler:^(id response, NSError *error) {
		dispatch_group_leave(group);
	}];
	//  table 接口
	dispatch_group_enter(group);
	[[BHNetReqManager sharedManager].bh_requestUrl(@"table") startRequestWithCompleteHandler:^(id response, NSError *error) {
		dispatch_group_leave(group);
	}];
	dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // 处理数据
    });
}
```

所以能看到在开发中线程同步是有很多种方式，甚至还可以使用自定义 NSOperation 通过设置依赖完成。而在 RAC 中也为我们提供了一个方式，这也是我比较推荐的，不仅仅是代码阅读起来可读性更高，另外一方面基于 RAC 开发的项目可以提供一个处理不同情况下的标准，让不同的人写出来的代码就看起来像是一个人写的一样。

``` objc
- (void)test
{
    RACSignal *signalA = [RACSignal createSignal:^RACDisposable *(id subscriber) {
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        subscriber sendNext:@"A"];
    });
        return nil;
    }];
    
    RACSignal *signalB = [RACSignal createSignal:^RACDisposable *(id subscriber) {
        [subscriber sendNext:@"B"];
        [subscriber sendNext:@"Another B"];
        [subscriber sendCompleted];
        return nil;
    }];
    
    [self rac_liftSelector:@selector(doA:withB:) withSignals:signalA, signalB, nil];
//    [self rac_liftSelector:@selector(doA:withB:) withSignalsFromArray:@[signalA,signalB]];
}

- (void)doA:(NSString *)A withB:(NSString *)B
{
    NSLog(@"A:%@ and B:%@", A, B);
}
```

## RAC 中的一些名词概念

上面只是简单举例说明 RAC 的处理一些事情的能力，当然它远远要比想象中的强大，但是在这之前我觉得还是很有必要了解一些 RAC 中涉及到的一些概念，既 RAC 中的四大核心组件。

#### 信号源：RACStream 及其子类

![Alt text](/assets/blogImg/rac_1.png)

翻阅的大部分的资料都会翻译成信号，其实这里如果直接理解成信号，会相当难理解出本意，所以我一般将其理解为电流。就像家里的墙上的插座一样，我们可以为在上面直接插上某件电器的插头，亦或者在外接一个插排，其实电流这玩意我们看不见摸不着，但是当你外接一个新插排，不关闭“开关”的情况下，新插排就直接可以使用了，甚至我们可以将两项/三项的插座转换为 USB 的插口供手机充电器使用。其实这个例子我觉得至少已经包含了，后面要提及到“信号的转换”和“订阅者”概念，是的每一个外接的新插排就是一个订阅者。

RACStream 是一个抽象类，开发中我们通常使用的是 RACSignal 和 RACSequence。

RACSignal 就是我们上面例子中插排电线中的电流，也就是开发中我们的数据的值。它可以向订阅者发送三种不同类型的事件：

* next: RACSignal 通过 next 事件向订阅者传送新的值，并且这个值可以为 nil。 
* error ：RACSignal 通过 error 事件向订阅者表明信号在正常结束前发生了错误。
* completed ：RACSignal 通过 completed 事件向订阅者表明信号已经正常结束，后续的值不会发送给订阅者。

这里会产生一个疑问，我们怎么给 RACSignal 添加订阅者？它有一个很核心的方法 subscribe，这是订阅者和信号源建立联系的关键，所以所有的子类都必须实现，并且我们发现所有的子类实现其实都会传递一个实现了 RACSubscriber 的类，并返回一个 RACDisposable 对象。

``` objc
- (RACDisposable *)subscribe:(id)subscriber 
{
  NSCAssert(NO, @"This method must be overridden by subclasses");
  return nil;
}
```

RACSubject 是一个可以手动控制的信号，这个相对 RACSignal 有了订阅者之后就会自动发送信号，而 RACSubject 是可控的，类似插排上面每个插头有额外添加了开关。类比代码中，就是它既是 RACSignal 的子类又实现了 RACSubscriber 协议。但正因为如此，官方反倒建议我们尽量少使用它。

RACSequence 从图上来看其实它跟 RACSignal 是并列的，也没有太直接的关系，而 RACSequence 通常是为了简化 Objective-C 中的集合操作。

#### 订阅者：RACSubscriber 的实现类及其子类

RACSubscriber 是一个协议，所有实现了该协议的类都可以成为订阅者。

其中 -sendNext: 、-sendError: 和 -sendCompleted 分别用来从 RACSignal 接收 next 、error 和 completed 事件，而 -didSubscribeWithDisposable: 则用来接收代表某次订阅的 disposable 对象。

#### 调度器：RACScheduler 及其子类

![Alt text](/assets/blogImg/rac_3.png)

主要为对 GCD 的封装，包含了一些同步异步及定时器的操作。

#### 清洁工：RACDisposable 及其子类

![Alt text](/assets/blogImg/rac_2.png)

最核心的方法 dispose，其实就是取消所有的订阅，类似于家里订阅了报纸、牛奶，一天你突然不想订了，取消掉。

相关资料：

[https://github.com/ReactiveCocoaChina/ReactiveCocoaChineseResources](https://github.com/ReactiveCocoaChina/ReactiveCocoaChineseResources)

[http://www.cocoachina.com/ios/20160106/14880.html](http://www.cocoachina.com/ios/20160106/14880.html)







