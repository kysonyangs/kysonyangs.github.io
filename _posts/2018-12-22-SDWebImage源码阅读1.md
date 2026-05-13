---
layout: post
title: "SDWebImage源码阅读1"
date: 2018-12-22 14:20:10 +0800
tags: ["源码"]
---

# [SDWebImage](https://github.com/rs/SDWebImage)

SDWebImage具有缓存支持的异步映像下载程序。并添加了像UI元素分类UIImageView、UIButton、MKAnnotationView，可以直接为这些UI元素添加图片。

**功能**

* 对UIImageView、UIButton、MKAnnotationView添加Web图像和高速缓存管理
* 异步图像下载器
* 具有自动缓存到期处理的异步内存+磁盘映像缓存
* 背景图像解压缩以避免帧率下降
* 逐步加载图像（包括GIF图）
* 缩略图图像解码可节省大图像的CPU和内存
* 可扩展的图像编码器以支持海量图像格式，例如WebP
* 动画图像的全栈解决方案，可在CPU和内存之间保持平衡
* 可以自定义和组合的转换，可在下载后立即应用于图像
* 可定制的多缓存系统
* 可以自定义加载器（如照片库）来扩展图像加载功能
* 图像加载指示器
* 图像加载过渡动画
* 保证不会多次下载相同的URL
* 保证不会重复尝试伪造的URL
* 保证主线程永远不会被阻塞
* 支持OC和Swift

**支持的图片格式**

