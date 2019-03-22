---
title: HTML常用标签
date: 2018-03-14 15:57:46
tags: 大前端实录
---

## 一、列表标签

### 1.1. 无序列表 `<ul>`
**作用：**
给一堆内容添加语义，表示这堆内容是一个无序列表（一个没有先后顺序整体），列表中的条目是部分先后顺序的

```html
<h4>选择一个中国的城市</h4>
<ul>
	<li>北京</li>
	<li>上海</li>
	<li>广州</li>
	<li>深圳</li>
</ul>
```

![](http://p4yixtz2j.bkt.clouddn.com/1520923767.png )

应用场景：

* 导航条
* 新闻列表
* 商品列表等

浏览器会给无序列表自动添加先导符号，但是一定要记得，ul标签的作用并不是给文字来添加小圆点，而是增加无序列表的语义。

ul是一个标签组，所以一定是一组一组出现的，也就是说li标签不能单独存在而必须要包裹在ul标签里面。ul标签和li标签是一个整体，所以ul标签里面不推荐包裹其他标签，最好只写li标签。

岁日安ul标签中只能写li标签，但是li标签是一个容器标签，li标签中却可以添加任意的标签，甚至可以添加ul标签。

**type属性：**

可以修改先导符号的样式，取值有：disc、square、circle几种。


### 1.2. 有序列表 `<ol>`
给一组内容添加语义，表示这组内容是一个有序列表（一个有顺序的整体），列表中的条目是有先后之分的。

```html
<h4>中国房价排行榜</h4>
<ol>
	<li>北京</li>
	<li>上海</li>
	<li>广州</li>
	<li>深圳</li>
</ol>
```

![](http://p4yixtz2j.bkt.clouddn.com/1520924246.png )

**应用场景：**

用来做排行榜（其实ol应用场景并不多，因为能用ol做的ul都能做）

start、type属性：可以修改先导符号的样式和序号，注意事项与ul标签的注意事项相同。

### 1.3. 定义列表 `<dl>`
给一组内容添加语义，表示这组内容是一个有顺序的整体，通过dt标签罗列出相应的条目，然后再通过dd标签给每个条目进行相应的描述。

```html
<dl>
	<dt>北京</dt>
	<dd>国家的首都，天安门广场</dd>
	<dt>上海</dt>
	<dd>中国的经济中心，俗称魔都</dd>
</dl>
```

**应用场景：**图文混排、网站底部的相关信息，单反看到一堆内容都是用于描述某一个内容的时候就要想到dl标签。

**注意事项：**

1. 由于dl标签和dt标签一样是容器标签，所以里面可以添加任意的标签。
2. 定义列表非常领过，可以给一个dt标签配置多个dd标签，但是最好不要出现多个dt标签对应了一个dd标签，dd标签的语义是描述离他最近的一个dt标签，所以其他dt标签相当于没有描述，而定义列表存在的意义就是既可以列出每一个条目，又可以对每一个条目进行描述。
3. 定义列表非常的灵活，可以将多个dt+dd标签组合拆分为多个dl标签。

## 二、表格标签
作用：以表格形式将数据显示出来

table定义表格、tr定义行、td定义单元格

```html
<table>
<tr>
	<td></td>
	<td></td>
</tr>
<tr>
	<td></td>
	<td></td>
</tr>
</table>
```

### 2.1. 表格中的属性

| 属性名 | 作用 
| --- | :---: |
| border | 指定边框宽度，默认情况下表格的边框宽度为0看不到 |
| cellspacing | 指定表格之间的间隙，默认值是2个像素 |
| cellpadding | 指定单元格边缘和内容之间的编剧，默认是1个像素 |
| width | 指定表格的宽度，默认情况下表格的宽度是由内容自动计算出来的 |
| height | 指定表格的高度，默认情况下表格的高度是由内容自动计算出来的 |
| align | 指定水平方向的对其方式，它的取值有：center、left、right |
| valign | 规定垂直方向对其方式，它的取值有：top、center、bottom |
| bgcolor | 规定表格的背景颜色 |

因为HTML仅仅用于说明语义，其实这些属性也仅仅作为了解，在企业开发中用css代替。

### 2.2. 表格中的其他标签

**caption标签：给整个表格设置标题**

一定要嵌套在table标签内部才有效，一定要紧跟在table标签的后面

**th标签：给每一列设置标题**

th标签中的内容文字加粗，居中。表格中有两种乐行的单元格，一种是标准的单元格td，一种是表头单元格th。

### 2.3. 表格的结构
* `thead`标签：用来存放每一列的表头
* `tbody`标签：用来存放页面中的主题数据，如果不写会自动加上
* `tfoot`标签：用来存放表格的页脚

表格的结构意义主要是用于SEO，便于搜索引擎指定哪部分的内容是需要抓取的重要内容，一般情况下，搜索引擎会优先抓取tbody中的内容。由于一部分浏览器对table这种结构支持不是很好，所以在企业开发中一般都不用严格的按照这种结构来编写。

### 2.4. 表格的结构
1. 纵向合并 colspan：合并当前列的哪几个单元格，向下合并
2. 横向合并rowspan：合并当前行的哪几个单元格，向右合并

## 三、表单标签 
专门用于收集用户信息，让用户填写、选择

`<form>
	所有的表单内容都要写在form标签里面
</form>`

form标签中有2个边角重要的属性：

1. action：表单提交的地址
2. method：提交表单的方式

### 3.1. 多行文本框 textarea标签
用于在表单中定义多行的文本输入控件

```html
<textarea cols="30" rows="10"> 默认
</textarea>
```

* cols属性规定文本区域内的可见宽度
* row是规定文本区域内的尅安行数

![](http://p4yixtz2j.bkt.clouddn.com/1520931405.png )

默认情况下文本区域可以拖拽缩放，可以利用CSS禁止缩放

```css
textarea {
    resize: none;
}
```

### 3.2. 下拉列表 select标签
select标签和ul、ol、dl一样，都是组标签，用于创建表单中的待选列表，可以从中选择某一个待选项。

```html
<select>
	<option>北京</option>
	<option>湖北</option>
	<option>贵州</option>
</select>
```

![](http://p4yixtz2j.bkt.clouddn.com/1520931787.png )


和radio标签、checkbox标签一样，select标签页可以设置默认值，通过selected属性来设置

```html
<select>
	<option>北京</option>
	<option>湖北</option>
	<option selected="selected">贵州</option>
</select>
```

给下拉列表添加分组：

```html
<select>
	<optgroup label="北京市">
		<option>海淀区</option>
		<option>朝阳区</option>
		<option>昌平区</option>
	</optgroup>
	<optgroup label="北京市">
		<option>天河区</option>
		<option>白云区</option>
	</optgroup>
</select>
```

![](http://p4yixtz2j.bkt.clouddn.com/1520932487.png )

## 四、input标签

### 4.1. 明文输入框
用户可以在输入框内输入内容，并且可以给输入框设置默认的值

```html
<input type="text" value="请输入用户名">
```

![](http://p4yixtz2j.bkt.clouddn.com/1521194126.png )

### 4.2 暗文输入框
用户可以在输入框内输入暗文内容

```html
<input type="password" value="请输入用户密码">
```

![](http://p4yixtz2j.bkt.clouddn.com/1521197319.png )

### 4.3 单选框(radio)
用户只能从众多选项中选择其中一个

```html
<input type="radio" name="xingbie"/> 男
```

单选按钮，天生就是不互斥的，如果想互斥，就必须要有相同的`name`属性

给单选设置默认值

```html
<input type="radio" name="xingbie"/ checked="checked"> 男
<input type="radio" name="xingbie"/> 女
<input type="radio" name="xingbie"/> 中性
```

![](http://p4yixtz2j.bkt.clouddn.com/1521197611.png )

### 4.4 多选框(checkbox)
用户只能从众多选项中选择多个

```html
<input type="checkbox" name="hobby" checked="checked"/> basketball
<input type="checkbox" name="hobby" checked="checked"/> football
<input type="checkbox" name="hobby"/> baseball
```

复选框最好也是有相同的那么(虽然不需要互斥，但是需要有相同的name)

![](http://p4yixtz2j.bkt.clouddn.com/1521617345.png )

### 4.5 lable标签
label标签不会向用户呈现任何特殊效果。不过，它为鼠标用户改进了可用性。

* 表单元素要有一个`id`，然后`label`标签就有了一个`for`属性，`for`属性和`id`相同就表示绑定了
* 所有表单元素都可以通过`label`绑定

```html
<label for="account"> 账号：</label>
<input type="text" id="account"/> 
```

![](http://p4yixtz2j.bkt.clouddn.com/1521618175.png )

### 4.6 按钮
定义可点击按钮，多数情况下，用于通过JavaScript启动脚本。

```html
<input type="button" value="我是按钮"/> 
```

![](http://p4yixtz2j.bkt.clouddn.com/1521618325.png )

### 4.7 图片按钮
定义图像形式的提交按钮
```html
<input type="image" src="http://p4yixtz2j.bkt.clouddn.com/1521618420.png"> 
```

![](http://p4yixtz2j.bkt.clouddn.com/1521618489.png )

### 4.8 重置按钮

这个按钮不需要写value自动就有`“重置”`文字，reset只对form表单中的表单项有效果。

```html
<input type="reset" /> 
```

![](http://p4yixtz2j.bkt.clouddn.com/1521618629.png )

### 4.9 提交按钮
定义提交按钮，提交按钮会把表单数据发送到action属性指定的页面

* 这个按钮不需要写`value`自动就有`“提交”`文字
* 要想通过`submit`提交数据到服务器，被提交的表单项都必须设置那么属性
* 默认明文传输`GET`不安全，可以将`method`属性设置为`POST`改为非明文

```html
<input type="submit" /> 
```

![](http://p4yixtz2j.bkt.clouddn.com/1521618996.png )


## 4.10 datalist 标签
给输入框绑定待选项

```html
<p/>输入您的车型   
<input type="text" list="cars">
<datalist id="cars">
	<option>奔驰</option>
	<option>宝马</option>
	<option>奥迪</option>
	<option>路虎</option>
	<option>宾利</option>
</datalist>
```

![](http://p4yixtz2j.bkt.clouddn.com/1521619042.png )

## 五、字符实体
在HTML中有的字符是被HTML保留的，有特殊含义的字符，是不能再浏览器中直接展示出来的，那么这些东西要想展示出来就必须通过字符实体。

1. `&lt;` 小于符号 `<`
2. `&gt` 大于符号 `>`
3. `&copy`版权符号 `©`

在HTML中对空格/回车/tab/不敏感，会把uduoge空格/回车/tab当做一个空格来处理，想输入多个空格就必须要输入 `&nbsp`

# 六、 新推出的标签
W3C又推出了一组新的标签，这些标签在现实上看似乎buis没什么区别，但是在语义上却又有重大的区别。

## 6.1 `strong` 标签
定义重要性强调的文字

```html
<strong>重要性内容</strong> 
```

![](http://p4yixtz2j.bkt.clouddn.com/1521620153.png )

## 6.2 `ins` 标签
`inserted`定义插入的文字

```html
<ins>新插入的内容</ins> 
```

![](http://p4yixtz2j.bkt.clouddn.com/1521620167.png )

## 6.3 `em` 标签
`Emphasized` 定义强调的文字

```html
<em>强调内容</em> 
```

![](http://p4yixtz2j.bkt.clouddn.com/1521620181.png )

## 6.4 `del` 标签
`Deleted`定义被删除的文字

```html
<del>已经删除的文本</del> 
```

![](http://p4yixtz2j.bkt.clouddn.com/1521620200.png )


## 七、 多媒体标签
### 7.1. vedio标签
用来播放视频

格式1：

```html
<video src="">
</video>
```

格式2：

```html
<video>
	<source src="" type=""></source> 
	<source src="" type=""></source> 
</video>
```
video标签的第二种格式存在的意义，就是为了解决浏览器适配的问题。

| 属性 | 作用 |
| --- | :---: |
| src | 告诉video标签需要播放的视频地址 |
| autoplay | 告诉video标签需要自动播放视频 |
| controls | 告诉video标签是否需要显示控制条 |
| poster | 告诉video标签视频没有播放之前显示的占位图片 |
| loop | 告诉video标签视频播放完毕之后是否需要循环播放 |
| preload | 预加载视频，preload和autoplay相冲突、互斥 |
| muted | 静音 |
| width/height | 和img标签中的一模一样 |

通过video标签的第二种格式虽然能够指定所有的浏览器都支持的视频格式，但是想让所有浏览器都通过video标签播放视频还有一个前提条件，就是浏览器必须都支持HTML5标签，负责同样无法播放。

过去的一些浏览器是不支持HTML5标签的，所以为了让过去的一些浏览器也能够通过video标签来播放视频，可以通过一个JS框架叫做`html5media`来实现。

### 7.2. audio标签
播放音频

格式1：

```html
<audio src="">
</video>
```

格式2：

```html
<audio src="">
	<source src="" type="">
</video>
```

audio标签的使用和video标签的使用基本一样，video中能够使用的属性在audio标签中大部分都能够使用，并且功能都一样只不过有3个属性不能用，`height/width/poster`

### 7.3. 详情和概要标签
利用summary标签来描述概要信息，利用details标签来描述详细信息。
默认情况下是折叠展示，想看见详情必须点击。

**格式：**

```html
<details>
	<summary>概要信息</summary>
	详细信息
</details>
```

![](http://p4yixtz2j.bkt.clouddn.com/1521629589.png )


### 7.4. marquee标签
跑马灯

```html
<marquee>内容</marquee>
```

| 属性 | 作用 |
| --- | :---: |
| direction | 设置滚动方向`left/right/up/down` |
| scrollamount | 设置滚动速度，值越大越快 |
| loop | 设置滚动次数，默认是-1，无限滚动 |
| behavior | 设置滚动类型 slide滚动到便捷就停止 alternate滚动到边界就弹回 |

marquee标签不是W3C推荐的标签，在W3C官方文档中也无法查询到这个标签，但是各大浏览器对这个标签的支持非常好。
 
## 八、被遗弃的标签
由于HTML现在只负责语义而不负责样式，但是HTML一开始有一部分标签连样式也包揽了，所以这部分标签都被废弃了。


| 标签 | 作用 | 格式 | 样式 |
| --- | :---: | --- | --- |
| b标签`Bold` | 将文本字体加粗 | `<b>加粗的文本</b>` |  <b>加粗的文本</b> |
| u标签`Underlined` | 将文本添加下划线 | `<u>下划线文本</u>` | <u>下划线文本</u> |
| i标签`Italic` | 将文本字体加粗 | `<i>斜体文本</i>` | <i>斜体文本</i> |
| s标签`Strikethrough` | 将文本添加删除线 | `<s>删除线文本</s>` | <s>删除线文本</s> |

**原则：不到万不得已，不会使用这些标签**
