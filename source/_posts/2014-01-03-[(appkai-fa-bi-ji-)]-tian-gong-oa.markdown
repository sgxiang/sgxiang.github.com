---
layout: post
title: "【APP开发笔记】天工OA"
date: 2014-01-03 22:03
comments: true
categories: 开发
keywords: app ios ios7 总结
description: 
---

从11月28号开始找到工作，就开始忙了，都无暇去顾及整理资料和学习。

在公司前一个星期做了个内部的OA应用，现在慢慢整理一下项目相关的代码片段以备用。

![](http://sgxiang.github.io/images/Screenshot_1.png) 

<!--more-->

![](http://sgxiang.github.io/images/Screenshot_2.png) 

![](http://sgxiang.github.io/images/Screenshot_3.png) 

![](http://sgxiang.github.io/images/Screenshot_4.png)




####UITabBar

```objc
self.tabBar.tintColor = UIColorFromRGB(WHITECOLOR);   //设置tabBar被选中时文字的颜色
UITabBarItem *item=[[UITabBarItem alloc] initWithTitle:@"我的主页" image:nil tag:0];
[item setFinishedSelectedImage:[UIImage imageNamed:@"toolbar_today_work_selected"] withFinishedUnselectedImage:[UIImage imageNamed:@"toolbar_today_work"]];
[item setTitleTextAttributes:[NSDictionary dictionaryWithObjectsAndKeys:UIColorFromRGB(WHITECOLOR), UITextAttributeTextColor,nil] forState:UIControlStateNormal];
```

####修复ios7视图控制器返回手势

在ios7中如果push一个视图控制器，然后设置了self.navigationItem.leftBarButtonItem,那么在push过去的视图上无法使用ios7默认的在最左边滑动返回的功能。

我的解决思路是：

在主视图控制器上

```objc
//.h文件
@interface MainViewController : UINavigationController<UIGestureRecognizerDelegate,UINavigationControllerDelegate>
@end
//.m文件
#define KindClass(x) ([viewController isMemberOfClass:[x class]])
- (void)viewDidLoad
{
    [super viewDidLoad];    
    //修复ios7手势
    __weak MainViewController *weakSelf = self;
    if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)]){
        self.interactivePopGestureRecognizer.delegate = weakSelf;
        self.delegate = weakSelf;
    }
}
- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
{
    //修复ios7手势
    if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)]){
        self.interactivePopGestureRecognizer.enabled = NO;
    }
    [super pushViewController:viewController animated:animated];
}
- (void)navigationController:(UINavigationController *)navigationController didShowViewController:(UIViewController *)viewController animated:(BOOL)animate
{
    
    //修复ios7手势
    if (KindClass(MyInfoTableViewController)||KindClass(UserWorkViewController)||KindClass(OrderViewController)||KindClass(SettingViewController)) {
        if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)]){
            self.interactivePopGestureRecognizer.enabled = NO;
        }
    }else{
        if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)]){
            self.interactivePopGestureRecognizer.enabled = YES;
        }
    }
}
```

在push的时候设置掉手势，当将展示视图的时候在需要手势的视图中开启手势。

####时间

```objc
NSDate *now = [NSDate date];
NSDateFormatter *formatter = [[NSDateFormatter alloc]init];
[formatter setDateFormat:@"HHmm"];
int nowTime = [[formatter stringFromDate:now]intValue];
```

####判断ios7

\#define IOS7 [[[UIDevice currentDevice]systemVersion] floatValue] >= 7.0

####设置颜色

\#define UIColorFromRGB(rgbValue) [UIColor colorWithRed:((float)((rgbValue & 0xFF0000) >> 16))/255.0 green:((float)((rgbValue & 0xFF00) >> 8))/255.0 blue:((float)(rgbValue & 0xFF))/255.0 alpha:1.0]

\#define UIColorFromRGBAndAlpha(rgbValue,alphaValue) [UIColor colorWithRed:((float)((rgbValue & 0xFF0000) >> 16))/255.0 green:((float)((rgbValue & 0xFF00) >> 8))/255.0 blue:((float)(rgbValue & 0xFF))/255.0 alpha:alphaValue]

####通知

```objc
//添加本地通知
-(void)addOrUpdateNotificationWithKey:(NSString *)key{
    [self removeNotificationWithKey:key];
    UILocalNotification *notification = [[UILocalNotification alloc]init];
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"HH:mm"];
    NSDate *pushDate = nil;
    if(notification!=nil){ 
        notification.timeZone = [NSTimeZone defaultTimeZone];
        notification.repeatInterval = kCFCalendarUnitDay;
        if (_user.isUseMusic) {
            notification.soundName = UILocalNotificationDefaultSoundName;//设置声音
        }
        //设置推送时间和内容
        if ([key isEqualToString:notificationOAKey]) {
            pushDate = [dateFormatter dateFromString:_user.oaTime];
            notification.alertBody = @"写日志的时间到了";
        }else{
            notification.alertBody = @"订餐时间到了";
            pushDate = [dateFormatter dateFromString:@"09:00"];
        }
        notification.fireDate = pushDate;
        //设置提示数目
        notification.applicationIconBadgeNumber += 1;
        //设置携带的信息
        NSDictionary *info = [NSDictionary dictionaryWithObject:key forKey:@"key"];
        notification.userInfo = info;
        UIApplication *app = [UIApplication sharedApplication];
        [app scheduleLocalNotification:notification];
    }
    dateFormatter = nil;
    notification = nil;
}
//解除通知
-(void)removeNotificationWithKey:(NSString *)key{
    UIApplication *app = [UIApplication sharedApplication];
    NSArray *localArray = [app scheduledLocalNotifications]; 
    //声明本地通知对象
    UILocalNotification *localNotification = nil;
    if (localArray) {
        for (UILocalNotification *noti in localArray) {
            NSDictionary *dict = noti.userInfo;
            if (dict) {
                NSString *inKey = [dict objectForKey:@"key"];
                if ([inKey isEqualToString:key]) {
                    if (localNotification){
                        localNotification = nil;
                    }
                    localNotification = [noti copy];
                    
                    break;
                }
            }
        }
        //判断是否找到已经存在的相同key的推送
        if (!localNotification) {
            //不存在初始化
            localNotification = [[UILocalNotification alloc] init];
        }
        
        if (localNotification) {
            //不推送 取消推送
            [app cancelLocalNotification:localNotification];
            return;
        }
    }   
}
-(void)removeAllNotification{
    [[UIApplication sharedApplication]cancelAllLocalNotifications];
}
```

程序在前台收到通知的处理：

```objc
- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification*)notification{
    UserObj *user = [[UsersInfoManage sharedInstance]readUser];
    if (!user) {
        return;
    }
    if (user.isUseMusic) {
        //声音
        AudioServicesPlaySystemSound(1007);
    }
    if(user.isUseShake){
        AudioServicesPlaySystemSound(kSystemSoundID_Vibrate);
    }
    if ([[notification.userInfo objectForKey:@"key"]isEqualToString:notificationOAKey]) {
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"OA提醒" message:notification.alertBody delegate:nil cancelButtonTitle:@"确定" otherButtonTitles:nil];
        [alert show];
    }else{
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"报餐提醒" message:notification.alertBody delegate:nil cancelButtonTitle:@"确定" otherButtonTitles:nil];
        [alert show];
    }
    application.applicationIconBadgeNumber -= 1;
}
```

当点击图标进入程序时：

```objc
- (void)applicationDidBecomeActive:(UIApplication *)application
{
    application.applicationIconBadgeNumber = 0;
}
```

当在后台点击通知时：

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
    self.window.backgroundColor = [UIColor whiteColor];
    [self.window makeKeyAndVisible];
    
    application.applicationIconBadgeNumber = 0;
    
    if(launchOptions!=nil){
        //从通知栏点击通知
        UILocalNotification* notification = [launchOptions objectForKey:UIApplicationLaunchOptionsLocalNotificationKey];
        if (notification != nil){
            [self application:application didReceiveLocalNotification:notification];
        }
    }
    return YES;
}
```

####后记

其实一直想找个机会记录一下，可是老是拖了好久之后就忘了。一下子就2014年了，这一年将是很关键的一年，工作，爱情，还有学业各方面都走到了很关键的一步，希望今年能够顺利一点。

有好几本书，还有好多文章没看，希望能够坚持下去看，踏实提升实力，慢慢成熟，努力实现和你的承诺。一起加油。

