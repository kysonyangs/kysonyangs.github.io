---
layout: post
title: "CoreAniamtion"
date: 2016-05-25 15:34:51 +0800
tags: ["iOS"]
---

## UIView vs CALayer
1. 每个 UIView 内部都有一个 CALayer 在背后提供内容的绘制和显示，两者都有树状层级结构，layer 内部有 SubLayers，View 内部有 SubViews;
2. 在 View显示的时候，UIView 做为 Layer 的 CALayerDelegate,View 的显示内容由内部的 CALayer 的 display;
3. View 可以接受事件，而 Layer 不行

### 属性说明
* @property(nullable, strong) id contents; : 可以将CGImage赋值给他，显示成为一张图片
```
layerView.layer.contents = (__bridge id _Nullable)(image.CGImage);
```
<!-- more -->

* `@property(copy) NSString *contentsGravity; : 类似于UIView的contentMode`
```
kCAGravityCenter
kCAGravityTop
kCAGravityBottom
kCAGravityLeft
kCAGravityRight
kCAGravityTopLeft
kCAGravityTopRight
kCAGravityBottomLeft
kCAGravityBottomRight
kCAGravityResize
kCAGravityResizeAspect
kCAGravityResizeAspectFill
```
* @property BOOL masksToBounds; : 超出位置是否裁切
* @property CGRect contentsRect;
这个属性决定图片的显示位置，默认是{0，0，1，1}从左上角到左下角显示，你也可以设为其他的,前两个是起点比例，后两个分别对应宽度和高度比例
(该属性一般用于给你一张有许多小图的大图，然后你根据自己的需要去拿你需要的图片)
* @property CGRect contentsCenter; : 类似于UIImage resizableImageWithCapInsets 属性！

---

* @property CGPoint position; : 类似于UIView的Center， 但是它作用的也是UIView的Center而不是Layer的Center哦
* @property CGPoint anchorPoint; : 我的理解是layer中心点的位置，默认为{0.5，0.5}，所以居中显示，但是将它设置为{0，0}，那么layer会以{0，0}为中心点显示哦！这时position没变哦！
* @property CGFloat zPosition;
* @property CGFloat anchorPointZ;
由于CALayer是存在于3维空间中的，所以我们可以改变他的z轴(你可以理解为Z轴就是垂直严你的屏幕的轴)

---

* @property CGFloat cornerRadius; : 圆角半径(我们一般给视图做圆角的时候，如果界面不是太多，可以使用该方法，但是如果是对Cell里面的视图进行圆角设置或者太多视图需要进行圆角设置，就别用这个属性了，因为layer的渲染会耗资源)
* @property CGFloat borderWidth;
* @property(nullable) CGColorRef borderColor;
边框的Width和颜色，不必多说
* @property CGFloat shadowRadius;  阴影模糊度
关于阴影的一些属性，建议如`cornerRadius`一样，使用阴影的时候别用maskToBounds哦！因为阴影在范围外，使用了会被裁剪掉哦！shadowPath
* @property(nullable) CGColorRef shadowColor; 阴影颜色
* @property float shadowOpacity;  阴影透明度
* @property CGSize shadowOffset;  阴影偏移量
* @property(nullable) CGPathRef shadowPath; 感觉该属性就是为了解决使用上述属性设置阴影好资源诞生的，推荐使用这个属性设置阴影哦！
```
layerView.layer.shadowOpacity = 0.5;
layerView.layer.shadowColor = [UIColor redColor].CGColor;
layerView.layer.shadowOffset = CGSizeMake(10, 10);
//    layerView.layer.shadowPath = CGPathCreateWithRect(layerView.layer.bounds, NULL);

//    layerView.layer.shadowPath = CGPathCreateWithRoundedRect(layerView.layer.bounds, layerView.layer.bounds.size.width * 0.5, layerView.layer.bounds.size.width * 0.5, NULL);
```

---

