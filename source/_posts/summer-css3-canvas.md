---
title: CSS3 & H5 Canvas
category: 2017暑假
tags: [暑假, 夏令营, 2017]
date: 2017-07-11
---

因为需要给大一捞仔们介绍下..CSS和 H5的东西,所以就顺便复习了下这些知识,写成学习记录blog,结合CSS揭秘和MDN,对CSS3的新特性也了解加深了..
<!-- more -->

## CSS3
### 选择器
 参考W3 
 http://www.w3school.com.cn/cssref/css_selectors.asp
 或者  MDN 
 https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference#选择器

### 背景和边框
+  border-radius
+  box-shadow
+  border-image
#### border-radius
border-radius 有四个参数  border-radius(top-left, top-right, bottom-right, bottom-left)
#### box-shadow
box-shadow（inset, offset-x, offset-y, blur-radius, spread-radius, color）
默认是outset 当第一个参数写了inset之后就变成阴影在边框内，背景之上内容之下。
offset-x, offset-y 是 水平和垂直的阴影偏移量
blur-radius  默认为0，边缘锋利，不能为负，值越大，模糊面积越大，阴影就越大越淡
spread-radius 取正值，阴影扩大，取负值，阴影收缩。默认为0，此时阴影与元素同样大
#### border-image 
### 文本效果
+  text-shadow  文本阴影
+  word-wrap: normal | break-word;  强制文本进行换行
+   text-overflow: clip | ellipsis | string;   (修剪| 省略号代替溢出的文本 | 使用给定的字符串而不是省略号来代替)
### @规则
#### @font-face规则 （使用自定义字体）
```
@font-face {
	font-family: myFont;
	src: url('xxxx.ttf');  //自定义的字体文件路径
}

div {
	font-family: myFont;
}
```

#### @media规则 （媒体查询，作用：在不同设备上使用不同的CSS）
```css
/*在打印机设备上显示*/
@media print {
}
/*在屏幕上显示*/
@media screen {
}


/*在700px~1024px的设备上显示以下样式*/
@media screen and (max-width: 1024px) and (min-width: 700px) {
}
```

#### @keyframes规则 （描述CSS动画的中间步骤）放在CSS3动画一起讲
### CSS3 变形 （2D&3D转换）
 **transform**属性： &nbsp 可以在不影响正常文档流的情况下改变作用内容的位置
#### 2D转换
+  translate(x,y)  元素从当前位置分别在水平和垂直移动 x、y 距离
+  rotate(x deg) 元素顺时针旋转给定的角度。负值时，元素将逆时针旋转
+  scale(x, y)  把尺寸转为原来的 x、y倍
+  skew(x deg, y deg) 围绕 X、Y轴翻转 
+  matrix()   前面的5个的综合

```css 
div {
	transform:translate(50px, 100px) || rotate(30deg) || scale(1,2) || skew(20deg, 30deg)
}
```

#### 3D转换
	1.  rotateX( )  围绕X轴 以给定的度数进行旋转
	2.  rotateY( )  围绕Y轴 以给定的度数进行旋转


### CSS3过渡
**transition**属性： &nbsp 简写属性，一般需要设置的是变化的属性值和变化时间
```css
div {
	width: 50px;
	heigth: 50px;	
	transition: width 2s;
}

div:hover {
	width: 100px;
}
```

### 动画	
#### @keyframes 规则（描述CSS动画的中间步骤）
```css
@keyframes myAnimate{
	from {
		color: red;
	}
	to {
		color: black;
	}
}

@keyframes Second {
	0% {
		background-color: red;
	}
	25% {
		background-color: green;
	}
	50% {
		background-color: blue;
	}
	75% {
		background-color: white;
	}
	100% {
		background-color: black;
	}
}
```

#### 动画属性
```css
div {
	animation-name: myAnimate;  /*规定@keyframes 动画的名称*/
	animation-duration: 5s;  /&规定动画完成一个周期所花费的时间*/
	animation-timing-function: ease; /*规定动画的速度曲线 默认为"ease"*/
	animation-delay： 100ms； /*表示延迟多久才开始动画*/
	animation-iteration-count: infinite; /*规定播放的次数，默认为1, infinite表示无限循环*/
	animation-direction： alternate； /*规定动画在下一周期是否逆向播放，默认为'normal'*/
	animation-play-state: running; /*规定动画是否在进行中，默认为running*/
}

/*简写属性*/
.second {
	animation: Second 5s linear 1s infinite alternate;
}
```

## Flex布局
>阮一峰Flex教程 http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html?utm_source=tuicool  

传统的布局解决方案是 display + position + float ,但这种方案要实现垂直居中就很麻烦.

