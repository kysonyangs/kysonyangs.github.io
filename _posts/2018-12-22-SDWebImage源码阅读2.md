---
layout: post
title: "SDWebImage源码阅读2"
date: 2018-12-22 15:00:10 +0800
tags: ["源码"]
---

## SDImageCache

SDImageCache 是 SDWebImage 中用来处理缓存的类，他是一个单例

SDWebImage 中关于缓存可以分为磁盘缓存 `id<SDDiskCache>` 和 内存缓存 `id<SDMemoryCache>`

```
@interface SDImageCache ()

#pragma mark - Properties
@property (nonatomic, strong, readwrite, nonnull) id<SDMemoryCache> memoryCache; // 内存缓存管理
@property (nonatomic, strong, readwrite, nonnull) id<SDDiskCache> diskCache;     // 磁盘缓存管理
@property (nonatomic, copy, readwrite, nonnull) SDImageCacheConfig *config;      // 基本设置
@property (nonatomic, copy, readwrite, nonnull) NSString *diskCachePath;         // 磁盘缓存路径
@property (nonatomic, strong, nullable) dispatch_queue_t ioQueue;                // io队列

@end
```

```
// 查询缓存操作
- (nullable NSOperation *)queryCacheOperationForKey:(nullable NSString *)key options:(SDImageCacheOptions)options context:(nullable SDWebImageContext *)context done:(nullable SDImageCacheQueryCompletionBlock)doneBlock {

    //不存在这个key，就调用完成block
    if (!key) {
        if (doneBlock) {
            doneBlock(nil, nil, SDImageCacheTypeNone);
        }
        return nil;
    }

    // 是否有转换操作，获取转换操作完成后图片的key
    id<SDImageTransformer> transformer = context[SDWebImageContextImageTransformer];
    if (transformer) {
        // grab the transformed disk image if transformer provided
        NSString *transformerKey = [transformer transformerKey];
        key = SDTransformedKeyForKey(key, transformerKey);
    }

    // 先检查内存是否存在该图片
    UIImage *image = [self imageFromMemoryCacheForKey:key];

    if (image) {
        if (options & SDImageCacheDecodeFirstFrameOnly) {
            // Ensure static image
            Class animatedImageClass = image.class;
            if (image.sd_isAnimated || ([animatedImageClass isSubclassOfClass:[UIImage class]] && [animatedImageClass conformsToProtocol:@protocol(SDAnimatedImage)])) {
#if SD_MAC
                image = [[NSImage alloc] initWithCGImage:image.CGImage scale:image.scale orientation:kCGImagePropertyOrientationUp];
#else
                image = [[UIImage alloc] initWithCGImage:image.CGImage scale:image.scale orientation:image.imageOrientation];
#endif
            }
        } else if (options & SDImageCacheMatchAnimatedImageClass) {
            // Check image class matching
            Class animatedImageClass = image.class;
            Class desiredImageClass = context[SDWebImageContextAnimatedImageClass];
            if (desiredImageClass && ![animatedImageClass isSubclassOfClass:desiredImageClass]) {
                image = nil;
            }
        }
    }

    // 是否只查找内存缓存
    BOOL shouldQueryMemoryOnly = (image && !(options & SDImageCacheQueryMemoryData));
    if (shouldQueryMemoryOnly) {
        if (doneBlock) {
            doneBlock(image, nil, SDImageCacheTypeMemory);
        }
        return nil;
    }

    // 2.查找内存缓存
    NSOperation *operation = [NSOperation new];
    // Check whether we need to synchronously query disk
    // 1. in-memory cache hit & memoryDataSync
    // 2. in-memory cache miss & diskDataSync
    BOOL shouldQueryDiskSync = ((image && options & SDImageCacheQueryMemoryDataSync) ||
                                (!image && options & SDImageCacheQueryDiskDataSync));
    void(^queryDiskBlock)(void) =  ^{
        if (operation.isCancelled) {
            if (doneBlock) {
                doneBlock(nil, nil, SDImageCacheTypeNone);
            }
            return;
        }

        @autoreleasepool {
            ///如果盘中存在图片，并且如果可以使用缓存
            NSData *diskData = [self diskImageDataBySearchingAllPathsForKey:key];
            UIImage *diskImage;
            SDImageCacheType cacheType = SDImageCacheTypeNone;
            if (image) {
                // the image is from in-memory cache, but need image data
                diskImage = image;
                cacheType = SDImageCacheTypeMemory;
            } else if (diskData) {
                cacheType = SDImageCacheTypeDisk;
                // decode image data only if in-memory cache missed
                diskImage = [self diskImageForKey:key data:diskData options:options context:context];
                if (diskImage && self.config.shouldCacheImagesInMemory) {
                    NSUInteger cost = diskImage.sd_memoryCost;
                    //将硬盘中的图片在缓存中存一份
                    [self.memoryCache setObject:diskImage forKey:key cost:cost];
                }
            }

            if (doneBlock) {
                if (shouldQueryDiskSync) {
                    doneBlock(diskImage, diskData, cacheType);
                } else {
                    dispatch_async(dispatch_get_main_queue(), ^{
                        doneBlock(diskImage, diskData, cacheType);
                    });
                }
            }
        }
    };

    // Query in ioQueue to keep IO-safe
    if (shouldQueryDiskSync) {
        dispatch_sync(self.ioQueue, queryDiskBlock);
    } else {
        dispatch_async(self.ioQueue, queryDiskBlock);
    }

    return operation;
}
```

