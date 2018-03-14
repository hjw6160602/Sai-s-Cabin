---
title: OC进化简述
date: 2016-09-28 16:37:46
---

`Objective-C` 最初起源于 `NeXTSTEP` 操作系统，之后乔布斯回到苹果，便将它在OS X和iOS中继承了下来。
20世纪80年代初，`Brad Box`和`Tom Love`以`SmallTalk-80`语言为基础发明了`Objective-C`，`Smalltalk`是历史上第二个面向对象的程序设计语言，也就是说：Objective-C就是C语言面向对象SmallTalk语法话的结果。
让我们来看看早起SmallTalk的语法特点：
> 在 Smalltalk 中一切皆对象，一切调用都是发消息

比如下面的表达式：
```smalltalk
2 + 3
```
它的意义是：向对象`2`发送消息`+`，参数为对象`3`。
再比如用一个工厂方法来实例化一个对象：
```smalltalk
p := Person name: 'Sai DiCaprio' age: 12
```
为Person类添加一个方法
```smalltalk
greet: name  
  | message | 
  message := 'Hello ', name. 
  Transcript show: message.
```
在方法定义里，管道` | `包着的本地变量，然后是方法的实现，把`‘Hello’`放到了变量`message`里，然后用逗号符把它和变量`name`连接起来。
```smalltalk
p := Person new.
p greet: 'Jack'.
```
这时Transcript会输出 `Hello Jack`。
从上可以看出，消息传递机制成了C语言进化为OC的最大障碍，要实现向一个` target ( class / instance ) `发送消息名` selector` 动态寻找到函数实现地址`IMP`并调用，为了解决这一难题，需要提供一系列在`Build Time`无法实现的运行时函数支持，这些函数慢慢封装完整，便出现了一套API：`Runtime`。
> 早期的Objective-C = C + Preprocessor + Runtime

既然已经形成了一套健全的面向对象的C语言体系，那么我们来看看早期的`Objective-C` 代码是如何书写的
```objc
@interface Person{
  NSString *_name;
  int _age;
}
- (NSString *)name;
- (void)setName:(NSString *)name;

- (int)age;
- (void)setAge:(int)age;
@end
```
上面的代码声明了一个类`Person`，他有2个成员变量，并分别为其提供了`set`和`get`方法。

秉着`谁创建，谁释放；谁引用，谁管理`的MRC原则，在实现文件中便有了下面的代码
```objc
@implementation Person
- (NSString *)name{
  return _name;
}
- (void)setName:(NSString *)name{
  if (_name != name){
    [_name release];
    _name = [name copy];
  }
  return _name;
}

- (int)age{
  return _age;
}
- (void)setAge:(int)age{
  _age = age;
}

@end
```
可以看到，仅仅创建了2个变量就为类带来了这么大的代码量，在成员变量多的情况下，通篇垃圾代码，于是`Objective-C 2.0`马上就出现了新的语法`@property`关键字
它用来让编译器自动帮我们生成成员变量和其对应的get和set方法的声明。
比如Person类的声明中便化为这样：
```objc
@interface Person
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) int age;
@end
```
> 注意：这时的@property并不像现在一样会帮我们自动生成成员变量的setter和getter方法的实现

在实现方面有另外一个关键字：`@synthesize`帮我们根据属性修饰符自动合成`setter`和`getter`的实现。
```objc
@implementation Person
@synthesize name = _name;
@synthesize age = _age;
@end
```
后来自动合成的`@synthesize`可以不用写了，默认就是上面的代码。这样一来，整个类变得轻快了很多。
但是也会有这样的需求：比如不希望系统帮我们自动实现`setter`和`getter`，而由我们自己来实现，便出现了`@dynamic`关键字，虽然编译器会通过，但是如果在运行过程中方法调用了对应的`setter`和`getter`方法，但是发现没有手动实现那么就会崩溃报`unrecognized selector sent to instance 0x.......`的错误。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。

##总结：
> * @synthesize 告诉编译器帮我们合成属性的getter和setter的实现 
* @dynamic 告诉编译器不要帮我自动合成属性，而由我们自己实现getter和setter的实现

##既然现在的@property会帮我们自动合成autosynthesize那么@dynamic和synthesize如今还有什么意义呢？

回答这个问题前，我们要搞清楚一个问题，什么情况下不会autosynthesis（自动合成）？
> * 同时重写了 setter 和 getter 时
* 重写了只读属性的 getter 时
* 使用了 @dynamic 时
* 在 @protocol 中定义的所有属性
* 在 category 中定义的所有属性
* 重载的属性

当同时重写了setter和getter或者重写了只读属性的getter时，系统就不会帮我们自动合成，就意味着ivar也不会被生成，所以，这种情况下有两种方案：
* 自己添加ivar
* 使用@synthesize

也就是说，当你想手动管理 `@property` 的所有内容时，你就会尝试通过实`@property` 的所有`存取方法`或者使用`@dynamic`来达到这个目的，这时编译器就会认为你打算手动管理 `@property`，于是编译器就禁用了自动合成。

#### 另外，当你在子类中重载了父类中的属性，你必须 使用 @synthesize 来手动合成ivar。