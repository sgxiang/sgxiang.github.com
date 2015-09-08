---
layout: post
title: "使用关联引用在类目中实现属性"
date: 2013-11-22 21:04
comments: true
categories: oc
keywords: category,oc,property
description: 
---

类目中不能添加实例变量，所以通过关联引用向对象添加键值数据。

用到的两个方法：

```objc
id objc_getAssociatedObject(id object, const void *key)
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
```

让一个对象与另外一个对象关联，`object`可以保存对`value`的引用并获得这个对象。

policy:

```objc
enum{
	OBJC_ASSOCIATION_ASSIGN = 0,
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1,
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,
    OBJC_ASSOCIATION_RETAIN = 01401,
    OBJC_ASSOCIATION_COPY = 01403
}
```

__eg__:

```objc
#import <objc/runtime.h>
@interface Person(EmailAddress)
@property(nonatomic,readwrite,copy)NSString *emailAddress;
@end

@implementation Person(EmailAddress)

static char emailAddressKey;

-(NSString *)emailAddress{
	return objc_getAssociatedObject(self,&emailAddressKey);
}

-(void)setEmailAddress:(NSString *)emailAddress{
	objc_setAssociatedObject(self,&emailAddressKey,emailAddress,OBJC_ASSOCIATION_COPY);
}
@end
```

那两个方法很方便的就实现了属性的getter和setter。

另外的一个例子：

```objc
//为alert关联一个对象
id someObj = ...;
UIAlertView *alert = [[UIAlertView alloc]
						initWithTitle:@"title"
						message:nil
						delegate:self
						cancelButtonTitle:@"ok"
						otherButtonTitles:nil];
//创建关联
objc_setAssociatedObject(alert,&someKey,someOjb,OBJC_ASSOCIATION_RETAIN_NONATOMIC);
[alert show];

//alert完成后的触发
-(void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex{
	//取得关联
	UIButton *sender = objc_getAssociatedObject(alertView,&someKey);
	NSLog(@"%@",[[sender titleLabel]text]);
}
```

