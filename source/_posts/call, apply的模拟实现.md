---
title: call, apply的模拟实现
category: JavaScript
tags: ['重学JS基础']
date: 2022-03-29
---

call, apply用法以及模拟实现

<!-- more -->


## call, apply

> call，apply都是在指定一个this和若干个指定的参数的前提下，执行某个函数或方法

`call`,  `apply` 方法第一个参数都是指定的 `this`

`call` 方法第二个参数开始是参数列表

`apply` 方法第二个参数是参数数组

## 模拟实现call

剖析call具体的作用，相当于把function挂载在指定的对象的一个属性属性上，通过对象属性执行该function，删除对象上的该属性。

```jsx
var obj = { name: 'obj' }
function sayName(str) {
  console.log(this.name, str)
}
// call(), 实现了以下代码同等的作用
obj.sayName = sayName // 挂载属性
obj.sayName('hello world') // 执行方法
delete obj.sayName // 删除对象上该属性
```

### 利用ES6语法

模拟call，还需要处理call的从第二个参数开始的若干参数值。

```jsx
Function.prototype.call2 = function(context, ...args) {
  context.fn = this // 将方法挂载在对象的fn属性上
  const result = context.fn(...args) // 给定指定的参数，执行方法
  delete context.fn // 删除fn属性
  return result // 返回结果
}
```

可以看到，我们用了ES6的语法来获取call的剩余参数，实属不妥，还是用ES3的语法来模拟，所以我们需要从arguments对象中来获取参数值。

```jsx
Function.prototype.call2 = function(context) {
  context.fn = this
  var args = [] // 最终形如 ['arguments[1]', 'argumetns[2]']
  for(let i = 1, len = arguments.length; i < len; i++) {
    args.push('arguments[' + i + ']')
  }
  // 使用eval来执行fn
  var result = eval('context.fn(' + args + ')') // 利用数组与字符类型相加的类型自动转换，数组元素会铺平，以逗号隔开
  // 相当于 context.fn(arguments[1], arguments[2]) 还有后续的参数就不展开了
  delete context.fn
  return result
}
```

call写出来了，那apply其实也简单，处理参数逻辑修改下即可

## 模拟实现apply

```jsx
Function.prototype.apply2 = function (context, arr) {
  context.fn = this;
  var result;
  if (!arr) {
    result = context.fn();
  } else {
    var args = [];
    for (let i = 0, len = arr.length; i < len; i++) {
      args.push('arr[' + i + ']');
    }
    result = eval('context.fn(' + args + ')');
  }

  delete context.fn;
  return result;
};
```

## 总结

在不同的学习阶段，回过头来看这些基础的知识点，会有另外的一种思路/看法。