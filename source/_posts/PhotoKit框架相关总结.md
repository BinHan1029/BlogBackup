---
title: PhotoKit框架相关总结
date: 2016-03-17 22:41:13
tags:
- 移动开发
- iOS
---

## PhotoKit中相关类介绍

* PHAsset：代表一个照片库中的一个资源，如一个照片/一个视频
* PHFetchResult：PHAsset 的集合，对应照片应用中的一个相册
* PHFetchOptions：可以通过配置对资源进行排序，如时间
* PHImageManager：用于从 PHFetchResult 中获取 PHAsset，可以设置相关参数，获取视频/图片/或者将一个PHAsset转化为NSData
* PHImageRequestOptions：获取 PHAsset 资源时的相关设置，获取资源的方式(同步/异步)，资源的质量等。

![Alt text](/assets/blogImg/PhotoKit_1.png)

<!-- more -->

## 获取相册

``` objc
	// 列出所有相册智能相册
    PHFetchResult *smartAlbums = [PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeSmartAlbum subtype:PHAssetCollectionSubtypeAlbumRegular options:nil];
    [smartAlbums enumerateObjectsUsingBlock:^(PHAssetCollection * _Nonnull collection, NSUInteger idx, BOOL *stop) {
        [self.dataArray addObject:collection];
    }];
    // 列出所有用户创建的相册
    PHFetchResult *topLevelUserCollections = [PHCollectionList fetchTopLevelUserCollectionsWithOptions:nil];
    [topLevelUserCollections enumerateObjectsUsingBlock:^(PHAssetCollection * _Nonnull collection, NSUInteger idx, BOOL *stop) {
        [self.dataArray addObject:collection];
    }];
``` 
当然在这里我们可以在加入集合前对某些相册进行过滤，如全景图片，最近删除我在项目中会直接将全景图片过滤掉，当然也可以只去在特定需求下仅仅展示特定的相册，这里主要会用到 PHAssetCollection 里 assetCollectionSubtype 和 assetCollectionType 属性，当然这些属性可以直接在 PhotosTypes.h 文件下找到：

如：

PHAssetCollectionSubtypeSmartAlbumUserLibrary 用户相册
PHAssetCollectionSubtypeSmartAlbumPanoramas 全景图片
PHAssetCollectionSubtypeSmartAlbumFavorites 个人收藏
PHAssetCollectionSubtypeSmartAlbumScreenshots 屏幕快照
PHAssetCollectionSubtypeSmartAlbumSelfPortraits 自拍
PHAssetCollectionSubtypeSmartAlbumPanoramas 全景照片
PHAssetCollectionSubtypeSmartAlbumTimelapses 延时视频
PHAssetCollectionSubtypeSmartAlbumRecentlyAdded 最近添加
PHAssetCollectionSubtypeSmartAlbumBursts 连拍
PHAssetCollectionSubtypeSmartAlbumVideos 视频

1000000201  最近删除

##### 获取相册名称

``` objc
NSString *localizedTitle = assetCollection.localizedTitle
``` 
此时我们获取的资源集合在PHAssetCollection中，我们需要将其转化为PHFetchResult：

``` objc
PHFetchResult *fetchResult = [PHAsset fetchAssetsInAssetCollection:assetCollection options:nil];
``` 
当然还可以对资源继续时间/描述等排序：

``` objc
/**
 对相册资源继续排序

 @param collection collection
 @return 返回按时间倒序后的集合
 */
-(PHFetchOptions *)fetchOptions
{
    PHFetchOptions *options = [[PHFetchOptions alloc] init];
    options.sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"creationDate" ascending:NO]];
    return options;
}
``` 

## 获取 PHAsset 资源
需要使用到PHImageManager，他是一个单例对象，通过 PHAsset 对象我们可以获取到我们真正使用的图像或者视频资源。

如相片：

``` objc
PHImageRequestOptions *imageRequestOptions = [[PHImageRequestOptions alloc] init];
imageRequestOptions.networkAccessAllowed = YES;
imageRequestOptions.resizeMode = PHImageRequestOptionsResizeModeFast;
imageRequestOptions.deliveryMode = PHImageRequestOptionsDeliveryModeOpportunistic;

[[PHImageManager defaultManager] requestImageDataForAsset:asset options:options resultHandler:^(NSData * _Nullable imageData, NSString * _Nullable dataUTI, UIImageOrientation orientation, NSDictionary * _Nullable info) {

}];
``` 
如视频：

``` objc
PHVideoRequestOptions *options = [[PHVideoRequestOptions alloc] init];
options.networkAccessAllowed = YES;
options.version = PHVideoRequestOptionsVersionOriginal;
options.deliveryMode = PHImageRequestOptionsDeliveryModeOpportunistic;
[[PHImageManager defaultManager] requestAVAssetForVideo:asset options:options resultHandler:^(AVAsset * _Nullable avAsset, AVAudioMix * _Nullable audioMix, NSDictionary * _Nullable info) {
 
}];
``` 

关于 options 的具体参数可以查看各自的头文件，这里很重要的一点是 requestImageDataForAsset/requestAVAssetForVideo 接口返回的 PHImageRequestID，可见 Apple 将开发者获取相册资源看做是一个请求，通常由于 PHAsset 都在本地，即使读取有一个 IO 及内存拷贝的操作，如果不是极限要求通常还是比较快的，但是我们知道相册中的资源是可以同步到 iCloud 中的，此时 requestID 的作用就体现出来了，我们可以在重新选择需要的资源时，可以将上次还没有从 iCloud 中同步过来的资源进行取消操作，就像这样

``` objc
[[PHImageManager defaultManager] cancelImageRequest:self.lastPHImageRequestID];
``` 
## 监听相册数据变化
当应用获取到用户授权可以访问相册资源时，很重要的一点就是我们如果同步相册中的资源，如用户将应用挂起后，重新拍了一张照片、录了一段新的视频、截图，甚至是将相册中的资源进行了删除，当然后者更为严重，这将导致极端情况下再返回应用时可能导致数据错乱甚至崩溃。好在 Apple 为我们提供了 PHPhotoLibraryChangeObserve。我们可以在程序中需要的地方添加监听，然后在不需要时进行移除。

``` objc
[[PHPhotoLibrary sharedPhotoLibrary] registerChangeObserver:self];

#pragma mark    PHPhotoLibraryChangeObserver

- (void)photoLibraryDidChange:(PHChange *)changeInstance
{

}

-(void)dealloc
{
    [[PHPhotoLibrary sharedPhotoLibrary] unregisterChangeObserver:self];
}

``` 

### 最后
PHAsset 是属于 iPhone 相册相关操作范围内的概念，PHAsset 并不是一个真正意义上的一个文件，我们通常获取到 PHAsset 后需要将真正的文件保存到沙盒中再将其上传到 CDN。推荐阅读
[ALAsset/PHAsset 中的图片和视频文件](http://io.upyun.com/2016/03/23/the-real-files-in-alasset-and-phasset/)

相关资料：
[https://developer.apple.com/reference/photos/phphotolibrarychangeobserver?language=objc](https://developer.apple.com/reference/photos/phphotolibrarychangeobserver?language=objc)








	
