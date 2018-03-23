---
title: 初识HTML
date: 2018-03-23 16:27:43
tags:
---

### 一、JavaScript(JS)简介:

`JavaScript`一种直译式脚本语言，是一种动态类型、弱类型、基于原型的语言，内置支持类型。

它的解释器被称为`JavaScript`引擎，为浏览器的一部分，广泛用于客户端的脚本语言，最早是在HTML网页上使用，用来给HTML网页增加动态功能。Node.js。 
在1995年时，由`Netscape`公司的`Brendan Eich`，在网景导航者浏览器上首次设计实现而成。因为`Netscape`与`Sun`合作，`Netscape`管理层希望它外观看起来像`Java`，因此取名为`JavaScript`。但实际上它的语法风格与`Self`及`Scheme`较为接近。 

为了取得技术优势，微软推出了`JScript`，`CEnvi`推出`ScriptEase`，与`JavaScript`同样可在浏览器上运行。为了统一规格，因为`JavaScript`兼容于`ECMA`标准，因此也称为`ECMAScript`。

#### 组成部分:

1. `ECMAScript`（核心）：`JavaScript` 语言基础规定了 `JavaScript` 脚本的核心语法，如 数据类型、关键字、保留字、运算符、对象和语句等，它不属于任何浏览器
2. `DOM`（文档对象模型）：规定了访问`HTML`和`XML`的接口(提供了访问 `HTML` 文档（如`body`、`form`、`div`、`textarea`等）的途径以及操作方法)
3. `BOM`（浏览器对象模型）：提供了独立于内容在浏览器窗口之间进行交互的对象和方法(提供了访问某些功能（如浏览器窗口大小、版本信息、浏览历史记录等）的途径以及操作方法)

#### 基本特点:

* 是一种解释性脚本语言（代码不进行预编译）。 
* 主要用来向HTML（标准通用标记语言下的一个应用）页面添加交互行为。 
* 可以直接嵌入HTML页面，但写成单独的js文件有利于结构和行为的分离。 

#### 日常用途:
* **网页特效**：嵌入动态的`HTML`页面
* **交互式操作**：对浏览器事件做出响应
* **表单验证**：在数据被提交到服务器之前验证数据
* web游戏（网页游戏）
* 服务器脚本开发等

---

### 二、JavaScript存在的位置:

#### 方式1: 
在`<html>`标签中,任何地方添加`<script> </script>`标签，标签中内容就是js代码. 虽然可以放在页面的任何地方,但是规范放在`<head>`标签中。

```html
<script type = "text/javascript">
    //当页面加载的时候，会执行大妈
    alert(123);
</script>
```

#### 方式2: 
单独使用一个文件来编写`javascript`代码(js文件),在需要使用的页面中引入该文件

```html
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
<script type = "text/javascript" src="index.js">
</script>
</head>
```

#### 方式3: 

把代码编写的`a标签`的`href属性`中,点击a标签的时候,就执行里面代码
```html
<a href="javascript:alert(123)">
</a>
```

![](http://p4yixtz2j.bkt.clouddn.com/1521797699.png )

### 三、基本语法 
#### 3.1. `JavaScript`中的标识符:
变量，常量，函数，语句块也有名字，我们统统称之为标识符。标识符可以由任意顺序的大小写`字母`、`数字`、`_` 和 `$` 组成，标识符不能以数字开头，不能是`JavaScript`中的保留字或关键字
	
**合法的标识符举例：**

`indentifier、username、user_name、_userName、$username`

**非法的标识符举例：**

`int、98.3、Hello World ` 

`JavaScript`严格区分大小写.`username`和`userName`是两个完全不同的符号 

**JavaScript程序代码的格式：**
	
每条功能执行语句的最后必须用分号 `;` 结束，每个词之间用空格、制表符、换行符或大括号、小括号这样的分隔符隔开。

语句块使用 `{ }` 来表示

**`JavaScript`程序的注释：**
`/*…*/`中可以嵌套 `//` 注释，但不能嵌套 `/*…*/` 、`/**..文档注释.*/`

#### 3.2. `JavaScript`中的变量:

声明变量要使用`var`关键字：`var name; //一般不使用name作为变量名`

声明变量的同时为其赋值：`var name = "will";`

对已赋值的变量赋予一个其他类型的数据：`name = 17;`

不事先声明变量而直接使用,报错 `not defined`.(未定义)

**提示：**

`javascript`定义变量无需指定类型，任何类型都使用`var`声明
变量的打印:
   方式1:使用alert(内容);
   方式2:使用console.debug(内容); 注意:只有W3C的浏览器才支持(IE不行),Firebug中,Google的控制台.
         好比System.out.println(内容);




