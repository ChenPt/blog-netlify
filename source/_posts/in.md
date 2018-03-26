---
title: Object.prototype.hasOwnProperty() 与 in操作符的比较

category: JavaScript

tags: Object

date: 2017-07-18 10:00

---



学习整理笔记.

<!-- more -->



### Object.prototype.hasOwnProperty()

>语法:  obj.hasOwnProperty('prop');  // prop: 要检测的属性
>若存在 则返回true, 不存在返回false.  

### in 运算符 
>语法: prop in obj // 存在返回true, 不存在返回false    
```javascript
//如果一个属性是从原型链上继承过来的,in运算符也会返回true
var a = {
  name: 'sd'
}
"toString" in a   //true 
```

因为对象a 继承了Object.prototype对象的所有属性和方法,所以toString存在于a对象中  



```javascript
var b = ['wow', 'my', 'mine'];
0 in b  // true
'wow' in b //false
'length' in b //true； length是一个数组属性
```

### hasOwnProperty()

```javascript
//hasOwnProperty()
var c = {
  name: 'cd',
  sayName: function() {
    return this.name;
  }
}

c.hasOwnProperty("name");  //true
c.hasOwnProperty("toString")  //false 
```



采用字面量法创建对象,c继承了Object原型对象的方法,所以可用hasOwnProperty()方法.
这个方法用来检测一个对象是否含有特定的自身属性.
该方法会忽略掉从原型链上继承到的属性  



```javascript
var d = Object.create(null);
d.name = 'cd';
d.hasOwnProperty('name')  // ERROR: d.hasOwnProperty is not a function(…) 
//解决办法,利用call
Object.prototype.hasOwnProperty.call(d, 'name'); //true
```

因为对象d用Object.create()方法创建,并且指定原型对象为null,所以d没有继承其他方法.只有一个特定的自身属性"name". 所以对象d是没有hasOwnProperty()方法的,所以报错.

解决办法是,利用call改变hasOwnProperty()this的指向, 使其指向对象d 的上下文, 并给hasOwnProperty()传参数 "name". 



### 相同点

两者都可以用来判断对象是否存在某个属性. 

### 不同点
Object.prototype.hasOwnProperty()方法会忽略掉从原型链上继承来的属性,而in运算符不会.