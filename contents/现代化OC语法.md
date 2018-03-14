---
title: 现代化OC语法
date: 2017-04-11 09:31:10
---

随着Swift的加入，OC在不也在不断向着新的编程方式靠拢，虽然OC是我很喜欢的一门很优雅的语言，但是毕竟编程讲究的是效率，时代的大浪下，OC的很多写法也顺应大潮做出了一些相应的改变。
下面我们来看一下都有哪些地方现代化OC语法做出的改变。

##1. 将一些单纯的方法转换为属性
```
- (BOOL)isGrade;
@property (NS_NONATOMIC_IOSONLY, getter=isGrade, readonly) BOOL grade;
```

## 2. 将一些中括号调用变为属性调用

#### 2.1. get方法调用的修改
```objc
[[NSBundle mainBundle] infoDictionary]
[NSBundle mainBundle].infoDictionary
```
```objc
[[UIScreen mainScreen] bounds]
[UIScreen mainScreen].bounds
```
```objc
[[[UIApplication sharedApplication] delegate] window]
[UIApplication sharedApplication].delegate.window
```
类似的还有`length`、`count`等等

#### 2.2. set方法调用的修改
将`setImage`这种类似的`setter`方法转换为.语法的`setter`赋值
```objc
image = [[UIImageView alloc] init];
[image setImage:[UIImage imageNamed:@"name.png"]];
image.image = [UIImage imageNamed:@"name.png"];
```
```objc
[btn setBackgroundColor:[UIColor redColor]];
btn.backgroundColor = [UIColor redColor];
```

## 3.将初始化 init 方法的返回值从 id 转换为转换为 instancetype

## 4.枚举宏定义的改变
将`enum`修改为`NS_ENUM`将枚举的后置修改为前置
#### Old Version
```objc
typedef enum { 
  EnumType, 
  EnumTypeA, 
  EnumTypeB 
} Type;
```
#### Modern Version:
```objc
typedef NS_ENUM (NSUInteger Type) { 
  EnumType, 
  EnumTypeA, 
  EnumTypeB 
};
```

## 6.字典和数组的一些赋值和取值的方式
### 6.1 字典
```objc
[dict setObject:value forKey:key];
dict[key] = value;
id value = [dict objectForKey: key]
id value = dict[key]
```
### 6.2 数组
```objc
[array objectAtIndex:index]
array[index]
```

## 7. 变量
```objc
@implementation Manager {
    UIButton *loginButton;
}
```
```objc
@interface HHMyClass ()
@property (nonatomic) UIButton *loginButton;
@end
@implementation HHMyClass
...
@end
```

## 8. 数组和字典的初始化

```objc
NSNumber *number = [NSNumber numberWithInteger:7];
NSNumber *modernNumber = @(7); // or @7

NSArray *array = [NSArray arrayWithObjects:number, modernNumber, nil];
NSArray *modernArray = @[number, modernNumber];

NSDictionary *dictionary = [NSDictionary dictionaryWithObjectsAndKeys:number, @"number"];
NSDictionary *modernDictionary = @{@"number" : number};
```