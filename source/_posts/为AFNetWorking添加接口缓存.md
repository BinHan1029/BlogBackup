title: 为AFNetWorking添加接口数据缓存
date: 2015-04-12 22:33:48
tags:
- 移动开发
- iOS
---

### 关于NSURLSession的缓存
NSURLSession是iOS7之后对NSURLConnection更进一步的优化封装，可通过NSURLSessionConfiguration对其进行初始化设置，其中requestCachePolicy属性设置就是配置获取得到NSURLResponse之后的缓存策略：

``` objc
typedef NS_ENUM(NSUInteger, NSURLRequestCachePolicy)
{
	// NSURLRequest默认的cache policy，使用Protocol协议定义
    NSURLRequestUseProtocolCachePolicy = 0,
    NSURLRequestReloadIgnoringLocalCacheData = 1,
    // 忽略缓存直接从原始地址下载
    NSURLRequestReloadIgnoringCacheData = NSURLRequestReloadIgnoringLocalCacheData,
	// 只有在cache中不存在data时才从原始地址下载
    NSURLRequestReturnCacheDataElseLoad = 2,
    // 只使用cache数据，如果不存在cache，请求失败;用于没有建立网络连接离线模式
    NSURLRequestReturnCacheDataDontLoad = 3,
    // 忽略本地和远程的缓存数据，直接从原始地址下载，与NSURLRequestReloadIgnoringCacheData类似，苹果未实现。
    NSURLRequestReloadIgnoringLocalAndRemoteCacheData = 4, // Unimplemented
	// 验证本地数据与远程数据是否相同，如果不同则下载远程数据，否则使用本地数据，苹果未实现
    NSURLRequestReloadRevalidatingCacheData = 5, // Unimplemented
};
```

<!-- more -->

### 关于AFNetWorking的缓存
其实AFNetWorking(特指3.x版本)本质上讲就是基于对NSURLSession的再一次封装，所以它默认就已经可以使用NSURLCache缓存，即我们可以直接使用NSURLCache进行缓存设置，而NSURLCache是一个NSURLRequest对应一个NSURLResponse进行缓存的。

``` objc
- (nullable NSCachedURLResponse *)cachedResponseForRequest:(NSURLRequest *)request;
```
我们正常的使用AFN进行get请求如下：

``` objc
	AFHTTPSessionManager *manager = [self setupAFHTTPSessionManager];
    // 此处实际上会对AFHTTPSessionManager进行一些初始化设置
    NSURLSessionDataTask *task = [manager GET:self.requestUrl parameters:self.parameters progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        handler(responseObject, nil);
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        handler(nil, error);
    }];
```

然后get方法创建完成后会返回一个NSURLSessionDataTask对象，我们通过task.originalRequest即可获取上一次的NSURLRequest对象，通过它我们即可以从cache中获取缓存数据。

``` objc
	NSCachedURLResponse *reaponse = [[NSURLCache sharedURLCache] cachedResponseForRequest:a.originalRequest];
	if (reaponse)
	{
		handler(reaponse.data, nil);
		[task cancel];
	}
```

但是我们在第一进行请求的时候肯定是没有缓存的，所以在第一次缓存成功后需要对返回的NSURLResponse对象进行缓存：

``` objc
 	NSData *data = [NSJSONSerialization dataWithJSONObject:responseObject options:0 error:nil];
	NSCachedURLResponse * cachedResponse = [[NSCachedURLResponse alloc] initWithResponse:task.response data:data];
	[[NSURLCache sharedURLCache] storeCachedResponse:cachedResponse forRequest:task.originalRequest];
```

### 封装我们自己的NSCache
主要最对缓存数据进行过期处理

``` objc
@interface BHCustomURLCache : NSURLCache

+ (instancetype)standardURLCache;

@end

#import "BHCustomURLCache.h"

static NSString * const CustomURLCacheExpirationKey = @"CustomURLCacheExpiration";
static NSTimeInterval const CustomURLCacheExpirationInterval = 600;

@interface BHCustomURLCache()

@end

@implementation BHCustomURLCache

+ (instancetype)standardURLCache {
    static BHCustomURLCache *_standardURLCache = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _standardURLCache = [[BHCustomURLCache alloc] initWithMemoryCapacity:(2 * 1024 * 1024) diskCapacity:(100 * 1024 * 1024) diskPath:nil];
    });
    return _standardURLCache;
}
                  
#pragma mark - NSURLCache
                  
- (NSCachedURLResponse *)cachedResponseForRequest:(NSURLRequest *)request
{
    NSCachedURLResponse *cachedResponse = [super cachedResponseForRequest:request];
    if (cachedResponse)
    {
      NSDate* cacheDate = cachedResponse.userInfo[CustomURLCacheExpirationKey];
      NSDate* cacheExpirationDate = [cacheDate dateByAddingTimeInterval:CustomURLCacheExpirationInterval];
      if ([cacheExpirationDate compare:[NSDate date]] == NSOrderedAscending)
      {
          [self removeCachedResponseForRequest:request];
          return nil;
      }
    }
    return cachedResponse;
}


- (void)storeCachedResponse:(NSCachedURLResponse *)cachedResponse forRequest:(NSURLRequest *)request
{
    NSMutableDictionary *userInfo = [NSMutableDictionary dictionaryWithDictionary:cachedResponse.userInfo];
    userInfo[CustomURLCacheExpirationKey] = [NSDate date];
    
    NSCachedURLResponse *modifiedCachedResponse = [[NSCachedURLResponse alloc] initWithResponse:cachedResponse.response data:cachedResponse.data userInfo:userInfo storagePolicy:cachedResponse.storagePolicy];
    
    [super storeCachedResponse:modifiedCachedResponse forRequest:request];
}

@end
```

### 最后
关于实际开发中，接口数据可能需要实时获取最新的数据，这就要需要我们自己修改接口数据刷新策略，一种常见的做法，每次接口请求的时候都带上上次请求的时间戳，当服务端有新数据返回时即解析最新的数据，重新加入缓存；当没有新数据时则可以通过返回状态码302的方式通知客户端直接获取缓存数据。

代码可以下载GITHUB中[BlogDemo](https://github.com/binhan666/BlogDemo)进行查看。