* - (void)setAffineTransform:(CGAffineTransform)m; 这个就是UIView的`transform`实现
```
CGAffineTransform 方法
CGAffineTransformMakeTranslation(CGFloat tx, CGFloat ty) // 移动
CGAffineTransformMakeScale(CGFloat sx, CGFloat sy) // 放缩
CGAffineTransformMakeRotation(CGFloat angle) // 旋转
CGAffineTransformIdentity  // 复位，初始状态
混合变换
CGAffineTransformTranslate(CGAffineTransform t, CGFloat tx, CGFloat ty) // 在t的基础上移动
CGAffineTransformScale(CGAffineTransform t, CGFloat sx, CGFloat sy) // 在t的基础上放缩
CGAffineTransformRotate(CGAffineTransform t, CGFloat angle) // 在t的基础上旋转
注意： 旋转时使用的是弧度而不是角度，你可以利用下列公式转换角度到弧度
#define RADIANS_TO_DEGREES(x) ((x)/M_PI*180.0)

```
* @property CATransform3D transform; 仿射变换  3D动画
```
CATransform3DMakeRotation(CGFloat angle, CGFloat x, CGFloat y, CGFloat z)
CATransform3DMakeScale(CGFloat sx, CGFloat sy, CGFloat sz)
CATransform3DMakeTranslation(Gloat tx, CGFloat ty, CGFloat tz)

CATransform3D的 m34 元素, 用来做透视,m34用于按比例缩放X和Y的值来计算到底要离视角多远。
```

### CAShapeLayer
* 渲染快，高效实用内存，不会被图层边界剪裁掉，不会出现像素化
* 一般用CGPath来绘制形状
```
UIBezierPath *path = [[UIBezierPath alloc] init];
 [path moveToPoint:CGPointMake(200, 100)];
 // 绘制圆
[path addArcWithCenter:CGPointMake(170, 100) radius:30 startAngle:0 endAngle:2*M_PI clockwise:YES];
// 绘制线条
[path moveToPoint:CGPointMake(150, 125)];
[path addLineToPoint:CGPointMake(150, 175)];
// 绘制，展示
CAShapeLayer *shapeLayer = [CAShapeLayer layer];
shapeLayer.strokeColor = [UIColor redColor].CGColor;
shapeLayer.fillColor = [UIColor clearColor].CGColor;
shapeLayer.lineWidth = 5;
shapeLayer.lineJoin = kCALineJoinRound;
shapeLayer.lineCap = kCALineCapRound;
shapeLayer.path = path.CGPath;
[self.view.layer addSublayer:shapeLayer];

// 再提一个  绘制圆角(可选4角)
CGRect rect = CGRectMake(50, 50, 100, 100);
CGSize radii = CGSizeMake(20, 20);
UIRectCorner corners = UIRectCornerTopRight | UIRectCornerBottomRight | UIRectCornerBottomLeft;
UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:rect byRoundingCorners:corners cornerRadii:radii];
```

### CATextLabel
图层的形式包含了UILabel几乎所有的绘制特性，**而且**比UILabel渲染的更快哦！(如果你有需求做一个自定制的Label,不防试试这个)
```
CATextLayer *textLayel = [CATextLayer layer];
    textLayel.frame = CGRectMake(100, 100, 200, 300);
    [self.view.layer addSublayer:textLayel];

    textLayel.foregroundColor = [UIColor blackColor].CGColor;
    textLayel.backgroundColor = [UIColor orangeColor].CGColor;
    textLayel.alignmentMode = @"justified";
    textLayel.wrapped = YES;

    UIFont *font = [UIFont systemFontOfSize:17];

    CFStringRef fontName = (__bridge CFStringRef)(font.fontName);
    CGFontRef fontRef = CGFontCreateWithFontName(fontName);
    textLayel.font = fontRef;
    textLayel.fontSize = font.pointSize;

    /**
     *  分辨率
     */
    textLayel.contentsScale = [UIScreen mainScreen].scale;

    CGFontRelease(fontRef);

    NSString *text = @"Kellen is a good boy! Kellen is a good boy! Kellen is a good boy! Kellen is a good boy! Kellen is a good boy! Kellen is a good boy! Kellen is a good boy! Kellen is a good boy! Kellen is a good boy! Kellen is a good boy!";

    NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:text];

    //set text attributes
    NSDictionary *attribs = @{
                              NSForegroundColorAttributeName : [UIColor blackColor],
                              NSFontAttributeName : font
                              };

    [string setAttributes:attribs range:NSMakeRange(0, [text length])];
    attribs = @{NSForegroundColorAttributeName : [UIColor redColor]};
    [string setAttributes:attribs range:NSMakeRange(6, 5)];

    textLayel.string = string;

```

