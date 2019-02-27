# 谈谈ES6前后的异步编程

- **本文作者：** 张炳
- **本文链接：** [http://blog.zhangbing.club/Javascript/谈谈ES6前后的异步编程/](http://blog.zhangbing.club/Javascript/%E8%B0%88%E8%B0%88ES6%E5%89%8D%E5%90%8E%E7%9A%84%E5%BC%82%E6%AD%A5%E7%BC%96%E7%A8%8B/)

Javascript 语言的执行环境是“单线程”的，如果没有异步编程，根本没法用，非卡死不可。

为了解决这个问题，Javascript语言将任务的执行模式分成两种：同步（Synchronous）和异步（Asynchronous）两种模式概念很好理解。

ES6 诞生以前，异步编程的方法，大概有下面四种：回调函数 ，事件监听 ，发布/订阅 ，Promise对象。

##  回调函数

这是异步编程最基本的方法。

假定有两个函数f1和f2，后者等待前者的执行结果。

```js
　　f1();
　　f2();
```

如果f1是一个很耗时的任务，可以考虑改写f1，把f2写成f1的回调函数。

```js
　　function f1(callback){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　callback();
　　　　}, 1000);
　　}
```

执行代码就变成下面这样：

```js
f1(f2);
```

采用这种方式，我们把同步操作变成了异步操作，f1不会堵塞程序运行，相当于先执行程序的主要逻辑，将耗时的操作推迟执行。

回调函数的优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（Coupling），流程会很混乱，而且每个任务只能指定一个回调函数。

## 事件监听

另一种思路是采用事件驱动模式。任务的执行不取决于代码的顺序，而取决于某个事件是否发生。

还是以f1和f2为例。首先，为f1绑定一个事件（这里采用的jQuery的写法）。

```js
f1.on('done', f2);
```

上面这行代码的意思是，当f1发生done事件，就执行f2。然后，对f1进行改写：

```js
　　function f1(){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　f1.trigger('done');
　　　　}, 1000);
　　}
```

f1.trigger(‘done’)表示，执行完成后，立即触发done事件，从而开始执行f2。

这种方法的优点是比较容易理解，可以绑定多个事件，每个事件可以指定多个回调函数，而且可以”去耦合”（Decoupling），有利于实现模块化。缺点是整个程序都要变成事件驱动型，运行流程会变得很不清晰。

## 发布/订阅

上一节的”事件”，完全可以理解成”信号”。

我们假定，存在一个”信号中心”，某个任务执行完成，就向信号中心”发布”（publish）一个信号，其他任务可以向信号中心”订阅”（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做”发布/订阅模式”（publish-subscribe pattern），又称”观察者模式”（observer pattern）。

这个模式有多种实现，下面采用的是Ben Alman的Tiny Pub/Sub，这是jQuery的一个插件。

首先，f2向”信号中心”jQuery订阅”done”信号。

```js
jQuery.subscribe("done", f2);
```

然后，f1进行如下改写：

```js
　　function f1(){
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　jQuery.publish("done");
　　　　}, 1000);
　　}
```

jQuery.publish(“done”)的意思是，f1执行完成后，向”信号中心”jQuery发布”done”信号，从而引发f2的执行。

此外，f2完成执行后，也可以取消订阅（unsubscribe）。

```js
jQuery.unsubscribe("done", f2);
```

这种方法的性质与”事件监听”类似，但是明显优于后者。因为我们可以通过查看”消息中心”，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。

## Promises对象

Promises对象是CommonJS工作组提出的一种规范，目的是为异步编程提供统一接口。

简单说，它的思想是，每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。比如，f1的回调函数f2,可以写成：

```js
f1().then(f2);
```

f1要进行如下改写（这里使用的是jQuery的实现）：

```js
　　function f1(){
　　　　var dfd = $.Deferred();
　　　　setTimeout(function () {
　　　　　　// f1的任务代码
　　　　　　dfd.resolve();
　　　　}, 500);
　　　　return dfd.promise;
　　}
```

这样写的优点在于，回调函数变成了链式写法，程序的流程可以看得很清楚，而且有一整套的配套方法，可以实现许多强大的功能。

比如，指定多个回调函数：

```js
f1().then(f2).then(f3);
```

再比如，指定发生错误时的回调函数：

```js
f1().then(f2).fail(f3);
```

而且，它还有一个前面三种方法都没有的好处：如果一个任务已经完成，再添加回调函数，该回调函数会立即执行。所以，你不用担心是否错过了某个事件或信号。这种方法的缺点就是编写和理解，都相对比较难。

------

ES6诞生后，出现了Generator函数，它将 JavaScript 异步编程带入了一个全新的阶段。ES6也将Promise 其写进了语言标准，统一了用法，原生提供了Promise对象。

故ES6异步编程的方法，大概有两种：Generator函数，Promise。

## Generator函数

特点： 带星号function，yield语句 ，next() 获取下一个yield表达式中yield后的值，拥有遍历器接口，与for..of可搭配使用

下面代码中，Generator函数封装了一个异步操作，该操作先读取一个远程接口，然后从JSON格式的数据解析信息。这段代码非常像同步操作，除了加上了yield命令

```js
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

*执行过程:*

首先执行Generator函数，获取遍历器对象，然后使用next 方法（第二行），执行异步任务的第一阶段。由于Fetch模块返回的是一个Promise对象，因此要用then方法调用下一个next 方法。

*缺点：*

可以看到，虽然Generator函数将异步操作表示得很简洁，但是流程管理却不方便（即何时执行第一阶段、何时执行第二阶段），即如何实现自动化的流程管理。

*补充拓展*

可以参考阮一峰的[ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/generator-async)用Thunk函数实现自动化流程管理,对Generator函数进行拓展，前提是每一个异步操作，都要是Thunk函数,进价就是再用CO模块来实现自动化流程管理，co模块其实就是将两种自动执行器（Thunk 函数和 Promise 对象），包装成一个模块。使用 co 的前提条件是，Generator 函数的yield命令后面，只能是 Thunk 函数或 Promise 对象。如果数组或对象的成员，全部都是 Promise 对象，也可以使用 co。后面，ES2017标准引入了async函数，对Generator再“语法升级”， async 函数是什么？一句话，它就是 Generator 函数的语法糖。async函数对 Generator 函数进行了改进，体现在以下四点：

- 内置执行器。

Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器。也就是说，async函数的执行，与普通函数一模一样，只要一行。

- 更好的语义。

async和await，比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。

- 更广的适用性。

co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。

- 返回值是 Promise。

async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作。进一步说，async函数完全可以看作多个异步操作，包装成的一个 Promise 对象，而await命令就是内部then命令的语法糖。

## Promise

ES6 规定，Promise对象是一个构造函数，用来生成Promise实例。

下面代码创造了一个Promise实例。

```js
const promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

Promise构造函数接受一个函数作为参数，该函数的两个参数分别是resolve和reject。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。

Promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

```js
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

then方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为resolved时调用，第二个回调函数是Promise对象的状态变为rejected时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受Promise对象传出的值作为参数。 Promise 的基本用法就谈到这，更深入用法，请参考阮一峰的[ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/promise)

*特别需要指出的是在ES6之前，promise是一套规范和原则，只要设计的库复合规范的要求就都可以算是promise, 目前比较流行的promise库（插件）有q和when，RSVP.js，jQuery的Deferred等。ES6后，将Promise 众多规范中的一种写入语言标准，ES6中的 Promise 是其中一种，各个 Promise 规范之间有细微的差别(主要是特性上的)*

参考来源：

- [ECMAScript 6 入门](http://es6.ruanyifeng.com/)
- [Javascript异步编程的4种方法](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)