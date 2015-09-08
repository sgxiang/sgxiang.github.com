---
layout: post
title: "【APP开发笔记】业务通"
date: 2014-01-07 20:27
comments: true
categories: 开发
keywords: 
description: 
---

目前还在做的一个项目。整理些资料以备用。

![](http://sgxiang.github.io/images/yewutong.png)

<!--more-->


####从自定的xib文件中加载view

CustomView *view =  [[NSBundle mainBundle] loadNibNamed:@"CustomView" owner:nil options:nil][0];

####UISearchBar适配

在ios7之前，UISearchBar的视图树是这样的：

UISearchBar => UISearchBarBackgroud、UISearchBarTextField

而在ios7中，视图树变成这样的：

UISearchBar => UIView => UISearchBarBackgroud、UISearchBarTextField

所以适配方面需要修改：

```objc
#import "ywtSearchBar.h"

@implementation ywtSearchBar

- (id)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        // Initialization code
        
        [self load];
    }
    return self;
}

-(void)load{
    self.keyboardType = UIKeyboardTypeDefault;
    //在ios7使提示语居中
    if (IOS7) {
        self.placeholder = [NSString stringWithCString:"搜索（请输入项目关键字）                        "  encoding: NSUTF8StringEncoding];
    }else{
        self.placeholder = [NSString stringWithCString:"搜索（请输入项目关键字）"  encoding: NSUTF8StringEncoding];
    }
}

//禁止默认的取消按钮，在之后可以自定义
- (void) addSubview:(UIView *)view {
    [super addSubview:view];
    if ([view isKindOfClass:UIButton.class]) {

        UIButton *cancelButton = (UIButton *)view;
        [cancelButton setTitle:@"        " forState:UIControlStateNormal];
        cancelButton.enabled = NO;
    }
    
}
//更改搜索框背景图片，适配ios7
- (void)layoutSubviews {
    UITextField *searchField;
    
    __weak UIView *view;
    //由于视图树改变了所以要做相应的判断
    if(!IOS7){
        view = self;
    }else{
        view = [self.subviews objectAtIndex:0];
    }
    
    NSUInteger numViews = [view.subviews count];
    for(int i = 0; i < numViews; i++) {
        if([[view.subviews objectAtIndex:i] isKindOfClass:[UITextField class]]) { //conform?
            searchField = [view.subviews objectAtIndex:i];
        }
    }
    if(!(searchField == nil)) {
        searchField.textColor = [UIColor blackColor];
        [searchField setBackground: [UIImage imageNamed:@"searchBg"] ];
        [searchField setBorderStyle:UITextBorderStyleNone];
    }
    
    [super layoutSubviews];
}
@end
```

而自定义搜索按钮，需要在UISearchDisplayController中修改

```objc
//在将要开始搜索的时候添加上自定义的按钮上去
-(void)searchDisplayControllerDidBeginSearch:(UISearchDisplayController *)controller{
    
    UIView *topView = controller.searchBar.subviews[0];
    
    if (_searchBtn == nil) {
        _searchBtn = [UIButton buttonWithType:UIButtonTypeCustom];
        [_searchBtn setTitle:@"     " forState:UIControlStateNormal];
        [_searchBtn setBackgroundImage:[UIImage imageNamed:@"roundBtn"] forState:UIControlStateNormal];
        [_searchBtn setFrame:CGRectMake(270, 7, 39, 27)];
        [_searchBtn addTarget:self action:@selector(searchBarSearchButtonClicked:) forControlEvents:UIControlEventTouchUpInside];
    }
    if (_searchBtn.superview ==nil) {
         [topView addSubview:_searchBtn];
    }
    
}
//在结束搜索的时候移除自定义的按钮
-(void)searchDisplayControllerWillEndSearch:(UISearchDisplayController *)controller{
    [_searchBtn removeFromSuperview];
}
```

####UIPageControl适配

由于在ios7中UIPageControl的子视图不在是UIImageView所欲无法直接的更改小圆点的图片。做了如下适配：

```objc
#import "YWTPageControl.h"

@implementation YWTPageControl

- (id)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        
        activeImage = [UIImage imageNamed:@"FullPoint"];
        inactiveImage = [UIImage imageNamed:@"NullPoint"];
        
        
        self.userInteractionEnabled = NO;
        
        if ([self respondsToSelector:@selector(setCurrentPageIndicatorTintColor:)] && [self respondsToSelector:@selector(setPageIndicatorTintColor:)]) {
            [self setCurrentPageIndicatorTintColor:[UIColor clearColor]];
            [self setPageIndicatorTintColor:[UIColor clearColor]];
        }
        
        [self setBackgroundColor:[UIColor clearColor]];
        
    }
    return self;
}


- (void)setCurrentPage:(NSInteger)currentPage
{
    [super setCurrentPage:currentPage];
    if (IOS7) {
    	//ios7中重绘视图
        [self setNeedsDisplay];
    }else{
    	//ios7之前之间更换图片
        [self updateDots];
    }
}

- (void)drawRect:(CGRect)iRect
{
    
    if (!IOS7) {
        return;
    }
    
    int i;
    CGRect rect;
    UIImage *image;
    
    iRect = self.frame;
    
    
    if (self.opaque) {
        
        [self.backgroundColor set];
        UIRectFill(iRect);
    }
    
    CGFloat _kSpacing = 5.0f;
    
    if (self.hidesForSinglePage && self.numberOfPages == 1) {
        return;
    }
    
    rect.size.height = activeImage.size.height;
    rect.size.width = self.numberOfPages * activeImage.size.width + (self.numberOfPages - 1) * _kSpacing;
    rect.origin.x = floorf((iRect.size.width - rect.size.width) / 2.0);
    rect.origin.y = floorf((iRect.size.height - rect.size.height) / 2.0);
    rect.size.width = activeImage.size.width;
    
    for (i = 0; i < self.numberOfPages; ++i) {
        image = (i == self.currentPage) ? activeImage : inactiveImage;
        [image drawInRect:rect];
        
        rect.origin.x += activeImage.size.width + _kSpacing;
    }
    
}

-(void) updateDots
{
    for (int i = 0; i < [self.subviews count]; i++)
    {

        UIImageView* dot = [self.subviews objectAtIndex:i];
        if (i == self.currentPage) dot.image = activeImage;
        else dot.image = inactiveImage;
    }
}
@end
```