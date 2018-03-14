---
title: iOS10中的通知
date: 2016-12-24 19:30:29
tags:
---

iOS10对推送通知的内容进行了一次大的重构，加入了一个新的框架 `UserNotifications.framework` 来集中管理和使用 iOS 系统中通知的功能，除此之外，还增加了很多额外的很好用的功能，比如：
* 撤回一条通知
* 更新已经展示的通知
* 中途修改通知内容
* 在通知中展示图片视频
* 自定义通知 UI 等


## 一、普通通知与其流程
远程通知 \(`Remote Notification`\)的工作流程：

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/remoteNotificationWorkFlow.png)

本地通知\(`Local Notification`\)的工作流程：

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/notification-flow.png)


### 1.1 基本流程

首先你需要向用户请求推送权限，然后发送通知。对于发送出的通知，如果你的应用位于后台或者没有运行的话，系统将通过用户允许的方式 \(弹窗，横幅，或者是在通知中心\) 进行显示。如果你的应用已经位于前台正在运行，你可以自行决定要不要显示这个通知。最后，如果你希望用户点击通知能有打开应用以外的额外功能的话，你也需要进行处理。

### 1.2 权限申请

#### 1.2.1 通用权限

iOS 8 之前，本地推送 \(`UILocalNotification`\) 和远程推送 \(`Remote Notification`\) 是区分对待的，应用只需要在进行远程推送时获取用户同意。iOS 8 对这一行为进行了规范，因为无论是本地推送还是远程推送，其实在用户看来表现是一致的，都是打断用户的行为。因此从 iOS 8 开始，这两种通知都需要申请权限。于是就有了下面这一段请求用户通知权限的代码：
```objc
#ifdef __IPHONE_8_0
    // 注册接收通知的类型
    UIUserNotificationType type = UIUserNotificationTypeBadge | UIUserNotificationTypeSound | UIUserNotificationTypeAlert;
    UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:type categories:nil];
    [application registerUserNotificationSettings:settings];
    
    // 注册允许接收远程推送通知
    [application registerForRemoteNotifications];
#else
    // 如果是iOS 3.0 ~ 7.0，使用以下方法注册
    [application registerForRemoteNotificationTypes:UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound];
#endif
    
```
iOS 10 里进一步消除了本地通知和推送通知的区别。向用户申请通知权限非常简单：

**Objective-C:**
```objc
[[UNUserNotificationCenter currentNotificationCenter] requestAuthorizationWithOptions:UNAuthorizationOptionBadge | UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionCarPlay completionHandler:^(BOOL granted, NSError * _Nullable error) {
    if (granted){
        //用户允许进行通知
    } 
}];
```
**Swift:**
```swift
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) {
    granted, error in
    if granted {
        // 用户允许进行通知
    }
}
```

注意：在使用 UN 开头的 API 的时候，不要忘记导入 UserNotifications 框架：

**Objective-C:**
```objc
#import <UserNotifications/UserNotifications.h>
```
**Swift:**
```swift
import UserNotifications
```

第一次调用这个方法时，会弹出一个系统弹窗。

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/Authorization.png)

要注意的是，一旦用户拒绝了这个请求，再次调用该方法也不会再进行弹窗，想要应用有机会接收到通知的话，用户必须自行前往系统的设置中为你的应用打开通知。

#### 1.2.2 远程推送

一旦用户同意后，你就可以在应用中发送本地通知了。不过如果你通过服务器发送远程通知的话，还需要多一个获取用户 token 的操作。你的服务器可以使用这个 token 将用向 `Apple Push Notification service` 苹果推送通知服务器(以下简称为APNs)提交请求，然后 APNs 通过 token 识别设备和应用，将通知推给用户。

> 注意：注册 APNs 的 deviceToken 的操作是iOS10中，**唯一** 不在新框架中的 API。

* 使用 `UIApplication` 的`registerForRemoteNotifications` 方法提交 token 请求来注册远程通知
* 在 `AppDelegate` 的`application:didRegisterForRemoteNotificationsWithDeviceToken:` 回调方法中获取用户 token

下面附上两张图来回顾下DeviceToken的获取过程。

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/getDeviceToken1.png)

手机设备向APNs发起获取token的请求，APNs通过描述文件里的手机设备信息生成deviceID，通过App信息生成token包，最终将其译成密文token发送给手机设备，然后手机设备再将token发送给公司服务器。

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/getDeviceToken2.png)


