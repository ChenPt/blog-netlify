---
title: 实现EventEmitter
category: JavaScript
date: 2018-3-15
tags: [JavaScript, 模拟实现]
---

# 实现一个EventEmitter (事件触发器)
算是从现在开始自己模仿的第一个轮子把，加油.

要实现一个EventEmitter，核心功能就是实现`on`，`emit`方法。

<!-- more -->
`on`方法用来为一个`事件`绑定一个`监听器`
``` javascript
const a = new EventEmitter()

var listener = function(arg) {
    console.log(arg)
}

//第一个参数为事件名称，string类型
//第二个参数为想要绑定的监听器函数，这个函数在这个事件被触发时执行.
a.on("sayHello", listener)

```
`emit`方法用来触发某个`事件`，并可以为这个事件所绑定的`监听器函数`传入参数。

``` javascript
a.emit("sayHello","hello World")
// “hello World”
```

## 构造函数EventEmitter
``` javascript
//eventEmitter.js

function EventEmitter () {
    this._event = {};
    /** 
     * 第一次设计的结构
     * {
     *     "event1": [fn1,fn2,fn3],
     *     "event2": [fn1,fn2,fn3]
     * }
    **/
    /**第二次设计的结构
     * {
     *      "event1": [{fn:fn1, once: false}]
     * } 
    **/
}
```
this._event对象存储'事件'与'监听器'的关系，因为每一个事件都可以绑定多个监听器，所以采用数组的形式来存储一个监听队列。结构如上代码注释. 

## 实现on方法
``` javascript
//eventEmitter.js

var proto = EventEmitter.prototype

/**
 * 
 * @param {string} eventName 事件名称
 * @param {function} listener 监听器函数
 */
proto.on = function(eventName, listener) {
    let listeners = this._event[eventName] =  this._event[eventName] || [];

    //将传入的listener参数添加到listeners队列。
    listeners.push(listener)

    //返回EventEmitter对象，从而可以进行链式调用
    return this;
}
```
如果传入的这个事件的监听队列存在，则直接将其赋值给`listeners`变量，若不存在，则给`listeners`创建一个新队列（空数组）。

再在`listeners`进行操作，将listener添加到监听队列中。

## 实现emit方法
``` javascript
proto.emit = function(eventName, args) {
    let listeners = this._event[eventName];
    
    //如果emit的事件不存在，则直接返回；
    if(!listeners) return;

    //遍历监听队列，将args传给监听函数并执行监听函数。
    listeners.forEach(listener => {

     //如果args是数组，则转化为参数列表
        if(Array.isArray(args)) {
            listener(...args)
        } 
        else {
            listener(args)
        }
    });

    //返回EventEmitter对象，从而可以进行链式调用
    return this; 
}
```
到这里就完成了一个简单的`EventEmitter`，后面就是进一步的完善了。

## 实现off方法

off方法用来移除某个事件的某个监听函数，需要传入参数为，`事件名`&`监听函数`
``` javascript
proto.off = function(eventName, listener) {
    let listeners = this._event[eventName];

    //如果事件对应的监听队列不存在，则直接返回
    if(!listeners) return;

    //采用filter方法将监听队列中监听函数为listener的给筛选掉.
    listeners = listeners.filter(e => {
        if(e !== listener) return  e;
    })

    return this;
}
```

## 实现once方法

once方法对某个特定事件只执行一遍监听函数，执行完毕就将监听函数从监听队列中移除。

``` javascript
/**
 * 
 * @param {string} eventName 
 * @param {function} listener 
 */
proto.once = function(eventName, listener) {
    return this.on(eventName, listener, true);
}
```
`once`方法内部调用`on`方法，不过传参的时候多传了个`once`参数为ture，所以需要修改`on`方法来配合`once`方法.

**修改后的on方法**

``` javascript
 proto.on = function(eventName, listener, once = false) {
     //once参数默认为false
    //当某个事件监听队列的不存在时，创建一个空队列。存在则直接赋值给listeners
    let listeners = this._event[eventName] =  this._event[eventName] || [];

    //将传入的listener参数添加到listeners队列。
    listeners.push({
        fn: listener,
        once: once
    });

    //返回EventEmitter对象，从而可以进行链式调用
    return this;
}
```

变化的地方就是`on`方法现在接受三个参数，第三个参数为once，默认值为`fasle`。`once`方法调用`on`方法传过来一个once参数为`true`。

现在不直接把监听函数`listener`添加到监听队列中，而是用一个`对象`来代表一个监听函数，对象的fn字段就是监听函数，once字段代表是否只能够执行一次.  
在emit的时候就判断监听队列中的item的once字段是否为true，如果是的话就调用off方法.

## 添加一个验证listener参数的函数
在调用EventEmitter的各个方法时，如果有需要传一个`listener`的参数的话，在内部需要判断这个参数是否为函数，如果不是函数，则抛出一个TypeError.

``` javascript
function isValid(listener) {
    if(typeof listener === 'function') {
        return true
    } else if (typeof listener === 'object' && listener) {
        return isValid(listener.fn)
    } else {
        return false
    }
}
```

在各个方法需要传入listener参数的就可以添加以下代码进行验证

``` javascript 
if(!isValid(listener)) {
    throw new TypeError("listener must be a function!");
}
```

## 实现Allof方法
``` javascript
/**
 * 
 * @param {string} eventName 事件名称，如果事件名称为空则删除
 */
proto.allOf = function(eventName) {
    //如果事件名称为空，则删除所有事件与监听函数的绑定关系。
    //若有指定某个事件，则删除此事件的监听队列，置为空。
    if(!eventName) {
        this._event = {}
    } else if(eventName && this._event[eventName]) {
        this._event[eventName] = []
    }
}
```

暴露EventEmitter对象
``` javascript
// eventEmitter.js

module.exports = EventEmitter
```

## Test.js
``` javascript
const eEmitter = require('./eventEmitter');

let event = new eEmitter();

let fn1 = function(arg) {
    console.log("fn1: ",arg);
}

let fn2 = function(arg) {
    console.log("fn2: ", arg)
}

let fn3 = function(arg) {
    console.log("fn3: ", arg);
}

event.on("event1", fn1)
    .on("event1", fn2)
    .once("event2", fn3)
    .on("event1", fn3)
    .on("event3", fn1)
    .on("event3", fn2)

console.log(event._event);
console.log("--------------------------------")

event.emit("event1", 1)
    .emit("event2", 'once')

console.log("--------------------------------")

event.allOf("event1")
console.log(event._event);


console.log("--------------------------------")
event.off("event3",fn1)
console.log(event._event)


```
## In the End
EventEmitter在Node.js的event模块中，而很多对象都是EventEmitter的实例，他们也就可以使用EventEmitter的方法.  

对于`EventEmitter`来说，最主要的还是`on`和`emit`和`off`方法，其他的方法都是在这几个的基础上扩展出来的.

通过这次实现，自己也学会了如何把一个任务拆解，并逐渐实现每个小任务，再对最后的结果加以适当的完善。

还有就是需要对整个任务的轮廓有个大概的理解。

呃...还是立个flag吧，实现一个Promise，现在只实现了简单的`then`和`resolve`。



