---
layout:     post
title:      "js异步编程史"
subtitle:   "js异步编程的发展"
date:       2018-04-19 22:00:00
author:     "iwalking11"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
	- 异步编程
---



## js单线程模型
众所周知javascript是单线程的，它的设计之初是为浏览器设计的GUI编程语言，GUI编程的特性之一是保证UI线程一定不能阻塞，否则体验不佳，甚至界面卡死。

在 JS 运行的时候可能会阻止 UI 渲染，这说明了两个线程是互斥的。这其中的原因是因为 JS 可以修改 DOM，如果在 JS 执行的时候 UI 线程还在工作，就可能导致不能安全的渲染 UI

所谓"单线程"，就是指一次只能完成一件任务。如果有多个任务，就必须排队，前面一个任务完成，再执行后面一个任务

## 什么叫异步？
所谓"异步"，简单说就是一个任务不是连续完成的，可以理解成该任务被人为分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头执行第二段。

比如，有一个任务是读取文件进行处理，任务的第一段是向操作系统发出请求，要求读取文件。然后，程序执行其他任务，等到操作系统返回文件，再接着执行任务的第二段（处理文件）。这种不连续的执行，就叫做异步。

相应地，连续的执行就叫做同步。由于是连续执行，不能插入其他任务，所以操作系统从硬盘读取文件的这段时间，程序只能干等着。。

## 异步的发展历程
ES6 诞生以前，异步编程的方法，大概有下面四种：回调函数 ，事件监听 ，发布/订阅 ，Promise对象。

### 1.回调函数（redux-thunk）
这是异步编程最基本的方法。

假定有两个函数f1和f2，后者等待前者的执行结果。


```
f1();
f2();
```

　　
如果f1是一个很耗时的任务，可以考虑改写f1，把f2写成f1的回调函数。


```
function f1(callback){
    setTimeout(function () {
　　　　　　// f1的任务代码
　　　　callback();
　　}, 1000);
}
```

执行代码就变成下面这样：


```
f1(f2);
```

采用这种方式，我们把同步操作变成了异步操作，f1不会堵塞程序运行，相当于先执行程序的主要逻辑，将耗时的操作推迟执行。


这种方式实现异步编程优点是思路清晰，以串行的思考方式进行编程，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（Coupling），形成回调地狱，过多的回调嵌套使得代码变得难以理解拆分和维护。

### 2.事件监听（观察者模式）
另一种思路是采用事件驱动模式。任务的执行不取决于代码的顺序，而取决于某个事件是否发生。

还是以f1和f2为例。首先，为f1绑定一个事件（这里采用的jQuery的写法）。


```
f1.on('done', f2);
上面这行代码的意思是，当f1发生done事件，就执行f2。然后，对f1进行改写：

　　function f1(){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　f1.trigger('done');
　　　　}, 1000);
　　}
```
（1）onclick方法
（2）addEvenListener方法

f1.trigger('done')表示，执行完成后，立即触发done事件，从而开始执行f2。

这种方法的优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以"去耦合"（Decoupling），有利于实现模块化。缺点是整个程序都要变成事件驱动型，运行流程会变得很不清晰。

### 3.发布/订阅模式
上一节的"事件"，完全可以理解成"信号"。

我们假定，存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信号，其他任务可以向信号中心"订阅"（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做"发布/订阅模式"（publish-subscribe pattern），又称"观察者模式"（observer pattern）。

这个模式有多种实现，下面采用的是Ben Alman的Tiny Pub/Sub，这是jQuery的一个插件。

首先，f2向"信号中心"jQuery订阅"done"信号。


```
jQuery.subscribe("done", f2);
然后，f1进行如下改写：

　　function f1(){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　jQuery.publish("done");
　　　　}, 1000);
　　}
```

jQuery.publish("done")的意思是，f1执行完成后，向"信号中心"jQuery发布"done"信号，从而引发f2的执行。

此外，f2完成执行后，也可以取消订阅（unsubscribe）。


```
jQuery.unsubscribe("done", f2);
```

这种方法的性质与"事件监听"类似，但是明显优于后者。因为我们可以通过查看"消息中心"，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

### 4.Promises对象（redux-promise）
Promises对象是CommonJS工作组提出的一种规范，目的是为异步编程提供统一接口。

简单说，它的思想是，每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。比如，f1的回调函数f2,可以写成：

f1().then(f2);

这样写的优点在于，回调函数变成了链式写法，程序的流程可以看得很清楚，而且有一整套的配套方法，可以实现许多强大的功能。

再比如，指定发生错误时的回调函数：

f1().then(f2).fail(f3);

而且，它还有一个前面三种方法都没有的好处：如果一个任务已经完成，再添加回调函数，该回调函数会立即执行。所以，你不用担心是否错过了某个事件或信号。这种方法的缺点就是编写和理解，都相对比较难。

