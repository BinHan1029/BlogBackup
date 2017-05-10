---
title: ReactiveCocoa 系列(二)特定场景 API 的使用
date: 2016-05-03 17:32:57
tags:
- 移动开发
- iOS
---

整理了开发中一些特定场景下使用 RAC 的便利之处，关于一些常用的 API 的使用网上的资源还是很多的，就不做举例了。

* 比如登录页面当用户输入的账号、密码长度合法的情况下才可以点击登录按钮，否则按钮不可点击，这里条件比较简单，即账号密码长度都大于5，其实还可以使用一些正则匹配账号格式是否正确等。combineLatest 就是合并多个信号，且每个信号至少发送过一次 sendNext，才会触发合并信号。

``` objc
RAC(self.loginBT, enabled) = [RACSignal combineLatest:@[self.accountTF.rac_textSignal, self.passWordTF.rac_textSignal] reduce:^(NSString *usernameValid, NSString *passwordValid) {
        return @(usernameValid.length > 5 && passwordValid.length > 5);
}];
```

<!-- more -->

* 使用 RACSubject 替代 Delegate，每调用一次 test 方法，则对应在 toVC2 方法中打印一次接受的 value 值。 这里利用的就是 RACSubject 既可以创建信号又可以发送信号的特性。

``` objc
@interface TestVC1 : UIViewController

@end

@implementation TestVC1

-(void)toVC2:(UIButton *)btn
{
	TestVC2 *VC2 = [[TestVC2 alloc] init];
	[VC2.subject subscribeNext:^(NSString *value) {
    	NSLog(@"value = %@", value);        
	}];
	[self.navigationController pushViewController:VC2 animated:YES]
}

@end

@interface TestVC2 : UIViewController

@property (nonatomic, strong) RACSubject *shareSubject;

@end

@implementation TestVC2

-(void)test:(UIButton *)btn
{
	[self.subject sendNext(@"test")];
}

-(RACSubject*)subject
{
    if (!_subject)
    {
        _subject = [RACSubject subject];
    }
    return _subject;
}
@end
```
* RACScheduler 计时器，这里看源码的话，可以看到最终是对 GCD 计时器的封装，当然也可以实现倒计时。

``` objc
- (void)start
{
	__block NSUInteger timerNum = 0;
	self.disposable = [[RACScheduler mainThreadScheduler] after:[NSDate date] repeatingEvery:1.f withLeeway:0 schedule:^{
		self.controlView.timerNum = timerNum;
		timerNum++;
		}];
}

-(void)stop
{
	[self.disposable dispose];
}

```

* 在控制器的 viewDidDisappear、dealloc 或者其他一些方法中执行某些操作。这个方法就是当监听到 self 的 viewDidDisappear 方法执行后会发送信号，然后做一些处理。某些方面讲这并不是一个好例子，这里也仅仅是对 rac_signalForSelector 使用的一个说明，就是通过此 API 可以监听某对象的在执行特定消息时可以回调给开发者。

``` objc
-(void)addObserve
{
	[[self rac_signalForSelector:@selector(viewDidDisappear:)] subscribeNext:^(id x) {
		[MBProgressHUD hideHUD];
	}];	
}
```

* 使用 RACCommand 执行网络请求，这里在实际开发中，需要对 response、error 进行判断。可以看到其实 RACCommand 内部封装的是一个信号，每当执行 RACCommand 时就会创建一个信号，所以一个 RACCommand 里包含了多个信号，如果想获取最新的信号，需要使用 executionSignals.switchToLatest，而执行一个 RACCommand 也很简单，调用 execute 即可，当然还可以传递参数。

``` objc
-(void)initialize
{
	[self.loadDataCommand.executionSignals.switchToLatest subscribeNext:^(id response) {
		@strongify(self)
		NSLog(@"response = %@", response);
	}];
}

-(RACCommand *)loadDataCommand
{
    if (!_loadDataCommand)
    {
        _loadDataCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
            return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
                [[BHNetReqManager sharedManager].bh_requestUrl(kNmpaiFound) startRequestWithCompleteHandler:^(NSURLSessionDataTask *task, id response, NSError *error) {
                    @strongify(self)
                    [subscriber sendNext:response];
                    [subscriber sendCompleted];
                }];
                return nil;
            }];
        }];

    }
    return _loadDataCommand;
}
```

* 为网络失败添加重试机制，当然重试机制的使用场景还是很多的

``` objc
__block int failedCount = 0;
//创建信号
RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id subscriber) {
if (failedCount < 9) 
	{
	failedCount++;
	NSLog(@"我失败了");
	//发送错误，才会要重试
	[[BHNetReqManager sharedManager].bh_requestUrl(@"http://binhan1029.github.io/") startRequestWithCompleteHandler:^(id response, NSError *error) {
		[subscriber sendError:nil];
	}];
	} 
	else 
	{
		[subscriber sendNext:@"成功"];
	}
	return nil;
	}];
RACSignal *retrySignal = [signal retry];
[retrySignal subscribeNext:^(id x) {
	NSLog(@"终于成功了");
}];
```







相关资料:

[http://tech.meituan.com/potential-memory-leak-in-reactivecocoa.html?from=timeline&isappinstalled=0](http://tech.meituan.com/potential-memory-leak-in-reactivecocoa.html?from=timeline&isappinstalled=0)

