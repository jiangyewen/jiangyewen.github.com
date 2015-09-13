---
title: "iOS 保存和读取本地数据"
date: 2015-06-07
layout: post
description: "本文记录了如何将iOS app中的数据,如图片和实例中的属性保存到本地,应用退出或者切换状态后,能够重新恢复数据"
tags: [iOS入门,NSCoding,应用状态,Object C]
---
### 类要遵循的[protocol]和实现的methods  (id:top)

>>要想将一个object中的property保存到本地, 这个object的对应的class应该遵循 NSCoding协议, 并实现其2个methods: 

* NSCoding protocol

```
@interface BNRItem : NSObject <NSCoding>
```

*  encodeWIthCoder 

将class中的property以key-value的形式进行encode, 并将数据流存储到本地的文件系统中, 其中key设定为property的名称.

```
- (void)encodeWithCoder:(NSCoder *)aCoder{
    
    [aCoder encodeObject:self.itemName forKey:@"itemName"];
    [aCoder encodeObject:self.serialNumber forKey:@"serialNumber"];
    [aCoder encodeObject:self.dateCreated forKey:@"dateCreated"];
    [aCoder encodeObject:self.itemKey forKey:@"itemKey"];
    [aCoder encodeInt:self.valueInDollars forKey:@"valueInDollars"];
}


```
* initWithCoder 

从encodeWithCoder中的数据中, 恢复数据, 并赋值给各个property,

```
- (id)initWithCoder:(NSCoder *)aDecoder{
    self = [super init];
    if (self) {
        _itemName = [aDecoder decodeObjectForKey:@"itemName"];
        _serialNumber = [aDecoder decodeObjectForKey:@"serialNumber"];
        _dateCreated = [aDecoder decodeObjectForKey:@"dateCreated"];
        _itemKey = [aDecoder decodeObjectForKey:@"itemKey"];
        _valueInDollars = [aDecoder decodeIntForKey:@"valueInDollars"];
    }
    
    return self;

}
```

### 选定保存数据的路径和目录
每一个应用都有其对应的sandbox, 只有本身可以访问,与其他app隔离, sandbox包含的目录有:

* application bundle: 只读, NIB和图标等文件

* Documents/: 存储的文件, 可用iCloud备份;

* Library/Caches/: 缓存的文件, 不能备份;

* Library/Preferences/: app的setting等设置

* tmp/ : 临时文件, 注意其清理

查找Document/的目录代码并指定保存的文件名称:

```c
- (NSString *) itemArchivePath {
   NSArray *documentDirs =  NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    
    NSString * documentDir = [documentDirs firstObject];
    NSString *fileName = @"item.archive";
    
    return  [documentDir stringByAppendingPathComponent:fileName];

}
```

### 如何开始将数据保存到本地

设定好要保存的property和路径后, 就需要在app进入后台的时候(双击home键), 触发保存事件.

```c
- (void)applicationDidEnterBackground:(UIApplication *)application
{
    // Use this method to release shared resources, save user data, invalidate timers, and store enough application state information to restore your application to its current state in case it is terminated later. 
    // If your application supports background execution, this method is called instead of applicationWillTerminate: when the user quits.
    BOOL success = [[BNRItemStore sharedStore] saveChanges];
    if (success) {
        NSLog(@"saved all items");
    }else{
        NSLog(@"Failed: can't sace items data!");
    }
}

/* save changes保存的函数
*  self.privateItems为一个数组, 里面包含了多个item
*/ NSKeyedArchiver 会递归的调用数组里每一个item的 encodeWithCoder 方法
- (BOOL) saveChanges{
    NSString * fileName = [self itemArchivePath]; 
    return [NSKeyedArchiver archiveRootObject:self.privateItems toFile:fileName];

}
```

### 如何将数据load到app中

```c
NSString *path = [self itemArchivePath];
_privateItems = [NSKeyedUnarchiver unarchiveObjectWithFile:path];
```

### 对于图片的保存, 可以采用NSData的writeToFile:path

```c
// 获取图片的保存目录
- (NSString *)imagePathForKey:(NSString *)key
{
    NSArray *documentDirectories =
            NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,
                                                NSUserDomainMask,
                                                YES);
    NSString *documentDirectory = [documentDirectories firstObject];
    return [documentDirectory stringByAppendingPathComponent:key];
}

// 保存图片到本地
NSString *imagePath = [self imagePathForKey:key];
 // Turn image into JPEG data
    NSData *data = UIImageJPEGRepresentation(image, 0.5);
    // Write it to full path
    [data writeToFile:imagePath atomically:YES];

// 读取本地的图片
NSString *imagePath = [self imagePathForKey:key];
 // Create UIImage object from file
  result = [UIImage imageWithContentsOfFile:imagePath];
```

返回[顶部](#top)

At 15:56 PM

##参考资料
* iOS Programming The Big Nerd Ranch Guide 4th Edition
