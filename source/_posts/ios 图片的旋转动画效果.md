---
title: "iOS图片的旋转动画效果"
date: 2015-09-14
layout: post
description: "本文实现了:根据当前时间, 旋转时钟的3指针(图片),显示当前时间."
tags: [iOS入门,CGAffineTransform,anchorPoint,Object C]
---


### 目的
> 根据当前时间, 实现钟表的时分秒3根指针的移动

### 界面
![Alt text](/1442160050001.png)

### 物体移动的计算方法
**旋转矩阵M**
> 把物体从A点(x,y)移动到B点(x1,y1)的计算方法如下:
> 矩阵B = 矩阵A * 移动矩阵M
> 对于顺时针旋转, M等于
> ![Alt text](/1442160401385.png)
> 在ios中,  顺时针旋转的矩阵M无需自己计算, 可由下面的方法得出:
> `CGAffineTransform matrix = CGAffineTransformMakeRotation(angle*M_PI/180);`
> 即CGAffineTransformMakeRotation输入的参数为要顺时针旋转的弧度,输出为旋转的矩阵M

**anchorPoint**

> **anchorPoint**为旋转点, 即以anchorPoint为中心, 对物体进行整体旋转.
> ![Alt text](/1442160889138.png)
> 上图中, 中心的anchorPoint为 (0.5,0.5),左上角为(0,0); 右下角为(1,1)

### 实现代码

-  输入三根指针的任意一根和旋转的角度,即可旋转
```Objective-C
- (void) moveHand :(UIView *)hand toAngle:(CGFloat) angle{
    [[hand layer] setAnchorPoint:CGPointMake(0.4f, 0.75f)];
    [[hand layer] setPosition:CGPointMake(160.0f, 170.0f)]; // 重新将指针置于表盘中间
    [UIView animateWithDuration:0.5 animations:^{
        CGAffineTransform matrix =  CGAffineTransformMakeRotation(angle*M_PI/180);
        [[hand layer] setAffineTransform:matrix]; //实现转动
    }];

}
```
anchorPoint点得位置
![Alt text](/1442161266216.png)

- 根据当前的时间,移动指针
```Objective-C
- (void) moveHandToLocalTime{
	//获取当前的时间
    NSCalendar *calendar = [NSCalendar currentCalendar];
    NSInteger comp = (NSCalendarUnitHour | NSCalendarUnitMinute | NSCalendarUnitSecond);
    NSDateComponents *components = [calendar components:comp fromDate:[NSDate date]];
    
    [self moveHand:self.hoursHand toAngle:[components hour] * 360.0f / 24.0f]; 
    //24h制度,12h旋转一圈, x h旋转 (x/2 /12)*360
    [self moveHand:self.minutesHand toAngle:[components minute] * 360.0f / 60.0f]; 
    // 60min旋转一圈, x min旋转 (x/60)*360
    [self moveHand:self.secondsHand toAngle:[components second] * 360.0f / 60.0f];
    
}
```

 - 当view移动到window上时, 触发Timer, 间隔为1s,重复执行
```
- (void) didMoveToWindow{
    [NSTimer scheduledTimerWithTimeInterval:1.0
                                     target:self                                                                   selector:@selector(moveHandToLocalTime)
                                   userInfo:nil
                                    repeats:YES];
    [self oscillatePendulum];
    

}
```

### 对于钟摆的移动, 可分为四个阶段:

![Alt text](/1442162089697.png)
> 1. 摆向15°减速
> 2. 摆向0° 加速
> 3. -摆向15°减速
> 4. 摆向0°加速

```
- (void)oscillatePendulum
{
    [[self.pendulum layer] setAnchorPoint:CGPointMake(0.5f, 0.0f)];
    [[self.pendulum layer] setPosition:CGPointMake(0.0f, 0.0f)];
    
    CAKeyframeAnimation *animation = [CAKeyframeAnimation animationWithKeyPath:@"transform.rotation.z"];
    animation.duration = 1.5f;
    animation.repeatCount = INT64_MAX;
    animation.values = @[@(0.0f * M_PI / 180.0f), @(15.0f * M_PI / 180.0f),
                         @(0.0f * M_PI / 180.0f), @(-15.0f * M_PI / 180.0f),
                         @(0.0f * M_PI / 180.0f)];
    
    animation.keyTimes = @[@(0.0f), @(0.26f), @(0.50f), @(0.74f), @(1.0f)];
    animation.timingFunctions = @[
                                  [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut],
                                  [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn],
                                  [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseOut],
                                  [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseIn],
                                  ];
    
    [[self.pendulum layer] addAnimation:animation forKey:nil];
}
```




