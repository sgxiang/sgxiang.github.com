---
layout: post
title: "ios崩溃报告收集"
date: 2014-08-11 13:16
comments: true
categories: oc
keywords: ios 崩溃报告 异常处理
description: ios崩溃报告收集
---

ios应用崩溃的时候，SDK提供了一个NSSetUncaughtExceptionHandler来处理异常。

`NSSetUncaughtExceptionHandler(handleRootException);`

绑定到`handleRootException`中让其函数来处理异常。

```objc
static void handleRootException( NSException* exception )
{
    NSString* name = [ exception name ];
    NSString* reason = [ exception reason ];
    NSArray* symbols = [ exception callStackSymbols ];
    NSMutableString* strSymbols = [ [ NSMutableString alloc ] init ];
    for ( NSString* item in symbols )
    {
        [ strSymbols appendString: item ];
        [ strSymbols appendString: @"\r\n" ];
    }
    
    NSString*   errorInfo = [NSString stringWithFormat:@" *** Terminating app due to uncaught exception %@ , reason: %@  \r\n  *** First throw call stack: \r\n(\r\n  %@\r\n)", name, reason, strSymbols];
    
    logToFile(errorInfo);
    
}
```
这个函数将崩溃的信息转入字符串之后存入文件中。

我封装了其中的方法，写了一个崩溃报告收集的接口。只需在应用开始时调用即可。

会将崩溃文件按时间保存在文件夹中，当应用每次启动的时候上传到服务器之后便删除（网络连接的自行加入）。

代码在github中：

[https://github.com/sgxiang/YSQLog](https://github.com/sgxiang/YSQLog)