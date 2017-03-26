title: 封装 AVPlayer (包含触摸滑动快进/快退 调节音量)及相关注意点
date: 2015-08-15 23:28:21
tags:
- 移动开发
- iOS
---

在iOS视频开发中，传统的方案可以直接使用系统的 MPMoviePlayerController 既可以直接将系统的播放页面掉出来，更贴心的为我们添加了控制条，全屏放大及暂停按钮。但是实际中我们可以需要针对播放器做更多的自定义设置，继而更多的是会采用 AVPlayer，因为 AVPlayer 提供了更为强大的功能，虽然在使用的过程会比较麻烦，但是确实能为我们的 app 提供更好的视频播放体验提供前提。

<!-- more -->

### 初步认识 AVPlayer
在这之前我们先介绍几个相关类：

* AVPlayerItem：一个视频资源对应一个 [AVPlayerItem](https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVPlayerItem_Class/)对象，当你需要循环播放多个视频资源时也需创建多个AVPlayerItem对象，并将其添加在一个数组中。我们可以通过静态方法 [playerItemWithURL](https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVPlayerItem_Class/) 进行创建，同样也可以通过 [AVAsset](https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVAsset_Class/index.html#//apple_ref/occ/cl/AVAsset)，而根据两种创建方法的不同，我们即知道加载本地视频资源还是网络视频资源。
* CMTime：关于 [CMTime](https://developer.apple.com/library/ios/documentation/CoreMedia/Reference/CMTime/) 我们可以很方便的获取到视频资源的精准播放长度长度，及快速让播放器定位到指定时间进行播放。
* AVPlayer: [AVPlayer](https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVPlayer_Class/) 我们能把播放器做得那么溜全靠它了，视频资源就是交给它进行处理的。
* AVPlayerLayer: [AVPlayerLayer](https://developer.apple.com/library/ios/documentation/AVFoundation/Reference/AVPlayerLayer_Class/) 其实 AVPlayer 仅仅处理视频资源，但是视频的画面其实是加载到AVPlayerLayer上的。



根据以上的简单介绍，我们可以做一个简单的视频播放器，只要仅仅能看到播放画面及听到声音就可以了，这里需要特别注意 AVPlayer 必须设置为成员变量，在局部变量中因为方法执行完后即被释放了会导致视频播放失败：

``` objc
NSString *filePath = [[NSBundle mainBundle] pathForResource:@"snow" ofType:@"mp4"];
NSURL *sourceMovieURL = [NSURL fileURLWithPath:filePath];
AVAsset *movieAsset = [AVURLAsset URLAssetWithURL:sourceMovieURL options:nil];
AVPlayerItem *playerItem = [AVPlayerItem playerItemWithAsset:movieAsset];
self.player = [AVPlayer playerWithPlayerItem:playerItem];
self.playerLayer = [AVPlayerLayer playerLayerWithPlayer:self.player];
self.playerLayer.frame = self.view.bounds;
self.playerLayer.videoGravity = AVLayerVideoGravityResizeAspect;
[self.view.layer insertSublayer:self.playerLayer atIndex:0];
[self.player play];
```

### 为我们的播放器添加控制条
这一步主要为在播放器上添加常规按钮，如返回、播放/暂停、全屏、播放时间进度条等一些为用户和播放器的交互试图。

首先我们自定义一个 BHPlayerControlView，这里面包含了所有的操作按钮及进度条、展示时间的 Label、当然还包括加载视频缓冲时的菊花 UIActivityIndicatorView，以下主要为设置控制条子试图约束的代码：

``` objc
/**
 *  设置子空间约束，采用 Masonry 设置约束，不需要重写 layoutSubviews 方法
 */
- (void)makeSubViewsConstraints
{
    [self.topImageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.leading.trailing.top.equalTo(self);
        make.height.mas_equalTo(80);
    }];
    [self.backBtn mas_makeConstraints:^(MASConstraintMaker *make) {
        make.leading.equalTo(self.mas_leading).offset(15);
        make.top.equalTo(self.mas_top).offset(5);
        make.width.height.mas_equalTo(30);
    }];
    [self.bottomImageView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.leading.trailing.bottom.equalTo(self);
        make.height.mas_equalTo(50);
    }];
    [self.startBtn mas_makeConstraints:^(MASConstraintMaker *make) {
        make.leading.equalTo(self.bottomImageView.mas_leading).offset(5);
        make.bottom.equalTo(self.bottomImageView.mas_bottom).offset(-5);
        make.width.height.mas_equalTo(30);
    }];
    [self.fullScreenBtn mas_makeConstraints:^(MASConstraintMaker *make) {
        make.width.height.mas_equalTo(30);
        make.trailing.equalTo(self.bottomImageView.mas_trailing).offset(-5);
        make.centerY.equalTo(self.startBtn.mas_centerY);
    }];
    [self.currentTimeLabel mas_makeConstraints:^(MASConstraintMaker *make) {
        make.leading.equalTo(self.startBtn.mas_trailing).offset(-3);
        make.centerY.equalTo(self.startBtn.mas_centerY);
        make.width.mas_equalTo(43);
    }];
    [self.totalTimeLabel mas_makeConstraints:^(MASConstraintMaker *make) {
        make.trailing.equalTo(self.fullScreenBtn.mas_leading).offset(3);
        make.centerY.equalTo(self.startBtn.mas_centerY);
        make.width.mas_equalTo(43);
    }];
    [self.progressView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.leading.equalTo(self.currentTimeLabel.mas_trailing).offset(4);
        make.trailing.equalTo(self.totalTimeLabel.mas_leading).offset(-4);
        make.centerY.equalTo(self.startBtn.mas_centerY);
    }];
    [self.videoSlider mas_makeConstraints:^(MASConstraintMaker *make) {
        make.leading.equalTo(self.currentTimeLabel.mas_trailing).offset(4);
        make.trailing.equalTo(self.totalTimeLabel.mas_leading).offset(-4);
        make.centerY.equalTo(self.currentTimeLabel.mas_centerY).offset(-1);
        make.height.mas_equalTo(30);
    }];
    [self.progressIndicatorLabel mas_makeConstraints:^(MASConstraintMaker *make) {
        make.width.mas_equalTo(160);
        make.height.mas_equalTo(40);
        make.center.equalTo(self);
    }];
    [self.activity mas_makeConstraints:^(MASConstraintMaker *make) {
        make.center.equalTo(self);
    }];
    [self.repeatBtn mas_makeConstraints:^(MASConstraintMaker *make) {
        make.center.equalTo(self);
    }];
}
```
当然还需要对外暴漏我们的控制器试图的隐藏及显示，及一些子试图初始某些属性设置方法：

``` objc
/**
 *  显示控制器
 */
- (void)showControlView
{
    self.topImageView.alpha = 1;
    self.bottomImageView.alpha = 1;
    self.backBtn.alpha = 1;
}

/**
 *  隐藏控制器
 */
- (void)hideControlView
{
    self.topImageView.alpha = 0;
    self.bottomImageView.alpha = 0;
    self.backBtn.alpha = 0;
}

/** 
 * 重置ControlView 
 */
- (void)resetControlView
{
    self.videoSlider.value = 0;
    self.progressView.progress = 0;
    self.currentTimeLabel.text = @"00:00";
    self.totalTimeLabel.text = @"00:00";
    self.progressIndicatorLabel.hidden = YES;
    self.repeatBtn.hidden = YES;
    self.backgroundColor = [UIColor clearColor];
}
```

### 封装 AVPlayer
以上已经自定义好了我们的控制器试图，下面我们开始封装 AVPlayer。

首先我们需要对 AVPlayerItem 设置监听，监听我们的视频资源的状态，这里通过 KVO 监听播放器的状态：

``` objc
    //视频添加kvo监听
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(moviePlayDidEnd:) name:AVPlayerItemDidPlayToEndTimeNotification object:self.playerItem];
    // 视频资源的加载状态
    [self.playerItem addObserver:self forKeyPath:@"status" options:NSKeyValueObservingOptionNew context:nil];
    // 获取缓冲区
    [self.playerItem addObserver:self forKeyPath:@"loadedTimeRanges" options:NSKeyValueObservingOptionNew context:nil];
    // 缓冲区空了，需要等待数据
    [self.playerItem addObserver:self forKeyPath:@"playbackBufferEmpty" options:NSKeyValueObservingOptionNew context:nil];
    // 缓冲区有足够数据可以播放了
    [self.playerItem addObserver:self forKeyPath:@"playbackLikelyToKeepUp" options:NSKeyValueObservingOptionNew context:nil];
```

然后在下面的回调方法中分别针对不同的状态进行不同的逻辑判断，主要为视频成功加载后需要为我们的播放器添加触摸手势，这里不建议直接重写 touches 的四个回调事件，因为这样会更新我们触摸逻辑的复杂度，其次视频如果加载失败，还需要额外的判断，不划算。所以这里建议直接使用手势，滑动事件监听手势的回调方法就可以了。

``` objc
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context；
```

关于计算缓冲进度：

``` objc
- (NSTimeInterval)availableDuration
{
    NSArray *loadedTimeRanges = [[_player currentItem] loadedTimeRanges];
    CMTimeRange timeRange = [loadedTimeRanges.firstObject CMTimeRangeValue];// 获取缓冲区域
    float startSeconds = CMTimeGetSeconds(timeRange.start);
    float durationSeconds = CMTimeGetSeconds(timeRange.duration);
    NSTimeInterval result = startSeconds + durationSeconds;// 计算缓冲总进度
    return result;
}
```
loadedTimeRanges 这个属性是一个数组，里面装的是本次缓冲的时间范围，这个用一个结构体 CMTimeRange 表示，start 表示本次缓冲时间的起点，duratin 表示本次缓冲持续的时间范围。

关于 CMTime,我们可以通过 CMTimeGetSeconds([_player currentTime]) 获取当前播放器的时间，但是通常我们可能需要换算为小时:分钟:秒这种格式。

关于触摸的回调事件，主要为控制左右滑动时视频的快进快退逻辑，及上下滑动时的音量控制逻辑，具体可参考代码：

``` objc
/**
 *  pan手势事件
 *
 *  @param pan UIPanGestureRecognizer
 */
- (void)panDirection:(UIPanGestureRecognizer *)pan
```
还有一个很重要的触摸事件，就是滑动我们的进度条 UISlider 的回调，这个回调里面主要为处理 NSTimer 及滑动过程中进度指示 Label 的文本内容设置：

``` objc
/**
 *  slider开始滑动事件
 *
 *  @param slider UISlider
 */
- (void)progressSliderTouchBegan:(UISlider *)slider;
 /**
 *  slider滑动中事件
 *
 *  @param slider UISlider
 */
- (void)progressSliderValueChanged:(UISlider *)slider
/**
 *  slider结束滑动事件
 *
 *  @param slider UISlider
 */
- (void)progressSliderTouchEnded:(UISlider *)slider;
```

最后就是关于视频播放完成后，我们需要对播放器及播放器控制器进行重置：

``` objc
/**
 *  播放完了
 *
 *  @param notification 通知
 */
- (void)moviePlayDidEnd:(NSNotification *)notification
{
    self.state = BHPlayerStateStopped;
    self.playDidEnd = YES;
    self.controlView.repeatBtn.hidden = NO;
    // 初始化显示controlView为YES
    self.isMaskShowing = NO;
    // 延迟隐藏controlView
    [self animateShow];
}
```
在这里提醒一点很重要的地方，当我们的时候及处于静音的时候需要做以下处理：

``` objc
	// 使用这个category的应用不会随着手机静音键打开而静音，可在手机静音下播放声音
    NSError *setCategoryError = nil;
    BOOL success = [[AVAudioSession sharedInstance] setCategory: AVAudioSessionCategoryPlayback error: &setCategoryError];
    if (!success) { /* handle the error in setCategoryError */ }
```
代码可以下载GITHUB中[BlogDemo](https://github.com/binhandev/BlogDemo)进行查看。 



