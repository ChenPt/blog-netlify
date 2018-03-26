---
title: JS高级程序设计 6.2 创建对象 的笔记
category: JavaScript
tags: 原型模式
date: 2016-07-26
---
## 原型模式
之前对原型这部分的内容感觉很懵,所以做笔记整理一下思路.


----------


我们创建的每个函数都有一个prototype(原型)属性,这个属性是一个指针,指向一个对象.这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法  所指的这个对象就是原型对象.
prototype 就是通过构造函数创建出来的实例的原型对象,使用原型对象的好处就是可以让所有对象实例共享它所包含的属性和方法,也就是不必在构造函数中定义对象实例的信息,而是将这些信息直接添加道原型对象中.
### 1.理解原型对象 

 - 创建一个新的函数,会默认创建一个prototype属性, 这个属性指向函数的原型对象.
 - 同时,所有原型对象会自动获得一个 constructor (构造函数)属性,这个属性是一个指针,指向prototype属性所在的函数.
 - 创建自定义的构造函数之后,其原型对象默认只会取得constructor属性
 - 使用构造函数构造一个新的对象实例后,该对象实例的内部将包含一个指针(内部属性)[[Prototype]] 指向构造函数的原型对象.

***
### 2.属性屏蔽
先说一下代码读取某个对象的某个属性的流程.

 1. 代码读取某个对象的某个属性,都会执行一次搜索,目标是具有给定名字的属性.
 2. 首先是在对象实例本身开始寻找是否具有那个属性.假如没有
 3. 转而继续搜索对象实例内部指针[[Prototype]]所指向的原型对象
 4. 找到该属性后,返回该属性的值
```javascript
function Person(){
}
Person.prototype.name = "ChenPt";
Person.prototype.age = 29;
Person.prototype.sayName = function(){
    alert(this.name);
};

var person1 = new Person();
var person2 = new Person();
Person1.name = "Wow";
alert(person1.name); // "WOw"来自对象实例
alert(person2.name);// "ChenPt"来自原型
```
由代码读取某个对象的某个属性的搜索过程我们可以知道,我们可以通过对象实例  访问 保存在原型中的值,但是我们不能通过对象实例重写原型中的值.如果我们往对象实例添加一个属性,这个属性已经存在于对象实例的原型中,结果会是,我们成功在对象实例中创建了该属性,该属性会屏蔽原型中的那个属性.
即当为对象实例添加一个属性时,这个属性就会屏蔽原型对象中保存的同名属性.添加这个属性只会阻止我们访问原型中的那个属性,但是不会修改那个属性,所以,叫做属性屏蔽
```javascript
delete.person1.name;
alert(person1.name);  // "ChenPt" 来自原型
```
可以删除对象实例中的属性,删除之后就恢复了对原型中name属性的连接
***
既然实例中和原型对象中都可以有同名属性,那我们可以 用hasOwnProperty()方法检测一个属性是存在于实例中还是存在于原型对象中. 属性存在于实例中时,hasOwnProperty()返回 true.
***
#### in操作符
 只要通过对象能够访问道属性就返回 true
 ***

     hasPrototypeProperty(person, "name")    // 如果实例拥有name 属性 则返回 false.
                                             // 如果实例没有name 属性 则返回 true.
***
### 3.简单的原型语法 
 (包含所有属性和方法的对象字面量来重写整个原型对象)
```javascript
function Person(){
}
Person.prototype = {
    name : "ChenPt",
    age : 19,
    sayName : function(){
        alert(this.name);
    }
}
```

注意 此时原型对象的constructor 属性不再指向Person了.这里我们本质上是完全重写了默认的prototype对象,因此 constructor属性也变成了新对象的constructor属性(指向Object构造函数) 不再指向Person函数.
你可以设置constructor的值为适当的值
```javascript
//重设构造函数
Object.defineProperty(Person.prototype, "constructor",{
    enumerable: false,
    value: Person
});
```
#### 原型的动态性
```javascript
function Person(){
}

var friend = new Person();

Person.prototype = {
    constructor: Person,
    name: "ChenPt",
    age: 19,
    sayName : function(){
        alert(this.name);
    }
};
friend.sayName();  //报错
```
我们先建了一个Person的一个实例,然后重新写了其原型对象. 
 `friend.sayName();`报错,因为friend指向的原型不包含sayName这属性.
friend指向的原型只有默认的constructor属性(指向Person这个构造函数).
这就说明了,重写原型对象切断了现有原型与之前已经存在的对象实例之间的联系,这些对象实例引用的仍然是最初的原型.
#### 原型模式的问题
**原型中所有属性都是被很多实例共享的**
对于包含**引用类型值**的属性来说,问题就会变得很突出了.什么叫引用类型值呢,我查阅的资料如下
**引用类型：对象、数组、函数**
我们来举个栗子(参照书的P158)
```javascript
function Person () {
}

Person.prototype = {
    constructor : Person,
    name : "ChenPt",
    age : 19,
    friends : ["Aaa","Bbb"],  //friends是包含引用类型值(数组)的属性
    sayName : function () {
        alert(this.name);
    }
};

var person1 = new Person();
var person2 = new Person();

person1.friends.push("Ccc");    //往数组再添加一个字符串

alert(person1.friends);     //"Aaa","Bbb","Ccc"  实例1改变了
alert(person2.friends);     //"Aaa","Bbb","Ccc"  实例2的变化是跟实例1一样的,他们共享一个数组
alert(person1.friends === person2.friends); //  true
```
所以说,在含有引用类型值的属性来说,修改此属性的值后,所有实例的此属性值也会跟着变化.
实例一般是要有属于自己的全部属性的.所以这时候就有了 组合使用构造函数模式和原型模式
***
## 组合使用构造模式和原型模式
通俗点讲,构造函数用于定义实例属性,而原型模式用于定义方法和共享的属性.
```javascript
function Person (name,age) {
    this.name = name;
    this.age = age;
    this.friends = ["Zzz","Yyy"];
}

Person.prototype = {
    constructor : Person,
    sayName : function () {
        alert(this.name)
    }
};

var person1 = new Person("Aaa",19);
var person2 = new Person("BBb",20);

person1.friends.push("Www");

alert(person1.friends);     // ["Zzz", "Yyy", "Www"]
alert(person2.friends);     // ["Zzz", "Yyy"]  
person1.sayName();          // Aaa
```
第17行代码修改了实例一的friends属性,往其中添加了一个新的字符串,但是并不会影响到对象实例 person2.friends,因为它们引用了不同的数组.
构造函数与原型混成的模式,可以说,`这是用来**定义引用类型**的一种默认模式.
***
**未完待续..........**
