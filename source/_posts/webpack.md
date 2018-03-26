---
title: webpack笔记
category: webpack
date: 2017-7-25
tags: [前端工程化]
---

这几天学习下了webpack,也做了demo,大概也了解webpack如何配置了...此篇是学习笔记...为了巩固刚学的知识..
<!-- more -->
这次使用的是webpack的新版本3.x

## 安装
新建一个文件夹， 使用`npm init` 快速配置package.json

文件目录是这样的... src用来放资源文件, build用来放打包后的文件. 

![](http://ww1.sinaimg.cn/large/ad9f1193gy1fhyoffw4egj206w0ayglx.jpg)

### 安装 webpack

安装webpack & webpack-dev-server
```bash
npm install webpack webpack-dev-server--save-v
```

安装webpack-dev-server（开发服务器)用于开发，便于调试..可以实时监听数据改变并刷新页面（live reloading）

至于 这里`--save`和 `--save-v`的区别  
`--save` 自动把模块和版本号添加到package.json配置文件的dependencies键对上  
`--save-v` 自动把模块和版本号添加到package.json的devdependencies键对上  
devdependencies下列出的模块是开发时用的，它们不会被部署到生产环境中，dependencies下的模块，则是生产环境中需要的依赖。(dev指的是开发的意思, prod是生产production的缩写)

永远不要在生产环境中使用开发工具...(我这里用在开发环境中的有webpack-dev-server)

## webpack.config.js

在项目目录里新建一个`webpack.config.js`文件用来配置webpack.  
webpack主要核心的东西就是 `entry(入口)`, `output(出口)`, `loader`, `plugins(插件)`
>webpack工作概念: 当 webpack 处理应用程序时，它会递归地构建一个**关系依赖图**,其中包含程序需要的每个模块,然后将这些模块打包成少量的bundle,通常只有一个文件,再由浏览器加载..
### entry（入口文件）
webpack创建关系依赖图,图的起点称作**入口起点**,它告诉webpack从哪里开始,并根据**关系依赖图**来确定需要打包的内容.也可以称作启动文件.  
单页面应用只有一个入口起点,多页面应用就有多个入口起点.  

`webpack-config.js`
```javascript
entry: {
    home:  __dirname + '/home.js',   //__dirname是node.js一个全局变量,指的是当前模块的目录名字.
    about: __dirname + '/about.js',
    setting: __dirname + '/setting.js'
}
```
只有一个入口文件可以写成  
```javascript
entry: __dirname + '/src/index.js'
```
### output (输出)
**输出你所打包的内容**
``` javascript
output: {
	path: __dirname + "/build",  //路径
	filename: "bundle.js"		 //文件名
},
```
### module 模块
在模块化编程中，开发者将程序分解成离散功能块，并称之为模块  
webpack把每个文件(.css,.html, .scss, .jpg, .png etc.)都作为模块处理, 然而,**webpack自身只理解JavaScript**  

