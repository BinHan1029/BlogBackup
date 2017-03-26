title: JSPatch 使用小结
date: 2015-12-03 20:40:15
tags:
- 移动开发
- iOS
---

众所周知，iOS 应用审核机制，所以当我们的线上应用面对突如起来的 bug 的时候会显得很手足无措，有时候仅仅可能就是添加一两行代码解决这个问题却要因为审核等待一周或者美国人过节就会有甚至更长的审核周期。所以hoxfix热修复的作用简直就是掉炸天的功能啊。而 JSPatch 就是这样一个极小的轻量级框架，它可以使用 JavaScript 调用任何 Objective-C 的原生接口，替换任意 Objective-C 原生方法。目前主要用于下发 JS 脚本替换原生 Objective-C 代码，实时修复线上 bug。

<!-- more -->

## 关于 JSPatch
[GITHUB地址](https://github.com/bang590/JSPatch)，除了查看demo运行外，在 [Wiki](https://github.com/bang590/JSPatch/wiki)里面我们也可以看到更详细的介绍说明。安装同样支持 cocoaPod，快捷方便。这里不具体更多的说明，只是记录下再使用的过程中的关键点。

## 服务端的patch文件部署问题
这里我们将补丁文件打包后通过存放在自己的服务器上，按照 [JSPatch Loader 使用文档](https://github.com/bang590/JSPatch/wiki/JSPatch-Loader-%E4%BD%BF%E7%94%A8%E6%96%87%E6%A1%A3)使用说明，采用版本名作为文件夹，然后将对应的补丁文件存放在对应文件夹下，并修改 JPLoader 的 rootUrl 为对应的服务器地址即可。可是考虑到服务器地址写死在本地可能后期补丁文件可能放在另一台服务器的情况或者不可控的因素，最终还是让服务器端将完整的补丁文件地址返回给客户端，而客户端也不去关心服务器端是否按照什么样的逻辑或者方式去存放补丁文件。而这样做我们就需要修改 JPLoader 下的 + (void)updateToVersion:(NSInteger)version callback:(JPUpdateCallback)callback；方法实现修改为下面的逻辑，虽然我通常不建议修改第三方库文件。



``` objc
+ (void)updateToVersion:(NSInteger)version patchpath:(NSString *)patchpath callback:(JPUpdateCallback)callback;
{
    NSString *appVersion = [[[NSBundle mainBundle] infoDictionary] objectForKey:@"CFBundleShortVersionString"];
    
    if (JPLogger) JPLogger([NSString stringWithFormat:@"JSPatch: updateToVersion: %@", @(version)]);
    
    // create url request
//    NSString *downloadKey = [NSString stringWithFormat:@"/%@/v%@.zip", appVersion, @(version)];
//    NSURL *downloadURL = [NSURL URLWithString:[rootUrl stringByAppendingString:downloadKey]];
    NSURL *downloadURL = [NSURL URLWithString:patchpath];
    NSURLRequest *request = [NSURLRequest requestWithURL:downloadURL cachePolicy:NSURLRequestReloadIgnoringLocalCacheData timeoutInterval:20.0];
    
    if (JPLogger) JPLogger([NSString stringWithFormat:@"JSPatch: request file %@", downloadURL]);
    *
    *
    *
    // success
            if (!isFailed) {
                if (JPLogger) JPLogger([NSString stringWithFormat:@"JSPatch: updateToVersion: %@ success", @(version)]);
                
                [[NSUserDefaults standardUserDefaults] setInteger:version forKey:kJSPatchVersion(appVersion)];
                [[NSUserDefaults standardUserDefaults] setObject:patchpath forKey:kLASTPATCHPATH(appVersion)];
                [[NSUserDefaults standardUserDefaults] synchronize];
                
                if (callback) callback(nil);
            }
            
            // clear temporary files
            [[NSFileManager defaultManager] removeItemAtPath:downloadTmpPath error:nil];
            [[NSFileManager defaultManager] removeItemAtPath:unzipVerifyDirectory error:nil];
            [[NSFileManager defaultManager] removeItemAtPath:unzipTmpDirectory error:nil];
        } else {
            if (JPLogger) JPLogger([NSString stringWithFormat:@"JSPatch: request error %@", error]);
            
            if (callback) callback(error);
        }
    }];
    [task resume];
}
``` 
这样在参数里面直接写入下载地址，而不考虑通过 rootUrl 和版本号拼接的方式。

## 更新机制
由于我们每次调用 JSPatch 的 updateToVersion 方法都会重新下载一次补丁文件，而这样显得并不是那么必要，所以客户端当检测到这次请求下载补丁地址与上一次的补丁地址相同的情况下，就不再调用 updateToVersion 方法，而上一次的的补丁地址我们会在上一次文件下载成功的回调里面通过 NSUserDefaults 保存起来，而原作者也是通过此方式将补丁的版本号保存在了 NSUserDefaults 里。

``` objc
                [[NSUserDefaults standardUserDefaults] setInteger:version forKey:kJSPatchVersion(appVersion)];
                [[NSUserDefaults standardUserDefaults] setObject:patchpath forKey:kLASTPATCHPATH(appVersion)];
                [[NSUserDefaults standardUserDefaults] synchronize];
``` 
## 更新频率
也就是我们刷新最新补丁接口的频率。首先我们应该按照作者的建议在程序启动的 -application:didFinishLaunchingWithOptions: 里第一句调用 run 这个接口，防止调用后执行 JSPatch 脚本过程中其他线程同时在执行相关代码，导致意想不到的问题。而刷新频率建议放在 applicationDidBecomeActive 这个方法里，并建议设置每隔1个小时检查一次。

## 最后
在线的 OC 转 JS 代码工具 [JSPatchConvertor](http://bang590.github.io/JSPatchConvertor/)，这里作者真得是很用心，基本的功能都实现了，更多的可以参考 [JSPatch 基础用法](https://github.com/bang590/JSPatch/wiki/JSPatch-%E5%9F%BA%E7%A1%80%E7%94%A8%E6%B3%95)，当然面对复杂的结构还是需要我们自己手动进行修改成JS代码的，因为它还不够聪明。
比如我们在使用SDWebImage框架的时候，工具修改后的imageView.sd_setImageWithURL，而这样会直接导致崩溃，需要修改为两个_下划线就可以了；而当抛出 'js exception: SyntaxError: At least one digit must occur after a decimal point'这样的错误可能就是因为我们出现的float数值后有了f字母。细节上更多的还是需要我们参考文档说明。

当然我们遇到了 bug 时，除了第一时间发布补丁，还是应该在新的版本中使用 OC 对 bug 进行修改，毕竟使用 JSPatch 还是一个紧急措施。


