---
title: JavaScript简介
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
#### 3.1. `JavaScript`中的标识符
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

#### 3.2. `JavaScript`中的变量

1. 声明变量要使用`var`关键字：`var name; //一般不使用name作为变量名`
2. 声明变量的同时为其赋值：`var name = "will";`
3. 对已赋值的变量赋予一个其他类型的数据：`name = 17;`
4. 不事先声明变量而直接使用,报错 `not defined`(未定义)

**提示：**

`javascript`定义变量无需指定类型，任何类型都使用`var`声明
变量的打印:

* 使用`alert`(内容);
* 使用`console.debug`(内容); 

注意:只有`W3C`的浏览器才支持(`IE`不行)

#### 3.2. `JavaScript`中的基本类型和常量

##### `Number`(数字类型)

##### 3.2.1. 整型常量(10进制\8进制\16进制)

* 十六进制以`0x`或`0X`开头，例如：`0x8a`。
* 八进制必须以`0`开头，例如：`0123`。
* 十进制的第一位不能是`0`（数字`0`除外），例如：`123`
	
##### 3.2.2. 实型常量

`12.32、193.98、 5E7、4e5`等。

**特殊数值：**

* `NaN`(Not a Number)、`Infinity`(除数为零)，所对应的判断函数`isNaN()`、`isFinite()`
* `Boolean`(布尔值) `true`和`false`。
* `String`字符串(

**注意：**

`js`中没有`char`类型，所以 `a` 也是一个字符串，“`a book of JavaScript`”、‘`a`’、 “`a`”、“”。
	
1. 字符串中的特殊字符，需要以反斜杠 ( \ ) 后跟一个普通字符来表示。
2. `null`常量:表示引用任何的内存空间.
3. `undefined`常量 (未定义,定义未赋值)


##### 3.2.3. `JavaScript`中的运算符

1. 算术运算符：`+,-,*,/`
2. 赋值运算符：`=`
3. 比较运算符：`>,<`
4. 逻辑运算符：`&&,||,!`
5. 三元运算符：`表达式 ? value1 : value2`
6. 位运算符


##### 3.2.4. JS中特殊存在的案例

**问题：**`==`和`===`的区别：

 1. `==`：用来比较变量或者常量的内容,不管数据类型.
 2. `===`：比较数据类型和内容,若数据类型相同,且内容相同返回`true`.

```js
console.debug(17 == "17");//true
console.debug(17 === "17");//false
```

**注意：**在逻辑运算中，`0`、""、`false`、`null`、`undefined`、`NaN`均表示`false`。

1. `a && b`：将a, b转换为`Boolean`类型, 再执行逻辑与, 若表达式结果为`true`返回b, `false`返回a
2. `a || b`：将a, b转换为`Boolean`类型, 再执行逻辑或, 若表达式结果为`true`返回a, `false`返回b

`&&` 和`||` 运算符的区别(开关)：

* `&&`操作：返回最后一个为`true`的值，或者第一个为`false`的值
* `||` 操作：返回第一个为`true`的值，或则最后一个为`false`的值


```js
console.debug(true && true) // true
console.debug(1 && true) //true
console.debug(1 && 2) //2
console.debug("A" && 2) //2
console.debug("" && 2)  //""
console.debug(null && "B") //null
console.debug("A" && "B") //B
console.debug(1 && 2 && 3) //3
console.debug(1 && null && 3) // null
console.debug("" && null && 0) // ""
```

#### 3.3. 函数

**作用：**

1. 将脚本编写为函数，就可以避免页面载入时执行该脚本。
2. 函数包含着一些代码，这些代码可以多次被调用。

**函数的定义和调用:**

* 定义函数分类:普通函数/匿名函数.
* 函数必须调用才会执行.

**定义语法：**

```js
function 函数名(参数列表)
{
	函数体
	[return 值/变量]
}
```

![](http://p4yixtz2j.bkt.clouddn.com/1522055214.png )


**函数的返回：**

* 如果在函数中使用了`return`,那么函数的返回结果就是 `return` 后的值.
* 如果在函数中没有使用`return`,说明该函数没有结果,不用接收,若接收,结果为:`undefined`.

**匿名函数(经常使用)：**

* 在定义函数的时候,没有定义函数名称
* 此时,可以把函数当做是一个值/变量,可以赋给另一个变量.

![](http://p4yixtz2j.bkt.clouddn.com/1522055150.png )


变量根据在`function`的内外定义位置不同,分成:

1. 全局变量：`function`外部定义的变量称为全局变量
2. 局部变量：`function`内部定义的变量称为局部变量

**访问变量原则：就近原则，谁离我最近我就使用谁。**

```html
<script language="javascript">
    var msg = "全局变量";
    function show()
      {
        msg = "局部变量";
      }
      show();
      alert(msg);//局部变量
</script>

<script language="javascript">
    var msg = "全局变量";
    function show()
    {
    var msg; 
    msg = "局部变量";
    }
    show();
    alert(msg);//全局变量
</script>
```

### 四、JavaScript的系统函数

在JS中已经预先定义好的,可以直接拿来使用的函数。

1. `encodeURI`及`encodeURIComponent`方法：返回对一个`URI`字符串编码后的结果。
2. `decodeURI`及`decodeURIComponent`()方法：将一个已编码的URI字符串解码成最初始的字符串并返回。 
3. `parseInt`方法：将一个字符串按指定的进制转换成一个整数，语法格式为：`parseInt(numString, [radix])`：如果没有指定第二个参数，则前缀为 ‘0x’ 的字符串被视为十六进制，前缀为 ‘`0`’ 的字符串被视为八进制，所有其他字符串都被视为是十进制。
4. `parseFloat`方法：将一个字符串转换成对应的小数。 
5. `isNaN`方法：检查一个值是否为`NaN`。 
6. `escape`方法(不推荐使用,改为`encodeURIComponent`)：返回对一个字符串进行编码后的结果字符串。所有空格、标点、重音符号以及任何其他非 `ASCII` 字符都用` %xx` 编码替换，其中`xx`等于表示该字符的`Unicode`编码的十六进制数，字符值大于`255`的字符以`%uxxxx`格式存储。 
7. `unescape` 方法(不推荐使用，改为`decodeURIComponent`)：将一个用escape方法编码的结果字符串解码成原始字符串并返回。
8. `escape`方法：编码的结果字符串解码成原始字符串并返回。
9. `eval` 方法：将参数字符串作为一个`JavaScript`表达式执行。

![](http://p4yixtz2j.bkt.clouddn.com/1522056443.png )

![](http://p4yixtz2j.bkt.clouddn.com/1522056560.png )

### 五、面向对象

对象中所包含的变量就是对象的`属性`，对象中所包含的对属性进行操作的函数就是对象的`方法`，对象的属性和方法都叫对象的成员。 

#### 5.1. 类
类是对某一类事物的描述，是抽象上的概念；而对象实例是一类事物中的具体个例。

能够被用来创建对象实例的函数就叫对象的`构造函数`。使用`new`关键字和对象的构造函数就可以创建对象实例，

语法格式如下：

```js
var objInstance = new ObjName([传递给该对象的实际参数列表]);
```

#### 概念区分

1. 函数：面向过程的,可以独立存在和运行
2. 方法：面向对象的,函数在面向对象中的称谓，此时不能独立存在,因为要依附于对象,也使用对象来调用
3. 构造函数：专门用来构建对象的
4. 类：就是一个函数,是一个特殊的函数,构造函数

**js创建一个类,只需要定义类的构造函数(方法)：**

![](http://p4yixtz2j.bkt.clouddn.com/1522056949.png )

#### 在JS中this表示什么? 

**this就表示当前对象**

1. `this`在构造器(类)中，此时`this`就表示当前被创建的对象
2. `this`在方法中，此时哪一个对象调用`this`所在的方法，`this`就是哪一个对象

![](http://p4yixtz2j.bkt.clouddn.com/1522057144.png )

#### 5.2. 内置对象

**Object:** 创建对象,并设置属性和方法

```js
var obj = new Object();
obj.name = "will";
obj.age = 17;
obj.sayHello = function() {
};
// 对象的构造函数
alert(obj.constructor);
// 是否有指定的属性
debug(obj.hasOwnProperty("name1"));
// 迭代对象所有的属性+方法:for...in
for (attr in obj) {
	alert(attr)//属性名
}
```

**Array**：数组类

**Boolean**：布尔类型

**Date**：时间类

```js
var d = new Date();
var day = d.getDate();
day = day<10?"0"+day:day;
var time = d.getFullYear() + "-" + (d.getMonth()+1) + "-" + day + " "
		+ d.getHours() + ":" + d.getMinutes() + ":" + d.getSeconds();
```

**Math**：数学公式相关API

```js
var num = Math.random();
```

**Number**：数字类

**String**：字符串类

```js
// 随机生成A到Z之间的字母:65+26
// 随机生成0~25
var num = parseInt(Math.random() * 26);//
num = num + 65;
alert(String.fromCharCode(num));
```

#### 5.3. 数组

**如何创建数组对象:**

![](http://p4yixtz2j.bkt.clouddn.com/1522058358.png )

**操作数组的属性和方法:**

* length－获得数组的长度；
* concat－连接数组；
* join－把数组转换成字符串；
* pop－删除最后一个元素；
* push－放入一个元素；
* reverse－颠倒数据中的元素顺序；
* shift－移出第一个元素；
* slice－截取数组;
* sort－排序数组;
* unshift－在前面追加元素；
* splice(start, deletecount, items) 方法用于插入、删除或替换数组的元素。

![](http://p4yixtz2j.bkt.clouddn.com/1522058452.png )

#### 5.4. prototype

需求:给数组提供一个修改指定索引位置元素的方法(set).

![](http://p4yixtz2j.bkt.clouddn.com/1522058807.png )

**使用prototype：** 可以理解为在类上增加方法。
方法属于类，属于该类所有的对象。








