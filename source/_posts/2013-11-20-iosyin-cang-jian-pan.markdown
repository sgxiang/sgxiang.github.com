---
layout: post
title: "ios隐藏键盘"
date: 2013-11-20 20:51
comments: true
categories: oc
keywords: oc
description: 
---
点击UIViewController的任意地方，就可以dismiss keyboard,最简单的办法就是在VC中override以下方法：

```objc
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
	[self.view endEditing:YES];
}
```

另外的方法：

```objc
//用于拿view比较困难的时候
[[UIApplication sharedApplication]sendAction:@selector(resignFirstResponder) to:nil from:nil forEvent:nil];
```

```objc
[[[UIApplication sharedApplication]keyWindow]endEditing:YES];
```