---
title: 开发一个组件库
category: webpack
tags: [webpack, 插件, npm]
date: 2018-8-02
---

组内负责的几个项目都有一些可复用的公共组件，所以就着手搭建了个公共组件开发的，第一次开发`library`，所以是参考着`iview`的配置来搭建的。记录如何使用`webpack4`搭建一个`library`的脚手架

<!-- more -->

## 前言

使用 webpack4，需要安装 webpack 和 webpack-cli

```shell
yarn add webpack webpack-cli -D
```

然后就是书写配置文件。

webpack4 会另写一篇总结

## library 结构

```
├─build
│      build.js  // 用于执行构建
│      check-versions.js    // vue-cli 留下的，主要就是检查npm版本和node版本
│      webpack.base.conf.js   // 通用配置
│      webpack.dev.conf.js   // 开发环境
│      webpack.dist.prod.conf.js //  用于生成library的代码 -- hbf.min.js
│      webpack.prod.conf.js  // 用于生成example文件的打包代码，这个其实是没有必要的.
│
├─dist
│   └─example // 生成的打包文件夹，因为现在习惯打包后用`anywhere`本地预览下打包后文件的资源加载是否有误
│     hbf.min.js // library 文件
│
├─example
│      App.vue
│      index.html
│      main.js
│
├─lib
│  │  index.js
│  │  README.md
│  │  
│  └─components // 公共组件
│
├─package.json  // 项目包依赖
```

对于需要打包成 library 的配置文件来说，output 选项需要设置 library

```javascript
// webpack.dist.pord.conf.js

output: {
  path: path.resolve(__dirname, '../dist'),
  filename: 'hbf.min.js',
  library: 'hbf',
  libraryTarget: 'umd'
},
```

先来了解下 webpack 编译后的代码。

```javascript
// webpack编译后的代码

;(function(modules) {
  function __webpack_require__(moduleId) {
    var module = {
      i: moduleId, // 模块ID
      l: false,
      exports: {}, // 作为结果返回.
    }
    modules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__
    )
    return module.exports
  }

  return __webpack_require__(0)
})([
  /** 省略了代码， 该数组的每一项代表一个模块，实际是一个函数，接受三个参数，module对象，module.exports对象，__webpack_require__函数 **/
])
```

webpack 编译后的代码的整体结构就是一个简单的`IIFE函数`，接受一个 `modules: Array`参数。

对于模块处理，无论是 `ES Module` 的 `import` 还是 `commonjs` 的 `require` 都转化为`__webpack_require__` 这个函数来引入模块。

`__webpack_require__` 函数，会从 `modules` 数组的第一个元素开始(moduleId 为 0，也就是入口文件)，执行该模块（实为一个函数）的逻辑，利用传入的`module.exports`的数据类型为引用类型`Object`，间接地给`module.exports`添加属性。

`return __webpack_require__(0)`

从入口文件，moduleId 为 0，开始引入依赖模块，最后返回入口模块的 `module.exports`

`library` 可以是字符串，也可以是对象，(对象仅限于 `libraryTarget` 的值为 `umd` 的情况下使用)

```javascript
output: {
  library: {
    root: Hbf,  // 暴露给
    commonjs: hbf. // 入口模块的module.exports 赋值给 exports['hbf']
    commonjs2: hbf,  // 入口模块的module.exports 赋值给 module.exports
  }
}
```

我们在引用别人的库的时候，通常都是可以通过多种方法引入的，比如 `<script>` 标签引入，通过 `commonJS` 引入，通过 `ES6 Module` 引入。

`libraryTarget`设置为`umd`(通用模块规范)的话，则打包后可以通过多种模块加载的方法加载 library，具有高兼容性。

## 依赖问题

如果我们的 library 是基于某某库的基础上开发的，比如说写一个基于`vue`的 UI 组件库，在开发的这个组件库的时候，我们需要引入`vue`，如果使用这个组件库的用户本身就已经引入了`vue`，那么`vue`就会被引入并打包两次，所以我们在开发一个`library`的时候，对于一些所依赖的模块，可以由引入`library`的使用者提供。所以我们需要将依赖的模块在 library 的打包构建中去除。

通过设置 `externals` 达到目的

```javascript
// webpack.dist.pord.conf.js

externals: {
  vue: {
    root: 'Vue',
    commonjs: 'vue',
    commonjs2: 'vue',
    amd: 'vue'
  },
  iview: {
    root: 'iView',
    commonjs: 'iview',
    commonjs2: 'iview',
    amd: 'iview'
  }
},
```

另外在 package.json 加多一个`peerDependencies`字段，作用是约定`library`所依赖的库的版本号，在使用者下载使用`library`的时候，如果所依赖的 iview 和 vue 的版本号不对，就会发出警告。

```json
"peerDependencies": {
  "iview": ">2.0.0",
  "vue": ">2.0.0"
},
```

对于 vue 的插件，需要有一个 install 方法，在暴露含有 install 方法的对象时，使用的是 `module.exports`，引用的时候报错`Uncaught TypeError: Cannot assign to read only property 'exports' of object '#<Object>'`。

因为我测试用的项目关闭了 `babel` 对 `ES Module` 的编译，通常情况下，没有手动关闭的话，`babel` 会将 `ES6 Module` 编译转换成 `commonjs` 规范。所以在关闭了`module`的转换的情况下，由于库的输出使用的是 `commonjs` 规范的 `module.exports`，而引入库使用的是 `import`，所以产生了报错，我将库的导出写成`export`关键字，就没报错了。
我也了解到了现在大多数库，都是用的 `commonjs` 规范，由于 `webpack` 的 `tree-shaking` 只对 `ES Module`起作用。

所以库的模块规范可以使用两种，利用 `package.json`的 `main` 和 `module` 字段分别定义库的两种规范的入口文件。 `main` 使用的是 `commonjs` 规范的文件，而 `module` 是使用 `ES6 module` 的文件。所以采用了`module`规范的库的使用也可以进行利用`webpack` 进行 `tree-shaking`，去除未引用的代码。
