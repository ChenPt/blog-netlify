---
title: JavaScript内置对象
tag: [JavaScript, Object]
date: 2017-07-18
category: JavaScript
---
## JavaScript内置对象

第二周周一...JavaScript内置对象

<!-- more -->


### Glboal对象 (全局对象)
它是一个特殊的对象,实际上,所有在全局作用域中定义的属性和函数,都是全局对象的属性和方法.  
在JavaScript中,全局对象就是window对象.
全局对象的方法:  

+ encodeURI() //对字符串进行编码形成可用的URI(通用资源标识符)
+ encodeURIComponent()  //将字符串编码为URI组件,与encodeURI的区别就是它的参数是URI的一部分(比如协议\主机名\路径\查询字符串) 它也会把不属于URI的特殊字符进行编码(如冒号,正斜杆,问号,#号),而encodeURI()不会.
+ decodeURI()
+ decodeURIComponent()
+ escape()  //将字符串进行编码
+ eval() //计算Js字符串,并把它作为脚本代码来执行
+ isFinite()  //判断某个值是否为有穷数
+ isNaN()      //is not a number  判断检查参数是否为非数字值.
+ Number()      //将对象的值转化为函数,如果无法转换则返回NaN
+ parseInt()    //将字符串转换为整数
+ parseFloat()  //将字符串转换为浮点数
+ String()      //将对象的值转化为字符串
+ unescape()    //将escape()编的码转化为字符串



+ NaN       // Not-A-Number
+ Infinity   //全局属性,表示无穷

## 引用类型
在ECMAScript中,引用类型是一种数据结构,用于将数据和功能组织在一起.引用类型有时候也被称为**对象定义**,因为它们描述的是一类对象所具有的属性和方法.  

**对象是某个特定引用类型的实例**,新对象是使用new操作符加一个构造函数生成的.

``` javascript 
var person = new Object(); 
```

这行代码创建一个 Object()类型的新实例, 保存到person变量中.  
构造函数Object为新对象定义了默认的属性和方法.
### Object 类型
#### 创建Object类型

创建Object类型有两种方法  
1.使用new操作符
``` javascript
var person = new Object();
```
2.对象字面量法
``` javascript 
var apple = {
    price: 100,
    color: 'red',
    from: 'GuangZhou',
    isFruit: true,
    to where: 'BeiJing'
}; 
```

#### 访问对象属性
``` javascript
apple.price;  //100
apple["price"]; //100  

var fruitColor = "color";  // 使用方括号语法可以通过变量来访问对象属性

apple[fruitColor]; // 'red'

//to where 含有空格,不能使用"."来访问,只能用方括号语法.  

apple["to where"]; // "BeiJing"
```

### Array 类型
#### 创建数组类型
**ECMAScript数组的每一项可以保存任何类型的数据**  
1.采用Array构造函数
``` javascript
var array = new Array();
// var array = Array();
array[0] = 1;
array[1] = 2;
// array = [1, 2];
```
2.数组字面量法
``` javascript
var a = [1, 2, 3, 5, 'Fuck'];
```
数组的length属性不是只读的,通过改变length属性的值可以从数组的末尾删除元素或者添加新元素.
``` javascript
var a = [1, 2, 3];
a.length = 2;
console.log(a); // [1, 2]
a.length = 3;
console.log(a[2]);  //undefined
a[a.length] = 4;
console.log(a) // [1, 2, undefined, 4];
```
#### 讲几个Array的方法
##### slice & splice
``` javascript
//Array.prototype.slice(beginIndex, endIndex); 不包括结束

var a = [1, 2, 3, 4];
a.slice(0,2);  // [1, 2]  把序号为0, 1 的值取出来,原数组不变
console.log(a)  // [1, 2, 3, 4];

//Array.prototype.splice(start, deleteCount)
//Array.prototype.splice(start, deleteCount,item1,....);
a.splice(2, 1);  // [3]
console.log(a); // [1, 2, 4]

a.splice(3, 0, 5); // [1, 2, 4, 5]  deleteCount为0表示不删除元素,即添加元素
```
##### pop & push & shift & unshift
``` javascript
var a = [1, 2, 3, 4];
a.push(5, 6)  // 返回a.length  == 6
console.log(a); // [1, 2, 3, 4, 5, 6];

a.pop(); // 数组的最后一项 6
console.log(a) // [1, 2, 3, 4, 5];

a.shift(); // 数组的第一项 1
console.log(a); [2, 3, 4, 5]

a.unshift(0, 1);  //返回a.length == 10
console.log(a); //[0, 1, 2, 3, 4, 5];
```
pop() & push() 可实现堆栈(LIFO).  
shift() & push() 可实现队列(FIFO).  
这四个方法都会改变数组的长度.

##### reduce() & map() & filter
reduce() 方法对累加器和数组中的每个元素 (从左到右)应用一个函数，将其减少为**单个值**。**不会改变原数组**  

>array.reduce(function(accumulator, currentValue, currentIndex, array), initialValue)


``` javascript
var a = [1, 2, 3, 4, 5];

a.reduce((x, y,idx, its) => {
    return x + y;
}, 0)   // 15
```
``` javascript
# map 
var a = [1, 2, 3, 4, 5];

a.map((item, idx, array) => {
    return item * 2;
})   // [2, 4, 6, 8, 10];

```

```javascript
var a = [3, 4, 5, 6, 7, 8, 10]

function islessThan(value) {
    return  value < 4;
}

a.filter(islessThan) 



```



### Date 类型 
```
var date = new Date();
console.log(date); //Mon Jul 17 2017 16:20:03 GMT+0800 (CST)
date.getFullYear(); //2017
```

### Function 类型 
http://chenpt.cc/2017/07/13/Js-Function/
### 基本包装类型(String,Number,Boolean)

#### String

1.  type1:"wow"
2.  type2:\`hello world\`  



``` javascript
var str1 = "wow"  + " fuck";    // "wow fuck"
```


### RegExp 构造函数创建了一个正则表达式对象RegExp
RegExp 构造函数创建了一个正则表达式对象  
三种方法: 字面量, 构造函数, 工厂符号
```javascript
/pattern/flags
new RegExp(pattern [, flags])
RegExp(pattern, [,flags]);

//pattern 正则表达式的文本
// flags 
// g: 找到所有匹配,而不是在第一个匹配后停止
```

``` javascript
var pattern = /*.com/g
var testStr = 'sdsdsdsada.com';
pattern.exec(testStr)
```