### 1.3 发送通知
让我们先来看看iOS10之前的通知是如何发送的：
```objc
// 1.创建本地推送通知对象
UILocalNotification *ln = [[UILocalNotification alloc] init];
// 2.设置通知属性
// 音效文件名
ln.soundName = @"sound.wav";
// 通知的具体内容
ln.alertBody = @"重大新闻：xxxx xxxx被调查了....";
// 锁屏界面显示的小标题（"滑动来" + alertAction）
ln.alertAction = @"查看新闻吧";
// 通知第一次发出的时间(5秒后发出)
ln.fireDate = [NSDate dateWithTimeIntervalSinceNow:5];
// 设置时区（跟随手机的时区）
ln.timeZone = [NSTimeZone defaultTimeZone];
// 设置app图标徽章数字
ln.applicationIconBadgeNumber = 5;
// 设置通知的额外信息
ln.userInfo = @{
                @"icon" : @"test.png",
                @"title" : @"重大新闻",
                @"time" : @"2016-11-14 11:19",
                @"body" : @"韩国民众阻止朴槿惠下台群众示威运动！"
                };
// 3.调度通知（启动任务）
[[UIApplication sharedApplication] scheduleLocalNotification:ln];
```
可以看到，iOS10之前的通知还是基于对象`UILocalNotification`来发送的。

那么在iOS10中，`UserNotifications.framework` 中对通知进行了统一。我们通过通知的内容 \(`UNNotificationContent`\)，发送的时机 \(`UNNotificationTrigger`\) 以及一个发送通知的 `String` 类型的标识符，来生成一个`UNNotificationRequest` 类型的发送请求。最后，我们将这个请求添加到`[UNUserNotificationCenter currentNotificationCenter]` 中，就可以等待通知到达了：

**Objective-C:**
```objc
// 1. 创建通知内容
UNMutableNotificationContent *content = [[UNMutableNotificationContent alloc]init];
content.title = @"时间戳通知";
content.body = @"我的第一条通知";
// 2. 创建发送触发器
UNTimeIntervalNotificationTrigger *trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:timeInterval repeats:NO];
// 3. 发送请求标识符
NSString *requestIdentifier = [NotificationHandler userNotiRawValue:self.notificationType];
// 4. 创建一个发送请求
UNNotificationRequest *request =[UNNotificationRequest requestWithIdentifier:requestIdentifier content:content trigger:trigger];
// 将请求添加到发送中心
[[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
    if (error) {
        [UIAlertController showConfirmAlertWithMessage:error.localizedDescription Controller:self]
    } else{
        [NSString stringWithFormat:@"时间戳通知\"%@\"已安排",requestIdentifier];
    }
}];
```
**Swift:**
```swift
// 1. 创建通知内容
let content = UNMutableNotificationContent()
content.title = "时间戳通知"
content.body = "我的第一条通知"
// 2. 创建发送触发器
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
// 3. 发送请求标识符
let requestIdentifier = "com.onevcat.usernotification.myFirstNotification"
// 4. 创建一个发送请求
let request = UNNotificationRequest(identifier: requestIdentifier, content: content, trigger: trigger)
// 将请求添加到发送中心
UNUserNotificationCenter.current().add(request) { error in
    if error == nil {
        print("Time Interval Notification scheduled: \(requestIdentifier)")
    }
}
```

1. 触发器 `trigger` 是只对本地通知而言的，远程推送的通知的话默认会在收到后立即显示。现在 `UserNotifications` 框架中提供了三种触发器，分别是：
    * `UNTimeIntervalNotificationTrigger` 在一定时间后触发
    * `UNCalendarNotificationTrigger` 在某月某日某时触发
    * `UNLocationNotificationTrigger` 在用户进入或是离开某个区域时触发

2. 请求标识符 `identifier` 可以用来区分不同的通知请求，在将一个通知请求提交后，通过特定 API 我们能够使用这个标识符来取消或者更新这个通知，而这些`CURD`的操作最重要的也就是这个标识符。

3. 在iOS10的通知框架`UserNotification.framework`中，苹果的设计思路借用了一部分网络请求的概念。组织并发送一个通知请求，然后将这个请求提交给 `UNUserNotificationCenter` 用户通知中心进行处理。之后会在 delegate 中接收到这个通知请求对应的 response，另外我们也有机会在应用的 extension 中对 request 进行处理。

在提交通知请求后，只要锁屏或者将应用切到后台，等待设定好的时间后，就能看到通知出现在通知中心或者屏幕横幅了：

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/timeInterval通知.png)

### 1.4 取消和更新
在创建通知请求时，我们已经指定了标识符 `Identifier`。这个标识符`Identifier`可以用来管理通知。在 iOS 10 之前，我们很难取消掉某一个特定的通知，也不能主动移除或者更新已经展示的通知。想象一下你需要推送用户账户内的余额变化情况，多次的余额增减或者变化很容易让用户十分困惑 - 到底哪条通知才是最正确的？又或者在推送一场比赛的比分时，频繁的通知必然导致用户通知中心数量爆炸，而大部分中途的比分对于用户来说只是噪音。


iOS 10 中，UserNotifications 框架提供了一系列管理通知的 API，你可以做到：
* 取消还未展示的通知
* 移除已经展示过的通知
* 更新还未展示的通知
* 更新已经展示过的通知

其中关键就在于在创建请求时使用同样的标识符`Identifier`。

#### 1.4.1 取消还未展示的通知
可以使用 `removePendingNotificationRequestsWithIdentifiers`这个方法，来取消还未展示的通知请求。

