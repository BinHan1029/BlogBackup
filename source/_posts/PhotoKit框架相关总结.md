---
title: PhotoKit框架相关总结
date: 2016-03-17 22:41:13
tags:
- 移动开发
- iOS
---

## PhotoKit中相关类介绍

* PHAsset：代表一个照片库中的一个资源，如一个照片/一个视频
* PHFetchResult：PHAsset 的集合，对应照片的一个相册
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
当然在这里我们可以在加入集合前对某些相册进行过滤，如全景图片，最近删除我在项目中会直接将全景图片过滤掉，当然也可以只去在特定需求下仅仅展示特定的相册，这里主要会用到 PHAssetCollection里assetCollectionSubtype和assetCollectionType属性，当然这些属性可以直接在PhotosTypes.h文件下找到：

如：

PHAssetCollectionSubtypeSmartAlbumPanoramas 全景图片

1000000201  最近删除枚举值

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

### 获取 PHAsset 资源
需要使用到PHImageManager，他是一个单例对象，在获取资源时我们可以获取一张照片







### 最后
PHAsset 是属于 iPhone 相册相关操作范围内的概念，PHAsset 并不是一个真正意义上的一个文件，我们通常获取到 PHAsset 后需要将真正的文件保存到沙盒中再将其上传到 CDN。推荐阅读
[ALAsset/PHAsset 中的图片和视频文件](http://io.upyun.com/2016/03/23/the-real-files-in-alasset-and-phasset/)









	
