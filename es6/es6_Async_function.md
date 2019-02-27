# ES6 异步操作和Async函数

ES6诞生**以前**，异步编程的方法，大概有下面四种。

- 回调函数
- 事件监听
- 发布/订阅
- Promise 对象

ES6将JavaScript异步编程带入了一个全新的阶段，ES7的`Async`函数更是提出了异步编程的终极解决方案。

## 基本概念

### 异步

**"异步"**，简单说就是一个任务分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头执行第二段。

比如，有一个任务T是读取文件进行处理，任务T的第一段是向操作系统发出请求，要求读取文件。然后，程序执行其他任务O，等到操作系统返回文件，再接着执行任务T的第二段（处理文件）。***这种不连续的执行，就叫做异步。***

相应地，连续的执行就叫做同步。由于是连续执行，不能插入其他任务，所以操作系统从硬盘读取文件的这段时间，程序只能干等着。

### 回调函数

JavaScript语言对异步编程的实现，就是回调函数。所谓回调函数，就是把任务的第二段单独写在一个函数里面，等到重新执行这个任务的时候，就直接调用这个函数。它的英语名字callback，直译过来就是"重新调用"。

读取文件进行处理，是这样写的。

```js
fs.readFile('/etc/passwd', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

上面代码中，readFile函数的第二个参数，就是回调函数，也就是任务的第二段。等到操作系统返回了`/etc/passwd`这个文件以后，回调函数才会执行。

由于执行分成两段，在这两段之间抛出的错误，程序无法捕捉，因此Node.js约定，错误对象err只能当作参数，传入第二段执行的函数。

回调函数本身并没有问题，它的问题出现在多个回调函数嵌套。假定读取A文件之后，再读取B文件，代码如下：

```js
fs.readFile(fileA, function (err, data) {
  fs.readFile(fileB, function (err, data) {
    // ...
  });
});
```

不难想象，如果依次读取多个文件，就会出现多重嵌套。代码不是纵向发展，而是横向发展，很快就会乱成一团，无法管理。这种情况就称为"回调函数噩梦"（callback hell）。

## Promise

Promise就是为了解决回调函数噩梦这个问题而提出的。它不是新的语法功能，而是一种新的写法，允许将回调函数的嵌套，改成链式调用。采用Promise，连续读取多个文件，写法如下。

```js
var readFile = require('fs-readfile-promise');

readFile(fileA)
.then(function(data){
  console.log(data.toString());
})
.then(function(){
  return readFile(fileB);
})
.then(function(data){
  console.log(data.toString());
})
.catch(function(err) {
  console.log(err);
});
```

可以看到，Promise 的写法只是对**回调函数的改进**，使用then方法以后，异步任务的两段执行看得更清楚了，除此以外，并无新意。

Promise 的最大问题是代码冗余，原来的任务被Promise 包装了一下，不管什么操作，一眼看去都是一堆 then，原来的语义变得很不清楚。

## Generator函数

### 协程

传统的编程语言。早有异步编程的解决方案（其实是多任务的解决方案）。其中一种叫做"协程"（coroutine）。意思是多个线程互相协作，完成异步任务。

协程有点像函数，又有点像线程。它的运行流程大致如下。

- 第一步：协程A开始执行
- 第二步：协程A执行到一半，进入暂停，执行权转移到协程B
- 第三步：协程B执行一段时间后，协程B交还执行权
- 第四步：协程A恢复执行，执行后一半。

上面流程的协程A，就是异步任务，因为它分成两段（或多段）执行。

举例来说，读取文件的协程写法如下。

```js
function *asyncJob() {
  // ...其他代码
  var f = yield readFile(fileA);
  // ...其他代码
}
```

上面代码的函数`asyncJob`是一个协程，它的奥妙就在其中的`yield`命令。它表示执行到此处，执行权将交给其他协程。也就是说，`yield`命令是异步两个阶段的分界线。

协程遇到`yield`命令就暂停，等到执行权返回，再从暂停的地方继续往后执行。它的最大优点，就是代码的写法非常像同步操作，如果去除yield命令，简直一模一样。

### Generator函数的概念

Generator函数是协程在ES6的实现，最大特点就是可以交出函数的执行权（即暂停执行）。

整个Generator函数就是一个封装的异步任务，或者说是**异步任务的容器**。

异步操作需要暂停的地方，都用`yield`语句注明。Generator函数的执行方法如下。

每次调用next方法，会返回一个对象，表示当前阶段的信息（value属性和done属性）。value属性是yield语句后面表达式的值，表示当前阶段的值；done属性是一个布尔值，表示Generator函数是否执行完毕，即是否还有下一个阶段。

### Generator函数的数据交换和错误处理

- next方法返回值的value属性，是Generator函数向外输出数据；

- next方法还可以接受参数，这是向Generator函数体内输入数据。

Generator 函数内部还可以部署错误处理代码，捕获**函数体外**抛出的错误。

```js
function* gen(x){
  try {
    var y = yield x + 2;
  } catch (e){
    console.log(e);
  }
  return y;
}