**Objective-C:**
```objc
UNTimeIntervalNotificationTrigger *trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:3 repeats:NO];
NSString *identifier = [NotificationHandler userNotiRawValue:UserNotificationTypePendingRemoval];
UNNotificationRequest *request = [UNNotificationRequest requestWithIdentifier:identifier content:self.title1Content trigger:trigger];
[[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
    if (error) {
        [UIAlertController showConfirmAlertWithMessage:error.localizedDescription Controller:self];
    } else{
        NSLog(@"通知请求 \"%@\" 被安排发送", identifier);
    }
}];
[self delay:2 Operation:^{
    NSLog(@"通知请求 \"%@\" 已经被移除", identifier);
    [[UNUserNotificationCenter currentNotificationCenter] removePendingNotificationRequestsWithIdentifiers:@[identifier]];
}];
```
**Swift:**
```swift
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 3, repeats: false)
let identifier = UserNotificationType.pendingRemoval.rawValue
let request = UNNotificationRequest(identifier: identifier, content: title1Content, trigger: trigger)

UNUserNotificationCenter.current().add(request) { error in
    if let error = error {
        UIAlertController.showConfirmAlert(message: error.localizedDescription, in: self)
    } else {
        print("通知请求\(identifier)被安排发送")
    }
}
delay(2) {
    print("通知请求\(identifier)已经被移除")
    UNUserNotificationCenter.current().removePendingNotificationRequests(withIdentifiers: [identifier])
}
```

#### 1.4.2 移除已经展示过的通知
可以使用 `removeDeliveredNotificationsWithIdentifiers`这个方法，来移除已经展示过的通知请求。

**Objective-C:**
```objc
UNTimeIntervalNotificationTrigger *trigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:3 repeats:NO];
NSString *identifier = [NotificationHandler userNotiRawValue:UserNotificationTypeDeliveredRemoval];
UNNotificationRequest *request = [UNNotificationRequest requestWithIdentifier:identifier content:self.title1Content trigger:trigger];
[[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
    if (!error) {
        NSLog(@"通知请求 \"%@\" 已被安排发出", identifier);
    }
}];
[self delay:4 Operation:^{
    NSLog(@"通知请求 \"%@\" 已经被移除", identifier);
    [[UNUserNotificationCenter currentNotificationCenter] removeDeliveredNotificationsWithIdentifiers:@[identifier]];
}];
```
**Swift:**
```swift
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 3, repeats: false)
let identifier = "com.onevcat.usernotification.notificationWillBeRemovedAfterDisplayed"
let request = UNNotificationRequest(identifier: identifier, content: content, trigger: trigger)
UNUserNotificationCenter.current().add(request) { 
    error in
    if error != nil {
        print("通知请求:\(identifier)已被安排发出")
    }
}
delay(4) {
    print("通知请求:\(identifier)已经被移除")
    UNUserNotificationCenter.current().removeDeliveredNotifications(withIdentifiers: [identifier])
}   
```

#### 1.4.3 更新通知
对于更新通知，可以使用 `add`这个方法，不论是否已经展示，都和一开始添加请求时一样，再次将请求提交给 `UNUserNotificationCenter` 即可来更新已经展示过的通知请求。

**Objective-C:**
```objc
[[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:request withCompletionHandler:^(NSError * _Nullable error) {
    if (!error) 
        NSLog(@"通知请求 \"%@\" 已被安排发出", identifier);
}];
[self delay:2 Operation:^{
    UNTimeIntervalNotificationTrigger *newTrigger = [UNTimeIntervalNotificationTrigger triggerWithTimeInterval:1 repeats:NO];
    UNNotificationRequest *newRequest = [UNNotificationRequest requestWithIdentifier:identifier content:self.title2Content trigger:newTrigger];
    [[UNUserNotificationCenter currentNotificationCenter] addNotificationRequest:newRequest withCompletionHandler:^(NSError * _Nullable error) {
        if (!error) 
            NSLog(@"通知 \"%@\" 已经被更新", identifier);
    }];
}];
```
**Swift:**
```swift
UNUserNotificationCenter.current().add(request) { error in
    if error == nil {
        print("通知请求\(identifier)已被安排发出")
    }
}
delay(2) {
    let newTrigger = UNTimeIntervalNotificationTrigger(timeInterval: 1, repeats: false)
    let newRequest = UNNotificationRequest(identifier: identifier, content:newContent, trigger: newTrigger)
    UNUserNotificationCenter.current().add(newRequest) { error in
        if error == nil {
            print("通知\(identifier)已经被更新")
        }
    }
}
```

### 1.5 处理通知
先来看看在iOS10之前通知的处理方式，本地通知和远程通知的点击分别进入了两个不同的回调方法：
* `application:didReceiveLocalNotification:`当用户点击本地通知进入app的时候调用
* `application:didReceiveRemoteNotification:` 当用户点击远程通知进入app的时候调用

可以看出，如果App正处于前台接收到了通知，虽然方法能够监听到通知，但是前台的通知是无法被展示出来的。

#### 1.5.1 应用内展示通知

iOS10下的现在，当应用处于前台时，收到的通知是可以通过操作将其展示在前台的。