ES6诞生后，出现了Generator函数，它将 JavaScript 异步编程带入了一个全新的阶段。ES6也将Promise 其写进了语言标准，统一了用法，原生提供了Promise对象。

### 5.Generator函数（redux-saga）
特点： 带星号function，yield语句 ，next()
获取下一个yield表达式中yield后的值，拥有遍历器接口，与for..of可搭配使用

下面代码中，Generator函数封装了一个异步操作，该操作先读取一个远程接口，然后从JSON格式的数据解析信息。这段代码非常像同步操作，除了加上了yield命令


```
var fetch = require('node-fetch');

function * gen() {
    var url = 'http://api.github.com/users/github';
    var result = yield fetch(url);
    console.log(result.bio);
}

var g = gen();
var result = g.next();

result.value.then(function(data) {
    return data.json();
}).then(function (data) {
    g.next(data);
});
```

执行过程:

首先执行Generator函数，获取遍历器对象，然后使用next 方法（第二行），执行异步任务的第一阶段。由于Fetch模块返回的是一个Promise对象，因此要用then方法调用下一个next 方法。

缺点：

可以看到，虽然Generator函数将异步操作表示得很简洁，但是流程管理却不方便（即何时执行第一阶段、何时执行第二阶段），即如何实现自动化的流程管理。


### 6.async函数

可以参考阮一峰的ECMAScript 6 入门用Thunk函数实现自动化流程管理,对Generator函数进行拓展，前提是每一个异步操作，都要是Thunk函数,进价就是再用CO模块来实现自动化流程管理，co模块其实就是将两种自动执行器（Thunk 函数和 Promise 对象），包装成一个模块。使用 co 的前提条件是，Generator 函数的yield命令后面，只能是 Thunk 函数或 Promise 对象。如果数组或对象的成员，全部都是 Promise 对象，也可以使用 co。后面，ES2017标准引入了async函数，对Generator再“语法升级”， async 函数是什么？一句话，它就是 Generator 函数的语法糖。async函数对 Generator 函数进行了改进，体现在以下四点：

1. 内置执行器。  
Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器。也就是说，async函数的执行，与普通函数一模一样，只要一行。

1. 更好的语义。  
async和await，比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。

1. 更广的适用性。  
co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。

1. 返回值是 Promise。  
async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。进一步说，async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。


### 7.Promise/A+规范
promise的特点：

- new Promise时需要传递一个executor执行器,执行器会立刻执行
执行器中传递了两个参数：resolve成功的函数、reject失败的函数，他们调用时可以接受任何值的参数value
- promise状态只能从pending态转onfulfilled,onrejected到resolved或者rejected，然后执行相应缓存队列中的任务
- promise实例,每个实例都有一个then方法，这个方法传递两个参数，一个是成功回调onfulfilled,另一个是失败回调onrejected
- promise实例调用then时，如果状态resolved，会让onfulfilled执行并且把成功的内容当作参数传递到函数中
- promise中可以同一个实例then多次,如果状态是pengding 需要将函数存放起来 等待状态确定后 在依次将对应的函数执行 (发布订阅)

链接：https://juejin.im/post/5b9645c75188255c463669e9


1. 每次调用返回的都是一个新的Promise实例(这就是then可用链式调用的原因)
1. 如果then中返回的是一个结果的话会把这个结果传递下一次then中的成功回调
1. 如果then中出现异常,会走下一个then的失败回调
1. 在 then中使用了return，那么 return 的值会被Promise.resolve() 包装(见例1，2)
1. then中可以不传递参数，如果不传递会透到下一个then中(见例3)
1. catch 会捕获到没有捕获的异常

特别需要指出的是在ES6之前，promise是一套规范和原则，只要设计的库复合规范的要求就都可以算是promise, 目前比较流行的promise库（插件）有q和when，RSVP.js，jQuery的Deferred等。ES6后，将Promise 众多规范中的一种写入语言标准，ES6中的 Promise 是其中一种，各个 Promise 规范之间有细微的差别(主要是特性上的)

promise的核心原理其实就是发布订阅模式，通过两个队列来缓存成功的回调（onResolve）和失败的回调（onReject）


### 8.多个异步任务依赖执行情况下，各种异步编程方式的区别

## 参考
- [异步编程的前世今生(异步流程历史)](https://zhuanlan.zhihu.com/p/33107664)
- [promise原理就是这么简单](https://juejin.im/post/5b9645c75188255c463669e9)
- [阮一峰 Generator 函数的异步应用](http://es6.ruanyifeng.com/#docs/generator-async)
- [JS的异步模式：1、回调函数；2、事件监听；3、观察者模式；4、promise对象](https://www.cnblogs.com/chengxs/p/6497575.html)
- [谈谈ES6前后的异步编程](https://segmentfault.com/a/1190000015440630#articleHeader0)
