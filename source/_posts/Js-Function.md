---
title: JavaScript函数
category: JavaScript
tags: [JavaScript, Function]
date: 2017-07-13
---

因为遇到个Bug发现自己了解的还是太少了,所以就重新回过头去学习了Function. 写了篇记录博客... 
<!-- more -->
## 函数声明式 和 函数表达式
```javascript
console.log(fn(2)); //Error： fn is not a function 
console.log(fnName(2));  // 4
//函数声明式
function fnName(x) {
	 return x*x;
}
//函数表达式
var fn = function(x) {
	return x*x;
} 
```
之前遇到的一个bug...就是因为写了函数表达式而且调用函数写在了函数表达式的前面..所以就报错了..后来了解到，只有函数声明式存在函数声明提升，调用写在函数声明式的前面也不会报错，因为解析器已经先读取了函数声明，把函数声明放在顶部，所以函数调用不会出错。所以以后尽量写要注意把函数的定义要写在函数调用的前面...
函数声明式存在函数声明提升（function declaration hoisting）， 解析器会先读取函数声明，而函数表达式必须等到JS引擎执行到它所在行时才会解析函数表达式。

除了什么时候可以通过变量访问函数这点之外，函数声明式和函数表达式其实是一样的。

## 立即执行函数
立即执行函数，我的理解就是，函数在被解析器解析的时候就已经执行了.
写法：
```javascript
(function(x){
	console.log(x*x*x);
})(2);   // 8

(function(x){
	console.log(x*x*x);
}(2));  //8

!function(x){
	console.log(x*x*x);
}(2);	//8

+function(x){
	console.log(x*x*x);
}(2);	//8

-function(x){
	console.log(x*x*x);
}(2);	//8

var fn = function(x){
	console.log(x*x*x);
}(2);	//8
```

（），！，+，-，=等运算符，都将函数声明转换为函数表达式，消除了JavaScript引擎识别函数表达式和函数声明的歧义，函数表达式可以在后面加括号，并立即执行.

## 函数的方法

### function.prototype.apply()
>**用法:** fun.apply(thisArg, [args]); -----参考 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

```javascript 
function sayDeatil(a, b, c){
	console.log('name:' + this.name + ' ，' + a + ' ' + b + ' ' + c);
}

var a = {
	name: 'apple',
	nums: [4, 3, 10]
}

var o = {
	name: 'orange',
	nums: [2, 4, 8]
}
var name = 'windows name';
var nums = [3, 3, 3];

sayDeatil.apply(a, a.nums); // name:apple, 4 3 10
sayDeatil.apply(o, o.nums); // name:orange, 2 4 8
sayDeatil.apply(this, nums); //name: windows name, 3 3 3
sayDeatil.apply(null, nums); //当前函数处于非严格模式下，thisArg这个参数赋为null或者undefined时会自动指向全局对象（浏览器中就是window对象）

```
fun.apply( )第一个参数传的是 函数体内this的指向（fun函数运行时指定的this值）。 上述例子分别指的是对象a，对象o和window 对象
第二个参数是参数数组或者类数组对象。
### function.prototype.call()
>**用法：** fun.call(thisArg[, arg1[, arg2[, ...]]]) ------参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

call（）和apply（）类似。他们两者传的第一个个参数一样。都是fun函数运行时指定的this值。而call第二个参数往后传的是参数列表.
```javascript 
function sayDeatil(a, b, c){
	console.log('name:' + this.name + ' ，' + a + ' ' + b + ' ' + c);
}
var o = {
	name: 'orange',
	nums: [2, 4, 8]
}

sayDeatil.call(o, 2, 4, 8); // call 和 apply的不同就是apply传的是参数数组或者类数组对象 而call传的是参数列表
```
### 常用用法
1.  **数组之间追加**
```javascript 
var array1 = [1, 2, 3, 4];
var array2 = [5, 6, 7, 8];

Array.prototype.push.apply(array1,array2);  //将array2融合到array1 相当于  array1.push(5, 6, 7, 8);
console.log(array1); // [1, 2, 3, 4, 5, 6, 7, 8];
```
2.  **获取数组中的最大值最小值**
```javascript 
var nums = [1, 2, 3, 5, 20];

var getMax = function(arg) {
	if(arg instanceof Array) {  //当arg是数组时
		let max = Math.max.apply(this, arg);
		console.log(max);
	}
	else { //arg不是数组，而是参数列表时，利用arguments对象
		let slice = Array.prototype.slice;
		let argArray = slice.apply(arguments); //利用Array.prototype.slice.apply() 把类数组对象转为真正的数组.
		let max = Math.max.apply(this, argArray); 
		console.log(max);
	}
}
getMax(nums); // 20
//#  求最小值类似,使用内置函数Math.min

```
3.  **类数组对象转化为真正的数组**
类数组对象最特殊的是function的arguments对象，具有length长度，不能使用pop,push等方法.. 解决办法就是将类数组对象转化
```javascript 
+function fn(a,b,c){
	let args = Array.prototype.slice.call(arguments);
	console.log(args); 
}(1, 2, 3)   // [1, 2, 3]
```

### 进一步提升
```javascript  
//定义个log方法，代理console.log方法
function log(msg) {
	console.log.apply(this,arguments);
};

log('asdasd2',4);  //asdasd2 4
```
### function.prototype.bind()
>**用法:** &nbsp fun.bind(thisArg[, arg1[, arg2[, ...]]]) -----参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
bind( )方法创建一个新的**函数**，当被调用时，将其this关键字设置为提供的值,在调用新函数时，在任何提供之前提供一个给定的参数序列。
```javascript 
var aFun = {
    x: 30,
    getX:function(){
        return this.x;
    }
}

var o = {
    x: 20
}

var newFun = aFun.getX.bind(o);  //
console.log(newFun) // function() { return this.x; }  跟aFun.getX一样的函数体 只不过newFun的this指向是o对象. 
typeof(newFun) // function
newFun();  // 20
// # 修改o对象的x值
o.x = 233;
newFun();  //233
```

### apply、call、bind总结

+  三者都是用来改变函数的this对象的指向的；
+  三者第一个参数都是this要指向的对象，也就是想指定的上下文
+  三者都可以利用后续参数传参
+  **bind是返回一个新函数**，便于稍后调用；apply、call则是立即调用


参考文章
[【优雅代码】深入浅出 妙用Javascript中apply、call、bind](http://www.cnblogs.com/coco1s/p/4833199.html) 
 
[ MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/)

>“对我来说，博客首先是一种知识管理工具，其次才是传播工具。我的技术文章，主要用来整理我还不懂的知识。我只写那些我还没有完全掌握的东西，那些我精通的东西，往往没有动力写。炫耀从来不是我的动机，好奇才是。"    ——阮一峰
>>  希望也成为我写记录博客的动力....