Flex是Flexbile Box的缩写 意思是"弹性布局"  
Flex现在的兼容性就已经很好了,所以可以很安全的使用flex布局而不用怎么考虑浏览器兼容性了.
### What is Flex?
任何容器都可以使用flex布局
```css
.box {
    display: flex;
}
```
```css
/*行内元素也可使用flex布局*/
.box {
    display: inline-flex;
}

```
**注意:任何使用flex布局的 float,clear,vertical-align的属性将失效**

### Flex基本概念

**Flex Container & Flex item**

采用Flex布局的元素, 称为**Flex Container -> 容器**, 此容器的所有子元素自动成为容器成员,称为**Flex item -> 项目**

<img src="http://osuuzm0m8.bkt.clouddn.com/2017-07-10%2011-52-19%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE.png">

容器默认存在两根轴,主轴(main axis) & 交叉轴(cross axis) 主轴的开始位置叫做main start 结束位置叫做main end
同理有 cross start & cross end.项目默认沿主轴排列.  单个项目占据的主轴空间叫做main size,占据的交叉轴空间叫做 cross size .

### 容器的属性

1.   flex-direction 决定主轴的方向 即item的排列方向  
row row-reverse column column-reverse  
2.   flex-wrap 
wrap、nowrap、 wrap-reverse
3.   flex-flow 
flex-flow属性是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap
4.   justify-content
5.   align-items
6.   align-content

### 项目的属性
+  order 定义项目的排列顺序。数值越小，排列越靠前
+  flex-grow  定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大.
+  flex-shrink 定义项目的缩小比例，默认为1，当空间不足时，该项目将缩小。
+  flex-basis
+  flex：  flex-grow & flex-shrink & flex-basis的简写，后两个属性可选。
+  align-self   可覆盖align-item属性
### 例子

## Canvas
### What is Canvas？ 
> canvas  标签只有两个属性—— width和height。这些都是可选的，并且同样利用 DOM properties 来设置。当没有设置宽度和高度的时候，canvas会初始化宽度为300像素和高度为150像素。该元素可以使用CSS来定义大小，但在绘制时图像会伸缩以适应它的框架尺寸：如果CSS的尺寸与初始画布的比例不一致，它会出现扭曲。

### 用法
@(HTML)
```html
<body onload = "draw();">
	<canvas id = "canvas" width="500" height="500"></canvas>
</body>
```
@(Js)

```javascript
function draw(){
	var canvas = document.getElementById("canvas"); //获取canvas元素
	var t = canvas.getContext('2d'); //canvas元素的getContext('2d')方法 获得渲染上下文和它的绘画功能
}
```
canvas的栅格和 坐标空间
![Markdown](http://i1.buimg.com/1949/e6019cbf5a8745b0.png)

绘制矩形
```javascript
t.fillStyle = 'rgba(130,120,244,.5)'; //设置填充样式
t.strokeStyle = 'rgba(100,200,130,.5)'; //设置描边样式

t.fillRect(50, 50, 30, 30); //绘制一个填充的矩形
t.strokeRect(75, 75, 30, 30); //绘制一个矩形的边框
```
结果如下图
![Markdown](http://i1.buimg.com/1949/71022b72e3961e1e.png)

### 使用路径
绘制圆形
```javascript  
t.beginPath();
t.arc(50, 50, 50, 0, Math.PI*2, true);  
// arc(x, y, radius, startAngle, endAngle, anticlockwise)  anticlockwise默认为true，顺时针 
t.closePath();	//关闭路径
t.fillStyle = 'rgba(231, 142, 56, .5)';  
t.fill(); //填充
```
如图
![Markdown](http://i1.buimg.com/1949/3a76773b6033b51c.png)

绘制线条
```javascript  
t.beginPath();
t.moveTo(50, 50);
t.lineTo(50,75);
t.closePath();
t.stroke();
```
![Markdown](http://i4.piimg.com/1949/50fb206a53c8caf2.png)

其他高级操作自行探索


### 清除画布

```
t.clearRect(0, 0, 500, 500) // （x, y, width, height）
```

### 利用canvas制作动画
1.  清空画布
2.  保存canvas状态
3.  绘制动画图形
4.  恢复canvas状态

**使用canvas制作时钟**


## LocalStorage
### What is LocalStorage？
localstorage 是用来存储客户端临时信息H5新特性.
```
typeof(window.localStorage)  //object
```
localstorage 存储的数据没有时间限制。
与之对应的sessionStorage 存储的数据当用户关闭浏览器窗口后，数据会被删除。
他们均只能存储字符串类型.
```javascript
var storage = localStorage;
storage.setItem("name",4);  // 写入
storage["temp"] = 1;		// 写入
console.log(storage.getItem("temp"));  // 1
console.log(storage["name"]);		// 4
console.log(typeof storage["name"]);  //string

// 删除localStorage的某个键值对
storage.removeItem("name")；
console.log(storage);  // { temp: '1', length:1}
// 将localStorage的所有内容删除
storage.clear();      // {}
```

