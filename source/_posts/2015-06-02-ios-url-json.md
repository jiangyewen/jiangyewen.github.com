---
title: "iOS URL请求和JSON的解析"
date: 2015-06-02
layout: post
description: "本文主要记录如何在iOS中使用webView来查看网页的内容"
tags: [iOS入门,webView,Object C]
---
##如何请求一个给定的url并处理返回的[数据](id:top)

*  将给定的url字符串转化为 NSURL 对象

```c
  NSString * requestString = @"http://bookapi.bignerdranch.com/courses.json";
  NSURL * url = [NSURL URLWithString:requestString];
```

*  初始化请求 NSURLRequest

NSURLRequest 包含了请求的各种参数,如http头部,超时时间,缓存策略等.

```c
  NSString * requestString = @"http://bookapi.bignerdranch.com/courses.json";
  NSURL * url = [NSURL URLWithString:requestString];
  NSURLRequest  *req = [NSURLRequest requestWithURL:url];

```

*  将一个请求看做一个任务,一个任务的初始化状态是suspend, 当发送resume消息给任务后,请求任务就开始执行. 其中,一个任务中包含了回调函数(completionHandler),回调函数对返回的数据(Json或者XML等)进行处理.

```c
- (void) fetchFeed {
    NSString * requestString = @"http://bookapi.bignerdranch.com/courses.json";
    NSURL * url = [NSURL URLWithString:requestString];
    NSURLRequest  *req = [NSURLRequest requestWithURL:url];
    
    NSURLSessionDataTask * dataTask = [self.session
                                       dataTaskWithRequest:req
                                       completionHandler:
					// 回调函数 block, 见下一步
                                       }];
    [dataTask resume]; // 请求正式开始

}
```

* 对JSON格式数据的解析

iOS提供了NSJSONSerialization类,对JSON进行解析,解析后的数据类型为字典(NSDictionary). 最终完整的代码为:

```c
- (void) fetchFeed {
    NSString * requestString = @"http://bookapi.bignerdranch.com/courses.json";
    NSURL * url = [NSURL URLWithString:requestString];
    NSURLRequest  *req = [NSURLRequest requestWithURL:url];
    
    NSURLSessionDataTask * dataTask = [self.session
                                       dataTaskWithRequest:req
                                       completionHandler:
					// 回调函数 block
                                       ^(NSData *data, NSURLResponse *response, NSError *error) { 
                                           NSDictionary *jsonObject = [NSJSONSerialization JSONObjectWithData:data options:0 error:nil];
                                           self.courses = jsonObject[@"courses"];
                                           NSLog(@"%@",self.courses);
                                       }];
    [dataTask resume];

}


```
返回[顶部](#top)

At 10:04 PM

##参考资料
* iOS Programming The Big Nerd Ranch Guide 4th Edition
