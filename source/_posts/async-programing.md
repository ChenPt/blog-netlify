---
title: 关于异步
category: 异步编程
date: 2018-07-23
tags: [JavaScript, 异步, async/await, Promise]
---

# 异步编程

发现自己还没写过关于异步的总结，由于在实际开发过程中，使用的异步操作是比较多的，所以回过头来总结一下。

<!-- more -->


## 传统的callback

众所周知，JS是单线程的，如果都采用同步来写I/O请求的话，因为I/O所花费的时间会比较长，所以，有可能阻塞网页，所以都是用异步来写I/O，请求I/O后这个调用会立刻返回，但是没有返回结果，当I/O请求结束后，会通过回调函数来通知调用方。

JS的异步编程，依靠的是`event-loop`，即事件循环。

参考下面这个视频

[youtube](https://www.youtube.com/watch?v=8aGhZQkoFbQ&t=940s)

![](https://ws1.sinaimg.cn/large/ad9f1193gy1ftjzbb1fy7j20na0d7jts.jpg)


举个简单的回调函数的例子

window对象监听`load`事件，当事件发生，执行回调函数，打印文字

``` javascript
window.addEventListener('load', function() {
  console.log('window is loaded')
})
```

传统的Ajax也是使用回调函数的一个例子。

传说中的`callback hell`，就是说回调函数里面再嵌套回调函数，当嵌套的层数多了，代码的可读性和可维护性也就降低了。

## Promise

为了解决这种回调函数存在的问题，社区就根据`Promise/A+规范`提出并实现了`Promise`。

`Promise`，字面意思，承诺，可以理解为一个容器，里面承载着未来必定会发生的某个动作的`结果`，无论成功或者失败。

对于我自己来说，ES6里用得比较多的就是Promise了。

Promise的写法，感官上就是把嵌套回调函数由横向变成纵向。

``` javascript
// 一个HTTP请求
axios.get('/getUser',{
  params: 'xx'
}).then(res => {
  // ...
  // ...
}).then(res => {
  // ...
}).catch(err => {
  // ...
})
```

## Generator

构造器函数也是ES6提出的，Generator函数可以看成一个状态机，内部的`yield`表达式定义不同的状态。Generator总会返回一个遍历器对象。使用`next`方法转换成下一个状态。`next`方法也可以看成将执行权移交给下一个协程。

> 如果将 Generator 函数当作协程，完全可以将多个需要互相协作的任务写成 Generator 函数，它们之间使用yield表示式交换控制权。 (阮一峰的《ECMAScript 6 入门》)

HTTP请求的`generator函数`写法

``` javascript
function* getUser() {
  try {
    var res = yield fetch('/getUser', {
      method: 'GET'
    })
    var data = yield caculate(res)
    return data.rows
  } catch(err) {
    // handle err
  }
}

// 执行getUser
var gen = getUser()  // gen为遍历器对象

gen.next() // {value: res, done: false}
gen.next() // {value: data, done: false}
gen.next() // {value: undefined, done: true}

```
因为generator函数不会自动执行，所以需要写执行函数，或者是利用已有的工具，如`co模块`，传入co函数的是一个`generator函数`，而不是`遍历器对象`

``` javascript
var co = require('co')

co(getUser)
```

co模块的原理，就是自动归还`执行权`，前面手动执行generator函数，都是调用`next`方法来归还执行权，有两种方法来让某个异步任务完成后自动归还执行权。一是利用`Thunk函数`，二是利用`Promise`

## Thunk函数

学习编译原理的时候有接触到`求值策略`，参数的`传值调用`和`传名调用`，前者的实现比较简单

``` javascript
let x = 1
let fun = (val) => {
  return val * 2
}

fun(x + 1)
```

当使用`传值调用`时，等价于`fun(2)`，也就是先计算表达式`x+1`的值，再作为函数参数传进去。

而`传名调用`，将表达式直接作为函数传给函数，当使用时，再进行求值。

`Thunk函数`就是使用`传名调用`时，计算参数值的`临时函数`

``` javascript
// 接上面的例子，等价于

// 临时函数
let thunk = () => {
  return x + 1
}

let fun = (thunk) => {
  return thunk() * 2
}
```

会将传进来的参数放在一个临时函数里，需要使用参数时再调用临时函数。

实际上，在JS中，利用thunk函数来将多参变成单参函数

``` javascript
var thunk = (eventName) => {
  return function(cb) {
    window.addEventListener(eventName, cb)
  }
}

var addEvent = thunk('load')

addEvent(console.log)
```

通用的thunk转换函数

``` javascript
var thunk = (fn) => {
  return function (...args) {
    return function (cb) {
      return fn.call(this, ...args, cb)
    }
  }
}

var readFileThunk = thunk(fs.readFile)

readFileThunk('aaa.txt')(console.log) // 等价于 window.addEventListener('error', console.error)
```

**thunk函数的第二个括号里的参数是`callback函数`**

`yield`后面需要跟着thunk函数。

``` javascript
function* g() {
  var a = yield readFileThunk('1.txt')  // 一个遍历器对象，value值为函数，接受的参数为一个callback函数
  var b = yield readFileThunk('2.js')
}
```

``` javascript
function autoRun (fn) {
  var gen = fn()

  function next () {
    var result = gen.next()
    if(result.done) return
    result.value(next)
  }

  next()
}

autoRun(g)
```

## Promise版本的自动执行

yield后面需要跟着一个可以返回`Promise`的操作

转化为Promise的方法

``` javascript
function toPromise(fn) {
  return function (...args) {
    return new Promise((resolve, reject) => {
      var result = fn.call(this, ...args)

      //根据是否成功来决定执行resolve或者reject
      if(result.type === 'success') {
        resolve(result)
      } else {
        reject(result)
      }
    })
  }
}
```

``` javascript
function runOfPromise (fn) {
  var gen = fn()
  
  function next () {
    var result = gen.next() // result的value为promise类型
    if(result.done) return
    result.value.then(next) // 利用then方法，前一个异步任务完成后自动执行下一个异步任务
  }
  next()
}

```

## Async/Await 大法

async/await 可以说是，真正的解决了异步的种种问题，而且代码可读性高，写法跟同步代码差不多。

`async/await` 可以看成`generator函数`的语法糖，实际上就是generator的进一步封装。

`await`的作用跟`yield`类似。 跟generator函数不同的是，async函数会自动执行，不需要利用`co`等工具。

async/await 进行HTTP请求的简单例子

``` javascript
async function getUser(url) {
  try {
    let res = await fetch(url) // await后面必须跟一个Promise对象

    return Promise.resolve(res) //async函数返回一个Promise对象，无论是返回什么值，都会自动转化为Promise对象
  } catch (err) {
    // handle error
  }
}

//只有async内部的所有await后面的promise执行完毕，才会执行then语句的回调函数.
getUser('/api/getUser').then(res => {
  console.log(res)
})
```

在await外层写`try catch`来捕获await后面的`Promise`发生的错误

对于我个人来说，目前使用比较多的还是Promise和async/await，基本...怎么没使用的就是generator函数了

## 最后写个经常出现的例子

多个异步操作顺序执行。如，按顺序请求一组资源。

### 构造Promise调用链

``` javascript
var urls = ['/aaa', '/bbb', '/ccc']
var getByOrder = (urls) => {
  var allPromises = urls.map(url => {
    return fetch(url)
  })

  allPromises.reduce((chain, promise) => {
    return chain.then(() => promise).then(res => console.log)
  }, Promise.resolve())
}

// 等价于
Promise.resolve(() => fetch('/aaa'))
  .then(res => console.log)
  .then(() => fetch('/bbb'))
  .then(res => console.log)
  .then(() => fetch('/ccc'))
  .then(res => console.log)
  .catch(err => {
    // handle error
  })
```


### 使用async/await

``` javascript
var getByOrder = async (urls) => {

  //并发读取url对应的资源，外部是并发的，并不受async的影响。整体请求所花费的时间取决于最多的那个请求
  var allPromises = urls.map(async (url) => {
    var res = await fetch(url)
    return Promise.resolve(res)
  })

  for(let item of allPromises) {
    let result = await item
    console.log(result) //按顺序输出
  }
  
}
```