`UNUserNotificationCenterDelegate` 提供了两个方法，分别对应如何在应用内展示通知，和收到通知响应时要如何处理的工作。我们可以实现这个接口中的对应方法来在应用内展示通知：
* `userNotificationCenter: willPresentNotification: withCompletionHandler:`通知即将被展示的时候调用
* `userNotificationCenter: didReceiveNotificationResponse: withCompletionHandler:`通知被操作点击的时候调用

**Objective-C:**
```objc
//__IOS_AVAILABLE(10.0) __TVOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0)
- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler{
//    typedef NS_OPTIONS(NSUInteger, UNNotificationPresentationOptions) {
//        UNNotificationPresentationOptionBadge   = (1 << 0),
//        UNNotificationPresentationOptionSound   = (1 << 1),
//        UNNotificationPresentationOptionAlert   = (1 << 2),
//    } __IOS_AVAILABLE(10.0) __TVOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0);
    // 如果不想显示某个通知，可以直接用空 options 调用 completionHandler(0)
    completionHandler(UNNotificationPresentationOptionAlert | UNNotificationPresentationOptionSound);
}
```
**Swift:**
```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, 
                   willPresent notification: UNNotification, 
                   withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) 
{
    completionHandler([.alert, .sound])

    // 如果不想显示某个通知，可以直接用空 options 调用 completionHandler:
    // completionHandler([])
}
```
**注意：实现这两个方法的实例对象是 `UNUserNotificationCenter` 的 `delegate`。推荐在`AppDelegate` 的 `application:didFinishLaunchingWithOptions:`方法中为代理赋值，至于为什么推荐在这个方法中为delegate赋值，在接下去会指出。**

**Objective-C:**
```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    self.notificationHandler = (id)[[NotificationHandler alloc]init];
    [UNUserNotificationCenter currentNotificationCenter].delegate = self.notificationHandler;
    return YES;
}
```
**Swift:**
```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    let notificationHandler = NotificationHandler()
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        UNUserNotificationCenter.current().delegate = notificationHandler
        return true
    }
}
```
#### 1.5.2 对通知进行响应
`UNUserNotificationCenterDelegate` 中还有一个方法，`userNotificationCenter:didReceive:withCompletionHandler:`。**这个代理方法会在用户与你推送的通知进行交互时被调用**，包括用户通过通知打开了你的应用，或者点击或者触发了某个 action (之后会提到 actionable 的通知)。因为涉及到打开应用的行为，所以实现了这个方法的 delegate 必须在 `applicationDidFinishLaunching:` 返回前就完成设置，这也是我们之前推荐将 `NotificationHandler` 尽早进行赋值的理由。

**Objective-C:**
```objc
// __IOS_AVAILABLE(10.0) __WATCHOS_AVAILABLE(3.0) __TVOS_PROHIBITED
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler{
    completionHandler();  // 交给系统来处理执行收到通知之后的操作
}
```
**Swift:**
```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    completionHandler()
}
```

## 二、可交互通知发送和处理
### 2.1 注册 Category
引入了可以交互的通知，这是通过将一簇 action 放到一个 category 中，将这个 category 进行注册，最后在发送通知时将通知的 category 设置为要使用的 category 来实现的。

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/notification-category.png)

注册一个 category :

**Objective-C:**
```objc
- (void)registerNotificationCategory{
    //注册Action Category
    NSString *inputActionId = [NotificationHandler saySomethingRawValue:SaySomethingCategoryActionInput];
    UNTextInputNotificationAction *inputAction = [UNTextInputNotificationAction actionWithIdentifier:inputActionId title:@"输入" options: UNNotificationActionOptionForeground textInputButtonTitle:@"发送" textInputPlaceholder:@"你想说什么..."];
    
    NSString *goodbyeActionId = [NotificationHandler saySomethingRawValue:SaySomethingCategoryActionGoodbye];
    UNNotificationAction *goodbyeAction = [UNNotificationAction actionWithIdentifier:goodbyeActionId title:@"再见" options:UNNotificationActionOptionForeground];
    
    NSString *cancelActionId = [NotificationHandler saySomethingRawValue:SaySomethingCategoryActionNone];
    UNNotificationAction *cancelAction = [UNNotificationAction actionWithIdentifier:cancelActionId title:@"取消" options:UNNotificationActionOptionForeground];
    
    NSString *saySomethingCategoryIdentifier = [NotificationHandler categoryRawValue:UserNotificationCategoryTypeSaySomething];
    UNNotificationCategory *saySomethingCategory = [UNNotificationCategory categoryWithIdentifier:saySomethingCategoryIdentifier actions:@[inputAction, goodbyeAction, cancelAction] intentIdentifiers:@[] options:UNNotificationCategoryOptionCustomDismissAction];
    
    //将类别集合注册 进入通知中心
    NSSet *categorySet = [NSSet setWithObjects:saySomethingCategory, nil];
    [[UNUserNotificationCenter currentNotificationCenter] setNotificationCategories:categorySet];
}
```
**Swift:**
```swift
private func registerNotificationCategory() {
    let saySomethingCategory: UNNotificationCategory = {
        let inputAction = UNTextInputNotificationAction(
            identifier: "action.input",
            title: "输入",
            options: [.foreground],
            textInputButtonTitle: "发送",
            textInputPlaceholder: "你想说什么...")

        let goodbyeAction = UNNotificationAction(
            identifier: "action.goodbye",
            title: "再见",
            options: [.foreground])

        let cancelAction = UNNotificationAction(
            identifier: "action.cancel",
            title: "取消",
            options: [.destructive])

        return UNNotificationCategory(identifier:"saySomethingCategory", actions: [inputAction, goodbyeAction, cancelAction], intentIdentifiers: [], options: [.customDismissAction])
    }()
    UNUserNotificationCenter.current().setNotificationCategories([saySomethingCategory])
}
```
1. `UNTextInputNotificationAction` 代表一个输入文本的 action，你可以自定义框的按钮 title 和 placeholder。你稍后会使用 identifier 来对 action 进行区分。
2. 普通的 `UNNotificationAction` 对应标准的按钮。
3. 为 category 指定一个 identifier，我们将在实际发送通知的时候用这个标识符进行设置，这样系统就知道这个通知对应哪个 category 了。

