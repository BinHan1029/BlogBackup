title: CoreSpotlight初识
date: 2015-04-27 21:47:03
tags:
- 移动开发
- iOS
---

## Spotloight是什么?
Spotlight在iOS9上做了一些新的改进, 也就是开放了一些新的API, 通过Core Spotlight Framework你可以在你的app中集成Spotlight。

![Alt text](/assets/blogImg/corespotlight_1.png)


<!-- more -->

## 如何集成Spotloight
* 引入CoreSpotlight框架

![Alt text](/assets/blogImg/corespotlight_2.png)

* 创建索引条目CSSearchableItem

``` objc

#import <CoreSpotlight/CoreSpotlight.h>

@interface CoreSpotlightController ()

@end

@implementation CoreSpotlightController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view.
    if(IOS9)
    {
        CSSearchableItemAttributeSet *attributeSet = [[CSSearchableItemAttributeSet alloc] initWithItemContentType:@"com.mobile.BlogDemo"];
        attributeSet.title = @"CoreSpotlightTestTitle";
        attributeSet.contentDescription = @"CoreSpotlightTestDescription";
        CSSearchableItem *item = [[CSSearchableItem alloc] initWithUniqueIdentifier:@"com.mobile.BlogDem" domainIdentifier:@"mxy" attributeSet:attributeSet];
        [[CSSearchableIndex defaultSearchableIndex] indexSearchableItems:@[item] completionHandler:^(NSError * _Nullable error) {
            
        }];
    }
}

@end

```

1. 关于CSSearchableItemAttributeSet更多的设置可以查看头文件及[文档](https://developer.apple.com/library/ios/documentation/CoreSpotlight/Reference/CSSearchableItemAttributeSet_Class/index.html)

2. 关于CSSearchableItem的uniqueIdentifier标识，当我们使用Spotloight搜索后，点击索引后进入应用可由此判断用户点击的为某一条

3. 关于CSSearchableItem的domainIdentifier标识，主要在我们删除CSSearchableItem时会用到，如下图可对指定的CSSearchableItem索引进行删除


``` objc

-(void)deleteSearchableItem
{
    //关于删除指定的spotlight
    [[CSSearchableIndex defaultSearchableIndex] deleteSearchableItemsWithDomainIdentifiers:@[@"mxy"] completionHandler:^(NSError * _Nullable error) {
        
    }];
    //删除所有spotlight
    [[CSSearchableIndex defaultSearchableIndex] deleteAllSearchableItemsWithCompletionHandler:^(NSError * _Nullable error) {
        
    }];
}

```

* 点击spotlight后的方法回调

``` objc

- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler
{
    if ([[userActivity activityType] isEqualToString:CSSearchableItemActionType])
    {
        NSString *uniqueIdentifier = [userActivity.userInfo objectForKey:CSSearchableItemActivityIdentifier];
        // 接受事先定义好的数值,如果是多个参数可以使用把json转成string传递过来,接受后把string在转换为json
        // 通过解析json后判断接下来的业务逻辑, 如页面跳转，提示框等。。。
        NSLog(@"传递过来的值%@", uniqueIdentifier);
    }
    return YES;
}

```

## 关于corespotlight向系统注册的索引条目限制

当插入的条目为200条时，会报警告，所以应在一定的条件下删除过时的CSSearchableItem

``` objc

<Warning>: BKSendHIDEvent: IOHIDEventSystemConnectionDispatchEvent error:0xE00002E8 -- Unknown event dropped

```

代码可以下载GITHUB中[BlogDemo](https://github.com/binhan666/BlogDemo)进行查看。