webpack的关系依赖图就是各个模块之间的依赖关系,模块之间的依赖关系可以通过这几种方式确定.  
1.  [ES6](http://es6.ruanyifeng.com/) 的 `import`语句  
2.  [commonJS](http://javascript.ruanyifeng.com/nodejs/module.html) 的`require()`语句  
3.  [AMD](http://zhaoda.net/webpack-handbook/amd.html)的 `define`和`require`语句  
4.  css/less/sass文件中的 `@import`语句  
5.  样式(`url(...)`)或 HTML 文件(` <img src=...> `)中的图片链接(image url)

#### module.rules
之前版本的webpack是使用loaders的，webpack 2.x之后就使用rules了。  
**rules** ：(数组) 创建模块时，匹配请求的规则数组，这些规则能够对模块应用loader。  (loader又是什么)

#### loader又是什么
loader描述了webpack如何处理非JavaScript(non-JavaScript)模块，并且在bundle中引入这些依赖.  
loader 有两个目标  
1.  识别出应该被对应的loader进行转换的那些文件(test属性)  
2.  使用某个loader转换这些文件,使其能够被加入到关系依赖图中去,并最终添加到bundle中 (<span id="use">use属性</span>)  



##### 安装style-loader & css-loader 

`npm install style-loader css-loader --save-v`

`webpack3的语法`

```json
module: {
    rules: [
        {
            test: /\.css$/,  //一般是正则表达式
            use: [
                {
                    loader: "style-loader"
                },
                {
                    loader: "css-loader?modules"  // ?modules 表示打开Css modules功能
                }
            ]
        },
    ]
}
```

用一句通俗的话来讲吧..就是:  "喂,webpack编译器吗? 当你碰到require()语句或者 import语句中被解析为'.css'的路径时,在打包添加到bundle之前,先用  css-loader给他搞一下..然后再交给style-loader处理添加到style标签里..嵌入到JS代码中,恩,然后搞完再添加到bundle去"  

**注:  模块的加载是从下到上的. 这里先加载css-loader 然后再加载style-loader**

###### 安装 Babel

`npm install babel-core babel-preset-es2015 babel-loader --save-v`  

-  babel-core 是Bebel的核心编译器,  
-  babel-preset-es2015 是一个配置文件,我们可以把 ES2015的代码转化成ES5    
-  babel-loader 用来编译JS文件

修改`package.json`  在其中添加一个"babel"键

```
{
    "babel": {
        "presets": {
            "es2015"
        }
    }
}
```



#### Rule 



Rule 能够根据rule筛选出来的模块对其应用loader

每个规则可以分为三部分，   条件（condition），结果（result），嵌套规则（nested rule）

 

#### Rule Conditions (规则条件)
条件有两种输入值   
1.  resource：请求文件的绝对路径。它根据resolve规则解析
2.  issuer: 被请求资源（requested the resource）的模块文件的绝对路径。是导入的位置

resource 有4个属性对其匹配 ：  
1.  `test`  ：使用正则表达式匹配loader所处理的文件的拓展名
2.  `include` ：添加必须处理的文件/文件夹
3.  `exclude`： 屏蔽不需要处理的文件/文件夹
4.  `resource`

条件可以是其中之一： 

```javascript
test: /\.js$/ // 使用正则表达式
exclude: /node_modules/  //屏蔽拉黑node_modules此文件夹...
```
#### Rule result
rule result 只在 rule condition 匹配时使用



用 [use](#use)属性

### dev-tool

dev-tool : 开发者辅助调试工具 

`string` 默认值: `false`

选择一种source-map来增强调试过程,以下各种选先的构建和重构建速度不同..

#### source-map是什么?

> [阮一峰的文章](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html) source-map是一个信息文件, 里面存储着位置信息,转换后的代码的每一个位置,所对应的转换前的位置.  出错的时候, 工具报错(浏览器控制台)将直接显示原始代码,而不是转换后的代码.    

source-map 文件可以存储位置信息,存储转换前的所有变量名和属性名, 存储转换前的文件.   

**两个文件的各个位置是如何一一对应的?**

source-map 文件也有一个属性用来存储转换前文件跟转换后文件的对应信息,如(对应的行号, 位置对应,)  

在位置对应里,每个位置使用5位,表示5个字段, 分别表示

-  这个位置在(转换后的代码)的第几列, 
-  这个位置属于转换前的文件的哪一个文件. 
-  这个位置属于转换前代码的第几行, 
-  这个位置属于转换前代码的第几列, 
-  这个位置属于转换前的所有变量和属性名的哪一个.   



简单了解了这个东西,不深入去探讨..等到以后用到的话,再去进一步学习.



![Alt text](http://osuuzm0m8.bkt.clouddn.com/webpack.png)  

```javascript
devtool: 'cheap-eval-source-map' 
```

我在开发环境下是使用这个的, 报错有对应的行号提示,构建,重构建速度也快,相比较于eval, eval映射到的是转换后的代码,所以没有行号提示. 综合起来还是 `"cheap-eval-source-map"`比较合适..生产环境下我使用`"cheap-source-map"`

###  devServer （开发服务器）
webpack-dev-server 由webpack提供的一个本地开发服务器，可以让浏览器监测代码的修改，并自动刷新修改后的结果，提高效率。  
webpack-dev-server.. 个人觉得最方便的就是自动刷新 & 热重载
`语法`
```javascript 
devServer: {
	contentBase； path.join(__dirname, 'src'), // 告知本地服务器从哪里提供内容.
	historyApiFallback: true,
	port: 5555  //默认是8080端口,可自定义
}
```

现在配置了开发服务器的话, 开发过程中监测着代码文件,当代码文件发生变化时, 可以自动刷新,重新构建网页.  

但是过于频繁的刷新,当你的项目比较大时,reload页面需要等待的时间就比较长了,所以,dev-server还可以设置开启热替代.可以利用 插件 Hot Module Replacement 来实现.  模块热替换在程序运行时对模块修改,添加或者删除时,不需要重新加载整个页面,只更新改变内容.  

有两种方式可以启用模块热替换,

- 在 `webpack.config.js`文件中,

   ```
  plugins: [
      new webpack.HotModuleReplacement()  //new一个HotModuleReplacement对象
  ]

  devServer: {
      ...
      inline: true, //采用inline模式.而不是iframe模式
      hot: true    //开启热替换
  }
   ```

  这样需要写的东西太对,所以我还是通过第二种方式, CLI命令行添加参数来开启

  所以我还是用第二种方式来开启..

- CLI添加参数.  顺便把这段开启webpack的代码写到`package.json`的 `"scripts"`键上,  如下

  添加参数 `--line --hot`

  ```json
  "scripts": {
  	"dev": "webpack-dev-server --inline --hot --env=dev --open"
  }
  ```

  直接在终端运行 `npm run dev`就可以开启开发服务器了.带有热重载的,不过此时热重载还没生效,因为还需要配置点其他东西.  

还需要修改的东西

修改entry入口数组,

```json
entry: {
  main: [
      'webpack/hot/only-dev-server',  //在热更新失败后,需要采取手动刷新
      'webpack-dev-server/client?http://127.0.0.1:5555/',  //资源服务器,我这里是本地服务器localhost,如果需要远程服务器的资源就相应的更改就好了.
      './src/index.js'   //入口文件
  ]
}
```

`index.js` (入口JS文件)  

为了让html文件也可以采用热替换,需要把html当作一个模块来引入到`index.js`内

`index.js`

````javascript
require('./index.tpl.html');
````

也要在rules新建一个一个规则  

html-loader需要先install

`npm install html-loader --save-v`

`webpack.config.js`

```javascript
module: {
    rules: [
        {
            test: /\.html$/,
            use: ["html-loader"]  //使用html-loader,将html内容存为js字符串.  
          						  //遇到  require ('./index.tpl.html') 
          						  //index.tpl.html的内容会被转成一个Js字符串,合并到JS文件中
        }
    ]
}
```

`index.js`

```javascript
if(moduel.hot) {
	//module.hot.accept(dependencies, callback); //如果某个模块热替换后需要调用回调函数的就需要写这个.
	module.hot.accept('./a.js', () => {
        console.log("change a.js");
	});
    //如果是某人index.js的所有模块都采用热替换的话,则只需写
    //module.hot.accept(); 这样所有模块都采用热替换.
	module.hot.decline('./component.js'); //强制component.js的修改热更新失败,只能reload整个页面.
}
```

这样就已经可以实现热替换功能了,无论html,css,js etc.发生改变都会热替换那个模块,如果热替换失败的话,浏览器会自动刷新或者需要手动刷新,HMR会在控制台输出信息,可以查阅..

#### 生产环境...& 开发环境...的配置文件.

呃...一开始我是没有注意到这个生产环境跟开发环境之间所引用的配置的文件需要区别的东西的,看了官方的文档才知道,生产环境跟开发环境所用的配置文件不是完全相同的,官网推荐的那种手动方式,分别使用webpack.dev.js 和 webpack.prod.js 去区别开发环境跟生产环境,由于两个文件重复的东西太多,所以感觉效率不是很好..(懒得写两个,其实几乎差不多的文件)

google了下,找到了一个我个人比较喜欢也容易理解的方法..

` webpack.config.js`

```javascript
var path = require('path');
var webpack = require('webpack');
var HtmlWebpackPlugin = require('html-webpack-plugin');

// 生产环境需要用到的配置文件的内容
const prodConfig = {
  
}

// 开发环境需要用到的配置文件的内容
function devConfig() {
  const config = {
      ...
  };
  
  return Object.assign({}, prodConfig, config, {plugins: prodConfig.plugins.concat(config.plugins)})
  /*采用 Object.assign的方法,把几个对象复制到一个空对象上,形成一个新对象,拥有两个对象所有属性,如果属性名,属性值以后面对象的值为准.(后面的对象的值覆盖掉前面对象的值.)  因为plugins是个数组,所以需要用concat方法来将两个对象的plugins数组连接起来.*/
}

// 再根据CLI时传的env的参数来确定使用什么环境下的配置文件
module.exports = function(env) {
  	console.log("env is: ",env);

	if(env === "dev") {
		return devConfig();
	}

	return prodConfig;
}
```



`package.json`

```json
{
    "scripts": {
    	"dev": "webpack-dev-server --inline --hot --env=dev --open",
    	"build": "rimraf build && webpack -p --env=prod"
    }
}
```

`rimraf`是用来每次build之前清空build文件夹的..

```bash
npm install rimraf --save-v
```

这两天的webpack的入门学习大概就这些了(之前看了一些webpack@1.x 和@2,x的学习教程,然后发现webpack已经出到3 了,干脆直接去官网啃文档了,不过这文档,真的是有点晦涩啊....)...webpack对于SPA(单页面应用来说还是挺不错的..),(主要是模块化),如果是多页面应用的话,用gulp来打包,压缩文件会更方便.. 等缓一缓再去学习下gulp,  ...等到....掌握了,再去考虑,鱼跟熊掌怎么兼得的问题了.   路漫漫其修远兮,前方道路远又远又远啊....

> 参考网站: [webpack中文网](https://doc.webpack-china.org/)
>
> ​		[webpack模块热替换](http://www.cnblogs.com/stoneniqiu/p/6496425.html)
>
> etc.. 有些也忘了...  
>
> 呃 还有参考github上一些项目的issues的解决办法, 发现....呃....英文真的很重要阿...orz..

