---
layout: post
title: "SDWebImage源码阅读3"
date: 2018-12-22 16:00:30 +0800
tags: ["源码"]
---

## 其他

### 判断是否是gif图

```
@implementation UIImage (GIF)

+ (nullable UIImage *)sd_imageWithGIFData:(nullable NSData *)data {
    if (!data) {
        return nil;
    }
    return [[SDImageGIFCoder sharedCoder] decodedImageWithData:data options:0];
}

@end
```
