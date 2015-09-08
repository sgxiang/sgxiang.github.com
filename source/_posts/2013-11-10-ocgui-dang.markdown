---
layout: post
title: "oc归档"
date: 2013-11-10 22:53
comments: true
categories: oc
---

###writeToFile:atomically:

``` objc
NSDictionary *glossary = [NSDictionary dictionaryWithObjectsAndKeys:@"TestClass",@"className",@"TestText",@"classText", nil];

if ([glossary writeToFile:@"glossary" atomically:YES]==NO) {
	NSLog(@"save to file failed");
}else{
    NSLog(@"succeed");
}
        
NSDictionary *gly =  [NSDictionary dictionaryWithContentsOfFile:@"glossary"];
    
for (NSString *key in gly) {
    NSLog(@"%@:%@",key,[glossary objectForKey:key]);
}
```
<!--more-->

###NSKeyedArchiver

```objc
NSDictionary *glossary = [NSDictionary dictionaryWithObjectsAndKeys:@"TestClass",@"className",@"TestText",@"classText", nil];

[NSKeyedArchiver archiveRootObject:glossary toFile:@"glossary.archive"];
        
NSDictionary *read = [NSKeyedUnarchiver unarchiveObjectWithFile:@"glossary.archive"];
        
for (NSString *key in read) {
    NSLog(@"%@:%@",key,[glossary objectForKey:key]);
}
```
    
###归档自定义对象

#####对象要实现NSCoding协议和encodeWithCoder:及initWithCoder:方法

```objc
-(void)encodeWithCoder:(NSCoder *)aCoder{
	[aCoder encodeObject:_name forKey:@"name"];
    [aCoder encodeInt:_age forKey:@"age"];
}
	
-(id)initWithCoder:(NSCoder *)aDecoder{
    _name = [aDecoder decodeObjectForKey:@"name"];
    _age = [aDecoder decodeIntForKey:@"age"];
    return self;
}
```
###使用NSData

```objc
NSMutableData *dataArea = [NSMutableData data];
NSKeyedArchiver *archiver = [[NSKeyedArchiver alloc]initForWritingWithMutableData:dataArea];
	
[archiver encodeObject:_name forKey:@"name"];
[archiver finishEncoding];
	
[dataArea writeToFile:@"fileName" atomically:YES];
```	
	
读取的时候

```objc
NSMutableData *dataArea = [NSMutableData data];
NSKeyedUnarchiver *unarchiver = [[NSKeyedUnarchiver alloc]initForReadingWithData:dataArea];
	
test = [unarchiver decodeObjectForKey:@"name"];
	
[unarchiver finishDecoding];
```