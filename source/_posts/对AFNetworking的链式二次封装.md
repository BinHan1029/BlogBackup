title: 对AFNetworking的链式二次封装
date: 2015-07-11 18:23:27
tags:
- 移动开发
- IOS
---

#### 写在前面
之前在使用[AFNetworking](https://github.com/AFNetworking/AFNetworking)都会进行二次封装便于开发使用，但是通常的结构是一种集中式的封装，如下：

``` objc
- (void)asyncWithQueryString:(NSString *)query params:(NSDictionary *)params 
 requestType:(RequestType)requestType 
 completionHandler:(void (^)(NSDictionary *result, NSError *error))handler；
```
这种结构的弊端在于，每次调用的时候都需要传递所有的参数，而即使没有参数也需要传递nil值站位，尤其如果一开始没有封装好导致后期要在方法里面添加一个参数，那么我们所有调用此方法的地方都需要进行修改，虽然这种可能性很小。所以后来采用了链式结构进行了封装。使用这种方法主要是借鉴了IOS中的布局适配框架[Masonry](https://github.com/SnapKit/Masonry)，关于链式编程更多的了解可以参考：

* [https://github.com/Wzxhaha/WZXProgrammingIdeas](https://github.com/Wzxhaha/WZXProgrammingIdeas)
* [http://www.ithao123.cn/content-1780874.html](http://www.ithao123.cn/content-1780874.html)

<!-- more -->

#### 链式结构的最终调用式例

``` objc
[[BHNetReqManager sharedManager].bh_requestUrl(@"http://binhan666.github.io/").
bh_requestType(GET).
bh_responseSerializer(HTTPResponseSerializer).
bh_parameters(nil) 
startRequestWithCompleteHandler:^(id response, NSError *error) {
}
```

如果我们想添加方法设置新的成员变量，则直接在后面添加方法，如：

``` objc
- (BHNetReqManager* (^)(id parameter))bh_newmothed
{
    return ^BHNetReqManager* (id parameter) {
        self.newvar = var;
        return self;
    };
}
```

而调用此方法则直接.bh_newmothed()即可

``` objc
[[BHNetReqManager sharedManager].bh_requestUrl(@"http://binhan666.github.io/").
bh_newmothed() 
startRequestWithCompleteHandler:^(id response, NSError *error) {
}
```

代码可以下载GITHUB中[BlogDemo](https://github.com/binhan666/BlogDemo)进行查看。            