---
title:  Realm数据库
date: 2017-09-13 10:31:10
tags:
---

	
### Realm是什么？

[Realm](https://realm.io/cn/)是由Y Combinator公司孵化出来的一款可以用于iOS(同样适用于Swift&Objective-C)和Android的跨平台移动数据库。支持的平台包括Java，Objective-C，Swift，React Native，[Xamarin](http://baike.baidu.com/link?url=zoUIwykilfP1IkvGMPcJcmllWyYEWT441ZR2zLriH7N9lhZUcN1MIv39S8hZVWEZhnegaeFYsTKANdnyV4R1CK)。
从2012年开发打造，2014年7月发布第一个版本，现在目前最新版是[Realm](https://realm.io/cn/) 2.1.2，像amazon,Google,intel,Alibaba,ebay等公司都在用，超过10亿用户+在使用。[Realm](https://realm.io/cn/)核心数据引擎用C++打造，并不是建立在SQLite之上的ORM框架设计，采用[MVCC]((https://en.wikipedia.org/wiki/Multiversion_concurrency_control))（Multi-Version Concurrency Control 多版本并发控制）机制，所以[Realm](https://realm.io/cn/)相比SQLite和CoreData而言更好、更快、更容易去使用和完成数据库的操作。想取代CoredData和sqlite,但它不是对CoreData的简单封装。相反[Realm](https://realm.io/cn/)它使用了它自己的一套持久化[存储引擎](https://realm.io/cn/news/jp-simard-realm-core-database-engine/) 。在[GitHUB](https://github.com/realm/realm-cocoa)上已收获9500多颗星，1200多个Fork，堪称*新一代移动客户端数据库王者*，[Realm](https://realm.io/cn/)单词中文翻译：领域、境界、王国。


### 性能对比(Realm官方提供)

每秒能在20万条数据中进行查询后count的次数。[Realm](https://realm.io/cn/)每秒可以进行30.9次查询后count。

![realm1](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/realm1.jpg)

在20万条中进行一次遍历查询，数据和前面的count相似：[Realm](https://realm.io/cn/)一秒可以遍历20万条数据31次，而coredata只能进行两次查询。

![realm2](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/realm2.jpg)

这是在一次事务每秒插入数据的对比，[Realm](https://realm.io/cn/)每秒可以插入9.4万条记录，在这个比较里纯SQLite的性能最好，每秒可以插入17.8万条记录。然而封装了SQLite的FMDB的成绩大概是[Realm](https://realm.io/cn/)的一半。

![realm3](http://lvioscode.com/ios_documents/documents/raw/master/知识库/images/realm3.jpg)



#### iOS的四大存储方式：

* NSKeyedArchiver（归档）
* NSUserDefaults（偏好设置）
* Write写入方式:永久保存在磁盘中(plist文件等)
* SQLite3 （CoreData本质上还是SQLite）

#### 有哪些优点？
* 有中文文档（OC支持到V2.0.3 Swift支持到V1.1.0）
* 轻量级,API简化很多，入门门槛低（语法简单）
* 相对于SQLite占用磁盘空间少
* 速度快（原因：不允许对象在不同的线程间共享，保证了数据的一致性，不加线程锁，保证了[Realm](https://realm.io/cn/)的在速度上遥遥领先），其实这也算得上是一个限制
* 跨平台（数据库可以在不同平台上使用）
* 可视化 可通过Realm Browser 查看数据库（在APPStore上可下载）
* 真正的懒加载：只有当你真正访问对象的值时候才真正从磁盘中加载进来


#### 试用范围：
适合一些业务逻辑不怎么复杂的场景应用。

<br>

------

### 一、 [安装]()
环境需要：iOS 7 及其以上版本，此外 [Realm](https://realm.io/cn/) 支持 tvOS 和 watchOS 的所有版本。需要使用 Xcode 7.3以及以上版本。

> 如果是混编项目，就需要安装OC的Realm，然后要把 Swift/RLMSupport.swift 文件一同编译进去。

支持四种安装方式：

### Dynamic Framework

>  1. 下载最新的[Realm](https://realm.io/cn/)发行版本，并解压；
前往Xcode 工程的”General”设置项中，从ios/dynamic/、osx/、tvos/或者watchos/中将’Realm.framework’拖曳到”Embedded Binaries”选项中。确认Copy items if needed被选中（除非您的项目中需要在多个平台中使用 [Realm](https://realm.io/cn/)），点击Finish按钮；
>  2. 在单元测试 Target 的”Build Settings”中，在”Framework Search Paths”中添加Realm.framework的上级目录；
>  3. 如果希望使用 Swift 加载 [Realm](https://realm.io/cn/)，请拖动Swift/RLMSupport.swift文件到 Xcode 工程的文件导航栏中并选中Copy items if needed；
>  4. 如果在 iOS、watchOS 或者 tvOS 项目中使用 [Realm](https://realm.io/cn/)，在应用程序的target的”Build Phases”中，创建一个新的”Run Script Phase”，并将
```bash "${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/Realm.framework/strip-frameworks.sh"```


###  CocoaPods（推荐,pod下来后大概接近63M）
>在`podfile`文件加入：
```pod 'Realm', '~> 2.1.2' ```最新tag是2.1.2 （这个方式可以看到C++部分代码）

###  Carthage
>需要安装 Carthage不推荐

###  Static Framework（不推荐，2.1.2版本大概422M）
>拖到项目中并添加 libc++.tbd库

------



### 二、Relam所有的[API](https://realm.io/docs/objc/latest/api/Classes.html)（都是以RLM开头）


&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMRealm](https://realm.io/docs/objc/latest/api/Classes/RLMRealm.html):Realm是框架的核心所在，是我们构建数据库的访问点，就如同CoreData的管理对象上下文（managed object context）或RAC的RACSignal。
	
&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMObject](https://realm.io/docs/objc/latest/api/Classes/RLMObject.html):[Realm](https://realm.io/cn/)数据库表的数据模型,新建Model必须继承RLMObject,[Realm](https://realm.io/cn/)中的Model就是Table（表）,这个Model中的属性就相当于表中的字段。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMResults](https://realm.io/docs/objc/latest/api/Classes/RLMResults.html):这个类是执行查询请求后所返回的类，包含了一系列的RLMObject对象。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMRealmConfiguration](https://realm.io/docs/objc/latest/api/Classes/RLMRealmConfiguration.html):设置数据库名字、存储位置、数据库加密的key等。默认的存储路径是：应用的documents文件夹下的default.realm。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMArray](https://realm.io/docs/objc/latest/api/Classes/RLMArray.html):RLMArray 的接口和 NSArray 非常类似，在 RLMArray 中的对象能够通过索引下标(indexed subscripting)进行访问，在定义Model时，在申明的末尾加入RLM_ARRAY_TYPE（MyModel）宏才能使用，一般在一对多关系中用到。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMNotificationToken](https://realm.io/docs/objc/latest/api/Classes/RLMNotificationToken.html):通知。集合的数据改变了，可以添加通知方式来更新UI。RLMRealm，RLMCollection，RLMResults ，RLMArray，RLMLinkingObjects可以使用通知，这几个类中提供addNotificationBlock。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMMigration](https://realm.io/docs/objc/latest/api/Classes/RLMMigration.html):数据库的乔迁。主要使用RLMMigrationBlock 这个Block去操作完成。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMLinkingObjects](https://realm.io/docs/objc/latest/api/Classes/RLMLinkingObjects.html):它表示链接到其父对象的对象的集合。用在反向关系等。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMCollectionChange](https://realm.io/docs/objc/latest/api/Classes/RLMCollectionChange.html):一般用在UITableView数据的更改，具体见[官方Demo](https://github.com/realm/realm-cocoa) TableView。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp; [RLMProperty](https://realm.io/docs/objc/latest/api/Classes/RLMProperty.html):[Realm](https://realm.io/cn/)支持的属性。[Realm](https://realm.io/cn/)支持以下的属性类型：BOOL、bool、int、NSInteger、long、long long、float、double、NSString、NSDate、NSData以及 被特殊类型标记的NSNumber。CGFloat属性的支持被取消了，因为它不具备平台独立性,不支持iOS平台上的集合类（NSArrary、NSDictionary、NSSet）。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMSortDescriptor](https://realm.io/docs/objc/latest/api/Classes/RLMSortDescriptor.html)：一般用在多个条件排序。

`+ (instancetype)sortDescriptorWithProperty:(NSString *)propertyName ascending:(BOOL)ascending;`

例:

		RLMResults  *results = [RLMResults sortedResultsUsingDescriptors:@[[RLMSortDescriptor sortDescriptorWithProperty:@"Date" ascending:YES],…]]。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMPropertyDescriptor](https://realm.io/docs/objc/latest/api/Classes/RLMPropertyDescriptor.html)：一般与linkingObjectsProperties配合使用，用在反向关系。

例:

		+ (NSDictionary *)linkingObjectsProperties
		{
		    return @{ @"owners": [RLMPropertyDescriptor descriptorWithClass:Person.class propertyName:@"dogs"] };
		}


&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMObjectSchema](https://realm.io/docs/objc/latest/api/Classes/RLMObjectSchema.html):具体Model（表）的字段说明，一般在数据迁移的时候用到。

		{
			age {
				type = int;
				objectClassName = (null);
				linkOriginPropertyName = (null);
				indexed = NO;
				isPrimary = NO;
				optional = NO;
			}
			sex {
				type = string;
				objectClassName = (null);
				linkOriginPropertyName = (null);
				indexed = NO;
				isPrimary = NO;
				optional = YES;
			}
		}


&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMSchema](https://realm.io/docs/objc/latest/api/Classes/RLMSchema.html):数据库中所有RLMObjectSchema的集合,只要继承RLMObject，都属于。


--

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;*[RLMSyn](https://realm.io/cn/docs/objc/latest/#realm--12)*系列一般联合服务器配合，本地数据库与服务器进行同步。具体见[官方Demo](https://github.com/realm/realm-cocoa)：Draw。 

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMSyncUser](https://realm.io/docs/objc/latest/api/Classes/RLMSyncUser.html):Realm 用户(RLMSyncUser) 是 [Realm](https://realm.io/cn/) 对象服务器 (Object Server) 当中的核心概念。如果要打开在[服务器](https://realm.io/docs/realm-object-server/)上的某个 Realm 数据库，那么首先您所需要做的就是以授权用户的身份进行登录。可以通过用户名/密码模式或者通过其他的[认证方法](https://realm.io/cn/docs/objc/latest/#authentication)来让 RLMSyncUser 能够通过认证，从而访问共享的 [Realm](https://realm.io/cn/) 数据库。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMSyncConfiguration](https://realm.io/docs/objc/latest/api/Classes/RLMSyncConfiguration.html)：设置本地服务器同步为服务器数据库。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;例：

		RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration]; 
		config.syncConfiguration = [[RLMSyncConfiguration alloc] initWithUser:user realmURL:syncServerURL];
		RLMRealm *realm = [RLMRealm realmWithConfiguration:config error:nil]; 

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMSyncCredentials](https://realm.io/docs/objc/latest/api/Classes/RLMSyncCredentials.html)：配置证书 (现支持Google Token 、iClound、FaceBook Token以及注册)。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;例：

	RLMSyncCredentials *usernameCredentials = [RLMSyncCredentials credentialsWithUsername:@"username"
                                                                             password:@"password"
                                                                              register:NO];
	RLMSyncCredentials *googleCredentials = [RLMSyncCredentials credentialsWithGoogleToken:@"Google token"];
	RLMSyncCredentials *facebookCredentials = [RLMSyncCredentials credentialsWithFacebookToken:@"Facebook token"];
	RLMSyncCredentials *cloudKitCredentials = [RLMSyncCredentials credentialsWithCloudKitToken:@"CloudKit token"];

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMSyncManager](https://realm.io/docs/objc/latest/api/Classes/RLMSyncManager.html)：服务同步管理

	// 监听服务器一些错误信息
	[RLMSyncManager sharedManager].errorHandler = ^(NSError *error, RLMSyncSession *session) {
	        NSLog(@"A global error has occurred! %@", error);
	};

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMSyncPermissionChange](https://realm.io/docs/objc/latest/api/Classes/RLMSyncPermissionChange.html)：更改同步服务器数据库的权限。

		RLMSyncPermissionChange *permissionChange = [RLMSyncPermissionChange permissionChangeWithRealmURL:realmURL    // The remote Realm URL on which to apply the changes
		                                                                                             userID:anotherUser // The user ID for which these permission changes should be applied
		                                                                                               read:@YES        // Grant read access
		                                                                                              write:@YES        // Grant write access
		                                                                                             manage:@NO];       // Grant management access
		  [realm addObject:permissionChange];
  
&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;[RLMSyncSession](https://realm.io/docs/objc/latest/api/Classes/RLMSyncSession.html)：封装服务器会话的对象

------
### 三、如何使用

*  创建默认[Realm](https://realm.io/cn/)数据库

Realm:

在程序中，如果调用了[Realm defaultRealm]后，会默认在Documents文件夹下创建2个文件（一个是default.realm数据库，一个default.realm.lock），一个文件夹（default.realm.management）。

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/realmDefault.jpg)


设置自定义的数据库：（要以.realm文件类型才行）

> 例:设置Cache文件夹下名为test.realm

`NSArray *cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);`

`NSString *fileName = [[[cachePath firstObject] stringByAppendingPathComponent:@"test"] stringByAppendingPathExtension:@"realm"];`
 

	- (void)setDefaultRealmWithPath:(NSString *)path {
		RLMRealmConfiguration *config = [RLMRealmConfiguration defaultConfiguration];
	    config.fileURL = [NSURL URLWithString:path];// 将这个配置应用到默认的 Realm 数据库当中
    	[RLMRealmConfiguration setDefaultConfiguration:config];
    }
*配置数据库的存储路劲,原默认的Document文件夹下的default.realm数据库如果是空的数据库会自动删除，否则不会删除。*

SQL:

	// 获得数据库文件的路径
	    NSString *cachePath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
	    NSString *fileName = [cachePath stringByAppendingPathComponent:@"default.sqlite"];
	    // 将OC字符串 转成 C语言字符串
	    const char *cfilename = fileName.UTF8String;
	    // 1.打开数据库（如果数据库文件不存在，sqlite3_open函数会自动创建数据库文件）
	    int result = sqlite3_open(cfilename, &_db);
	    if (result == SQLITE_OK) { // 打开成功
	        // 2.创表
	        const char *sql = "CREATE TABLE IF NOT EXISTS Student (id integer PRIMARY KEY AUTOINCREMENT, name text NOT NULL, age integer NOT NULL);";
	        char *erroMsg = NULL;
	        result = sqlite3_exec(self.db, sql, NULL, NULL, &erroMsg);
	        if (result == SQLITE_OK) {
	            NSLog(@"成功创表");
	        } else {
	            NSLog(@"创表失败->%s",erroMsg);
	        }
	    } else {
	        NSLog(@"打开数据库失败");
	    }

FMDB:

	// 1.获得数据库文件的路径
    NSString *doc = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
    NSString *filename = [doc stringByAppendingPathComponent:@"defalut.sqlite"];
    // 2.得到数据库
    FMDatabase *db = [FMDatabase databaseWithPath:filename];
    // 3.打开数据库
    if ([db open]) {
        // 4.创表
        BOOL result = [db executeUpdate:@"CREATE TABLE IF NOT EXISTS Student (id integer PRIMARY KEY AUTOINCREMENT, name text NOT NULL, age integer NOT NULL);"];
        if (result) {
            NSLog(@"成功创表");
        } else {
            NSLog(@"创表失败");
        }
    } else {
        NSLog(@"打开数据库失败");
    }
<br>

* 获取[Realm](https://realm.io/cn/)数据库

		[RLMRealmConfiguration defaultConfiguration].fileURL 或 [RLMRealm defaultRealm].configuration.fileURL


* 查看数据库

	通过`Realm Broswer`工具可以查看。在AppStore可以[下载](https://itunes.apple.com/cn/app/realm-browser/id1007457278?mt=12)。
	
	![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/realmBrowser.png)

* 构建基于realm的数据model
>Realm Model插件（在[官方Demo](https://github.com/realm/realm-cocoa)中）

	通过这个插件可快速生成RLMModelObject
	
	![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/realmModelObject.png)
	
		@interface Student :RLMObject
		@property  NSString *id; 
		@property  NSString *name;
		@property  NSInteger *age;
		@property  NSString *ignoredPro;
		@property  RLMArray <Day *> *days;
		@end
		
		RLM_ARRAY_TYPE(Student)
		
		@implementation Student
		
		// 设置主键
		+ (NSString *)primaryKey {
		    return @"id";
		}
		
		// 设置属性默认值
		+ (NSDictionary *)defaultPropertyValues{ 
		    return @{@"name":@"周武",@"age":@26};
		}
		
		// 设置忽略属性,即不存到realm数据库中
		+ (NSArray<NSString *> *)ignoredProperties {
		    return @[@"ignoredPro"];
		}
		
		// 如果设置了必须要的属性、这个属性的值不能传nil 
		+ (NSArray *)requiredProperties { 
		    return @[@"name"];
		}
		
		//设置索引,可以加快检索的速度
		+ (NSArray *)indexedProperties {
		    return @[@"id"];
		}
		@end
		
> 不用加对属性的声明，如strong,copy,assin等。`@property  NSString *id; `


*  存储数据（插入数据）

Realm:


##### 存储创建对象有4中方式能存储：

>创建Dog对象，依次设置各个属性

	Dog *myDog = [[Dog alloc] init];
	
	myDog.name  = @"haha";
	
	myDog.age   = 10;


> 使用字典创建Dog对象

	Dog *myDog = [[Dog alloc]initWithValue:@{@"name" :@"xiaoxiao", @"age"  :@11 }];

> 用数组的方式

	Dog *myDog = [[Dog alloc]initWithValue:@[@"小小",@12]];

> 支持嵌套的对象（在一对多关系中用到）

	Person *person = [[Person alloc] initWithValue:@[@"Thomas", @[aDog, anotherDog]]];
	
	RLMRealm *realm = [RLMRealm defaultRealm];
	
	[realm beginWriteTransaction];
	
	[realm addObject:myDog];// 把对象存到realm数据库
	
	[realm commitWriteTransaction];

注意点：initWithValue包数据时，每个属性依次对象包数据，不能为nil,如果是@{}方式，无需按数据给每个属性赋值，可以随意，但必须为每个属性都要赋值。

SQL：

	NSString *name = @"周武";
    int age = 26;
    NSString *sql = [NSString stringWithFormat:@"INSERT INTO Student (name, age) VALUES ('%@', %d);", name, age];
    // 2.执行SQL语句
    char *erroMsg = NULL;
    sqlite3_exec(self.db, sql.UTF8String, NULL, NULL, &erroMsg);
    if (erroMsg) {
        NSLog(@"插入数据失败->%s", erroMsg);
    } else {
        NSLog(@"成功插入数据");
    }
    
FMDB:

	NSString *name = @"周武";
	int age = 26;
	[self.db executeUpdate:@"INSERT INTO Student (name, age) VALUES (?, ?);", name, age];
	//或
	[self.db executeUpdate:@"INSERT INTO Student (name, age) VALUES (?, ?);" withArgumentsInArray:@[name, @(age)]];
	//或
	[self.db executeUpdateWithFormat:@"INSERT INTO Student (name, age) VALUES (%@, %d);", name, age];    
	
<br>

* 修改数据

Realm:

> 内容直接更新
	
	Person *p =[ [Person allObjects] firstObject];
	RLMRealm *realm = [RLMRealm defaultRealm];
	[realm beginWriteTransaction];
	p.name = @“周武”; // 更新Person表第一个对象的名称为’周武’
	[realm commitWriteTransaction];

> 通过主键更新(必须在表中设置了主键，否则不能用OrUpdate相关方法)

	[realm beginWriteTransaction];
	
	[Person createOrUpdateInRealm:realm withValue:@{@"name": @“周武”}]; 
	
	或[Person createOrUpdateInRealm:realm withValue:p];
	
	[realm commitWriteTransaction];	

> 键值编码（KVC）更新

	[p setValue:@“周武” forKeyPath:@"name"];

	
FMDB:

	[self.db executeUpdate:@"UPDATE Student SET id = ? WHERE name = ?;", @0, @"Thomas"];
	
		
<br>
		
* 删除数据

Realm:

> 删除指定的数据:

	[realm deleteObject:(RLMObject *)object];
	
例：

	RLMResults *results = [Student allObjects];
	[realm deleteObject:[results objectAtIndex:1]];

> 删除一组数据:
	
	[realm deleteObjects:(id)array];
	
例：
	
	[realm deleteObjects:[Student objectsWhere:@"age > 5"]];

> 删除全部数据:
	
	[realm deleteAllObjects()];
	
FMDB:

	[self.db executeUpdate:@"DELETE FROM Student;"];
	// 或
    [self.db executeUpdate:@"DROP TABLE IF EXISTS Student;"];
    
    [self.db executeUpdate:@"CREATE TABLE IF NOT EXISTS Student (id integer PRIMARY KEY AUTOINCREMENT, name text NOT NULL, age integer NOT NULL);"];

<br>

* 查询数据（一般使用objectsWhere 或者objectsWithPredicate方法查询）
 
Realm:
 
> 查询PersonModel表中所有的数据

			RLMResults *results = [PersonModel allObjects]；// 在默认的数据库中查询 如果config修改fileUrl路径后 默认查询的是修改后路径的数据库。或者指定Realm数据库：
			RLMRealm *realm = [RLMRealm realmWithPath:realmPath];
			RLMResults *results = [PersonModel allObjectsInRealm:realm];

> 条件查询 (Realm支持[NSPredicate](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Predicates/AdditionalChapters/Introduction.html)查询。)
    
	    1、 比较操作数(comparison operand)可以是属性名称或者某个常量，但至少有一个操作数必须是属性名称；
	    
	    2、比较操作符 ==、<=、<、>=、>、!=, 以及 BETWEEN 支持 int, long, long long, float, double, 以及 NSDate 属性类型的比较，比如说 age == 45；
	    
	    3、相等比较 ==以及!=，比如说[Employee objectsWhere:@"company == %@", company]
	    
	    4、 比较操作符 == and != 支持布尔属性；
	    
	    5、 对于 NSString 和 NSData 属性来说，我们支持 ==、!=、BEGINSWITH、CONTAINS 以及 ENDSWITH 操作符，比如说
	        name CONTAINS ‘Ja’；
	    
	    6、 字符串支持忽略大小写的比较方式，比如说 name CONTAINS[c] ‘Ja’ ，注意到其中字符的大小写将被忽略；
	
	    7、 Realm 支持以下复合操作符：“AND”、“OR” 以及 “NOT”。比如说 name BEGINSWITH ‘J’ AND age >= 32；
	
	    8、 包含操作符 IN，比如说 name IN {‘Lisa’, ‘Spike’, ‘Hachi’}；
	
	    9、 ==、!=支持与 nil 比较，比如说 [Company objectsWhere:@"ceo == nil"]。注意到这只适用于有关系的对象，这里 ceo 是 Company 模型的一个属性。
	
	    10、 通过 ==, != 进行空值比较，比如说 [Company objectsWhere:@"ceo == nil"]； 注意，Realm 将 nil 视为一个特殊的值而不是“缺失值”，不像 SQL 那样 nil 等于自身。
	
	    11、 ANY 比较，比如说 ANY student.age < 21
	
	    12、 RLMArray 以及 RLMResults 属性支持集合表达式：@count、@min、@max、@sum 以及 @avg，例如 [Company objectsWhere:@"employees.@count > 5"] 用以寻找所有超过 5 名雇员的公司。
	
	    13、 支持子查询，不过有限制：
	
	    14、 @count 是唯一能应用在 SUBQUERY 表达式中的操作符
	
	    15、 SUBQUERY(…).@count 表达式必须与常量进行比较
		
	    16、 相关子查询目前还不支持


	```
		// 使用NSPredicate字符串查询
		RLMResults<Dog *> *tanDogs = [Dog objectsWhere:@"color = '棕黄色' AND name BEGINSWITH '大'"];
		// 使用 NSPredicate 查询
		NSPredicate *pred = [NSPredicate predicateWithFormat:@"color = %@ AND name BEGINSWITH %@",
		                                                     @"棕黄色", @"大"];
		tanDogs = [Dog objectsWithPredicate:pred];
		
	```


> 条件排序（一般使用`[RLMResults sortedResultsUsingProperty:ascending:]` 或 `[RLMResults sortedResultsUsingDescriptors:@[RLMSortDescriptor,…..]]` ) 

```
RLMResults *results = [[PersonModel allObjects] sortedResultsUsingProperty:@"age" ascending:YES]// 按照age升序
RLMResults *results = [[PersonModel allObjects] sortedResultsUsingDescriptors:@[[RLMSortDescriptor sortDescriptorWithProperty:@"sex" ascending:YES],[RLMSortDescriptor sortDescriptorWithProperty:@"age" ascending:NO]]]; // 按照sex字段升序 age字段降序排序
```

FMDB:

	// 1.执行查询语句
    FMResultSet *resultSet = [self.db executeQuery:@"SELECT * FROM Student"];
    // 2.遍历结果
    while ([resultSet next]) {
        int ID = [resultSet intForColumn:@"id"];
        NSString *name = [resultSet stringForColumn:@"name"];
        int age = [resultSet intForColumn:@"age"];
        NSLog(@"%d %@ %d", ID, name, age);
    }


* 链式查询(RLMResults一层一层的过滤)

	```
	RLMResults<Dog *> *tanDogs = [Dog objectsWhere:@"color = '棕黄色'"];
	RLMResults<Dog *> *tanDogsWithBNames = [tanDogs objectsWhere:@"name BEGINSWITH '大'"];
	跟
	// 使用 NSPredicate 查询
	NSPredicate *pred = [NSPredicate predicateWithFormat:@"color = %@ AND name BEGINSWITH %@",
	                                                     @"棕黄色", @"大"];
	tanDogs = [Dog objectsWithPredicate:pred]; 结果是一样的。
	```
	
FMDB:
	
	[self.db executeUpdate:@"SELECT * FROM Student WHERE color = ? AND name LIKE '%@%'",@"棕黄色",@"大"];
	
<br>

*  移动端平台[Realm对象服务](https://realm.io/cn/docs/realm-object-server/) (专业版)

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/realmObectServer.png)

1、先去Realm[下载](https://static.realm.io/downloads/mobile-platform/latest/realm-mobile-platform.zip) 服务器工具。

2、下载解压后打开双击start-object-server.command这个文件。(请勿关闭打开的终端，否则无法访问[http://localhost:9080/#/](http://localhost:9080/#/)

3、提示注册。注册成功后会看到这个 上面截图 Realm Dashboard

4、运行[官方Demo](https://github.com/realm/realm-cocoa) Draw

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/realmDraw.gif)

> *注意点：*

* 修改kIPAddress常量 改为本机的IP 在终端运行：ifconfig | grep "inet " | grep -v 127.0.0.1 可获取。
* 修改用户名和密码。
* 连接成功后 会在应用的Document文件夹下生成realm-object-server文件夹

![](http://lvioscode.com/ios_documents/documents/raw/master/%E7%9F%A5%E8%AF%86%E5%BA%93/images/realmSynCode.png)

------

### 四、特性：

* 支持KVC和KVO

  RLMObject、RLMResult以及 RLMArray都遵守键值编码(Key-Value Coding)（KVC）机制。
  
 `[p setValue:@“周武” forKeyPath:@"name"];`
 
 [Realm](https://realm.io/cn/)对象的大多数属性都遵从 KVO 机制。所有 RLMObject子类的持久化(persisted)存储（未被忽略）的属性都是遵循 KVO 机制的，并且 RLMObject以及 RLMArray中 无效的(invalidated)属性也同样遵循（然而 RLMLinkingObjects属性并不能使用 KVO 进行观察）。
 
* 支持数据库加密

[Realm](https://realm.io/cn/)支持在创建[Realm](https://realm.io/cn/)数据库时采用64位的密钥对数据库文件进行 AES-256+SHA2 加密。这样硬盘上的数据都能都采用AES-256来进行加密和解密，并用 SHA-2 HMAC 来进行验证。每次要获取一个 Realm 实例时，需要提供一次相同的密钥。

	RLMRealmConfiguration *configuration = [RLMRealmConfiguration defaultConfiguration];
    configuration.encryptionKey = [self getKey];
    RLMRealm *realm = [RLMRealm realmWithConfiguration:configuration
                                                     error:nil];
                                                     
>不过，加密过的 Realm 只会带来很少的额外资源占用（通常最多只会比平常慢10%）。


* 通知

>申请一个strong属性持有这个通知。

`@property (nonatomic, strong) RLMNotificationToken *notification;`

	self.notification = [realm addNotificationBlock:^(NSString *notification, RLMRealm * realm) {
	     // do something
	}];

	- (void)dealloc {
		[_notification stop];// 移除通知
	}

RLMRealm，RLMCollection，RLMResults ，RLMArray，RLMLinkingObjects 这些类支持添加通知。

* 数据库迁移

见[官方Demo](https://github.com/realm/realm-cocoa) `Migration` 例子。

第一个是合并字段的例子，第二个是增加新字段的例子，第三个是原字段重命名的例子。

		RLMMigrationBlock migrationBlock = ^(RLMMigration *migration, uint64_t oldSchemaVersion) {
		        if (oldSchemaVersion < 1) {
		            [migration enumerateObjects:Person.className block:^(RLMObject *oldObject, RLMObject *newObject) {
		                if (oldSchemaVersion < 1) {
		                    // combine name fields into a single field
		                    newObject[@"fullName"] = [NSString stringWithFormat:@"%@ %@", oldObject[@"firstName"], oldObject[@"lastName"]];
		                }
		            }];
		        }
		        if (oldSchemaVersion < 2) {
		            [migration enumerateObjects:Person.className block:^(RLMObject *oldObject, RLMObject *newObject) {
		                // give JP a dog
		                if ([newObject[@"fullName"] isEqualToString:@"JP McDonald"]) {
		                    Pet *jpsDog = [[Pet alloc] initWithValue:@[@"Jimbo", @(AnimalTypeDog)]];
		                    [newObject[@"pets"] addObject:jpsDog];
		                }
		            }];
		        }
		        if (oldSchemaVersion < 3) {
		            [migration enumerateObjects:Pet.className block:^(RLMObject *oldObject, RLMObject *newObject) {
		                // convert type string to type enum if we have outdated Pet object
		                if (oldObject && oldObject.objectSchema[@"type"].type == RLMPropertyTypeString) {
		                    newObject[@"type"] = @([Pet animalTypeForString:oldObject[@"type"]]);
		                }
		            }];
		        }
		        NSLog(@"Migration complete.");
		    };

	    RLMRealmConfiguration *configuration = [RLMRealmConfiguration defaultConfiguration];
	
	    // set the schema verion and migration block for the defualt realm
	    configuration.schemaVersion = 3;
	    configuration.migrationBlock = migrationBlock;
	
	    [RLMRealmConfiguration setDefaultConfiguration:configuration];



------

### 五、缺点/限制

* 类名称的长度最大只能存储 57 个 UTF8 字符。

* 属性名称的长度最大只能支持 63 个 UTF8 字符。

* NSData以及 NSString属性不能保存超过 16 MB 大小的数据

* 字符集的限制（对字符串进行排序以及不区分大小写查询只支持“基础拉丁字符集”、“拉丁字符补充集”、“拉丁文扩展字符集 A” 以及”拉丁文扩展字符集 B“（UTF-8 的范围在 0~591 之间））

* [Realm](https://realm.io/cn/) 对象不支持跨线程

* [Realm](https://realm.io/cn/)对象的 Setters & Getters 不能被重载

* [Realm](https://realm.io/cn/) 没有自动增长属性

* 所有的数据模型必须直接继承自RealmObject

* [Realm](https://realm.io/cn/)不支持集合类型（NSArray，NSMutableArray，NSDictionary，NSMutableDictionary，NSSet，NSMutableSet等）

* 文件大小 & 版本跟踪
<br>

------

### 六、相关文档

Realm Demo：[https://github.com/realm/realm-cocoa](https://github.com/realm/realm-cocoa)

Realm官网：[https://realm.io/](https://realm.io/)   中文官网 [https://realm.io/cn/](https://realm.io/cn/)

Objective-C文档：[https://realm.io/docs/objc/latest/](https://realm.io/docs/objc/latest/)   2.0.3版本支持中文文档。

Realm新闻：[https://realm.io/news/](https://realm.io/news/)  这里面都是技术博文，不只是讲Realm的，里面还包含其他的技术知识。

Realm 核心数据库引擎探秘:[https://realm.io/cn/news/jp-simard-realm-core-database-engine/](https://realm.io/cn/news/jp-simard-realm-core-database-engine/) （在Realm新闻里面可找到）

深入理解Realm的多线程机智：[https://realm.io/cn/news/threading-deep-dive/](https://realm.io/cn/news/threading-deep-dive/)  建议看看（在Realm新闻里面可找到）
<br>

------

### 七、常用疑问

>1、*为什么 Realm 对象不能在线程间传递？*

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;&nbsp; &nbsp; &nbsp;[Realm](https://realm.io/cn/)官方解释：如果 [Realm](https://realm.io/cn/) 允许对象可在线程间共享，Realm 会无法确保数据的一致性，因为不同的线程会在不确定的什么时间点同时改变对象的数据。

>2、*是否开源？*

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;&nbsp; &nbsp; &nbsp;已经开源，只是大部分已经开源，c++核心代码没有开源。使用cocoapods方式安装可以查看c++部分代码。

>3、*realm的支持库有多大？*<br />

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;&nbsp; &nbsp; &nbsp;一旦你的app编译完成， realm的支持库应该只有1 MB左右。 发布的时候各个版本不一，最新的2.1.2版本framework 62M多， 那是因为它们还包含了对其他构架的支持（ARM， ARM64，模拟器的是X86）和一些编译符号。 这些都会在你编译app的时候被Xcode自动清理掉。

>4、*是否安全？*

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;&nbsp; &nbsp; &nbsp;支持加密  在RLMRealmConfiguration 属性encryptionKey 设置加密的密钥。

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;&nbsp; &nbsp; &nbsp;加密/解密：

			RLMRealmConfiguration *configuration = [RLMRealmConfiguration defaultConfiguration];
	        configuration.encryptionKey = myKey;
	        RLMRealm *realm = [RLMRealm realmWithConfiguration:configuration
	                                                     error:nil];

>5、*需要付费吗？*

&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;&nbsp; &nbsp; &nbsp;个人以及商业使用是免费的。专业版前2个月免费，后每月$1500，具体看Realm[价格](https://realm.io/cn/pricing/)。 

&nbsp;&nbsp; &nbsp;专业版多出的功能：

* 响应活动对象中更改的触发事件（Trigger events in response to change in your live objects）

* 与现有基础架构和第三方API集成 （Integrate with existing infrastructure and 3rd-party APIs）

* 对同步的数据启用业务逻辑（Enable business logic on synced data）

* 获得对服务器端的Realm对象的访问（Gain access to Realm objects server-side）

* 备份数据（Backup your Realm data for disaster recovery）

<br>

------


### 八、总结

- 所有的操作（添加、删除、更新、插入等）必须在提交事物(`transaction`)中完成。

- 所有的`Model`必须继承`RLMObject`。

- 跨线程，不能使用同一个`RLMRealm` 对象，单独在线程中创建一个`[RLMRealm defaultRealm]`对象;

- 创建的Model最好要有主键方便更新数据。

- 创建的Model最好在申明末尾加入这个宏`RLM_ARRAY_TYPE(Class)`，方便以后的扩展使其实现多对关系。