当然，不要忘了在程序启动时调用`registerNotificationCategory`这个方法进行注册

### 2.2 发送一个带有 action 的通知
在完成 category 注册后，发送一个 actionable 通知就非常简单了，只需要在创建 `UNNotificationContent` 时把 categoryIdentifier 设置为需要的 category id 即可：`content.categoryIdentifier = @"saySomethingCategory"`
尝试展示这个通知，在下拉或者使用 3D touch 展开通知后，就可以看到对应的 action 了：

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/Actionable.png)

远程推送也可以使用 category，只需要在 `payload` 中添加 `category` 字段，并指定预先定义的 `category id` 就可以了：
```json
{
  "aps":{
    "alert":"Please say something",
    "category":"saySomething"
  }
}
```
### 2.3 处理 actionable 通知
和普通的通知并无二致，actionable 通知也会走到 `userNotificationCenter: didReceiveNotificationResponse: withCompletionHandler:` 这个代理方法，我们通过 request 中包含的 `categoryIdentifier` 和 response 里的 `actionIdentifier` 就可以轻易判定是**哪个通知的哪个操作被执行了**。对于 `UNTextInputNotificationAction` 触发的 response，直接将它转换为一个 `UNTextInputNotificationResponse`，就可以拿到其中的用户输入了：

**Objective-C:**
```objc
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void(^)())completionHandler{
    if (response.notification.request.content.categoryIdentifier.length > 0) {//判断是不是category类型的通知
        UserNotificationCategoryType categoryType = [NotificationHandler notiCategoryTypeWithRawValue:response.notification.request.content.categoryIdentifier];
        switch (categoryType) {
            case UserNotificationCategoryTypeSaySomething:
                [self handleSaySomthing:response];
                break;
            default:
                break;
        }
    }
    completionHandler();  // 交给系统来处理执行收到通知之后的操作
}

- (void)handleSaySomthing:(UNNotificationResponse *)response{
    NSString *text = @"";
    SaySomethingCategoryAction actionType = [NotificationHandler categoryActionWithRawValue:response.actionIdentifier];
    
    if (actionType) {
        switch (actionType) {
            case SaySomethingCategoryActionInput:
                text = [(UNTextInputNotificationResponse *)response userText];
                break;
            case SaySomethingCategoryActionGoodbye:
                text = @"再见";
                break;
            case SaySomethingCategoryActionNone:
                text = @"";
                break;
            default:
                break;
        }
    }
    if (text.length > 0) {
        [UIAlertController showConfirmAlertFromTopViewControllerWithMessage:[NSString stringWithFormat:@"你刚刚说：\"%@\" ", text]];
    }
}
```
**Swift:**
```swift
func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
    if let category = UserNotificationCategoryType(rawValue: response.notification.request.content.categoryIdentifier) {
        switch category {
        case .saySomething:
            handleSaySomthing(response: response)
        }
    }
    completionHandler()
}

private func handleSaySomthing(response: UNNotificationResponse) {
    let text: String
    if let actionType = SaySomethingCategoryAction(rawValue: response.actionIdentifier) {
        switch actionType {
        case .input: text = (response as! UNTextInputNotificationResponse).userText
        case .goodbye: text = "再见"
        case .none: text = ""
        }
    } else {
        text = ""
    }
    if !text.isEmpty {
        UIAlertController.showConfirmAlertFromTopViewController(message: "你刚刚说：\(text)")
    }
}
```
上面的代码先判断通知响应是否属于 "saySomething"，然后从用户输入或者是选择中提取字符串，并且弹出一个 alert 作为响应结果。

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/saySomething.png)

当然了，在实际的App开发中，这种 actionable通知 的情况可能更多的是用户的一个action会发送一个对应的网络请求，或者更新一些 UI 等。