### CAGradientLayer
处理颜色渐变
```
CAGradientLayer *gradientLayer = [CAGradientLayer layer];
gradientLayer.frame = CGRectMake(100, 100, 100, 100);
[self.view.layer addSublayer:gradientLayer];
gradientLayer.colors = @[(__bridge id)[UIColor redColor].CGColor, (__bridge id)[UIColor blueColor].CGColor, (__bridge id)[UIColor blackColor].CGColor];
gradientLayer.locations = @[@0,@0.5,@1];
gradientLayer.startPoint = CGPointMake(0, 0);
gradientLayer.endPoint = CGPointMake(1, 1);
```
4. CAEmitterLayer : 粒子引擎(直播那些❤️动画之类的)
```
CAEmitterLayer *emitterLayer = [CAEmitterLayer layer];
// 发射器在xy平面的中心位置
emitterLayer.emitterPosition = CGPointMake(self.view.frame.size.width-50,self.view.frame.size.height-50);
// 发射器的尺寸大小
emitterLayer.emitterSize = CGSizeMake(20, 20);
// 渲染模式
emitterLayer.renderMode = kCAEmitterLayerUnordered;
// 开启三维效果
//    _emitterLayer.preservesDepth = YES;
NSMutableArray *array = [NSMutableArray array];
// 创建粒子
for (int i = 0; i<10; i++) {
    // 发射单元
    CAEmitterCell *stepCell = [CAEmitterCell emitterCell];
    // 粒子的创建速率，默认为1/s
    stepCell.birthRate = 1;
    // 粒子存活时间
    stepCell.lifetime = arc4random_uniform(4) + 1;
    // 粒子的生存时间容差
    stepCell.lifetimeRange = 1.5;
    // 颜色
    // fire.color=[[UIColor colorWithRed:0.8 green:0.4 blue:0.2 alpha:0.1]CGColor];
    UIImage *image = [UIImage imageNamed:[NSString stringWithFormat:@"good%d_30x30", i]];
    // 粒子显示的内容
    stepCell.contents = (id)[image CGImage];
    // 粒子的名字
    //            [fire setName:@"step%d", i];
    // 粒子的运动速度
    stepCell.velocity = arc4random_uniform(100) + 100;
    // 粒子速度的容差
    stepCell.velocityRange = 80;
    // 粒子在xy平面的发射角度
    stepCell.emissionLongitude = M_PI+M_PI_2;;
    // 粒子发射角度的容差
    stepCell.emissionRange = M_PI_2/6;
    // 缩放比例
    stepCell.scale = 0.3;
    [array addObject:stepCell];
}

emitterLayer.emitterCells = array;
[self.view.layer addSublayer:emitterLayer];
```

## 隐式动画
就是当你改变某个属性是时，他会自动平滑的过渡到新的值。而你不需要去开启动画。
比如： CALayer的backgroundColor

>`事务`实际上是Core Animation用来包含一系列属性动画集合的机制，任何用指定事务去改变可以做动画的图层属性都不会立刻发生变化，而是当事务一旦提交的时候开始用一个动画过渡到新值。
事务是通过CATransaction类来做管理，这个类的设计有些奇怪，不像你从它的命名预期的那样去管理一个简单的事务，而是管理了一叠你不能访问的事务。CATransaction没有属性或者实例方法，并且也不能用+alloc和-init方法创建它。但是可以用+begin和+commit分别来入栈或者出栈。

```
// 使用事务改变执行时间（默认 0.25s）
[CATransaction begin];(入栈)
[CATransaction setAnimationDuration:1.0];

// 完成之后在旋转90度(0.25s完成)
[CATransaction setCompletionBlock:^{
    NSLog(@"改变颜色完成，开始旋转");
    在颜色改变之后(入栈)
    CGAffineTransform transform = self.colorLayer.affineTransform;
    self.colorLayer.affineTransform = CGAffineTransformRotate(transform, M_PI_2);
    (出栈)
}];

CGFloat red = arc4random() / (CGFloat)INT_MAX;
CGFloat green = arc4random() / (CGFloat)INT_MAX;
CGFloat blue = arc4random() / (CGFloat)INT_MAX;
self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;
[CATransaction commit];(出栈)
```
你也可以改变隐式动画的行为（如何渐变）
```
 // 行为
CATransition *transition = [CATransition animation];
transition.type = kCATransitionPush;
transition.subtype = kCATransitionFromLeft;
self.colorLayer.actions = @{@"backgroundColor": transition};
```

但是你把上述的`self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;`
改成`self.colorLayer.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:1.0].CGColor;`你会发现它是瞬间改变的，而不是平滑的动画。因为隐式动画被UIView关联图层给禁了！
因为处理UIView动画用UIView的动画函数，而不依赖CATransaction。


## 显式动画
1. CABasicAnimation(基本动画)
```
CABasicAnimation *basicA = [CABasicAnimation animation];
basicA.keyPath = @"backgroundColor";
basicA.toValue = (__bridge id _Nullable)(color.CGColor);
basicA.delegate = self;
basicA.duration = 1.0;
[self.layerView.layer addAnimation:basicA forKey:nil];

// 代理，监听动画完成，还有个动画开始的监听
- (void)animationDidStop:(CABasicAnimation *)anim finished:(BOOL)flag {
    NSLog(@"animation end");
    self.layerView.layer.backgroundColor = (__bridge CGColorRef _Nullable)(anim.toValue);
}
```

