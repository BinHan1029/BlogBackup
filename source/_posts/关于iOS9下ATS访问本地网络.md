---
title: 关于 iOS9 下 ATS 访问本地网络
date: 2017-01-03 22:05:00
tags:
- 移动开发
- iOS
---

这段时间可能所有的开发 iOS 的公司都在做应用的 HTTPS 适配，因为在 WWDC 16 中，Apple 表示从 2017 年 1 月 1 日起，所有的新提交 app 默认是不允许使用 NSAllowsArbitraryLoads 来绕过 ATS 限制的，也就是说，我们最好保证 app 的所有网络请求都是 HTTPS 加密的，否则可能会在应用审核时遇到麻烦。

由于 Apple 在这之前预留了足够的时间让我们来进行网络适配，所以在这之前我们大部分情况下将 NSAllowsArbitraryLoads 设置为 YES，这样网络请求不受 ATS 的限制了。


<!-- more -->


## ATS属性配置
当然 ATS 相关 NSAppTransportSecurity 下的诸多属性都是可选的，毕竟通常我们并不能保证我们跳转的web页面、多媒体资源、一些第三方的 SDK 等是一定支持 HTTPS 的，所以在这方面 Apple 给予了足够的可配置选项，当然在提交审核时我们需要提供一个"合理的解释"。

例如:

* NSAllowsArbitraryLoadsForMedia，默认值为NO，置为 YES 后，使用 AV Foundation 框架载入资源时不受 ATS 的限制；（iOS 10.0及以上支持，测试发现真机可行，模拟器未起作用）
* NSAllowsArbitraryLoadsInWebContent，默认值为 NO，置为 YES 后，使用 web view 的网络请求不受 ATS 限制；（iOS 10.0及以上支持）
* NSAllowsLocalNetworking，默认值为 NO，置为 YES 后，本地网络请求不受 ATS 限制；（iOS 10.0及以上支持）

