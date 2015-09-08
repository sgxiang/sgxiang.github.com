---
layout: post
title: "整洁的表视图"
date: 2014-01-06 23:55
comments: true
categories: oc
keywords: 
description: 
---

##1.轻量化的表视图控制器

1. 将Data Source和其他Protocols剥离开视图控制器
2. 将业务的逻辑处理移到Model层
3. 创建Store类，用来处理数据加载，缓存，数据库配置等。
4. 将Web服务逻辑移到Model层
5. 将View代码移到View Layer


<!--more-->



##2.整洁的表视图


###2.1子视图控制器

可以将一个表视图添加到另一个视图控制器，这样表视图控制器只需管理表视图，而父视图可以管理其他额外的界面元素。

```objc
- (void)addPhotoDetailsTableView{
	DetailsViewController *details = [[DetailsViewController alloc] init];
	details.photo = self.photo;
	details.delegate = self;
	[self addChildViewController:details];
	CGRect frame = self.view.bounds;
	frame.origin.y = 110;
	details.view.frame = frame;
	[self.view addSubview:details.view];
	[details didMoveToParentViewController:self];
}
```

表视图可以设置代理让父视图去响应。

###2.2分离关注点

####桥接模型对象和单元之间的差距

我们经常将一个处理视图显示数据的任务放在表示图的数据来源里：

```objc
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
	PhotoCell *cell = [tableView dequeueReusableCellWithIdentifier:@"PhotoCell"];
	Photo *photo = [self itemAtIndexPath:indexPath];	cell.photoTitleLabel.text = photo.name;
	NSString* date = [self.dateFormatter stringFromDate:photo.creationDate];
	cell.photoDateLabel.text = date;
}
```

在上面的代码中，数据源和cell的设计逻辑绑定在一起。我们可以将这部分放在cell的类别里重构：

```objc
@implementation PhotoCell (ConfigureForPhoto)
- (void)configureForPhoto:(Photo *)photo{
	self.photoTitleLabel.text = photo.name;
	NSString* date = [self.dateFormatter stringFromDate:photo.creationDate];
	self.photoDateLabel.text = date;
}
@end
```

这样在调用的时候数据源的代码就会很清晰，同时将关注点分离开了。

###2.3使cell可重用

####在cell中处理cell的状态

比如设置cell高亮和选中状态等。可以把这段逻辑放在cell中。

####处理多种cell类型

```objc
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    NSString *key = self.keys[(NSUInteger) indexPath.row];
    id value = [self.photo valueForKey:key];
    UITableViewCell *cell;
    if ([key isEqual:PhotoRatingKey]) {
        cell = [self cellForRating:value indexPath:indexPath];
    } else {
        cell = [self detailCellForKey:key value:value];
    }
    return cell;
} 
- (RatingCell *)cellForRating:(NSNumber *)rating indexPath:(NSIndexPath *)indexPath{
    // ...
}
- (UITableViewCell *)detailCellForKey:(NSString *)key value:(id)value{
    // ...
}
```

####table view的编辑

在编辑过程中，表数据源通过代理方法来获得通知。将处理数据的任务放在模型层上，控制器扮演视图和模型层之间的协调者。这样逻辑模型会很容易测试，不用理会和控制器层的交互任务。