* Apple系统支持的图像格式（JPEG，PNG，TIFF，HEIC等），包括GIF / APNG / HEIC动画
* WebP格式，包括动画WebP（使用SDWebImageWebPCoder项目）
* 支持可扩展的编码器插件，用于BPG，AVIF等新图像格式。以及矢量格式，例如PDF，SVG。查看图像编码器插件列表中的所有[列表](https://github.com/SDWebImage/SDWebImage/wiki/Coder-Plugin-List)

**更多模块**

查看 [GitHub](https://github.com/SDWebImage/SDWebImage)，有许多基于 SDWebImage 的三方库。

## 源码分析

SDWebImage架构图
![](https://raw.githubusercontent.com/SDWebImage/SDWebImage/master/Docs/Diagrams/SDWebImageHighLevelDiagram.jpeg)

时序图
![](https://raw.githubusercontent.com/SDWebImage/SDWebImage/master/Docs/Diagrams/SDWebImageSequenceDiagram.png)

从SDWebImage中提供的架构图中我们可以大概的看出，整个库分为两层，Top Level、Base Module。

* Top Level：当UIImageView调用加载image方法，会进入SDWebImage中的UIImageView分类，在分类中调用负责加载UIImage的核心代码块ImageManager，其主要负责调度Image Cache/Image Loader，这两者分别从缓存或者网络端加载图片，并且又进行了细分。Cache中获取图片业务，拆分到了memory/disk两个分类中；Image Loader中又分为从网络端获取或者从系统的Photos中获取。
* Base Module：获取到图片的二进制数据处理，二进制解压缩，二进制中格式字节判断出具体的图片类型。

**核心类**

* UIImageView+WebCache：SDWebImage加载图片的入口，随后都会进入sd_setImageWithURL: placeholderImage: 中。
* UIView+WebCache：由于分类中不能添加成员变量，所以runtime关联了sd_imageURL图片的url、sd_imageProgress下载进度；sd_cancelCurrentImageLoad方法取消当前下载，sd_imageIndicator设置当前加载动画，sd_internalSetImageWithURL: 内部通过SDWebImageManager调用加载图片过程并返回调用进度
* SDWebImageManager：内部分别由SDImageCache、SDWebImageDownloader来处理下载任务，

### SDWebImage 入口

面向UIImageView的是UIImageView+WebCache这个分类，将图片的URL，占位图片直接给这个类，该类提供众多的设置方法，不过最后都会调用以下这个方法.

``` objc

UIImageView+WebCache.m

- (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                   context:(nullable SDWebImageContext *)context
                  progress:(nullable SDImageLoaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
```

最后进入的是 UIView+WebCache 的 sd_internalSetImageWithURL方法中

```objc

UIView+WebCache.m

- (void)sd_internalSetImageWithURL:(nullable NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholder
                           options:(SDWebImageOptions)options
                           context:(nullable SDWebImageContext *)context
                     setImageBlock:(nullable SDSetImageBlock)setImageBlock
                          progress:(nullable SDImageLoaderProgressBlock)progressBlock
                         completed:(nullable SDInternalCompletionBlock)completedBlock {
        // 1.取消之前的下载请求
        // 1.1生成self类名的key值
        // 1.2通过key值取消之前的下载请求
        // 2.设置placeholder
        // 3.使用SDWebImageManager来管理是下载/缓存获取图片
        // 3.1重置下载进度
        // 3.2展示loading
        // 3.3使用SDWebImageManager来管理是下载/缓存获取图片
        // 3.4同步loading与下载进度
        // 3.5图片下载完成进行转换展示
        // 4.保存当前下载操作

        // 下面将SDWebImgaeManager的逻辑
}
```

UIView+WebCache通过关联为UIView扩展了几个属性

* sd_imageURL: 当前图片的URL
* sd_imageProgress: 当前view中图片loading的下载进度
* sd_imageTransition: 图片的过渡属性，内部包含了很多的属性duration、过渡的动画样式
* sd_imageIndicator: 图片loading的indicator图标
* sd_latestOperationKey: 最后一次的操作Key

#### UIView+WebCacheOperation

`UIView+WebCacheOperation` 中为UIView通过关联添加了一个 `sd_operationDictionary` 属性，是一个 `MapTable` 列表，类似于字典<Key:id<SDWebImageOperation>>

主要提供一下几个方法

```
/// 通过key获取图片下载操作
- (nullable id<SDWebImageOperation>)sd_imageLoadOperationForKey:(nullable NSString *)key;

/// 保存下载操作
- (void)sd_setImageLoadOperation:(nullable id<SDWebImageOperation>)operation forKey:(nullable NSString *)key;

/// 取消下载操作
- (void)sd_cancelImageLoadOperationWithKey:(nullable NSString *)key;

/// 删除下载操作
- (void)sd_removeImageLoadOperationWithKey:(nullable NSString *)key;
```

#### SDWebImageOptions: 设置的图片加载以及缓存策略，这里只看几个常用的，具体请看 `SDWebimageDefine.h -> SDWebImageOptions`

```
typedef NS_OPTIONS(NSUInteger, SDWebImageOptions) {
    //失败后重试
    SDWebImageRetryFailed = 1 << 0,
    //UI交互期间开始下载，导致延迟下载比如UIScrollView减速。
    SDWebImageLowPriority = 1 << 1,
    //这个标志可以渐进式下载,显示的图像是逐步在下载
    SDWebImageProgressiveLoad = 1 << 2,
    //刷新缓存,如果设置了该类型，缓存策略依据NSURLCache而不是SDImageCache，所以可以通过NSURLCache进行缓存了
    SDWebImageRefreshCached = 1 << 3,
    //后台下载
    SDWebImageContinueInBackground = 1 << 4,
    SDWebImageHandleCookies = 1 << 5,
    //允许使用无效的SSL证书
    SDWebImageAllowInvalidSSLCertificates = 1 << 6,
    //优先下载
    SDWebImageHighPriority = 1 << 7,
    //延迟占位符
    SDWebImageDelayPlaceholder = 1 << 8,
    //改变动画形象
    SDWebImageTransformAnimatedImage = 1 << 9,
   //手动在图片下载完成后设置图片
    SDWebImageAvoidAutoSetImage = 1 << 10,
    SDWebImageScaleDownLargeImages = 1 << 11,
    //查询内存缓存
    SDWebImageQueryMemoryData = 1 << 12,
    //同步查询内存缓存
    SDWebImageQueryMemoryDataSync = 1 << 13,
    //同步查询磁盘缓存
    SDWebImageQueryDiskDataSync = 1 << 14,
    SDWebImageFromCacheOnly = 1 << 15,
    SDWebImageForceTransition = 1 << 17,
    SDWebImageAvoidDecodeImage = 1 << 18,
    SDWebImageDecodeFirstFrameOnly = 1 << 19,
    SDWebImagePreloadAllFrames = 1 << 20,
    SDWebImageMatchAnimatedImageClass = 1 << 21,
}
```

### `SDWebImageManager`

上面关于图片缓存和下载的调用的是SDWebImageManager里面的方法

```
SDWebImageManager.m

- (SDWebImageCombinedOperation *)loadImageWithURL:(nullable NSURL *)url
                                          options:(SDWebImageOptions)options
                                          context:(nullable SDWebImageContext *)context
                                         progress:(nullable SDImageLoaderProgressBlock)progressBlock
                                        completed:(nonnull SDInternalCompletionBlock)completedBlock {
    // 断言，completedBlock不能为空
    NSAssert(completedBlock != nil, @"If you mean to prefetch the image, use -[SDWebImagePrefetcher prefetchURLs] instead");

    // 容错机制，如果url是字符串转为URL
    if ([url isKindOfClass:NSString.class]) {
        url = [NSURL URLWithString:(NSString *)url];
    }

    // 防崩溃处理，如果不是NSString和NSURL类型，就赋值为nil，防止app崩溃
    if (![url isKindOfClass:NSURL.class]) {
        url = nil;
    }

    // 绑定的操作类，下面介绍
    // 实现了 SDWebImageOperation 协议，含有取消方法
    // 包含一个cache操作和下载操作
    SDWebImageCombinedOperation *operation = [SDWebImageCombinedOperation new];
    operation.manager = self;

    BOOL isFailedUrl = NO;
    if (url) {
        // 加锁操作，失败URLSet是够包含该URL
        SD_LOCK(self.failedURLsLock);
        isFailedUrl = [self.failedURLs containsObject:url];
        SD_UNLOCK(self.failedURLsLock);
    }

    // 如果是无效的URL。调用完成操作，返回一个ERROR
    if (url.absoluteString.length == 0 || (!(options & SDWebImageRetryFailed) && isFailedUrl)) {
        [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorInvalidURL userInfo:@{NSLocalizedDescriptionKey : @"Image url is nil"}] url:url];
        return operation;
    }

    // 将operation添加到正在运行的操作Set中
    SD_LOCK(self.runningOperationsLock);
    [self.runningOperations addObject:operation];
    SD_UNLOCK(self.runningOperationsLock);

    // 获取一个操作结果
    SDWebImageOptionsResult *result = [self processedResultForURL:url options:options context:context];

    // Start the entry to load image from cache
    [self callCacheProcessForOperation:operation url:url options:result.options context:result.context progress:progressBlock completed:completedBlock];

    return operation;
}

// 查找缓存操作
- (void)callCacheProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                 url:(nonnull NSURL *)url
                             options:(SDWebImageOptions)options
                             context:(nullable SDWebImageContext *)context
                            progress:(nullable SDImageLoaderProgressBlock)progressBlock
                           completed:(nullable SDInternalCompletionBlock)completedBlock {
    // Grab the image cache to use
    id<SDImageCache> imageCache;
    if ([context[SDWebImageContextImageCache] conformsToProtocol:@protocol(SDImageCache)]) {
        imageCache = context[SDWebImageContextImageCache];
    } else {
        imageCache = self.imageCache;
    }
    // 是否不找缓存，直接下载
    BOOL shouldQueryCache = !SD_OPTIONS_CONTAINS(options, SDWebImageFromLoaderOnly);
    if (shouldQueryCache) {
        NSString *key = [self cacheKeyForURL:url context:context];
        @weakify(operation);
        operation.cacheOperation = [imageCache queryImageForKey:key options:options context:context completion:^(UIImage * _Nullable cachedImage, NSData * _Nullable cachedData, SDImageCacheType cacheType) {
            @strongify(operation);
            if (!operation || operation.isCancelled) {
                // 返回用户取消缓存错误
                [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorCancelled userInfo:@{NSLocalizedDescriptionKey : @"Operation cancelled by user during querying the cache"}] url:url];
                // 安全的从正在运行中的操作SET中删除该操作
                [self safelyRemoveOperationFromRunning:operation];
                return;
            }
            // 下载操作
            [self callDownloadProcessForOperation:operation url:url options:options context:context cachedImage:cachedImage cachedData:cachedData cacheType:cacheType progress:progressBlock completed:completedBlock];
        }];
    } else {
        // 下载操作
        [self callDownloadProcessForOperation:operation url:url options:options context:context cachedImage:nil cachedData:nil cacheType:SDImageCacheTypeNone progress:progressBlock completed:completedBlock];
    }
}

// Download process
- (void)callDownloadProcessForOperation:(nonnull SDWebImageCombinedOperation *)operation
                                    url:(nonnull NSURL *)url
                                options:(SDWebImageOptions)options
                                context:(SDWebImageContext *)context
                            cachedImage:(nullable UIImage *)cachedImage
                             cachedData:(nullable NSData *)cachedData
                              cacheType:(SDImageCacheType)cacheType
                               progress:(nullable SDImageLoaderProgressBlock)progressBlock
                              completed:(nullable SDInternalCompletionBlock)completedBlock {
    // Grab the image loader to use
    id<SDImageLoader> imageLoader;
    if ([context[SDWebImageContextImageLoader] conformsToProtocol:@protocol(SDImageLoader)]) {
        imageLoader = context[SDWebImageContextImageLoader];
    } else {
        imageLoader = self.imageLoader;
    }
    // Check whether we should download image from network
    BOOL shouldDownload = !SD_OPTIONS_CONTAINS(options, SDWebImageFromCacheOnly);
    shouldDownload &= (!cachedImage || options & SDWebImageRefreshCached);
    shouldDownload &= (![self.delegate respondsToSelector:@selector(imageManager:shouldDownloadImageForURL:)] || [self.delegate imageManager:self shouldDownloadImageForURL:url]);
    shouldDownload &= [imageLoader canRequestImageForURL:url];
    if (shouldDownload) {
        if (cachedImage && options & SDWebImageRefreshCached) {
            // If image was found in the cache but SDWebImageRefreshCached is provided, notify about the cached image
            // AND try to re-download it in order to let a chance to NSURLCache to refresh it from server.
            // 先设置缓存数据在进行下载操作
            [self callCompletionBlockForOperation:operation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
            // Pass the cached image to the image loader. The image loader should check whether the remote image is equal to the cached image.
            SDWebImageMutableContext *mutableContext;
            if (context) {
                mutableContext = [context mutableCopy];
            } else {
                mutableContext = [NSMutableDictionary dictionary];
            }
            mutableContext[SDWebImageContextLoaderCachedImage] = cachedImage;
            context = [mutableContext copy];
        }

        // 下载
        @weakify(operation);
        operation.loaderOperation = [imageLoader requestImageWithURL:url options:options context:context progress:progressBlock completed:^(UIImage *downloadedImage, NSData *downloadedData, NSError *error, BOOL finished) {
            @strongify(operation);
            if (!operation || operation.isCancelled) {
                // Image combined operation cancelled by user
                // 用户取消了下载操作
                [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:SDWebImageErrorDomain code:SDWebImageErrorCancelled userInfo:@{NSLocalizedDescriptionKey : @"Operation cancelled by user during sending the request"}] url:url];
            } else if (cachedImage && options & SDWebImageRefreshCached && [error.domain isEqualToString:SDWebImageErrorDomain] && error.code == SDWebImageErrorCacheNotModified) {
                // Image refresh hit the NSURLCache cache, do not call the completion block
            } else if ([error.domain isEqualToString:SDWebImageErrorDomain] && error.code == SDWebImageErrorCancelled) {
                // Download operation cancelled by user before sending the request, don't block failed URL
                [self callCompletionBlockForOperation:operation completion:completedBlock error:error url:url];
            } else if (error) {
                // 下载失败了
                [self callCompletionBlockForOperation:operation completion:completedBlock error:error url:url];
                BOOL shouldBlockFailedURL = [self shouldBlockFailedURLWithURL:url error:error options:options context:context];

                // 加入失败的URLSet中
                if (shouldBlockFailedURL) {
                    SD_LOCK(self.failedURLsLock);
                    [self.failedURLs addObject:url];
                    SD_UNLOCK(self.failedURLsLock);
                }
            } else {
                // 从失败SET中移除URL
                if ((options & SDWebImageRetryFailed)) {
                    SD_LOCK(self.failedURLsLock);
                    [self.failedURLs removeObject:url];
                    SD_UNLOCK(self.failedURLsLock);
                }
                // Continue store cache process
                // 将图片缓存
                [self callStoreCacheProcessForOperation:operation url:url options:options context:context downloadedImage:downloadedImage downloadedData:downloadedData finished:finished progress:progressBlock completed:completedBlock];
            }

            if (finished) {
                [self safelyRemoveOperationFromRunning:operation];
            }
        }];
    } else if (cachedImage) {
         // 直接使用缓存
        [self callCompletionBlockForOperation:operation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
        [self safelyRemoveOperationFromRunning:operation];
    } else {
        // 删除操作，且没数据
        // Image not in cache and download disallowed by delegate
        [self callCompletionBlockForOperation:operation completion:completedBlock image:nil data:nil error:nil cacheType:SDImageCacheTypeNone finished:YES url:url];
        [self safelyRemoveOperationFromRunning:operation];
    }
}

```

大概流程

```
// 1.判断并处理url
// 2.初始化一个操作
// 3.坏url直接返回成功操作，并带回一个error
// 4.将当前操作添加到runningOpearations中
// 5.获取 SDWebImageOptionsResult 对象
// 6.调用callCacheProcessForOperation方法
// 6.1.1 有缓存，从缓存里面找 [imageCache queryImageForKe...]，并设置operation.cacheOperation
// 6.1.2 拿到缓存，并执行下一步操作callDownloadProcessForOperation方法下载
// 6.2 调用callDownloadProcessForOperation方法下载
// 7.调用callDownloadProcessForOperation方法下载
// 7.1 判断是否只从缓存获取 判断是否需要结合SDWebImageRefreshCached,来读取缓存
// 7.2 如果不需要下载，有缓存数据，直接调用成功回调
// 7.3 如果不需要下载，又没有缓存数据，也调用成功回调
// 7.4 需要下载，进行下载，成功后调用成功回调，并执行存储方法callStoreCacheProcessForOperation
```

#### SDWebImageCombinedOperation

SDWebImageCombinedOperation 实现了 SDWebImageOperation 代理，而 SDWebImageOperation 里面只有一个cancel方法，上面那个NSMapTable里面的value 类型是 id<SDWebImageOperation> 不难猜测存储的就是 SDWebImageCombinedOperation 类型，便于调用cancel方法取消任务

```
/// 取消当前的operation任务
* (void)cancel;

/// 表示当前的图片是从缓存中查找到的，将从缓存中获取任务放到cacheOperation属性中
@property (strong, nonatomic, nullable, readonly) id<SDWebImageOperation> cacheOperation;

/// 当前的图片需要下载,将下载任务放入loaderOperation属性中
@property (strong, nonatomic, nullable, readonly) id<SDWebImageOperation> loaderOperation;

//SDWebImageCombinedOperation
//当前对象中取消当前的下载任务

* (void)cancel {

        @synchronized(self) {
                if (self.isCancelled) {
                        return;
                }
                self.cancelled = YES;
                //是从缓存中读取数据 SDImageCache
                if (self.cacheOperation) {
                        [self.cacheOperation cancel];
                        self.cacheOperation = nil;
                }
                //下载中读取数据 SDWebImageDownloadToken
                //取消NSOperation中的任务
                //取消SDWebImageDownloaderOperation中的自定义回调任务
                if (self.loaderOperation) {
                        [self.loaderOperation cancel];
                        self.loaderOperation = nil;
                }
                //从当前正在执行的operation列表中移除当前的SDWebImageCombinedOperation任务
                [self.manager safelyRemoveOperationFromRunning:self];
        }
}
```



