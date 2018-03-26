---

title: Process

category: Node.js

tabs: [Node.js, process]

date: 2017-07-30 21:00

---



最近在学gulp,接触到了Node.js的process对象,stream  

所以就看了Node.js官方文档学习..做下笔记

<!-- more -->

把目前接触到的process的相关属性和方法整理下..

Process对象是一个global对象,提供有关信息,控制当前nodejs进程.它作为一个全局对象,对于nodejs应用程序始终是可用的,所以无需使用require().



**Process对象是EventEmitter的实例,**
在 NodeJS 中，使用 process 代表 NodeJS 应用程序

## process Event

### Event: 'exit'

`exit`事件监听器

```
process.on('exit', (code) => {
	console.log(`code is ${code}`);
})
```

`'exit'`事件监听器的回调函数只有一个入参, 这个参数可以使process.exitCode的属性值,或者是调用process.exit()方法传入的`code`值.    

`'exit'`事件监听器的回调函数只允许包含同步操作.当`'exit'`事件被调用后,所有在event loop队列中排队的事件都会被强制丢弃,然后Node.js进程会马上结束.  

```javascript
process.on('exit', (code) => {
	setTimeout(() => {
    	console.log('hello ,this will not run');
	}, 0)
});
console.log('wow');
process.exit(1);

// only printf "wow"
```



## process.argv
`a.js`

``` javascript
process.argv.forEach((item, idx) => {
    console.log(`${idx}: ${item}`);
})
```

``` shell 
node a.js a b c d

0: /usr/local/bin/node
1: /home/chenpengteng/gulpstudy/src/js/a.js
2: a
3: b
4: c
5: d

```
第一个元素是`process.execPath`属性,返回启动Node.js进程的可执行文件所在的绝对路径  
第二个元素是 当前Node.js进程的当前工作目录 使用 `process.cwd()`方法也可以得到

## process.stdin 标准输入流对象

process.stdin 是一个stream.writable类对象

## process.stdout 标准输出流对象
``` javascript 
process.stdout.write('hello stdout\n'); 
//在终端输出
hello stdout

```
## process.env 环境变量
process.env 属性返回一个包含用户环境信息的对象

## process.exit([code]) & process.exitCode



code为结束状态码,默认为0.(success状态码.)

显式使用process.exit()会强制进程尽快结束,即时还有在等待中的异步操作没有全部执行完成,如 process.stdout

尽量不要显式调用process.exit. 当Node.js的事件轮询队列没有在等待中的工作,Node.js进程会自行结束.  

process.exitCode是进程结束的状态码.   

当进程正常结束或者通过process.exit()方法结束进程而没有传参时,process.exitCode代表进程结束的状态码.  

当使用process.exit()结束进程,并指定了一个状态码,则会覆盖process.exitCode的原有值.