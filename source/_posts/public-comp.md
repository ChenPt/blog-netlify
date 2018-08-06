---
title: 记搭建一个Vue组件库
category: webpack
tags: [webpack, library, npm]
date: 2018-8-02
---

组内负责的几个项目都有一些一样的公共组件，所以就着手搭建了个公共组件开发脚手架，第一次开发 library，所以是参考着 [iview](https://github.com/iview/iview) 的配置来搭建的。记录如何使用`webpack4`搭建一个`library`的脚手架

<!-- more -->

## 前言

使用 webpack4，需要安装 webpack 和 webpack-cli

```shell
yarn add webpack webpack-cli -D
```

然后就是书写配置文件。

webpack4 会另写一篇总结

## library 结构

我写的 library 的目录结构如下，仅供参考，主要是模仿 `iview` 的结构，其中部分配置参考了 `vue-cli`的 webpack 配置文件。

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
│   └─example // example生成的打包文件夹，因为现在习惯打包后用`anywhere`本地预览下打包后文件的资源加载是否有误
│     hbf.min.js // library 文件
│
├─example
│      App.vue
│      index.html
│      main.js
│
├─lib
│  │  index.js  // 全量引入公共组件，并暴露出来，包含install方法可供vue引入使用该插件
│  │  README.md
│  │  
│  └─components // 公共组件
│
├─package.json  // 项目包依赖
```

## 经过 webpack 编译后的代码

为了好理解 library，先来了解下 webpack 编译后的代码。

```javascript
// webpack编译后的代码

/*
* @param {Array} modules
*/
;(function(modules) {
  function __webpack_require__(moduleId) {
    var module = {
      i: moduleId, // 模块ID
      l: false,
      exports: {}, // 作为结果返回.
    }
    // 调用modules数组的某个元素（类型为函数）
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

webpack 编译后的代码的整体结构就是一个`IIFE函数`，接受一个 `modules: Array`参数。

对于模块处理，无论是 `ES Module` 的 `import` 还是 `commonjs` 的 `require` 都转化为`__webpack_require__` 这个函数来引入模块。

`__webpack_require__` 函数，会从 `modules` 数组的第一个元素开始(moduleId 为 0，也就是入口文件)，执行该模块（实为一个函数）的逻辑，利用传入的`module.exports`的数据类型为引用类型`Object`，间接地给`module.exports`添加属性。

`return __webpack_require__(0)`

从入口文件开始，逐个引入依赖模块，最后返回入口模块的 `module.exports`

此时这个编译后的 js 文件，是无法被其他模块所引用的，只在当前作用域内有效， `webpack` 就提供了创建 library 的方式，就是在`output`里定义`library` 和 `libraryTarget`。使得构建完的 js 可以供其他模块引入使用。

对于作为一个 library 使用的项目来说，output 选项需要设置 library

```javascript
// webpack.dist.pord.conf.js

output: {
  path: path.resolve(__dirname, '../dist'),
  publicPath: '/dist/',
  filename: 'hbf.min.js',
  library: 'hbf',
  libraryTarget: 'umd'
},
```

`library` 可以是字符串，也可以是对象，(对象仅限于 `libraryTarget` 的值为 `umd` 的情况下使用)

```javascript
output: {
  library: {
    root:'Hbf',  // 暴露给全局变量，window.Hbf进行调用
    commonjs: 'hbf-public-components'
  },
  libraryTarget: 'umd'
}
```

`commonjs` 和 `commonjs2` 的区别。

`commonjs` 规范就是定义了一个 `exports` 对象，而 `nodejs` 在实现的时候，在 `commonjs` 规范的前提下做了一些扩展，定义了 `module.exports` ，从而也叫这种为 `commonjs2` 规范。

我们在引用别人的库的时候，通常都是可以通过多种方法引入的，比如 `<script>` 标签引入，通过 `commonJS`模块 引入，通过 `ES6 Module` 引入。

`libraryTarget`设置为`umd`(通用模块规范)的话，则打包后可以通过多种模块加载的方法加载 library，具有高兼容性。

## library 的依赖问题

如果我们的 library 是基于某某库的基础上开发的，比如说写一个基于`vue`的 UI 组件库，在开发的这个组件库的时候，我们需要引入`vue`，如果使用这个组件库的用户本身就已经引入了`vue`，那么`vue`就会被引入并打包两次，所以我们在开发一个`library`的时候，对于一些所依赖的模块，可以由引入`library`的使用者提供。所以我们需要将依赖的模块在 library 的打包构建中去除。

`externals` 的作用，防止将某些 `import` 的包打包到 bundle 中，而是在运行时再去外部获取这些扩展依赖。

通过设置 `externals`，从输出的 `bundle` 中排除 `vue` 和 `iview`。

这些外部依赖可能是以下的任何一种形式。

- `root` 全局变量访问
- `commonjs` 作为一个`commonjs`模块引入
- `commonjs2` 与`commonjs`类似，不过导出的是 `module.exports`
- `amd` 使用`amd`模块规范引入

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

另外在 package.json 加多一个`peerDependencies`字段，作用是约定`library`所依赖的库的版本号，在使用者下载使用`library`的时候，如果所依赖的 `iview` 和 `vue` 的版本号不对，就会发出警告。

```json
"peerDependencies": {
  "iview": ">2.0.0",
  "vue": ">2.0.0"
},
```

对于这两个依赖，写到开发环境依赖中 ，不然安装时，会在库的目录下安装`vue` 和 `iview`，这也不符合让 `library` 的引用者提供 `library` 的依赖这个想法。

## vue 插件库/ 组件库

对于 vue 的插件库 / 组件库来说，如果想要全局引入的话，需要有一个`install`方法。install 内部逻辑就是通过参数传进来的`vue`对象，注册所有组件。然后最后将所有公共组件连同`install`方法组成一个新对象暴露出去。

```javascript
// 引入公共组件
import publicMenu from './components/public-menu'
import tablePage from './components/table-page'
import sliderCustom from './components/slider-custom'

const components = {
  publicMenu,
  tablePage,
  sliderCustom,
}
const Hbf = Object.assign({}, components)
const install = function(Vue, opts) {
  if (install.installed) return

  Object.keys(components).forEach(component => {
    Vue.component(component, component)
  })
}

// 用于script标签引入
if (typeof window !== 'undefined' && window.Vue) {
  install(window.Vue)
}

// 将install方法赋给Hbf对象
Hbf.install = install

// 输出default变量，用于全量引入，也可以在引入的时候选择使用 * 来全量引入
export default Hbf
// 输出各个组件，用于按需引入
export { publicMenu, tablePage, sliderCustom }
```

在暴露含有 install 方法的对象时，一开始使用的是 `module.exports`，引用 library 的时候报错`Uncaught TypeError: Cannot assign to read only property 'exports' of object '#<Object>'`。

![](https://ws1.sinaimg.cn/large/ad9f1193gy1ftwbbi4fb0j20hm0903yv.jpg)

因为我测试用的项目关闭了 `babel` 对 `ES Module` 的编译，通常情况下，没有手动关闭的话，`babel` 会将 `ES6 Module` 编译转换成 `commonjs` 规范。所以在关闭了`module`的转换的情况下，由于库的输出使用的是 `commonjs` 规范的 `module.exports`，而引入库使用的是 ES6 模块规范的`import`关键字，所以产生了报错，我将库的导出写成`export`关键字，就没报错了。

以后书写某一模块的导入导出，还是使用同一种规范的，不要混用。

我也了解到了现在大多数库，都是用的 `commonjs` 规范，由于 `webpack` 的 `tree-shaking` 只对 `ES Module`起作用。而`webpack` 的 `tree-shaking` 实际上是由`Uglylify`来实现的。

所以库的模块规范可以使用两种，利用 `package.json`的 `main` 和 `module` 字段分别定义库的两种模块规范的入口文件。 `main` 使用的是 `commonjs` 规范语法书写的文件，而 `module` 是使用 `ES6 module` 语法书写的文件，**`module`字段目前还是一个提案**。所以采用了 `ES2015` 模块语法的库的，当我们只使用到 `library`的部分代码，则可以利用`webpack` 进行 `tree-shaking`，去除未引用的代码，减少打包文件体积。

## 发布 npm 包

首先就是需要注册一个 npm 账号。

如果之前使用的是淘宝镜像的话，需要先切回 npm 官方源。不然是发不了包的。

打开命令行

切换官方源 `npm config set registry https://registry.npmjs.org/`

执行`npm login`，然后输入你的账号信息。

可以配置`.npmignore` 忽略一些不需要上传的文件，写法跟`.gitignore` 相同。

需要保证项目有正确的`package.json`文件和`README.md`文件

然后执行`npm publish`进行发包。

发完包就可以切回淘宝镜像源

`npm config set registry https://registry.npm.taobao.org`

## 引用组件库

### Script 标签引入

一开始是将 JS 文件放在本地测试，发现在`HTML`文件的第一行就报错，`Unexpected token <`，[StackOverflow](https://stackoverflow.com/questions/31529446/unexpected-token-in-first-line-of-html)说的原因是引入路径不正确，所以我就把 JS 文件放到 CDN 上了，

```html
<script src="http://osuuzm0m8.bkt.clouddn.com/hbf.min.js"></script>
<script>  
  console.log(window.Hbf) // 会看到你导出的对象
</script>
```

输出

![](https://ws1.sinaimg.cn/large/ad9f1193gy1fu05614vx2j20i706njru.jpg)

### 全量引用

可以像使用其他 vue 插件库/组件库一样使用。

```javascript
import hbf from 'hbf-public-components'

// 使用use方法触发hbf的intall方法，注册全部组件
Vue.use(hbf)
```

如果是没有导出`default`变量，则使用另外一种方式全量引入

```javascript
import * as hbf from 'hbf-public-components'
```

### 按需引用

```javascript
import { publicMenu } from 'hbf-public-components'
```

按需引用，如果 library 使用的是`ES2015 Module`规范，则不需要安装任何插件，webpack 会对其进行`tree-shaking`，去除未引用的代码。

前面提过，`webpack`的`tree-shaking`是由`Uglylify`插件实现的，我在开发环境下，没有启用`Uglylify`来压缩代码，所以查看模块打包图，会发现整个库都被引入了，虽然我只引入了一个组件。`webpack4`在生产环境下，才会进行`tree-shaking`，设置`mode`的值为`production`就会开启生产环境下的优化。

如果是使用 `commonjs` 规范的 library 则需要一个插件支持，[babel-plugin-import](https://github.com/ant-design/babel-plugin-import)。该插件是`ant`官方开发的。许多 UI 组件库的按需引入也是依赖于这个插件。

安装

`yarn add babel-plugin-import -D`

修改`.babelrc`文件，

```json
"plugins": [
  ["import", {
    "libraryName": "hbf-public-components",
    "libraryDirectory": "lib/components"
  }]
],
```

## 总结

其实如果对于一个不是很稳定，需要一直`迭代更新`的`公用组件库`来说，使用 npm 包的话，会比较不方便，经常更新的公共组件代码可以使用`Git subtree`（[教程](https://tech.youzan.com/git-subtree/)）来维护，可以等到一定地步之后，公共组件库稳定下来之后再考虑发布一个 npm 包。

而开发一个组件库，也可以使用 [rollup.js](https://www.rollupjs.com/guide/zh) 来搭脚手架，`rollup.js` 默认使用的就是 `ES2015 Module`，可以进行静态分析，去除未引用的代码，`tree-shaking` 也是 `rollup.js` 先提出的。`Rollup`相比较于`Webpack`，更适合用于构建`library`，`Vue.js`就是使用 `Rollup` 构建的。`Webpack` 在代码分割这方面比较有优势，所以`webpack` 相对来说比较适合构建应用程序，不过使用 webpack 构建 library 也是可以的。

这个项目也可以用做 webpack 构建 library 的通用脚手架。下次再尝试`Rollup`构建 library

项目地址： https://github.com/huya-base-fed/public-components

npm 地址：https://www.npmjs.com/package/hbf-public-components

[关于模块规范，以及 webpack，babel 是如何编译转换模块的文章](https://juejin.im/post/5a2e5f0851882575d42f5609)