## 三、Notification Extension
iOS 10 中添加了很多 extension，作为应用与系统整合的入口。与通知相关的 extension 有两个：
* `Service Extension` 可以让我们有机会在收到远程推送的通知后，展示之前对通知内容进行修改
* `Content Extension` 用来自定义通知视图的样式

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/notification-extensions.png)

### 3.1 截取并修改通知内容

**Objective-C:**
```objc
- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    
    if (self.bestAttemptContent) {
        if ([request.identifier isEqualToString:@"mutableContent"]) {
            ;
            self.bestAttemptContent.body = [NSString stringWithFormat:@"%@, 科比·布莱恩特",self.bestAttemptContent.body];
        } else if ([request.identifier isEqualToString:@"media"]){
            NSString *imageURLString = [self.bestAttemptContent.userInfo objectForKey:@"image"];
            NSURL *URL = [NSURL URLWithString:imageURLString];
            if (URL) {
                __weak typeof(self) weakSelf = self;
                [self downloadAndSave:URL handler:^(NSURL *localURL) {
                    if (localURL) {
                       UNNotificationAttachment *attachment = [UNNotificationAttachment attachmentWithIdentifier:@"image_downloaded" URL:localURL options:nil error:nil];
                        weakSelf.bestAttemptContent.attachments = @[attachment];
                    }
                    weakSelf.contentHandler(self.bestAttemptContent);
                }];
            }
        }
    }
    self.bestAttemptContent.title = [NSString stringWithFormat:@"%@ [modified]", self.bestAttemptContent.title];
}
```
**Swift:**
```swift
class NotificationService: UNNotificationServiceExtension {
    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?
    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)

        if let bestAttemptContent = bestAttemptContent {
            if request.identifier == "mutableContent" {
                bestAttemptContent.body = "\(bestAttemptContent.body), 科比·布莱恩特"
            }
            contentHandler(bestAttemptContent)
        }
    }
    override func serviceExtensionTimeWillExpire() {
        if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }
}
```
1. `didReceiveNotificationRequest withContentHandler:` 方法中有一个等待发送的通知请求，我们通过修改这个请求中的 content 内容，然后在限制的时间内将修改后的内容调用通过 contentHandler 返还给系统，就可以显示这个修改过的通知了。
2. 在一定时间内没有调用 contentHandler 的话，系统会调用`serviceExtensionTimeWillExpire`这个方法，来告诉你大限已到。你可以选择什么都不做，这样的话系统将当作什么都没发生，简单地显示原来的通知。可能你其实已经设置好了绝大部分内容，只是有很少一部分没有完成，这时你也可以像例子中这样调用 contentHandler 来显示一个变更“中途”的通知。

Service Extension 现在只对远程推送的通知起效.
你可以在推送 payload 中增加一个 mutable-content 值为 1 的项来启用内容修改：

### 3.2 多媒体通知推送
#### 3.2.1 发送本地多媒体通知
相比于旧版本的通知，iOS 10 中另一个亮眼功能是多媒体的推送。开发者现在可以在通知中嵌入图片或者视频，这极大丰富了推送内容的可读性和趣味性。

为本地通知添加多媒体内容十分简单，只需要通过本地磁盘上的文件 URL 创建一个 UNNotificationAttachment 对象，然后将这个对象放到数组中赋值给 content 的 attachments 属性就行了：

**Objective-C:**
```objc
UNMutableNotificationContent *content = [[UNMutableNotificationContent alloc]init];
content.title = @"精彩瞬间";
content.body = @"科比·布莱恩特！夺冠时刻！";
NSArray *imageNames = @[@"image", @"onecat"];
NSMutableArray *attachments = [NSMutableArray array];
for (NSString *imgName in imageNames) {
    NSURL *imgURL = [[NSBundle mainBundle]URLForResource:imgName withExtension:@"jpg"];
    NSString *identifier = [NSString stringWithFormat:@"image-%@",imgName];
    UNNotificationAttachment *attachment = [UNNotificationAttachment attachmentWithIdentifier:identifier URL:imgURL options:nil error:nil];
    [attachments addObject:attachment];
}
content.attachments = attachments;
```
**Swift:**
```swift
let content = UNMutableNotificationContent()
content.title = "精彩瞬间"
content.body = "科比·布莱恩特！夺冠时刻！"
if let imageURL = Bundle.main.url(forResource: "image", withExtension: "jpg"),
   let attachment = try? UNNotificationAttachment(identifier: "imageAttachment", url: imageURL, options: nil)
{
    content.attachments = [attachment]
}
```
在显示时，横幅或者弹窗将附带设置的图片，使用 3D Touch pop 通知或者下拉通知显示详细内容时，图片也会被放大展示：

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/mediaNotification.png)
![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/mediaNotificationDetail.png)

除了图片以外，通知还支持音频以及视频。你可以将 MP3 或者 MP4 这样的文件提供给系统来在通知中进行展示和播放。
> 不过，这些文件都有尺寸的限制，
> * 图片不能超过 5MB，
> * 视频不能超过 50MB

