---
title: 函数柯里化 & 偏函数应用
date: 2017-07-12 01:16:50
tags: [JavaScript,Function]
category: JavaScript
---

在学习function.prototype.bind()方法的时候看到了bind() 的一个作用是实现偏函数应用,顺便了解到了函数柯里化.
<!-- more -->

## 函数柯里化（Currying）& 偏函数应用（Partial Application）
偏函数解决的问题是，如果有函数是具有多个参数的，希望能固定其中某几个参数的值。
而函数柯里化，由数学家curry发明的这种使用技巧，所以以他的名字命名。（滑稽.jpg）（库里不但会投三分还会数学哇）。函数柯里化是将一个接受多个参数的函数转变为一组支持链式调用的函数链，其中每个函数仅有一个入参。也可以说，将单参数函数实现为多参数函数的方法。

### 偏函数应用
>**引用mdn**：mdn上的bind( )的其中一种用法就是实现偏函数应用。
>bind( )简单用法之一就是使一个函数拥有预设的初始参数，这些参数作为bind( )二个参数跟在this（或者其他对象）后面，之后它们会被插入到目标函数的参数列表的开始位置，传递给绑定函数的参数会跟在他们后面。

```javascript
// bind()实现偏函数应用1.
// Array.prototype.slice的应用之一：把类数组对象转化为真正的数组。
var slice = Array.prototype.slice;

function fn() {
	// arguments对象是一个类数组对象，除了长度之外没有任何数组属性。但是可以被转换为一个真正的数组
	return slice.call(arguments);  
}

var fn1 = fn(1, 2, 3);

var fn37 = fn.bind(undefined, 37);  

fn37(1, 2, 3);  // [37, 1, 2, 3]  cause：37会插入到目标函数的参数列表的开始位置，而1,2,3是传给绑定函数即fn37()的参数

// bind()实现偏函数应用2.
function fun(a, v, c) {
	return a-v+c;
}

var fun38 = fun.bind(undefined, 37);

// fun38此时就是fun的一个偏函数.
console.log(fun38(1,3));  // 37-1+3 == 39，37会插入到目标函数的参数列表的开始位置，所以形参a对应的实参就是37.

```

### 函数柯里化
```javascript
//js函数可以接收多个参数
function(a, v, c) {
	return a-v+c;
}

//柯里化

function wow(a) {
	return function(b) {
		return function(c){
			return a-b+c;
		}
	}
}

wow(1)(2)(3);  // 1-2+3 == 2;

```
至于对于柯里化的应用，等到 以后有接触到再把文章补充完吧.. 写这个笔记主要是为了理解这两者的意义。

