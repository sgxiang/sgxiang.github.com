---
layout: post
title: "Mantle使用"
date: 2014-01-11 22:15
comments: true
categories: oc
keywords: 
description: 
---

Mantle可以很方便的去书写一个模型层的代码。

使用它可以很方便的去反序列化JSON或者序列化为JSON(需要在`MTLModel`子类中实现`<MTLJSONSerializing>`协议)

使用一个解释器`MTLJSONAdapter`去转换模型对象。

```objc
NSError *error = nil;
MyObject *myObject = [MTLJSONAdapter modelOfClass:MyObject.class fromJSONDictionary:JSONDictionary error:&error];  //把JSONDictionary转换为模型对象

NSDictionary *JSONDictionary = [MTLJSONAdapter JSONDictionaryFromModel:myObject];   //将模型对象转为JSON字典
```

###`+JSONKeyPathsByPropertyKey`

这个方法返回一个字典，指定着你的模型对象和JSON中关键字的映射关系。

一个例子：

```objc
@interface XYUser : MTLModel

@property (readonly, nonatomic, copy) NSString *name;
@property (readonly, nonatomic, strong) NSDate *createdAt;

@property (readonly, nonatomic, assign, getter = isMeUser) BOOL meUser;

@end

@implementation XYUser

+ (NSDictionary *)JSONKeyPathsByPropertyKey {
    return @{
        @"createdAt": @"created_at",   //这里代表createdAt属性映射JSON中的created_at关键字
        @"meUser": NSNull.null        //meUser不会从JSON中反序列化
    };
}

@end
```

当我们用字典反序列化的时候

```objc
NSDictionary *JSONDictionary = @{
    @"name": @"john",
    @"created_at": @"2013/07/02 16:40:00 +0000",
    @"plan": @"lite"
};

XYUser *user = [MTLJSONAdapter modelOfClass:XYUser.class fromJSONDictionary:JSONDictionary error:&error];

//user中的name为john  createdAt为2013/07/02 16:40:00 +0000 meUser为0 plan将会被忽略
```

###`+JSONTransformerForKey:`

实现这个方法去用一个不一样的类型转换器去转化属性

```objc
+ (NSValueTransformer *)JSONTransformerForKey:(NSString *)key {
    if ([key isEqualToString:@"createdAt"]) {
        return [NSValueTransformer valueTransformerForName:XYDateValueTransformerName];
    }
    return nil;
}
```

对于一些值的转换：

```objc
+ (NSValueTransformer *)createdAtJSONTransformer {
    return [MTLValueTransformer reversibleTransformerWithForwardBlock:^(NSString *str) {
        return [self.dateFormatter dateFromString:str];
    } reverseBlock:^(NSDate *date) {
        return [self.dateFormatter stringFromDate:date];
    }];
}
```

###`+classForParsingJSONDictionary:`

如果定义了一个类簇，可以实现这个方法去转换。

```objc
@interface XYMessage : MTLModel

@end

@interface XYTextMessage: XYMessage

@property (readonly, nonatomic, copy) NSString *body;

@end

@interface XYPictureMessage : XYMessage

@property (readonly, nonatomic, strong) NSURL *imageURL;

@end

@implementation XYMessage

+ (Class)classForParsingJSONDictionary:(NSDictionary *)JSONDictionary {
    if (JSONDictionary[@"image_url"] != nil) {
        return XYPictureMessage.class;
    }

    if (JSONDictionary[@"body"] != nil) {
        return XYTextMessage.class;
    }

    NSAssert(NO, @"No matching class for the JSON dictionary '%@'.", JSONDictionary);
    return self;
}

@end
```

使用

```objc
NSDictionary *textMessage = @{
    @"id": @1,
    @"body": @"Hello World!"
};

NSDictionary *pictureMessage = @{
    @"id": @2,
    @"image_url": @"http://example.com/lolcat.gif"
};

XYTextMessage *messageA = [MTLJSONAdapter modelOfClass:XYMessage.class fromJSONDictionary:textMessage error:NULL];

XYPictureMessage *messageB = [MTLJSONAdapter modelOfClass:XYMessage.class fromJSONDictionary:pictureMessage error:NULL];
```
###数据的持久化

`MTLModel`已经遵循了`<NSCoding>`协议，所以可以`NSKeyedArchiver`归档这个模型对象。



