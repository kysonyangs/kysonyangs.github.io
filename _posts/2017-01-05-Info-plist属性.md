---
layout: post
title: "Info.plist属性"
date: 2017-01-05 20:00:03 +0800
tags: ["Xcode"]
---

![](http://ozhr26838.bkt.clouddn.com/hexo/info-plist.png)

<!-- more -->

- `Application does not run in background`: 默认 `NO`， 自从iOS4.0之后，当你在应用程序执行的时候按下Home键，应用程序并不会中断目前的应用，而是放到后台去了。因此希望使用者在按下Home键之后就要退出当前应用的请勾选这个选项。
- `Localization native development region`: 本地化相关
- `Bundle display name`: 应用程序本地化的显示名称，预设值为${PRODUCT_NAME}。
- `Bundle name`: 应用程序的短名称，通常就是你的应用程序名称。
- `Executable file`: 程序安装包的名称, 一般不用改
- `Bundle identifier`: 用来标示应用程序的唯一ID，通常是以反向的DNS方式命名的，例如：com.myCompany.myApp
- `InfoDictionary version`: info.plist格式的版本。一般来说，我们不会变动这个数值。
- `Bundle versions string, short`: 应用程序的版本，通常是以三个数字來表示版本号，例如：1.0.1。
- `Bundle version`: 标识编译版本(Bundle number)，你可以使用任何字串格式来表示这个版本。例如使用一个数字来表示编译次数。
- `Bundle OS Type code`: 用用来标识整个封包的(bundle)的类型。在Mac裡面，一个封包可能是一个档案或目录，其目的在于将软体使用到的资源包在一起。例如应用程序应标识为APPL。
- `Bundle creator OS Type code`: 开发者对应用程序的标识
- `Application requires iPhone environment`: 如果应用程序不能在iPad、Touch上运行，设置此项为true;
- `Application uses WIFI`: 如果应用程序需要WiFi才能工作，应该将此属性设置为true。这么做会提示用户，如果没有代开WiFi的话，打开WiFi，为了节省电力，iPhone会在30分钟后自动关闭程序中的任何WiFi。设置这一属性可以防止这种情况发生，并且保持连接处于活动状态。
- `Icon already includes gloss and bevel effects`:  应用程序设置玻璃效果，设置为true可以阻止这么做。
- `Main nib file base name`: 应用程序首次启动载入的xib文件
- `Laumch image`: 用以指定应用程序启动时的图片文件。
- `Initial interface orientation`: 指定应用程序打开时的方向。
- `Localizations`: 用以指定应用程序所支持的语言。
- `Localization native development region`: 应用程序原始的语言版本。
- `Status bar is initially hidden`: 设置是否隐藏状态栏
- `Status bar style`: 选择三种不同格式中的一种
- `URL types`: 应用程序支持的URL标识符的一个数组（用于程序回调）
- `supported interface orientation`: 程序的默认支持方向
- `View controller-based status bar appearance`: YES/NO
```
// 设为 `YES`, 这时 view controller中对status bar的设置优先级高于application的设置，用下面的方式隐藏status bar:
- (BOOL)prefersStatusBarHidden {
    return YES;
}
// 设为 `NO`, 这时application的设置优先级最高，用下面的方式隐藏status bar:
[[UIApplication sharedApplication] setStatusBarHidden:YES withAnimation:NO];
```
- `Required background modes`: 设定当应用程序进入后台执行后，哪些动作要继续在背景执行。这个键值是一个阵列类型的设定，可设定动作包括：audio，locateon，voip。
- `App Transport Security Settings`: https网络，这是个字典，如果需要请求 http 需要设置下列字典项
    * `Allow Arbitrary Loads`: YES
- `Privacy ...`: 一些权限
```
Privacy - Media Library Usage Description               // 获取用户媒体库说明
Privacy - Bluetooth Peripheral Usage Description        // 蓝牙外设使用描述
Privacy - Calendars Usage Description                   // 日历的使用说明
Privacy - Camera Usage Description                      // 相机使用叙述说明
Privacy - Contacts Usage Description                    // 联系人使用说明
Privacy - Health Share Usage Description                // 健康分享使用描述
Privacy - Location Always Usage Description             // 后台定位(在iOS设置中为'永久')
Privacy - Location Usage Description                    // 需要定位
Privacy - Location When In Use Usage Description        // 前台定位(在iOS设置中为'使用期间')
Privacy - Health Update Usage Description               // 健康更新使用描述
Privacy - HomeKit Usage Description                     // HomeKit使用描述
Privacy - Microphone Usage Description                  // 麦克风的使用说明
Privacy - Motion Usage Description                      // 运动使用的描述
Privacy - Photo Library Usage Description               // 照片库使用说明
Privacy - Reminders Usage Description                   // 提醒使用描述
Privacy - TV Provider Usage Description                 // 电视提供商使用的描述 (貌似国内用不到)
Privacy - Speech Recognition Usage Description          // 语音识别权限
Privacy - Reminders Usage DescriptionPrivacy - Reminders Usage Description  // 提醒事项权限
Privacy - Motion Usage Description                      // 运动与健身

// iOS11新增
Privacy - NFC Reader Usage Description         //NFC使用描述
Privacy - Face ID Usage Descriptio                   //使用Face ID
Privacy - Photo Library Additions Usage Description     //  保存图片到图库中 （重要）
```

