关于支持的文件格式和尺寸，可以在文档中进行确认。在创建`UNNotificationAttachment` 时，如果遇到了不支持的格式，SDK 也会抛出错误。

#### 3.2.2 远程推送多媒体通知
这要借助 `Notification Service Extension` 来修改推送通知内容的技术。
一般做法是，我们在推送的 payload 中指定需要加载的图片资源地址，这个地址可以是应用 bundle 内已经存在的资源，也可以是网络的资源。不过因为在创建 `UNNotificationAttachment` 时我们只能使用本地资源，所以如果多媒体还不在本地的话，我们需要先将其下载到本地。在完成 UNNotificationAttachment 创建后，我们就可以和本地通知一样，将它设置给 attachments 属性，然后调用 contentHandler 了。

示例 payload 如下：
```json
{
  "aps":{
    "alert":{
      "title":"Image Notification",
      "body":"Show me an image from web!"
    },
    "mutable-content":1
  },
  "image": "https://onevcat.com/assets/images/background-cover.jpg"
}
```
`mutable-content` 表示我们会在接收到通知时对内容进行更改，`image` 指明了目标图片的地址。
在 `NotificationService` 里，加入如下代码来下载图片，并将其保存到磁盘缓存中：

**Objective-C:**
```objc
- (void)downloadAndSave:(NSURL *)url handler:(void(^)(NSURL *localURL))handler{
    NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithURL:url completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        NSURL *loccalURL;
        if (data) {
            NSString *ext = url.absoluteString.pathExtension;
            NSURL *cacheURL = [NSURL fileURLWithPath:NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject];
            
            NSURL *urlPath = [[cacheURL URLByAppendingPathComponent:url.absoluteString.md5] URLByAppendingPathExtension:ext];
            
            if ([data writeToURL:urlPath atomically:YES]) {
                loccalURL = urlPath;
            };
        }
        handler(loccalURL);
    }];
    [task resume];
}
```
**Swift:**
```swift
private func downloadAndSave(url: URL, handler: @escaping (_ localURL: URL?) -> Void) {
    let task = URLSession.shared.dataTask(with: url, completionHandler: {
        data, res, error in

        var localURL: URL? = nil

        if let data = data {
            let ext = (url.absoluteString as NSString).pathExtension
            let cacheURL = URL(fileURLWithPath: FileManager.default.cachesDirectory)
            let url = cacheURL.appendingPathComponent(url.absoluteString.md5).appendingPathExtension(ext)

            if let _ = try? data.write(to: url) {
                localURL = url
            }
        }

        handler(localURL)
    })

    task.resume()
}
```
然后在 `didReceiveNotificationRequest: withContentHandler` 中，接收到这类通知时提取图片地址，下载，并生成 attachment，进行通知展示：

**Objective-C:**
```objc
NSString *imageURLString = [self.bestAttemptContent.userInfo objectForKey:@"image"];
NSURL *URL = [NSURL URLWithString:imageURLString];
if (URL) {
    __weak typeof(self) weakSelf = self;
    [self downloadAndSave:URL handler:^(NSURL *localURL) {
        if (localURL) {
           UNNotificationAttachment *attachment = [UNNotificationAttachment attachmentWithIdentifier:@"image_downloaded" URL:localURL options:nil error:nil];
            weakSelf.bestAttemptContent.attachments = @[attachment];
        }
        weakSelf.contentHandler(self.bestAttemptContent);
    }];
}
```
**Swift:**
```swift
if let imageURLString = bestAttemptContent.userInfo["image"] as? String,
   let URL = URL(string: imageURLString)
{
    downloadAndSave(url: URL) { localURL in
        if let localURL = localURL {
            do {
                let attachment = try UNNotificationAttachment(identifier: "image_downloaded", url: localURL, options: nil)
                bestAttemptContent.attachments = [attachment]
            } catch {
                print(error)
            }
        }
        contentHandler(bestAttemptContent)
    }
}
```

### 3.3 自定义通知视图样式
iOS 10 SDK 新加的另一个 `Content Extension` 可以用来自定义通知的详细页面的视图。新建一个 `Notification Content Extension`，Xcode 为我们准备的模板中包含了一个实现了：
* `NotificationViewController` : `UNNotificationContentExtension` 的 `UIViewController` 子类;
* `MainInterface.stroyboard`：自定义UI视图的一个storyboard;
* `info.plist`：用来配置一些内容。

#### 3.3.1 普通自定义UI的通知实现
这个 extension 中有一个必须实现的方法 `didReceiveNotification:notification`，在系统需要显示自定义样式的通知详情视图时，这个方法将被调用，你需要在其中配置你的 UI。而 UI 本身可以通过这个 extension 中的 `MainInterface.storyboard` 来进行定义。自定义 UI 的通知是和通知 category 绑定的，我们需要在 extension 的 Info.plist 里指定这个通知样式所对应的 category 标识符：

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/info_plist.png)