所以我们发现这些字段都是 iOS10 下新添加的，这样就会引发我们应用在 iOS9 下可能访问 HTTP 资源失败。。。。这篇文章也并不是告诉我们怎么来升级应用的 HTTPS 的，关于 AFNetWorking 升级 HTTPS 二次验证证书的资源也有很多，这里主要记录我所在 iOS9 下所踩得坑，当然在写这篇博客的时候，Apple 也将 [ATS 无限期延期](https://developer.apple.com/news/?id=12212016b)了。

![Alt text](/assets/blogImg/ats_1.png)

## iOS9下访问本地网络 192.168.1.1
我们的项目是需要访问本地网络的，APP 运行时连接硬件产品发出的热点，而连接采用的是 IP 地址访问，测试的时候发现 iOS10 下无论开启/关闭 ATS，IP下的网络都能正常访问，而在 iOS9 下无论怎么配置 NSAppTransportSecurity 的相关参数都无法建立网络连接，当然我们尝试了设置 NSExceptionDomains 进行设置但是并没有效果，后来觉得既然是过滤域名显然对 IP 是无效的，而不同的是 Apple 在 iOS10 下直接让 IP 连接绕过了 ATS。

其实从某种侧面考虑，仅仅是涉及到局域网内的访问貌似也不会设计到数据安全问题，所以个人猜测 iOS9 下可能是 Apple 一开始推出 ATS 而没有全面考虑而导致本地网络无法访问的。当然也发现有更多的人受到同样问题的困扰，苹果的官方解释貌似还是需要使用 NSAllowsArbitraryLoads。。。

![Alt text](/assets/blogImg/ats_2.png)

## ATS支持诊断工具
HTTPS适配完成后，可以先使用/usr/bin/nscurl（OS X v10.11及以上系统支持）工具模拟进行ATS网络连接状况诊断，命令如下：

``` objc
/usr/bin/nscurl --ats-diagnostics [--verbose] URL
```

* ats-diagnostics 参数的设定，会模拟ATS属性的不同配置场景（NSAllowsArbitraryLoads、NSExceptionMinimumTLSVersion、NSExceptionRequiresForwardSecrecy 和 NSExceptionAllowsInsecureHTTPLoads 的不同组合）进行连接；
* verbose 指定时，可显示ATS不同配置场景的详细信息。

例，检测百度官网 https://www.baidu.com

```
Starting ATS Diagnostics

Configuring ATS Info.plist keys and displaying the result of HTTPS loads to https://www.baidu.com.
A test will "PASS" if URLSession:task:didCompleteWithError: returns a nil error.
================================================================================

Default ATS Secure Connection
---
ATS Default Connection
ATS Dictionary:
{
}
Result : PASS
---

================================================================================

Allowing Arbitrary Loads

---
Allow All Loads
ATS Dictionary:
{
    NSAllowsArbitraryLoads = true;
}
Result : PASS
---

================================================================================

Configuring TLS exceptions for www.baidu.com

---
TLSv1.2
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionMinimumTLSVersion = "TLSv1.2";
        };
    };
}
Result : PASS
---

---
TLSv1.1
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionMinimumTLSVersion = "TLSv1.1";
        };
    };
}
Result : PASS
---

---
TLSv1.0
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionMinimumTLSVersion = "TLSv1.0";
        };
    };
}
Result : PASS
---

================================================================================

Configuring PFS exceptions for www.baidu.com

---
Disabling Perfect Forward Secrecy
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionRequiresForwardSecrecy = false;
        };
    };
}
Result : PASS
---

================================================================================

Configuring PFS exceptions and allowing insecure HTTP for www.baidu.com

---
Disabling Perfect Forward Secrecy and Allowing Insecure HTTP
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionAllowsInsecureHTTPLoads = true;
            NSExceptionRequiresForwardSecrecy = false;
        };
    };
}
Result : PASS
---

================================================================================

Configuring TLS exceptions with PFS disabled for www.baidu.com

---
TLSv1.2 with PFS disabled
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionMinimumTLSVersion = "TLSv1.2";
            NSExceptionRequiresForwardSecrecy = false;
        };
    };
}
Result : PASS
---

---
TLSv1.1 with PFS disabled
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionMinimumTLSVersion = "TLSv1.1";
            NSExceptionRequiresForwardSecrecy = false;
        };
    };
}
Result : PASS
---

---
TLSv1.0 with PFS disabled
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionMinimumTLSVersion = "TLSv1.0";
            NSExceptionRequiresForwardSecrecy = false;
        };
    };
}
Result : PASS
---

================================================================================

Configuring TLS exceptions with PFS disabled and insecure HTTP allowed for www.baidu.com

---
TLSv1.2 with PFS disabled and insecure HTTP allowed
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionAllowsInsecureHTTPLoads = true;
            NSExceptionMinimumTLSVersion = "TLSv1.2";
            NSExceptionRequiresForwardSecrecy = false;
        };
    };
}
Result : PASS
---

---
TLSv1.1 with PFS disabled and insecure HTTP allowed
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionAllowsInsecureHTTPLoads = true;
            NSExceptionMinimumTLSVersion = "TLSv1.1";
            NSExceptionRequiresForwardSecrecy = false;
        };
    };
}
Result : PASS
---

---
TLSv1.0 with PFS disabled and insecure HTTP allowed
ATS Dictionary:
{
    NSExceptionDomains =     {
        "www.baidu.com" =         {
            NSExceptionAllowsInsecureHTTPLoads = true;
            NSExceptionMinimumTLSVersion = "TLSv1.0";
            NSExceptionRequiresForwardSecrecy = false;
        };
    };
}
Result : PASS
---

================================================================================
```


相关资料：

[https://yq.aliyun.com/articles/62563](https://yq.aliyun.com/articles/62563)

[http://stackoverflow.com/questions/30903923/app-transport-security-and-ip-addresses-in-ios9](http://stackoverflow.com/questions/30903923/app-transport-security-and-ip-addresses-in-ios9)

[https://forums.developer.apple.com/thread/6205](https://forums.developer.apple.com/thread/6205)





