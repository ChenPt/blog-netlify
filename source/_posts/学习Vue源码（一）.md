---
title: 学习Vue源码
category: Vue
tags: [Vue, JavaScript]
date: 2018-3-23
---

记一次阅读Vue源码的总结.

<!-- more -->

## 响应式原理

Vue通过响应式在修改数据的时候更新视图。

在官网copy的介绍响应式原理的图.
![](http://ww1.sinaimg.cn/large/ad9f1193gy1fpcluuphejj20xc0kugnm.jpg)

流向为： model -> view
可以对表单元素`v-model`来进行双向数据绑定.


### 数据劫持 & 将数据变成可观察的（Observable）

``` javascript
//   observer/index.js

/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */
// Observer是一个可以附加到每一个被观察的对象的类。一次附加，观察者将会将目标对象的属性键转化为拥有getter/setters的访问器属性，可以用来收集依赖（getter）和在属性有变化时分派通知更新（setter）
export class Observer {
  value: any;
  dep: Dep; //dep为一个Dep（依赖）类型
  vmCount: number; // number of vms that has this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep() // 创建新的Dep实例
    this.vmCount = 0

    //前面还有一些对传入构造函数的参数的类型的判断，而这里执行的话，value是个对象，从下面的walk方法可以看出
    this.walk(value)
  }    

    /**
   * Walk through each property and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  //尤大这里注释的意思是，Wlak方法遍历传进来的对象的每一个属性，把它们转化为访问器属性（getter/setter），这个方法只在参数是对象类型的时候被调用.
  walk (obj: Object) {
    //用Object.keys方法获取obj对象的所有属性.
    const keys = Object.keys(obj)
    //遍历属性，defineReactive方法处理.
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }
}


/**
 * Attempt to create an observer instance for a value,
 * returns the new observer if successfully observed,
 * or the existing observer if the value already has one.
 */

/**
 * 尝试为一个value创建一个观察者实例，如果value已经拥有一个存
 * 的观察者实例，则返回它。
 * 如果成功创建了，则返回新的观察者实例.
 */
export function observe (value: any, asRootData: ?boolean): Observer | void {
  //ob是一个Observer实例
  let ob: Observer | void
  // ...
  // ...
  //上面忽略的部分主要是防止为一个data重复创建Observer实例
  ob = new Observer()
  return ob  //返回一个Observer实例
}


/**
 * Define a reactive property on an Object.
 */
/** 此方法用来定义一个对象的响应式属性 这里的reactive我理解为响应式.
 *  defineReactive方法
 */

export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function, //这里使用的是flow，一种静态类型检查工具.
  shallow?: boolean  //shallow 浅的意思，应该是来判断val是一个基本属性还是引用属性吧.
) {
  //对obj对象里的key属性描述进行检索，返回一个描述符对象
  const property = Object.getOwnPropertyDescriptor(obj, key)

  //如果该属性描述符对象的configurable（能否修改属性）为false，则返回
  if (property && property.configurable === false) {
    return
  }

  //获取该属性已经定义了的 getter/setters 函数
  // cater for pre-defined getter/setters
  const getter = property && property.get
  if (!getter && arguments.length === 2) {
    //没有传入val参数且 getter不存在的话，val的值为key的value.
    val = obj[key]
  }
  const setter = property && property.set

  // 如果shallow为假，代表说val为一个对象，则递归调用ovserve方法来将子对象里的属性变成可观察的.
  let childOb = !shallow && observe(val)

  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      //如果getter存在，则调用getter获取到值
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend() //将Dep.target指的watch实例与当前val的dep实例连接起来。将watch实例push进dep实例维护的订阅者数组.
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      return value
      // 主要是实现依赖收集。下面第二节分析Dep.
    },
    set: function reactiveSetter (newVal) {

      //取得旧值value
      const value = getter ? getter.call(obj) : val
      /* eslint-disable no-self-compare */
      //旧值value与newVal比较，若一样，跳出,不继续执行往下的操作。
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      
      if (setter) {
        //如果setter存在, 执行setter函数，传入obj的this，和传入参数newVal
        setter.call(obj, newVal)
      } else {
        //不存在就直接把新值赋给val
        val = newVal
      }

      // 如果shallow为假，代表说val为一个对象，则递归调用ovserve方法来将子对象里的属性变成可观察的.
      childOb = !shallow && observe(newVal)

      //通知在订阅者数组内的订阅者
      dep.notify()
    }
  })
}   

```
### 小结

[initData源码](https://github.com/vuejs/vue/blob/dev/src/core/instance/state.js#L107)
``` javascript

function initData (vm: Component) { 
    //...
    //...
    // observe data
    observe(data, true /* asRootData */)
}
```
`Vue`在其初始化时调用`observe`方法，为`data`对象的每个属性利用`Object.defineProperty()`方法给每个属性添加`getter`, `setter`，让每个属性都变成`observable`(可观察的).如果属性的`value`为一个对象的话，那么内部会递归调用`observe`方法来给子对象的属性创建观察者实例，使其变为`observable`的.
(其实还会将template上的每个`v-`指令还有`computed`，`props`的数据全部转化为`observale`)

当Vue进行render时，需要取数据，就会触发`getter`，进行依赖收集， （将订阅者(watcher)和观察者观察的（data）绑定起来）,修改某一个可观察属性时，就会触发该属性的`setter`，通知订阅者（watcher）进行更新操作.




## 依赖（dependence）Dep

这个dep对象上可以挂载多个订阅者
``` javascript
/**
 * A dep is an observable that can have multiple
 * directives subscribing to it.
 */
/**
 * 这里我理解为，一个dep实例可以挂载多个订阅者. 
 */
export default class Dep {
  static target: ?Watcher;
  id: number;  //用来表示
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  // 将订阅者添加到subs数组
  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  //从subs里移除某个订阅者
  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  //用与将watcher与Dep的绑定
  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  //通知， 通知subs里的所有订阅者（watcher实例）执行update操作
  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

// the current target watcher being evaluated.
// this is globally unique because there could be only one
// watcher being evaluated at any time.
// 这里定义了个全局变量 Dep.target，用于确定当前的watcher实例.任何时候只有一个watcher实例可以给评估.
// 因为在watcher内部,当它要把自己加进某个data的订阅者数组里时，会将其自身赋给Dep.target，然后触发data的getter，这时就会将其push进subs数组，然后watch再把Dep.target置空.
Dep.target = null

//一个target栈，用以评估watcher实例
const targetStack = []

//这两个方法暴露给watcher使用.用于来给Dep.target指示一个watcher.
export function pushTarget (_target: ?Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}

export function popTarget () {
  Dep.target = targetStack.pop()
}
```
### 小结

读取数据，触发`observable` `data`的getter，通过`Dep`将data与订阅者添加一个依赖关系。一个data对应一个`Dep`实例，其有一个uniqueID，用来标识，其内部维护一个`subs`订阅者数组。当data改变，触发setter，会通过`Dep`实例来给subs里的所有订阅者通知更新。

## Watcher 订阅者
订阅者主要作用：
1. 通过Dep（依赖）来为每个数据添加订阅者，当数据变化时，会通过`Dep`来通知订阅者进行`update`操作.
2. 创建订阅者实例会传入一个cb函数，当update执行完后会执行这个cb函数。cb函数是用来执行re-render操作的，用于将新数据重新渲染到view上.

这里就贴我自己写的`watch`
``` javascript
import Dep from './dep'

export default class Watcher {
    constructor(vm, exp, cb) {
        this.vm = vm  //在vue源码里，这里是一个component
        this.exp = exp //某个属性
        this.cb = cb
        this.value = this.get()
    }

    /**
     * 调度者接口，当订阅数据更新时执行
    */
    update() {
        console.log('订阅者里的update操作.');
        this.run()
    }

    //用来触发getter获取到最新的新值.
    get() {
        Dep.target = this
        let value = this.vm[this.exp] //触发key的getter，将自己添加为订阅者.
        //添加完后
        Dep.target = null;

        return value;

    }

    /**
     * 调度者工作接口，由调度者接口调用
    */
    // vue源码里 run执行后执行cb函数，视图重新渲染
    run() {
        let value = this.get()
        console.log('run: ', value)
        let oldVal = this.value

        if(oldVal !== value) {
            this.value = value
            this.cb.call(this.vm, value, oldVal)
            console.log("执行完毕回调");
        }
    }
}
```
### 对三者的一个小结
![](http://ww1.sinaimg.cn/large/ad9f1193gy1fpcluuphejj20xc0kugnm.jpg)

首先是组件渲染函数，进行一次render操作就会取到所有的数据，这时就会触发getter，(这里只取到了视图渲染需要的数据触发getter),从而进行依赖收集，`Data`可以看成与`Watcher`绑定在一起，但是我们知道他们中间是有个`Dep`在维护的他们的关系的，当触发getter时，将watcher添加到Dep的`subs`数组。数据改变时，触发setter，`Dep`通知`subs`里的所有watcher【你订阅的数据改变了】，watcher会触发update，而实际工作是由`run`来执行，判断新旧数据是否一致，如果数据真的改变了，则会触发watcher实例的cb函数，继续将工作交给compiler来做，最终结果会重新渲染视图（一般是渲染部分视图），因为vue也采用了virtual DOM技术，（用JS对象来模仿DOM节点，避免多次直接DOM操作，提高性能）监听到VNode变化时，会先通过diff算法，判断初始虚拟DOM树和改变后的虚拟DOM树是否有差别，具体哪里发生改变，思想是修改尽量少的DOM，进行尽量少的DOM操作，最后把发生改变的虚拟DOM元素应用到真实DOM上.（`patch`操作）

再看上图. `touch`

在源码`watcher.js`也看到`touch`，关于这个我自己是这样理解的.
``` javascript
//observer/watcher.js

// ...

// "touch" every property so they are all tracked as
// dependencies for deep watching
if (this.deep) {
    traverse(value)
}
```

"touch" every property so they are all tracked as dependencies for deep watching

这里应该是来对值为`object` or `array`的进一步处理, 触发每个深层对象的依赖.

图中的`touch`也可以理解为访问'touch' every needed property，多了个needed，就是访问所有需要用到的属性. 


## 自己的实现

模仿Vue完成小作业， 因为还没有去看Vue的模版渲染，所以就只实现了Observer、Dep、Watcher这三者的逻辑关系，最终结果一个简单的观察订阅者模式的demo。

[demo in github]()

## 题外话

浅尝辄止，这个词，来形容过去的学习生活，可以说是很形象很贴切了。

最近也是在忙春招，也是会有点紧张和压抑，原因，`浅尝辄止`。

反正，加油啦。自我安慰（学东西，慢一点没关系啦）。

也是自己第一篇源码相关的总结，学习到不少，这篇主要还是给源码根据自己的理解添加中文注释，可能由很多地方有误。

## link
[Vue - github](https://github.com/vuejs/vue/blob/dev/src/core/observer)

[柒陌 - github](https://github.com/answershuto/learnVue/blob/master/docs/%E4%BB%8E%E6%BA%90%E7%A0%81%E8%A7%92%E5%BA%A6%E5%86%8D%E7%9C%8B%E6%95%B0%E6%8D%AE%E7%BB%91%E5%AE%9A.MarkDown)

[segmentfault](https://segmentfault.com/a/1190000006599500#articleHeader0)