var g = gen(1);
g.next();
g.throw('出错了');
// 出错了
```

上面代码的最后一行，Generator函数体外，使用指针对象的throw方法抛出的错误，可以被函数体内的try ...catch代码块捕获。

这意味着，出错的代码与处理错误的代码，实现了时间和空间上的分离，这对于异步编程无疑是很重要的。

### 异步任务的封装

下面看看如何使用 Generator 函数，执行一个真实的异步任务。

```javascript
var fetch = require('node-fetch');

function* gen(){
  var url = 'https://api.github.com/users/github';
  var result = yield fetch(url);
  console.log(result.bio);
}
```

上面代码中，Generator函数封装了一个异步操作，该操作先读取一个远程接口，然后从JSON格式的数据解析信息。就像前面说过的，这段代码非常像同步操作，除了加上了yield命令。

执行这段代码的方法如下。

```js
var g = gen();
var result = g.next();

result.value.then(function(data){
  return data.json();
}).then(function(data){
  g.next(data);
});
```

上面代码中，首先执行Generator函数，获取遍历器对象，然后使用next 方法（第二行），执行异步任务的第一阶段。由于Fetch模块返回的是一个Promise对象，因此要用then方法调用下一个next 方法。

可以看到，虽然 Generator 函数将异步操作表示得很简洁，

但是流程管理却不方便（即何时执行第一阶段、何时执行第二阶段）。

## Thunk函数的含义

编译器的"传名调用"实现，往往是将参数放到一个临时函数之中，再将这个临时函数传入函数体。这个临时函数就叫做Thunk函数。

详细内容请参加相关链接

## CO模块

[co模块](https://github.com/tj/co)是著名程序员TJ Holowaychuk于2013年6月发布的一个小工具，用于Generator函数的自动执行。

详细内容请参加相关链接

## async函数

ES7提供了`async`函数，使得异步操作变得更加方便。

`async`函数是什么？一句话，`async`函数就是Generator函数的**语法糖**。

```js
var fs = require('fs');

var readFile = function (fileName) {
  return new Promise(function (resolve, reject) {
    fs.readFile(fileName, function(error, data) {
      if (error) reject(error);
      resolve(data);
    });
  });
};

