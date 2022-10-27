### 一、CSS选择器
##### 1、基本选择器
| 类型 | 选择器 |
|--------|--------|
|   通配选择器     |     *   |
|   元素选择器     |     E   |
|   ID选择器     |     #id   |
|   类选择器     |     .class   |
|   群组选择器     |     s1, sn   |

##### 2、层次选择器
| 类型 | 选择器 |
|--------|--------|
|    后代选择器    |    E F    |
|    子选择器    |    E > F    |
|    相邻兄弟选择器    |    E + F    |
|    通用选择器    |    E ~ F    |
##### 3、伪类
###### 3.1、动态伪类选择器
分为链接伪类选择器(E:link, E:visited)和用户行为伪类选择器(E:active, E:hover, E:focus)
###### 3.2、目标伪类选择器
E:target
###### 3.3、语言伪类选择器
###### 3.4、UI状态伪类选择器
E:checked  E:enable  E:disable
###### 3.5、否定伪类选择器
E:not(F)
###### 3.6、结构伪类选择器
| 选择器 | 功能 |
|--------|--------|
|    E：first-child    |     匹配第一个   |
|    E：last-child    |     匹配最后一个   |
|    E：root    |     根元素 一般为html   |
|    E F：nth-child(n)    |     父元素E的第n个元素F n从1开始 |
|    E F：nth-last-child(n)    |     父元素E的倒数第n个元素F |
|    E：nth-of-type(n)    |     指定类型的第n个元素   |
|    E：first-of-type    |     指定类型的第n个元素   |
|    E：only_child    |     父元素只有一个子元素且子元素匹配E   |
|    E：only-of-type    |     父元素只有一个同类型的子元素且子元素匹配E   |
|    E：empty    |     没有子元素的元素   |

div > p:nth-child(2){样式} div下如果第二个元素为p 则匹配样式

###### 3.6、伪元素
css的伪元素：```:first-line  :first-letter  :before  :after```
css3在之前的基础上增加了一个：
| 元素 | 功能 |
|--------|--------|
|    ::first-line    |    选择文本块的第一行    |
|    ::first-letter    |    选择文本块的第一个字母    |
|    ::before    |    插入文本    |
|    ::after    |    插入文本    |
|    ::slelction    |    改变选中文字的颜色和背景色    |

##### 4、属性选择器
| 选择器 | 功能 |
|--------|--------|
|    E[attr]    |    arrt为属性 E可以省略    |
|    E[attr=val]    |    val为值    |
|    E[attr I= val]    |    val或val开头的属性    |
|    E[attr~=val]    |    attr具有多个值且有空格分隔 有一个为val    |
|    E[attr*=val]    |    包含val    |
|    E[attr^=val]    |    val为开头    |
|    E[attr$=val]    |    val为结尾    |

### 二、渐变
线性渐变、径向渐变、重复渐变
http://www.runoob.com/css3/css3-gradients.html

### 三、变形
http://www.runoob.com/css3/css3-3dtransforms.html
##### 1、2D变形
translate()
rotate()
scale()
skew()
matrix()

##### 2、3D变形
matrix3d(n,n,n,n,n,n,n,n,n,n,n,n,n,n,n,n)
translate3d(x,y,z)
scale3d(x,y,z)
rotate3d(x,y,z,angle)
perspective(n)  定义 3D 转换元素的透视视图

##### 2、属性
| 属性 | 作用 |
|--------|--------|
|    transform    |     顺序执行多个动画   |
|    transform-origin(x,y,z)    |     指定变形的中心位置   |
|    transform-style(flat I preserve-3d)    |     执行效果为2D/3D   |
|    perspective:像素    |     元素距离视图的距离   |
|    perspective-origin    |     定义 3D 元素所基于的 X 轴和 Y 轴   |
|    backface-visibility    |     元素不面向屏幕时是否可见   |

### 四、过渡
transition: property duration timing-function delay;
property：属性
duration:持续时间
timing-function:加速度
delay：延迟

加速度表：

| 属性 | 作用 |
|--------|--------|
|    linear    |    线性    |
|    ease    |    慢-快-慢    |
|    ease-in    |    加速   |
|    ease-out    |    减速    |
|    ease-in-out    |    快-慢-快    |
|    cubic-bezier(n,n,n,n)    |    三次贝塞尔    |

### 五、动画
animation: name duration timing-function delay iteration-count direction fill-mode play-state;

name:由@keyframes定义
duration：持续时间
timing-function：加速度
delay：延迟
iteration-count：循环次数
direction：播放方向
fill-mode：应用完成或开始前的延迟的状态
play-state：播放状态

##### 5.1 keyframes
```css
@keyframes mymove
{
    0%   {top:0px;}
    50%   {top:0px;}
    100% {top:0px;}
}
@keyframes mymove
{
    from   {top:0px;}
    to   {top:0px;}
}
```


