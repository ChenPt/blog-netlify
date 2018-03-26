---
title: JavaScript 继承 的学习笔记
date: 2016-07-26 01:20:46
tags: 继承
category: JavaScript
---
>我是结合慕课和JS高级程序设计这本书去学习继承这部分的内容的。在慕课《JavaScript 深入浅出》那一教程的OOP（Object Oriented Programming 面向对象程序设计）章节。
## 原型链
首先我们了解原型链。在ECMAScript中描述了原型链的概念，并将原型链作为实现继承的主要方法。
```javascript
function foo() {
}
foo.prototype.z = 3;
var obj = new foo();
obj.x = 1;
obj.y = 2;
```
![image](http://note.youdao.com/yws/res/342/WEBRESOURCE0ae9a61d44df15812fcf1ae06d7dc33b)
---
这边我们创建一个 foo构造函数，并创建了一个对象实例obj，对象obj的内部有一内部属性[[prototype]] （跟图中的[[proto]]是同一个东西） ，该属性是一个指针，因为是用new命令实例化对象，我们来回顾下new命令在使用时进行的操作.
1. 创建一个新的对象
2. 为这个新建对象进行[[prototype]]连接
3. 将this值绑定到这个新建对象上
4. 如果函数没有返回其他对象就返回这个对象


执行new命令操作的第2步中，将对象obj的[[prototype]]属性，（它是一个指针） 指向到其对应的原型对象 foo.prototype

foo.prototype 这个原型对象其内部也有一个[[prototype]]属性，指向Object.prototype   
我们可以用isPrototypeOf()方法来确定对象之间是否存在这种关系

```javascript
console.log(foo.prototype.isPrototypeOf(obj));               // true
console.log(Object.prototype.isPrototypeOf(foo.prototype));  // true
```
我们用原型对象的isPrototypeOf（ ）方法测试了obj和foo.prototype，
对象实例obj内部有一个指针指向foo.prototype 所以返回了true
因为foo.prototype内部有一个指针指向Object.prototype 所以返回了true

Object.prototype 指向null，这条原型链的尽头就是Object.prototype
所以上图就是一个简单的完整的原型链~  明确了原型链是什么之后，我们就来了解如何利用原型链实现继承。

---
## 实现继承
>继承的本质在于重写原型对象，往往通过重写构造函数的原型对象去实现继承
### 实现继承的方法
#### 利用原型链
---
```javascript
//继承了Human  以下两种方式都可以实现继承
Person.prototype = new Human();  //新建一个Human的实例  ，并且Human的实例也指向了Human.prototype  。  所以Person.prototype也指向了Human.prototype
Person.prototype = Object.creat(Human.prototype);  //较为理想的方法，创建一个空对象，并且规定此新建对象指向的对象原型为Human.prototype
```
```javascript
Object.create(proto, [ propertiesObject ]）  //语法
```

Object.create() 方法创建一个拥有指定原型和若干个指定属性的对象。
**参数**
proto ： 一个对象，作为新建对象的原型
propertiesObject ： 可选。该参数对象是一组属性与值，
举个例子
```javascript
function Human() {
}
Human.prototype = {
    constructor: Human,
    a: 99,
    b: 100
}
function Person() {
    this.type = false;
}
Person.prototype = new Human();   //Person 继承了Human
Person.prototype.getValue = function () {
    return this.a;
}
var person1 = new Person();
var person2 = new Person();
console.log(person1.a);
console.log(person2.getValue());
//首先从person2本身需找此方法，找不到再从person2的原型Person.prototype查找getValue方法，
//转而寻找Person.prototype所继承的Human原型对象，找到并引用。
```

利用原型链的确可以，但是我的上一篇笔记有说过，原型模式的问题是当构造函数含有引用类型值（如数组）的属性时，修改这种属性的值，所有实例共享属性，所有实例也会发生变化。这显然不是我们想要的





