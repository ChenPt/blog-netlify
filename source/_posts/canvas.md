---
title: canvas元素的学习笔记
date: 2016-07-26 01:16:50
tags: canvas
category: HTML 5
---


## 什么是canvas
HTML5新增了 canvas元素，专门用来绘制图形，在页面上放置一个canvas元素，相当于在页面上放置一块"画布"，画布是一个矩形区域，你可以在其中进行图形的绘制。canvas元素放在html页面中，跟其他标签没太大区别。canvas 拥有多种绘制路径、矩形、圆形、字符以及添加图像的方法。
## 创建canvas元素
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Canvas~</title>
</head>
<body>
	<canvas id="mycanvas" width="200" height="200"></canvas>
</body>
</html
```
canvas 需指定 ID、width、height 三个属性值
### 绘制矩形
**canvas本身是没有绘制能力的，需要通过JavaScript来进行绘制。
用canvas元素绘制图形时，步骤如下**

 - 取得canvas元素（创建canvas元素）

```javascript
var canvas = document.getElementById("mycanvas");
```
 - 取得上下文对象
```javascript
var context = canvas.getContext("2d"); 
```
利用canvas对象的getContext方法来获得图形上下文，将参数设为“2d”,不能够设置为3d或4d。
>getContext("2d") 对象是内建的 HTML5 对象，拥有多种绘制路径、矩形、圆形、字符以及添加图像的方法。

 - 设定绘图样式
```javascript
context.fillStyle = "AAA";  //填充的样式,填入填充的颜色值
context.strokeStyle = "FF0000"; //图形边框的样式,填入边框的颜色值
```
 - 绘制图形
```javascript
context.fillRect(0, 0, 50, 50) // 填充矩形
context.strokeRect(0, 0, 50, 50) //绘制矩形边框
```
**x指矩形起点的横坐标，y指矩形起点的纵坐标，width指矩形的长度，height指矩形的高度**
```
context.clearRect(x,y,width,height)
```
clearRect ，该方法可以擦除指定的矩形区域的图形，使得该区域中的颜色全部变为透明。

**设定绘图样式必须在绘制图形之前，如果在绘制图形之后的话，绘图样式会使用默认值，而不是我们设定的那个值**
```javascript
context.fillRect(0, 0, 50, 50);
context.fillStyle("FF0000");
//最终图形显示的填充颜色值是默认的黑色而不是我们设定的红色（#FF0000）
```
## 使用路径
### 绘制圆形

 1. 开始创建路径
 2. 创建圆形路径
 3. 路径创建完成后，关闭路径
 4. 设定绘制样式，调用绘制方法，绘制路径
```javascript
var canvas = document.getElmentById("mycanvas");
var context = canvas.getContext("2d");

context.beginPath();    //创建路径
context.arc(50, 50, 50, 0, Math.PI*2, true);    //创建圆形路径
context.closePath();  //关闭路径
context.fillStyle = "rgba(100,100,100,0.2)";    //填充样式，给定颜色值
context.strokeStyle = "#000";   //绘制图形边框样式，给定颜色值
context.fill();     //因为路径已经确定了图形的大小，所以不用再使用参数去指定图形的大小,也可以使用stroke 方法
```
如果没有关闭路径，创建的路径会永远保留着，如果进行多次绘制，则创建的图形会一次又一次地进行重叠。
```javascript
var canvas = document.getElementById("mycanvas");
var context = canvas.getContext("2d");

context.fillStyle = "#EEEEFF";
context.fillRect(0, 0,500, 500);
for(var i = 0;i < 10; i++)
{
    context.arc(i*25, i*25, i*10, 0, Math.PI*2, true);
    context.fillStyle = "rgba(100,100,100,0.2)";
    context.fill();
}
```
如图
![image](http://note.youdao.com/yws/res/557/WEBRESOURCE880df486be209ef49fe78dd2b41739e1)
### 绘制直线

 1. moveTo (x,y)  将光标移动到指定坐标点，绘制直线的时候以这个坐标点为起点
 2. lineTo (x,y)   以moveTo方法设定的点为起点，到自己的参数中指定的点之间绘制一条直线  x表示直线终点的横坐标，y表示终点纵坐标

```javascript
var canvas = document.getElementById("mycanvas");
var context = canvas.getContext("2d");
context.beginPath();
context.moveTo(20,20);
context,lineTo(50,100);
context.lineWidth = 3;  //设置直线的宽度   ps：需要在stroke前面调用
context.stokeStyle = "#FF0000";
context.stroke(); //绘制直线
``` 
### 绘制渐变图形
#### 绘制线性渐变
使用图形上下文的createLinearGradient方法
context.createLinearGradient(xStart, yStart, xEnd, yEnd);
然后再使用LinearGradient对象的addColorStop方法
addColorStop(offset, color);
offset为设定的颜色离开渐变起始点的偏移量，范围为0~1；
```javascript
var g1 = context.createLinearGradient(0, 0, 200, 200);
g1.addColorStop(0,"#FF0000"）;  //起点偏移量为0，颜色为红色
g1.addColorStop(1,"#FFF"); //终点偏移量为1，颜色为白色
```
如图：
![image](http://note.youdao.com/yws/res/560/WEBRESOURCEd2681bd319ed28c140e09ac9e4f84fac)
#### 绘制径向渐变
使用图形上下文的createRadiaGradient方法
context.createLinearGradient(xStart, yStart, radiusStart, xEnd, yEnd, radiusEnd);
xStart为渐变开始圆的圆心横坐标，yStart为圆心纵坐标
同理xEnd，yEnd为渐变结束圆的圆心横纵坐标。
radiusStart为渐变开始圆的半径，radiusEnd 为渐变结束圆的半径
```javascript
var canvas = document.getElementById("mycanvas");
var context = canvas.getContext("2d");
var g2 = context.createRadialGradient(40,40,50,500,500,300);
g2.addColorStop(0,"blue");
g2.addColorStop(0.8,"red");
g2.addColorStop(1,"rgba(100,100,105,0.5)");
context.fillStyle = g2;
context.fillRect(0, 0,500, 500);
```
![image](http://note.youdao.com/yws/res/554/WEBRESOURCEec11c248c50ee94447354dc148bbf7f1)
### 绘制变形图形
#### 坐标变换

 - 平移  context.translate(x,y);
 - 扩大 context.scale(x,y) x为水平方向扩大的放大倍数，y是垂直方向的放大倍数
 - 旋转 context.rotate(angle);  angle 是指旋转的角度
利用这个坐标变换我们可以做很多好玩的图形。
```javascript
	var canvas = document.getElementById("mycanvas");
	var context = canvas.getContext("2d");

	context.fillStyle = "#888";
	context.fillRect(0,0,500,500);
	context.translate(250,100);
	context.fillStyle = "rgba(255,0,0,0.25)";
	for(var i = 0;i<60; i++)
	{
		context.translate(25,25);
		context.scale(0.93,0.95);
		context.rotate(Math.PI/10);
		context.fillRect(0,0,150,40);
	}
```
如此图
![image](http://note.youdao.com/yws/res/548/WEBRESOURCEf56b01eeb0973467321aee9dee3a7d1b)
未完待续...

