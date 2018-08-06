---
title: CommonJS 模块规范与 ES Module 的差异
category: ECMAScript
tags: [Module, CommonJS]
date: 2018-08-06
---

在 ES Module 还未全面支持的时候，我们写 ES Module 的语法，`import`，`export`需要通过`babel`来编译，转化成`CommonJS`规范。现在由于`ES Module`普及度已经够广了。在复习的过程中也学习到了两者的区别。

<!-- more -->

## Babel 转换 ES Module 为 CommonJS 规范

```javascript
// a.js
var a = 2
var b = '123'
var obj = {
  test: '12',
}
export default obj
export { a, b }
```

Babel 会将其转换为

```javascript
exports.default = {
  test: '12'
}
exports.a = 2
exports.b = '123'
export._esModule = true // 有一个标记，说明该模块是由ES Module转换的
```

可以手动设置关闭 babel 对`ES Module`的转换，设置`modules`的值为`false`

```json
"presets": [
    [
      "env",
      {
        "modules": false
      }
  ]
```

## Webpack 对`ES Module`的转换

不仅仅可以通过`Babel`可以转换`ES Module`，实际上 Webpack 在处理文件的时候，也会对模块语法进行一定的处理。最终使用`webpack`内部定义的`__webpack_require__`函数来引入模块，从入口文件开始引入的模块都存放到一个叫`module`对象的`exports`属性里，`exports`也是一个对象。
格式如下，每引入一个模块都会给`exports`添加一个属性，最后`__webpack_require__`函数返回该`module.exports`对象。

```javascript
var module = {
  i: moduleId, // 模块ID
  l: false,
  exports: {}, // 作为结果返回.
}
```

## 两者的区别

### ES Module

`import` 是编译阶段执行的，所以可以进行静态分析。
import 语句不能够使用变量，表达式，因为编译阶段这些都还未存在。
`export` 导出的值其实是一个`值的引用`，可以理解为存放着具体值的`地址`。也可以理解为`符号链接`，[是一类特殊的文件， 其包含有一条以绝对路径或者相对路径的形式指向其它文件或者目录的引用](https://zh.wikipedia.org/wiki/%E7%AC%A6%E5%8F%B7%E9%93%BE%E6%8E%A5)

对于 ES 模块导出的值，举个例子

```javascript
// a.js
export var a = 1
export function caculate() {
  a++
}

// b.js
import { a, caculate } from 'a.js'

console.log(a) // 1
caculate()
console.log(a) // 2
```

引入 a.js 的变量 a 和 caculate，通过 caculate 修改 a 的值，b.js 引入的变量`a`也发生了改变，说明了，`import`导入的只是一个`值的引用`。当该值发生变化时，会同步该变化。

### CommonJS

`require`是在运行时加载的，只有在运行的时候才能够得到 CommonJS 模块导出的对象 exports，可以使用变量，表达式，动态加载模块。
`module.exports`导出的其实是一个`值的拷贝`，引入模块后修改其导出的值，不会影响模块原本的值。
举个例子，改写上面的例子

```javascript
var a = 1
var caculate = () => {
  a++
}
module.exports = { a, caculate }
// b.js
var { a, caculate } = require('./a.js')

console.log(a) // 1
caculate()
console.log(a) // 1
```

但是需要记住的是，一个模块暴露提供的值，不能通过手动赋值的方式改变其的值，会报错`Assignment to constant variable`，只有其内部模块的方法可以修改值，这也是一种特殊的情况，其他情况下我们都可以将一个模块的暴露的值当成`const`常量，其引入后不可以被修改。

## Node.js

现在 Node 对 ES Module 的支持还不是很理想，还在测试阶段。所以在`Node`环境下还是继续使用`CommonJS`模块规范。