var gen = function* (){
  var f1 = yield readFile('/etc/fstab');
  var f2 = yield readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

写成`async`函数，就是下面这样。

```js
var asyncReadFile = async function (){
  var f1 = await readFile('/etc/fstab');
  var f2 = await readFile('/etc/shells');
  console.log(f1.toString());
  console.log(f2.toString());
};
```

比较就会发现，`async`函数就是将Generator函数的星号（`*`）替换成`async`，将`yield`替换成`await`，仅此而已。

（1）内置执行器。Generator函数的执行必须靠执行器，所以才有了`co`模块，而`async`函数自带执行器。也就是说，`async`函数的执行，与普通函数一模一样，只要一行。

```js
var result = asyncReadFile();
```

上面的代码调用了`asyncReadFile`函数，然后它就会自动执行，输出最后结果。这完全不像Generator函数，需要调用`next`方法，或者用`co`模块，才能得到真正执行，得到最后结果。

（2）更好的语义。`async`和`await`，比起星号和`yield`，语义更清楚了。`async`表示函数里有异步操作，`await`表示紧跟在后面的表达式需要等待结果。

（3）更广的适用性。 `co`模块约定，`yield`命令后面只能是Thunk函数或Promise对象，而`async`函数的`await`命令后面，可以是Promise对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）。

（4）返回值是Promise。`async`函数的返回值是Promise对象，这比Generator函数的返回值是Iterator对象方便多了。你可以用`then`方法指定下一步的操作。

进一步说，`async`函数完全可以看作多个异步操作，包装成的一个Promise对象，而`await`命令就是内部`then`命令的语法糖。

### 语法

（1）`async`函数返回一个Promise对象。

`async`函数内部`return`语句返回的值，会成为`then`方法回调函数的参数。

函数内部的return 和 async　函数的返回是两个东西！！！

```js
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"
```

上面代码中，函数`f`内部`return`命令返回的值，会被`then`方法回调函数接收到。

`async`函数内部抛出错误，会导致返回的Promise对象变为`reject`状态。抛出的错误对象会被`catch`方法回调函数接收到。

```js
async function f() {
  throw new Error('出错了');
}

f().then(
  v => console.log(v),
  e => console.log(e)
)
// Error: 出错了
```

（2）`async`函数返回的Promise对象，必须等到内部所有`await`命令的Promise对象执行完，才会发生状态改变。也就是说，只有`async`函数内部的异步操作执行完，才会执行`then`方法指定的回调函数。有些类似Promise.all()。

下面是一个例子。

```js
async function getTitle(url) {
  let response = await fetch(url);
  let html = await response.text();
  return html.match(/<title>([\s\S]+)<\/title>/i)[1];
}
getTitle('https://tc39.github.io/ecma262/').then(console.log)
// "ECMAScript 2017 Language Specification"
```

（3）正常情况下，`await`命令后面是一个Promise对象。如果不是，会被转成一个立即`resolve`的Promise对象。

```js
async function f() {
  return await 123;
}

f().then(v => console.log(v))
// 123
```

`await`命令后面的Promise对象如果变为`reject`状态，则`reject`的参数会被`catch`方法的回调函数接收到。

```js
async function f() {
  await Promise.reject('出错了');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// 出错了
```

注意，上面代码中，`await`语句前面没有`return`，但是`reject`方法的参数依然传入了`catch`方法的回调函数。这里如果在`await`前面加上`return`，效果是一样的。

只要一个`await`语句后面的Promise变为`reject`，那么整个`async`函数都会中断执行。

```js
async function f() {
  await Promise.reject('出错了');
  await Promise.resolve('hello world'); // 不会执行
}
```

为了避免这个问题，可以将第一个`await`放在`try...catch`结构里面，这样第二个`await`就会执行。

```js
sync function f() {
  try {
    await Promise.reject('出错了');
  } catch(e) {
  }
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// hello world
//e，log不到

```

另一种方法是`await`后面的Promise对象再跟一个`catch`方面，处理前面可能出现的错误。

```js
async function f() {
  await Promise.reject('出错了')
    .catch(e => console.log(e));
  return await Promise.resolve('hello world');
}

f()
.then(v => console.log(v))
// 出错了
// hello world
```

如果有多个`await`命令，可以统一放在`try...catch`结构中

```js
async function main() {
  try {
    var val1 = await firstStep();
    var val2 = await secondStep(val1);
    var val3 = await thirdStep(val1, val2);

    console.log('Final: ', val3);
  }
  catch (err) {
    console.error(err);
  }
}
```

（4）如果`await`后面的异步操作出错，那么等同于`async`函数返回的Promise对象被`reject`。

```javascript
async function f() {
  await new Promise(function (resolve, reject) {
    throw new Error('出错了');
  });
}

f()
.then(v => console.log(v))
.catch(e => console.log(e))
// Error：出错了
```

防止出错的方法，也是将其放在`try...catch`代码块之中。

```js
async function f() {
  try {
    await new Promise(function (resolve, reject) {
      throw new Error('出错了');
    });
  } catch(e) {
  }
  return await('hello world');
}
```

### async函数的实现

async 函数的实现，就是将 Generator 函数和自动执行器，包装在一个函数里。

```js
async function fn(args){
  // ...
}

// 等同于

function fn(args){
  return spawn(function*() {
    // ...
  });
}
```

