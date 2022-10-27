### 一、css创建
###### 1、外部样式表
当样式需要应用于很多页面，使用`<link>`标签
```html
<head>
<link rel="stylesheet" type="text/css" href="css文件.css">
</head>
```

###### 2、内部样式表
使用`<style>`标签
```html
<head>
<style>
hr {color:sienna;}
p {margin-left:20px;}
body {background-image:url("images/back40.gif");}
</style>
</head>
```

###### 3、内联样式
将标签和样式放在一起，不容易修改，谨慎使用
```html
<p style="color:sienna;margin-left:20px">这是一个段落。</p>
```

###### 4、样式优先级
样式可以继承，如果有重复的属性值不同，优先级为：
内联样式 > 内部样式表 > 外部样式表

### 二、选择器
###### 1、元素选择器
直接作用于标签
`html {color:black;}`

也可以将多个标签设置相同的属性，且各个标签的可以重写
`body, h2, p, table, th, td, pre, strong, em {color:gray;}`

###### 2、类选择器
类名前有一个点号（.）
```html
<h1 class="important">
This heading is very important.
</h1>

<p class="important">
This paragraph is very important.
</p>
```
可以指定标签的类`h1.important {color:blue;}`
也可以选择所有class为important  `.important {color:blue;}`
`h1,p.important {color:red;}`

也可以对一个标签设置多个类
```html
<h1 class="important urgent warning">
This heading is very important.
</h1>
```

###### 3、ID选择器
类名前有一个点号（#）
一个文档中只能使用一次，且不能使用列表
```html
<p id="intro">This is a paragraph of introduction.</p>

#intro {font-weight:bold;}
```

###### 4、属性选择器
匹配标签的属性
```html
img[alt] {border: 5px solid red;}

a[href="http://www.w3school.com.cn/"][title="W3School"] {color: red;}
```

###### 5、后代选择器
匹配当前标签中的标签
```html
<h1>This is a <em>important</em> heading</h1>

h1 em {color:red;}
```

###### 5、子元素选择器
只能选择作为某元素子元素的元素
```html
<h1>This is <strong>very</strong> <strong>very</strong> important.</h1>

h1 > strong {color:red;}
```

与后代选择器的区别是子元素必须为当前标签的直接子元素
```html
多了一层p标签 子元素选择器将无法选定 但是后代选择器可以
<h1>This is <p><strong>very</strong> <strong>very</strong></p> important.</h1>
```

###### 6、相邻兄弟选择器
选择紧接在另一个元素后的元素，而且二者有相同的父元素
```html
<div>
  <ul>
    <li>List item 1</li>
    <li>List item 2</li>
    <li>List item 3</li>
  </ul>
  <ol>
    <li>List item 1</li>
    <li>List item 2</li>
    <li>List item 3</li>
  </ol>
</div>

li + li {font-weight:bold;}
第二个和第三个列表项变为粗体，第一个列表项不受影响
```

###### 7、伪类
浏览器内核预定义的选择符，功能与类相同且可以被占用
如&lt;a> 的link、visited属性

###### 8、伪元素
CSS 伪元素用于向某些选择器设置特殊效果。
如：before在元素之前添加内容    after正在元素之后添加元素

> 伪类与伪元素的特性及其区别：
伪类本质上是为了弥补常规CSS选择器的不足，以便获取到更多信息；
伪元素本质上是创建了一个有内容的虚拟容器；
CSS3中伪类和伪元素的语法不同；
可以同时使用多个伪类，而只能同时使用一个伪元素；
https://www.cnblogs.com/ihardcoder/p/5294927.html

### 三、CSS属性的结构和重叠
###### 1、特殊性
```html
<h1>This is a <em>important</em> heading</h1>
<p>This is a <em>important</em> paragraph.</p>

em {color: red;}
h1 em {color: blue;}

h1中的em标签内容为蓝色  其他标签的em中为红色
```
###### 2、继承
相同标签的属性有继承性

当标签有相同属性时，约束越具体，优先级越高
优先级相同的情况下，顺序在后面的起效果

### 四、值和单位
###### 1、值
值可以使用具体的数字或者百分比

