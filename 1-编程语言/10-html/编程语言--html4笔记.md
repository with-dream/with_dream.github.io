### 一、概述
``` html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
 
<h1>我的第一个标题</h1>
 
<p>我的第一个段落。</p>
 
</body>
</html>
```


<!DOCTYPE html>	为头部声明
 HTML5
<!DOCTYPE html>
HTML 4.01
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">

&lt;html> 元素是 HTML 页面的根元素
&lt;head> 元素包含了文档的元（meta）数据，如 &lt;meta charset="utf-8"> 定义网页编码格式为 utf-8。
&lt;title> 元素描述了文档的标题
&lt;body> 元素包含了可见的页面内容

### 二、基本标记
###### 标题: &lt;h#>内容&lt;/h#>   #=1~6  数字越大 字体越小

###### 段落: &lt;p>内容&lt;/p> 段落之间会有一个空行的效果

###### 换行: &lt;br/>
###### 强制不换行: &lt;nobr>&lt;/nobr>

###### 粗体：&lt;b>&lt;/b>

###### 斜体：&lt;i>&lt;/i>

###### 强调：&lt;em>&lt;/em>              显示为斜体，不要为了使用斜体而是用em

###### 强调：&lt;strong>&lt;/strong>    比em更强烈的强调，显示为粗体,不要为了使用粗体而是用strong

### 三、字体标记
无序列表：
&lt;ul>
&lt;li>内容&lt;/li>
&lt;/ul>

有序列表：
&lt;ol>
&lt;li>内容&lt;/li>
&lt;/ol>

带标题的列表
&lt;dl>
&lt;dt>列表标题&lt;/dt>
&lt;dd>列表内容&lt;/dd>
&lt;dd>列表内容&lt;/dd>
&lt;/dl>

### 四、图像
&lt;img src="图片路径" alt="无图片时提示" width="宽" height="高">

### 五、超链接
&lt;a href="url">内容&lt;/a>

### 六、表单
&lt;form>
	&lt;input type="">
&lt;/form>

type可以为
> text：文本域
password：密码
radio：单选按钮
checkbox：复选框
submit：提交按钮
option:下拉表

### 七、表格
&lt;table border="1">
	&lt;tr>
        &lt;th>标题&lt;/th>
    &lt;/tr>
    &lt;tr>
        &lt;td>内容&lt;/td>
    &lt;/tr>
&lt;/table>

th和td中跨多行:rowspan
th和td中跨多列:colspan

### 八、框架
https://blog.csdn.net/doigt_love/article/details/52869218
&lt;frameset rows="50%,*">
   &lt;frame   name="frame1"   src="test1.htm"/>
   &lt;frame   name="frame2"   src="test2.htm"/>
&lt;/frameset>

### 九、区块
内联元素：内联元素在显示时通常不会以新行开始。
实例: &lt;b>, &lt;td>, &lt;a>, &lt;img>

区块元素：块级元素在浏览器显示时，通常会以新行来开始（和结束）。
实例: &lt;h1>, &lt;p>, &lt;ul>, &lt;table>

&lt;div> 元素：没有特定的含义，用作容器，是区块元素
&lt;span> 元素：内联元素，可用作文本的容器




参考：http://www.runoob.com/html/html-blocks.html