2. CAKeyframeAnimation (关键帧动画)
不限制于设置一个起始和结束的值，而是可以根据一连串随意的值来做动画。
```
CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
animation.keyPath = @"backgroundColor";
animation.duration = 2.0;
animation.values = @[
                     (__bridge id)[UIColor blueColor].CGColor,
                     (__bridge id)[UIColor redColor].CGColor,
                     (__bridge id)[UIColor greenColor].CGColor,
                     (__bridge id)[UIColor blueColor].CGColor];
[self.layerView.layer addAnimation:animation forKey:nil];
```

```
UIBezierPath *bezierPath = [[UIBezierPath alloc] init];
[bezierPath moveToPoint:CGPointMake(0, 150)];
[bezierPath addCurveToPoint:CGPointMake(300, 150) controlPoint1:CGPointMake(75, 0) controlPoint2:CGPointMake(225, 300)];
CAKeyframeAnimation *animation = [CAKeyframeAnimation animation];
animation.keyPath = @"position";
animation.duration = 4.0;
animation.path = bezierPath.CGPath;
animation.rotationMode = kCAAnimationRotateAuto;
[shipLayer addAnimation:animation forKey:nil];
```

3. 动画组
CABasicAnimation和CAKeyframeAnimation仅仅作用于单独的属性，而CAAnimationGroup可以把这些动画组合在一起。
```
UIBezierPath *bezierPath = [[UIBezierPath alloc] init];
[bezierPath moveToPoint:CGPointMake(0, 150)];
[bezierPath addCurveToPoint:CGPointMake(300, 150) controlPoint1:CGPointMake(75, 0) controlPoint2:CGPointMake(225, 300)];
//draw the path using a CAShapeLayer
CAShapeLayer *pathLayer = [CAShapeLayer layer];
pathLayer.path = bezierPath.CGPath;
pathLayer.fillColor = [UIColor clearColor].CGColor;
pathLayer.strokeColor = [UIColor redColor].CGColor;
pathLayer.lineWidth = 3.0f;
[self.view.layer addSublayer:pathLayer];
//add a colored layer
CALayer *colorLayer = [CALayer layer];
colorLayer.frame = CGRectMake(0, 0, 64, 64);
colorLayer.position = CGPointMake(0, 150);
colorLayer.backgroundColor = [UIColor greenColor].CGColor;
[self.view.layer addSublayer:colorLayer];
//create the position animation
CAKeyframeAnimation *animation1 = [CAKeyframeAnimation animation];
animation1.keyPath = @"position";
animation1.path = bezierPath.CGPath;
animation1.rotationMode = kCAAnimationRotateAuto;
//create the color animation
CABasicAnimation *animation2 = [CABasicAnimation animation];
animation2.keyPath = @"backgroundColor";
animation2.toValue = (__bridge id)[UIColor redColor].CGColor;
//create group animation
CAAnimationGroup *groupAnimation = [CAAnimationGroup animation];
groupAnimation.animations = @[animation1, animation2];
groupAnimation.duration = 4.0;
//add the animation to the color layer
[colorLayer addAnimation:groupAnimation forKey:nil];
```
4. 过度动画
属性动画只对图层的可动画属性起作用，所以如果要改变一个不能动画的属性（比如图片），或者从层级关系中添加或者移除图层，属性动画将不起作用。
```
type值:
kCATransitionFade,   // 平滑过渡
kCATransitionMoveIn,
kCATransitionMoveIn,
kCATransitionReveal,

subtype值: 控制方向
kCATransitionFromRight，
kCATransitionFromLeft，
kCATransitionFromTop,
kCATransitionFromBottom

CATransition *transition = [CATransition animation];
transition.type = kCATransitionFade;
[self.imgv.layer addAnimation:transition forKey:nil];
UIImage *currentImage = self.imgv.image;
NSUInteger index = [arr indexOfObject:currentImage];
index = (index + 1) % [arr count];
self.imgv.image = arr[index];

// 过渡动画和之前的属性动画或者动画组添加到图层上的方式一致，都是通过-addAnimation:forKey:方法。
// 但是和属性动画不同的是，对指定的图层一次只能使用一次CATransition，
// 因此，无论你对动画的键设置什么值，过渡动画都会对它的键设置成“transition”，也就是常量kCATransition
```