### 先看一下 SDImageCacheConfig

SDImageCacheConfig 是一个关于缓存设置相关的类

```
- (instancetype)init {
    if (self = [super init]) {
        _shouldDisableiCloud = YES; // 忽略iCloud
        _shouldCacheImagesInMemory = YES; // 缓存到内存
        _shouldUseWeakMemoryCache = YES;  // 缓存到磁盘
        _shouldRemoveExpiredDataWhenEnterBackground = YES; // 进入后台模式删除过期的数据
        _diskCacheReadingOptions = 0;
        _diskCacheWritingOptions = NSDataWritingAtomic;
        _maxDiskAge = kDefaultCacheMaxDiskAge; // 磁盘缓存最大存储时间 1WEEK
        _maxDiskSize = 0; // 最大磁盘缓存大小 0表示无上限
        _diskCacheExpireType = SDImageCacheConfigExpireTypeModificationDate;
        _memoryCacheClass = [SDMemoryCache class];
        _diskCacheClass = [SDDiskCache class];
    }
    return self;
}
```

### SDMemoryCache 内存缓存

SDMemoryCache 继承自 NSCache，实现了 SDMemoryCache 协议，封装了一系列方法，便于扩展

```
@interface SDMemoryCache <KeyType, ObjectType> : NSCache <KeyType, ObjectType> <SDMemoryCache>

@property (nonatomic, strong, nonnull, readonly) SDImageCacheConfig *config;

@end

@interface SDMemoryCache <KeyType, ObjectType> ()

@property (nonatomic, strong, nullable) SDImageCacheConfig *config;
#if SD_UIKIT
@property (nonatomic, strong, nonnull) NSMapTable<KeyType, ObjectType> *weakCache; // strong-weak cache
@property (nonatomic, strong, nonnull) dispatch_semaphore_t weakCacheLock; // a lock to keep the access to `weakCache` thread-safe
#endif
@end
```

SDMemoryCache 也使用 NSMapTable 来储存缓存的数据，并加了锁来保证线程安全

那为什么 SDMemoryCache 继承自 NSCache 还要维护一个 NSMapTable 呢，因为 NSCache 对于 value 是强引用的，而 NSMapTable 对于value 是弱引用的！

可以通过config 的 shouldUseWeakMemoryCache 来关闭使用 NSMapTable 管理。

### SDDiskCache 磁盘缓存

SDMemoryCache 实现了 SDDiskCache 协议，封装了一系列方法，便于扩展
