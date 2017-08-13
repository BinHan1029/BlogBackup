---
title: TCP/UDP 通讯下包与流
date: 2016-09-18 22:39:03
tags:
- 移动开发
- iOS
---

### 关于 TCP 流与包的概念
首先需要明确一个概念，TCP 是以二进制流式传送数据的，既发送端与接收端成功建立连接下即可以不停的发送数据包，不同数据包间并没有明确的边界定义；而 UDP 发送数据的时候是按照一个一个的数据包去发送的。所以 TCP 发送数据是一个二进制流，而管道内的数据是一个个封装好的数据包。另外很重要的一点是，无论是发送端发送数据，还是接收端接收数据，都存在一个数据的缓冲区，而当发送端等待缓冲区满才发送数据，就会造成缓冲区有多个数据包，数据包可能会在开启 Nagle 算法的情况下进行合并发送；而接收端不及时处理缓冲区的数据包时，既同样造成缓冲区存在多个数据包，应用层可能会一次 read 完多个数据包。所以在发送端，我们需要对数据按照事先约定好的协议进行合理的封装，在接收端，我们需要按照协议对数据进行拆分。

<!-- more -->

### 关于 Nagle 算法
TCP 默认是会开启 Nagle 算法的，Nagle 算法主要做两件事：

* 只有上一个分组得到确认，才会发送下一个分组，这样的做法是为了优化网络；既当发送多个小数据包时，若上一个数据包的 ACK 返回较慢导致缓冲区数据包过多时，此算法会将多个小数据包合并为一个大包进行发送。因为在 TCP 协议下即使你发送一个字节，那么按照协议他也会被封装为一个数据包进行发送，这样额外添加 TCP header (传输层)和 IP header（网络层）信息来组装包，这样不光消耗流量，还可能造成网络拥堵。

* 另外接收端可能也会对 ACK 进行延迟返回，甚至可能将 ACK 添加到发送的数据包内进行返回。