系统在接收到通知后会先查找有没有能够处理这类通知的 content extension，如果存在，那么就交给 extension 来进行处理。
另外，在构建 UI 时，我们可以通过 Info.plist 控制通知详细视图的尺寸，以及是否显示原始的通知。

**虽然我们可以使用包括按钮在内的各种 UI，但是系统不允许我们对这些 UI 进行交互。点击通知视图 UI 本身会将我们导航到应用中。**

![](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/customUINoti.png)

不过我们可以通过 action 的方式来对自定义 UI 进行更新。`UNNotificationContentExtension` 为我们提供了一个可选方法 `didReceiveNotificationResponse: completionHandler:`，它会在用户选择了某个 action 时被调用，你有机会在这里更新通知的 UI。

**Objective-C:**
```objc
- (void)didReceiveNotification:(UNNotification *)notification {
    UNNotificationContent *content = notification.request.content;
    
    NSArray *items = [content.userInfo objectForKey:@"items"];
    NSInteger i = 0;
    for (NSDictionary *item in items) {
        NSString *title = [item objectForKey:@"title"];
        NSString *text = [item objectForKey:@"text"];
        if (!title.length || !text.length) {
            continue;
        }
        NSURL *url = content.attachments[i].URL;
        
        NotificationPresentItem *presentItem = [NotificationPresentItem itemWithURL:url Title:title Text:text];
        [self.items addObject:presentItem];
        i++;
    }
    [self updateUI:0];
}

- (void)updateUI:(NSInteger)index{
    NotificationPresentItem *item = self.items[index];
    if (item.url.startAccessingSecurityScopedResource){
        self.imageView.image = [UIImage imageWithContentsOfFile:item.url.path];
        [item.url stopAccessingSecurityScopedResource];
    }

    self.label.text = item.title;
    self.textView.text = item.text;
    
    self.index = index;
}
```
**Swift:**
```swift
func didReceive(_ notification: UNNotification) {
    let content = notification.request.content
    if let items = content.userInfo["items"] as? [[String: AnyObject]] {
        for i in 0..<items.count {
            let item = items[i]
            guard let title = item["title"] as? String, let text = item["text"] as? String else {
                continue
            }
            if i > content.attachments.count - 1 {
                continue
            }
            let url = content.attachments[i].url
            
            let presentItem = NotificationPresentItem(url: url, title: title, text: text)
            self.items.append(presentItem)
        }
    }
    updateUI(index: 0)
}
    
private func updateUI(index: Int) {
    let item = items[index]
    if item.url.startAccessingSecurityScopedResource() {
        imageView.image = UIImage(contentsOfFile: item.url.path)
        item.url.stopAccessingSecurityScopedResource()
    }
    label.text = item.title
    textView.text = item.text
    self.index = index
}
```

#### 3.3.2 有UI更新的自定义通知
如果有 UI 更新，那么在方法的 completionHandler 中，开发者可以选择传递 `UNNotificationContentExtensionResponseOptionDoNotDismiss` 来保持通知继续被显示。如果没有继续显示的必要，可以选择 
* `UNNotificationContentExtensionResponseOptionDismissAndForwardAction` 把通知的 `action` 继续传递给应用的 `UNUserNotificationCenterDelegate` 中的`userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:` 
* `UNNotificationContentExtensionResponseOptionDismiss`，直接解散这个通知。


**Objective-C:**
```objc
- (void)didReceiveNotificationResponse:(UNNotificationResponse *)response completionHandler:(void (^)(UNNotificationContentExtensionResponseOption))completion{
    if ([response.actionIdentifier isEqualToString:@"switch1"]) {
        NSInteger nextIndex;
        if (self.index == 0) {
            nextIndex = 1;
        } else{
            nextIndex = 0;
        }
        [self updateUI:nextIndex];
        completion(UNNotificationContentExtensionResponseOptionDoNotDismiss);
    } else if ([response.actionIdentifier isEqualToString:@"open2"]) {
        completion(UNNotificationContentExtensionResponseOptionDismissAndForwardAction);
    } else if ([response.actionIdentifier isEqualToString:@"dismiss3"])  {
        completion(UNNotificationContentExtensionResponseOptionDismiss);
    } else {
        completion(UNNotificationContentExtensionResponseOptionDismissAndForwardAction);
    }
}
```
**Swift:**
```swift
func didReceive(_ response: UNNotificationResponse, completionHandler completion: @escaping (UNNotificationContentExtensionResponseOption) -> Void) {
    if response.actionIdentifier == "switch" {
        let nextIndex: Int
        if index == 0 {
            nextIndex = 1
        } else {
            nextIndex = 0
        }
        
        updateUI(index: nextIndex)
        completion(.doNotDismiss)
    } else if response.actionIdentifier == "open" {
        completion(.dismissAndForwardAction)
    } else if response.actionIdentifier == "dismiss" {
        completion(.dismiss)
    } else {
        completion(.dismissAndForwardAction)
    }
}
```

> 注：如果你的自定义 UI 包含视频等，你还可以实现 `UNNotificationContentExtension` 里的 media 开头的一系列属性，它将为你提供一些视频播放的控件和相关方法。


