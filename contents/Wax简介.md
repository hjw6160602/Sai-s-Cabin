# `Wax`简介

## `Wax`是什么？
`Wax`是一个用[Lua](http://www.lua.org/)语言编写本地`iPhone` `app`的框架。它使用`OC`运行时将`OC`和`Lua`进行绑定。通过`Wax`，你可以用`Lua`做任何`OC`可以做到的事情。

# Wax-lua的语言优势：
1. 自动垃圾回收：再也不用使用`alloc`、`retain`和`release`；
2. 代码少：没有头文件、没有`static`类型、`array`常量、`dictionary`常量
3. 能够使用任何一个`framework`，例如`cocoa`、`UITouch`、`Fondation`等，任何用`oc`写的`framework`，`wax`自动将其暴露给`lua`使用
4. 超级简单的`http`请求： 和`REST`的`web` `service`一起交互使用
5. `lua`支持函数闭包
6. `lua`有内置的`Regex-like` 模式匹配`library`
7. `64位`支持
8. 线程安全
9. 其他一些特性：
  *  `Lua function` 转化成 `oc block`，
  *  在`Lua`中调用`oc block`
  *  `getting/setting` 私有成员变量
  *  内置通用的C函数
  *  支持`Lua`代码`debug`

    
## `Lua` 在`Mac OS` 系统上安装

在终端执行如下命令：

```shell
curl -R -O http://www.lua.org/ftp/lua-5.3.0.tar.gz tar zxf lua-5.3.0.tar.gz
cd lua-5.3.0
make macosx test
make install
```

## Wax安装

`GitHub`下载：[https://github.com/alibaba/wax](https://github.com/alibaba/wax)

### 使用`Cocoapods`

`pod 'wax', :git=>'https://github.com/alibaba/wax.git', :tag=>'1.2.1'`
	
### 使用`Static framework`
	
 * [下载Wax](https://github.com/alibaba/wax), `cd`到项目的`tools/Framework`
 * 执行`rake package` 命令
 *  到项目的`framework`文件夹下找到`wax.framework` 拖到自己的项目中即可。

### 手动加入`Lib`

 * [下载Wax](https://github.com/alibaba/wax)
 * 在`Wax`根目录找到`lib`文件夹并拖拽到自己的项目中。
 * 在`TARGETS`->`Build Phases`->`Compile Sources`加入`-fno-objc-arc`标志
 * 如想支持`SQLite`需添加`libsqlite3`，在`Build Phases`->`Link Binary With Libraries`.
 * 如需支持`xml`,需要添加`libxml2` 在` Build Phases`->`Link Binary With Libraries` 和添加` ${SDKROOT}/usr/include/libxml2` 在 `Build settings->Head Search Path`中。

## 语法

`Wax`语法介绍：[https://github.com/alibaba/wax/wiki/Overview](https://github.com/alibaba/wax/wiki/Overview)

### 1、注释：

通过`--`来注释
`--`注释的内容

### `wax` 中新建对象：

```lua
waxClass{"MyClass", NSObject}

添加协议 
waxClass{"MyClass", NSObject, protocols = {"UITableViewDelegate", "UITableViewDataSource"}}
```
	 
### `wax` 中方法的参数，第一个默认必须是 `self`

这样，在该方法中，就可以通过 `self` 来调用一些东西。

```lua
waxClass{"MyClass", NSObject}
		 
function storeWords(self, words)
  self.words = words
  self.super:words(self)
end
```
	  
### 使用 冒号`：`代替点.

OC

```lua
UIApplication:sharedApplication() 
```

```lua
UIApplication.sharedApplication(UIApplication)
``` 
	  
### `oc` 有多参数的方法中，通过使用 `_` 下划线来代替

OC

```objc
[UIAlertView initWithTitle:@"title" message:@"message" delegate:nil];
```
LUA
```lua
UIAlertView:initWithTitle_message_delegate("title", "message", nil) 
```  
### `wax` 中不使用 `oc` 中的属性，`wax`强制要求使用方法来获取或者赋值  
	
```lua
someView.frame -- 不会工作
	
//使用 setter/getter 来代替
someView:frame()
someView:setFrame(someFrame)
```  

### 可以通过 `.` 的操作，来给对象赋值任何的值。

```lua
//方法通过 ： 来获取
local view = UIView:init()

view.someRandomVariable = "YEAH!"
//可以赋任何值 并且该值持久存在
```

### `wax` 会强制将 `oc` 的语法写成 `lua` 的语法。 
即：一个方法的参数是 `NSString` 类型，那么，在 `wax` 中，应该传入的参数为 `lua` 的 `string`

```lua
local fileManager = NSFileManager:defaultManager()

local contents = fileManager:directoryContentsAtPath(".")
//directoryContentsAtPath 返回一个数组，wax 中编程 table 类型
	
print(contents[1]) --> "info.plist"
```

 * NSDictionaries 成为 Lua 中的 tables
 * NSArrays 成为 Lua 中的 tables
 * NSStrings 成为 Lua 中的 strings
 * NSNumbers 成为 Lua 中的 numbers
    
### 如果需要将 `wax` 类型强制转换成为 `oc` 类型，可以使用 `toobjc` 这个方法

```lua
//如果尝试去调用 oc 的方法，这将会失败

local string = "a string"
string:capitalizedString()

//... 应该这个 string 已经被强制转为 lua 的 string

//使用 toobjc  ，string 会被转换为 NSString
toobjc(string):capitalizedString()

local length = toobjc(str):length();
if not toobjc(tempDeviceID):isEqualToString(self:deviceID()) then 
    --xx
end    
```
		
### 枚举：

系统定义的枚举都在 `wax/stdlib/enums.lua` 这个文件中    
    
### `selector` 使用 `string` 进行传递。

OC 中的写法
```objc
@selector(this:is:a:method)
```

在 `lua` 中写成

```lua
this:is:a:method   
```
如：
```lua
local backItem = UIBarButtonItem:alloc():initWithTitle_style_target_action("返回",UIBarButtonItemStylePlain,self,"backAction");
self:navigationItem():setLeftBarButtonItem(backItem);
```
			
`32/64` 可以使用 `wax.isArm64` 来判断当前 `APP` 运行的设备是否是`64位`的 `cpu`

`Structs` 大部分都定义好了，在 `APP_ROOT/wax/wax-scripts/structs.lua` 路径下面，不需要重新创建。特别注意 `NSInteger` 、`CGFloat` ，它们在`32/64`位中的设备上，表现不一样。

在存在的对象中，`hook` 其中的一个存在方法，直接写该方法，就会覆盖掉 `oc` 中已经写好的方法 。 如果你想调用原来的方法 ，你可以在新的方法中，增加 `ORIG` 的前缀

```lua
waxClass{“MyController"}
function viewDidLoad(self)
//—做一些事情在原方法之前
self:ORIGviewDidLoad()--self is no need
//做一些事情在原方法之后
end
```

### `hook` 一个扩展出来的方法
和 `hook` 一个对象中的方法类似

OC `extension`

```objc
@interface MTPServer (Async4j)
- (void)startAsync4jRequest;
@end	
```

Lua

```lua
waxClass{"MTPServer"}
function startAsync4jRequest(self)
--做一些事情在原方法之前
 self:ORIGstartAsync4jRequest()
--做一些事情在原方法之后
end
```

### 如果 `oc` 的方法中有 `_`，那么，在 `lua` 中，使用 `UNDERxLINE` 来代替

OC

```objc
view.backgroundColor = [UIColor yellowColor];
[view mas_makeConstraints:^(MASConstraintMaker *make) {
    make.center.equalTo(self.view);
    make.width.offset(50);
    make.height.offset(50);
}];
```

Lua

```lua
view:masUNDERxLINEmakeConstraints(toblock(
function ( make )
  make:center():equalTo()(self:view())
  make:width():offset()(50)
  make:height():offset()(50)
  -- make:centerOffset()(CGPoint(10, 20))
end
,{"void", "MASConstraintMaker *"}))
```
### 如果有使用 `$` 符号，使用`DOLLARxMARK`来替代
OC
```objc
(NSString *)$testDolorMethod;
```
Lua
```lua
function DOLLARxMARKtestDolorMethod( self )
	print("lua $testDolorMethod")
	self:ORIGDOLLARxMARKtestDolorMethod()
	return "abc"
end
```

### 为对象添加属性

		function myProperity(self)
	    return self._myProperity
		end
		function setMyProperity(self, x)
		    self._myProperity = x
	end
	
### 执行热修复代码

	wax_start("fileName.lua", 需加载的库......);
	
	如：
	wax_start("init.lua", luaopen_wax_http,luaopen_wax_json,luaopen_wax_filesystem);
	或：
	-- 执行字符串
	wax_startWithNil();
	int i = wax_runLuaString(contentStr.UTF8String);
    if(i){
        NSLog(@"error=%s", lua_tostring(wax_currentLuaState(), -1));
    }
    -- 执行某个文件
        int i = wax_runLuaFile(path.UTF8String);
    if(i){
        NSLog(@"error=%s", lua_tostring(wax_currentLuaState(), -1));
    }

	

## 注意事项
### 1. `waxClass` 函数第一个参数必须传`self`
###  2. `wax` 数组索引值从`1`开始而不是`0`
###  3. `wax block` 如果返回值和参数都是`void` 可不写。

```objc
UIView:animateWithDuration_animations_completion(1, 
	toblock(
		function()
			
		end
	),
	toblock(
		 	function(finished)
				
			end
	,{"void", "BOOL"})
)
```
### 4. 当实例化一个对象时，`alloc`方法可以不需要。
	
#### 例子

创建一个红色的View

```lua
local view = UIView:initWithFrame(CGRect(0, 0, 320, 100))

view:setBackgroundColor(UIColor:redColor())
```
	
### 5. 有多个参数的方法

```lua
UIApplication:sharedApplication():setStatusBarHidden_animated(true, false)
```
		
### 6. 数组/字符串/字典赋值

```lua
images = {"myFace.png", "yourFace.png", "theirFace.png"}
dic = {key="value",key2="value2"} // key的名称不能是字符串 dic = {"key"="value","key2"="value2"} // 错误
str = "hello"
imageView = UIImageView:initWithFrame(CGRect(0, 0, 320, 460))
imageView:setAnimationImages(images)
```
		
		
### 7. 创建一个自定义`UIViewController`

```lua
waxClass{"MyController", UIViewController}

function init()
  -- to call a method on super, simply use self.super
  self.super:initWithNibName_bundle("MyControllerView.xib", nil)
  return self
end

function viewDidLoad()
  self.super:viewDidLoad()
  -- xxxxx
end
```
	
### 8. 发送一个`HTTP`请求

```lua
url = "http://search.twitter.com/trends/current.json"

wax.http.request{url, callback = function(json, response)
if response and response:statusCode() == 200 then
  self.trends = json["trends"]
end

--do something
end}
```
  	
### 9. 如果你不想`OC`对象被自动转换，则可以使用`toobjc`方法。

```lua
local testString = "Hello lua!"
local bigFont = UIFont:boldSystemFontOfSize(30)
local size = toobjc(testString):sizeWithFont(bigFont)
```
	
### 10. 获取/设置成员变量

```lua
self:getIvarInteger("_aInteger")
self:setIvar_withObject("_title", "abcdefg")
self:setIvar_withObject("_infoDict", {k11="v11", k22="v22"})
```
  
### 11. 调用`C`函数

```objc
luaSetWaxConfig({openBindOCFunction=true})--bind built-in C function

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), 
        toblock(
            function( )
                print(string.format("dispatch_async"));
            end)
    );

UIGraphicsBeginImageContext(CGSize(200,400));
UIApplication:sharedApplication():keyWindow():layer():renderInContext(UIGraphicsGetCurrentContext());
local aImage =UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();

local imageData =UIImagePNGRepresentation(aImage);
local image = aImage;
local data = UIImageJPEGRepresentation(image, 0.8);
print("data.length=", data:length());
```

### 12. `Lua`语言在`Xcode`显示高亮和提示
  * [下载Wax-In-Xcode](https://github.com/intheway/Wax-In-Xcode)，找到Add-Lua.sh, Lua.xclangspec文件。
  * 打开 Add-Lua.sh 并更改 DVTFOUNDATION_PATH的路径（一般不需要修改）
  * 终端执行Add-Lua.sh 例如：sudo ./Add-Lua.sh 注意：Xcode需关闭
  * 打开 Xcode 然后打开一个Lua文件, 在"Editor->Syntax Coloring" 点击'Lua。
	
	