###### 2、颜色
颜色可以使用名称 如：{color: red;}
可以使用rgb值{color: #FFFF0000;}
可以使用color:rgb(0,0,255);
或者按百分比{color: rgb(0%,0%,50%);}

###### 2、单位
绝对长度单位包括有： cm, mm, q, in, pt, pc, px
相对长度单位包括有： em, ex, ch, rem, vw, vh, vmax, vmin
最常使用的是px

> em是定义文字大小的值，也就是文本中font-size属性的值。例如：定义某个元素的文字大小为12pt，那么，对于这个元素来说1em就是12pt。单位em的实际大小是受到字体尺寸的影响的。

> ex和em类似，指的是文本中字母x的高度，因为不同的字体的x的高度是不同的，所以ex的实际大小，受到字体和字体尺寸两个因素的影响。
https://blog.csdn.net/u013909970/article/details/47124801

#### 五、字体
###### 1、字体系列
通用字体系列 - 拥有相似外观的字体系统组合（比如 "Serif" 或 "Monospace"）
特定字体系列 - 具体的字体系列（比如 "Times" 或 "Courier"）

`body {font-family: 字体;}`

如果想使用特定字体，但是担心有些浏览器不支持，可以使用多个字体解决
`h1 {font-family: 特定字体, 通用字体}`

字体支持正则匹配，但是需要加上引号`body {font-family: 'Seri%';}`

###### 2、字体的加粗、大小、风格
略~

### 六、文本属性
###### 1、文本缩进		text-indent 值可以为负
###### 2、行高			line-height
###### 3、文本对齐：
text-align 主要控制左右
vertical-align 主要控制上下
https://blog.csdn.net/ssisse/article/details/52493319
###### 4、字间隔/word-spacing	字母间隔/letter-spacing
###### 5、空白符处理	white-space
###### 5、文本阴影	text-shadow

### 七、视觉格式化
margin/border/padding-->https://www.cnblogs.com/wzhiq896/p/6020329.html
改变元素的显示方式：display		可以更改标签的块级元素或者行内元素

### 八、浮动和定位
###### 1、定位 position
https://www.cnblogs.com/keyi/p/5817748.html
static		默认
relative	元素偏移某个位置,未偏移前的位置仍保留
absolute	偏移某个位置，但是不会保留原来的空间
fixed		固定位置 与absolute类似 只是不会滑动
left/top/right/bottom

###### 2、Flex
http://www.runoob.com/w3cnote/flex-grammar.html
display: flex;	//块元素
display: inline-flex;	//行内元素

* flex-direction
	+ row（默认值）：主轴为水平方向   column垂直
* flex-wrap	主轴放不下时的情况
	+ nowrap（默认）：不换行
	+ wrap 换行，第一行在上方
	+ wrap-reverse 换行，第一行在下方
* flex-flow
	+ flex-direction和flex-wrap的简写，默认值为row nowrap。
	+ 写法`flex-flow: <flex-direction> <flex-wrap>`
* justify-content
	+ flex-start | flex-end | center
	+ space-between 两端对齐，项目之间的间隔都相等
	+ space-around	每个项目两侧的间隔相等
* align-items
	+ 项目在交叉轴上如何对齐
	+ flex-start | flex-end | center | baseline
	+ stretch 如果项目未设置高度或设为auto，将占满整个容器的高度
* align-content
	+ 多根轴线的对齐方式 只有一条不起作用
	+ flex-start | flex-end | center
	+ space-between | space-around
	+ stretch

###### 3、float
定义元素向左/右方向浮动，会脱离标准流。浮动元素会生成一个块级框，而不论它本身是何种元素
清除float的作用：`clear:both;`

###### 3、裁剪 overflow 内容溢出元素框时的操作 /clip
###### 4、元素可见性	visibility
###### 5、层级		z-index

### 九、其他
###### 1、block，inline和inline-block
https://www.cnblogs.com/KeithWang/p/3139517.html
block:块元素  独占一行  有width,height,margin,padding
inline：内联元素	不会独占一行 没有width,height   margin,padding只有左右有效
inline-block:表现为内联对象 但是有块元素的作用

###### 2、盒模型
css3: https://www.cnblogs.com/chengzp/p/cssbox.html
```css
/* 标准模型 */
box-sizing:content-box;
 /*IE模型*/
box-sizing:border-box;
```

###### 3、图片
background-origin	控制图片的相对参考位置
















参考：http://www.w3school.com.cn/css/css_selector_grouping.asp