这么做很重要的一个优点就是优化网络，减少小数据包的发送，但是这样一来一回都增加了延迟处理，就大大增加了网络延迟。当然发送端是可以对此算法继续关闭的，关闭方法，[点这里](https://github.com/robbiehanson/CocoaAsyncSocket/issues/325)。

### 对数据包进行边界划分
这里相对传统的做法，即是对数据包进行划分，然后进行添加“头部描述”。

下面是 [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket) UDP 下发送数据最终调用的方法，已经对参数进行注释。

``` objc
/**
 UDP 发送数据

 @param data 数据包
 @param host 接收端 host 地址
 @param port 接收端 port 端口
 @param timeout 超时  -1 即为不超时
 @param tag 标识
 */
- (void)sendData:(NSData *)data toHost:(NSString *)host port:(uint16_t)port
     withTimeout:(NSTimeInterval)timeout tag:(long)tag
```

举个例子，为数据添加“头部描述”，这里的做法其实就是在每次发送数据前，添加一个描述数据大小长度的数据包，以此来告诉接收端，有一点类似 HTTP 协议中的 Request Line 头，是对数据包大小长度的描述，当然我们还可以添加别的描述，类图数据类型，如是否为文本(txt)、图片(img)、文件(file)等，本质上是一个字典。

下面是发送端代码：

``` objc
- (void)sendMsg
{
	NSData *textData  = [@"测试文本" dataUsingEncoding:NSUTF8StringEncoding];
	[self sendData:data :@"txt"];
	NSData *imgData  = [NDData dataWithContentsOfFile:imageFile];
	[self sendData: imgData :@"img"];
	NSData *fileData  = [NDData dataWithContentsOfFile:file];
	[self sendData: fileData :@"file"];
}

/**
 发送数据

 @param data 数据包
 @param type 数据包类型
 */
- (void)sendData:(NSData *)data :(NSString *)type
{
    NSUInteger size = data.length;
    NSMutableDictionary *headDic = [NSMutableDictionary dictionary];
    [headDic setObject:type forKey:@"type"];
    [headDic setObject:[NSString stringWithFormat:@"%ld",size] forKey:@"size"];
    NSString *jsonStr = [self dictionaryToJson:headDic];
    NSData *lengthData = [jsonStr dataUsingEncoding:NSUTF8StringEncoding];
    NSMutableData *mData = [NSMutableData dataWithData:lengthData];
    //其实 [GCDAsyncSocket CRLFData] 就是一个 \r\n。我们用它来做头部的边界。
    [mData appendData:[GCDAsyncSocket CRLFData]];
    [mData appendData:data];
    [_udpSocket sendData:data toHost:kIp port:port withTimeout:-1 tag:-1];
}

/**
 将字典转化为字符串

 @param dic “头部描述”字典
 @return 描述文件内容
 */
- (NSString *)dictionaryToJson:(NSDictionary *)dic
{
    NSError *error = nil;
    NSData *jsonData = [NSJSONSerialization dataWithJSONObject:dic options:NSJSONWritingPrettyPrinted error:&error];
    return [[NSString alloc] initWithData:jsonData encoding:NSUTF8StringEncoding];
}
```

下面是接收端代码，主要思路为先将数据包的“头部描述”解析出来，然后在解析数据：

``` objc
- (void)udpSocket:(GCDAsyncUdpSocket *)sock didReceiveData:(NSData *)data
        fromAddress:(NSData *)address withFilterContext:(nullable id)filterContext
{
	 // 获取数据包“头部描述”
    if (!currentPacketHead) 
    {
        currentPacketHead = [NSJSONSerialization JSONObjectWithData:data
                             options:NSJSONReadingMutableContainers error:nil];
        NSUInteger packetLength = [currentPacketHead[@"size"] integerValue];
        [sock readDataToLength:packetLength withTimeout:-1 tag:-1];
        return;
    }
    if (!currentPacketHead) 
    {
        NSLog(@"error：当前数据包的头为空");
        return;
    }
    NSUInteger packetLength = [currentPacketHead[@"size"] integerValue];
    //数据包出错
    if (packetLength <= 0 || data.length != packetLength) 
    {
        NSLog(@"error：当前数据包数据大小不正确");
        return;
    }
    NSString *type = currentPacketHead[@"type"];
    if ([type isEqualToString:@"img"]) 
    {
    	UIImage *image = [UIImage imageWithData:data];
       NSLog(@"数据包为图片");
    }
    else if ([type isEqualToString:@"file"]) 
    {
        NSLog(@"数据包为文件");
    }
    else
    {
        NSString *msg = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"数据包为文本");
    }
    currentPacketHead = nil;
```

### 另外一种形式对数据包边界划分
在某些项目中，发送端与接收端的数据包的内容可能是事先定义好协议，规定数据包某一字节为数据包长度。例如传递每一个字节都是是 byte 类型数据，我们可以对数据的某一位定义为该条数据包的长度，那么接收端在接收到数据时，先解析数据事先约定好的长度标志位，即可获取到数据包的长度，伪代码如下：

``` objc
uint8_t b = 0;
NSMutableData *data = [[NSMutableData alloc] init];
b = 0xA0; [ret appendBytes:&b length:1];
b = 0x04; [ret appendBytes:&b length:1];
b = 0xE1; [ret appendBytes:&b length:1];
b = 0x02; [ret appendBytes:&b length:1];
[_udpSocket sendData:data toHost:kIp port:port withTimeout:-1 tag:-1];
```

我们可以看到发送的 data 数据数据长度是4位，这里假定事先约定长度的为第 2 个字节。

而接受端伪代码如下，用此次获取 data 数据的实际长度与约定位数长度的值进行比对，当然如果数据的长度大于约定位数的长度时，我们需要递归拆包：

``` objc
- (void)udpSocket:(GCDAsyncUdpSocket *)sock didReceiveData:(NSData *)data
        fromAddress:(NSData *)address withFilterContext:(nullable id)filterContext
{
	uint8_t *buf = (uint8_t*)[data bytes];
	// 此次接收的 data 数据包大小
	NSUInteger bufLen = data.length;
	//其中第一条数据的大小
	uint8_t len = buf[1];
	if (bufLen < len) 
	{
        NSLog(@"error：当前数据包数据大小不正确");
        return;
	}
	[self cutData:data];
}

- (void)cutData:(NSData *)data
{
	uint8_t *buf = (uint8_t*)[data bytes];
	NSUInteger bufLen = data.length;
	uint8_t len = buf[1];
	if (length == cmdLength) 
	{
		NSLog(@"数据正好完整");
	}
	else
	{
		NSLog(@"数据包较大时，递归拆包获取每一个小数据包");
		NSData *data = [[NSData alloc] initWithBytes: buf +cmdLength length:length-cmdLength];
		[self cutData:data];
	}
}

```

当然对数据的封装、拆分还可以有其他的策略，比如采用特殊的字符对数据包进行分割，只要这个字符在数据包不会出现，在某些特定的场景也是可以的。







