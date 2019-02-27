# ES6 Generator

## 简介

Generator函数是ES6提供的一种异步编程解决方案，语法行为与传统函数完全不同。本章详细介绍Generator函数的语法和API，它的异步编程应用请看《异步操作》一章。

Generator函数有多种理解角度。从语法上，首先可以把它理解成，Generator函数是一个**状态机**，封装了多个内部状态。

执行Generator函数会返回一个遍历器对象，也就是说，Generator函数除了状态机，还是一个遍历器对象生成函数。

Generator函数是一个普通函数，但是有两个特征。一是，`function`关键字与函数名之间有一个星号；二是，函数体内部使用`yield`语句，定义不同的内部状态（yield语句在英语里的意思就是“产出”）。

**总结**

1. Generator函数是一个普通函数
2. Generator函数的function 关键字和函数名间有一个*号
3. Generator函数内部使用yield定义不同的状态
4. Generator函数是一个状态机，封装了多个内部状态。
5. Generator函数是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历Generator函数内部的每个状态。

```js
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();
```

上面代码定义了一个Generator函数`helloWorldGenerator`，它内部有两个`yield`语句“hello”和“world”，即该函数有**三个状态**：hello，world和return语句（结束执行）

Generator函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。

**不同的是**，调用Generator函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是上一章介绍的遍历器对象（Iterator Object）。

下一步，必须调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用`next`方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个`yield`语句（或`return`语句）为止。换言之，Generator函数是分段执行的，`yield`语句是暂停执行的标记，而`next`方法可以恢复执行。

```js
hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

第一次调用，Generator函数开始执行，直到遇到第一个`yield`语句为止。`next`方法返回一个对象，它的`value`属性就是当前`yield`语句的值hello，`done`属性的值false，表示遍历还没有结束。

第二次调用，Generator函数从上次`yield`语句停下的地方，一直执行到下一个`yield`语句。`next`方法返回的对象的`value`属性就是当前`yield`语句的值world，`done`属性的值false，表示遍历还没有结束。

第三次调用，Generator函数从上次`yield`语句停下的地方，一直执行到`return`语句（如果没有return语句，就执行到函数结束）。`next`方法返回的对象的`value`属性，就是紧跟在`return`语句后面的表达式的值（如果没有`return`语句，则`value`属性的值为undefined），`done`属性的值true，表示遍历已经结束。

第四次调用，此时Generator函数已经运行完毕，`next`方法返回对象的`value`属性为undefined，`done`属性为true。以后再调用`next`方法，返回的都是这个值。

ES6没有规定，`function`关键字与函数名之间的星号，写在哪个位置。这导致下面的写法都能通过。

```js
function * foo(x, y) { ··· }

function *foo(x, y) { ··· }

function* foo(x, y) { ··· }

function*foo(x, y) { ··· }
```

由于Generator函数仍然是普通函数，所以一般的写法是上面的第三种，即星号紧跟在`function`关键字后面。

## yield语句

由于调用Generator函数会返回一个遍历器对象。

只有调用该遍历器对象的`next`方法才会遍历下一个内部状态，`yield`语句就是暂停标志。

`yield`语句后面的表达式，只有当调用`next`方法、内部指针指向该语句时才会执行，因此等于为JavaScript提供了**手动的“惰性求值”**（Lazy Evaluation）的语法功能。

```js
function* gen() {
  yield  123 + 456;
}
```

上面代码中，yield后面的表达式`123 + 456`，不会立即求值，只会在`next`方法将指针移到这一句时，才会求值。

Generator生成了一系列的值，这也就是它的名称的来历（在英语中，generator这个词是“生成器”的意思）。

Generator函数**可以不用`yield`语句**，这时就变成了一个单纯的暂缓执行函数。

```js
function* f() {
  console.log('执行了！')
}

var generator = f();

setTimeout(function () {
  generator.next()
}, 2000);
```

上面代码中，函数`f`**如果**是普通函数，在为变量`generator`赋值时就会执行。但是，函数`f`是一个Generator函数，就变成只有调用`next`方法时，函数`f`才会执行。

注意，`yield`语句不能用在普通函数中，否则会报错。

```js
var arr = [1, [[2, 3], 4], [5, 6]];

var flat = function* (a) {
  a.forEach(function (item) {
    if (typeof item !== 'number') {
      yield* flat(item);
    } else {
      yield item;
    }
  }
};

for (var f of flat(arr)){
  console.log(f);
}
```

上面代码会报错，因为`forEach`方法的参数是一个普通函数，但是在里面使用了`yield`语句

`yield`语句如果用在一个表达式之中，必须放在圆括号里面。

```js
console.log('Hello' + yield); // SyntaxError
console.log('Hello' + yield 123); // SyntaxError

console.log('Hello' + (yield)); // OK
console.log('Hello' + (yield 123)); // OK
```

`yield`语句用作函数参数或赋值表达式的右边，可以不加括号。

```js
foo(yield 'a', yield 'b'); // OK
let input = yield; // OK
```

## Generator函数和Iteratir接口的关系

任意一个对象的`Symbol.iterator`方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。

由于**Generator函数就是遍历器生成函数**，因此可以把Generator赋值给对象的`Symbol.iterator`属性，从而使得该对象具有Iterator接口。

```js
var myIterable = {};
myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};

[...myIterable] // [1, 2, 3]
```

Generator函数执行后，返回一个遍历器对象。该对象本身也具有`Symbol.iterator`属性，执行后返回自身。

```js
function* gen(){
  // some code
}

var g = gen();

g[Symbol.iterator]() === g
// true
```

## next方法的参数

`yield`句本身没有返回值，或者说总是返回`undefined`。`next`方法可以带一个参数，该参数就会被当作**上一个`yield`语句**的返回值。

注意`var tmp =yield xxx`中 ,tmp 即yield语句的返回值。

```js
function* f() {
  for(var i=0; true; i++) {
    var reset = yield i;
    if(reset) { i = -1; }
  }
}

var g = f();

g.next() // { value: 0, done: false }
g.next() // { value: 1, done: false }
g.next(true) // { value: 0, done: false }
```

这个功能有很重要的语法意义。Generator函数从暂停状态到恢复运行，它的上下文状态（context）是不变的。通过`next`方法的参数，就有办法在Generator函数开始运行之后，继续向函数体内部注入值。也就是说，可以在Generator函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。

```js
function* dataConsumer() {
  console.log('Started');
  console.log(`1. ${yield}`);
  console.log(`2. ${yield}`);
  return 'result';
}

let genObj = dataConsumer();
genObj.next();
//打印Started。程序运行至L3,yield, next()返回undefine
genObj.next('a')
//L3 yield返回a，运行　L3，打印 1. a，程序运行至L4,next()返回undefine
genObj.next('b')
// L4 yield返回b，运行　L4，打印 2. b，程序运行至L5,next()返回'result'. 
```
一定要试运行以上程序。若不清楚，参照简介。

## for..of循环

`for...of`循环可以自动遍历Generator函数时生成的`Iterator`对象，且此时不再需要调用`next`方法。

```js
function *foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

这里需要注意，一旦`next`方法的返回对象的`done`属性为`true`，`for...of`循环就会中止，且不包含该返回对象，所以上面代码的`return`语句返回的6，不包括在`for...of`循环之中。

利用`for...of`循环，可以写出遍历任意对象（object）的方法。原生的JavaScript对象没有遍历接口，无法使用`for...of`循环，通过Generator函数为它加上这个接口，就可以用了。

```js
function* objectEntries(obj) {
  let propKeys = Reflect.ownKeys(obj);

  for (let propKey of propKeys) {
    yield [propKey, obj[propKey]];
  }
}

let jane = { first: 'Jane', last: 'Doe' };

for (let [key, value] of objectEntries(jane)) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
```

上面代码中，对象`jane`原生不具备Iterator接口，无法用`for...of`遍历。这时，我们通过Generator函数`objectEntries`为它加上遍历器接口，就可以用`for...of`遍历了。加上遍历器接口的另一种写法是，将Generator函数加到对象的`Symbol.iterator`属性上面。

```js
function* objectEntries() {
  let propKeys = Object.keys(this);

  for (let propKey of propKeys) {
    yield [propKey, this[propKey]];
  }
}

let jane = { first: 'Jane', last: 'Doe' };

jane[Symbol.iterator] = objectEntries;

for (let [key, value] of jane) {
  console.log(`${key}: ${value}`);
}
// first: Jane
// last: Doe
```

除了`for...of`循环以外，扩展运算符（`...`）、解构赋值和`Array.from`方法内部调用的，都是**遍历器接口***。这意味着，它们都可以将Generator函数返回的Iterator对象，作为参数。

```js
function* numbers () {
  yield 1
  yield 2
  return 3
  yield 4
}

// 扩展运算符
[...numbers()] // [1, 2]

// Array.form 方法
Array.from(numbers()) // [1, 2]

// 解构赋值
let [x, y] = numbers();
x // 1
y // 2

// for...of 循环
for (let n of numbers()) {
  console.log(n)
}
// 1
// 2
```



## Generator.prototype.throw()

详细请参见参考链接

## Generator.prototype.return()

Generator函数返回的遍历器对象，还有一个`return`方法，可以返回给定的值，并且终结遍历Generator函数。

```js
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

var g = gen();

g.next()        // { value: 1, done: false }
g.return('foo') // { value: "foo", done: true }
g.next()        // { value: undefined, done: true }
```

如果`return`方法调用时，不提供参数，则返回值的`value`属性为`undefined`。

如果Generator函数内部有`try...finally`代码块，那么`return`方法会推迟到`finally`代码块执行完再执行。

```js
function* numbers () {
  yield 1;
  try {
    yield 2;
    yield 3;
  } finally {
    yield 4;
    yield 5;
  }
  yield 6;
}
var g = numbers()
g.next() // { done: false, value: 1 }
g.next() // { done: false, value: 2 }
g.return(7) // { done: false, value: 4 }
g.next() // { done: false, value: 5 }
g.next() // { done: true, value: 7 }
```

上面代码中，调用`return`方法后，就开始执行`finally`代码块，然后等到`finally`代码块执行完，再执行`return`方法。

## yield*语句

如果在Generater函数内部，调用另一个Generator函数，默认情况下是没有效果的。

```js
function* foo() {
  yield 'a';
  yield 'b';
}

function* bar() {
  yield 'x';
  foo();
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "y"
```

这个就需要用到`yield*`语句，用来在一个Generator函数里面执行另一个Generator函数。

```js
function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "a"
// "b"
// "y"
```



## 作为对象属性的Generator函数

如果一个对象的属性是Generator函数，可以简写成下面的形式。

```js
let obj = {
  * myGeneratorMethod() {
    ···
  }
};
```

上面代码中，`myGeneratorMethod`属性前面有一个星号，表示这个属性是一个Generator函数。

它的完整形式如下，与上面的写法是等价的。

```js
let obj = {
  myGeneratorMethod: function* () {
    // ···
  }
};
```



## Generator函数的`this`

高级内容，详细请参见相关链接。

## 含义

### Generator与状态机

Generator是实现状态机的最佳结构。比如，下面的clock函数就是一个状态机

```js
ar ticking = true;
var clock = function() {
  if (ticking)
    console.log('Tick!');
  else
    console.log('Tock!');
  ticking = !ticking;
}
```

上面代码的clock函数一共有两种状态（Tick和Tock），每运行一次，就改变一次状态。这个函数如果用Generator实现，就是下面这样。

```js
var clock = function*() {
  while (true) {
    console.log('Tick!');
    yield;
    console.log('Tock!');
    yield;
  }
};
```

上面的Generator实现与ES5实现对比，可以看到少了用来保存状态的外部变量`ticking`，这样就更简洁，更安全（状态不会被非法篡改）、更符合函数式编程的思想，在写法上也更优雅。Generator之所以可以不用外部变量保存状态，是因为它本身就包含了一个状态信息，即目前是否处于暂停态。

### Generator与协程(有助于理解协程)

协程（coroutine）是一种程序运行的方式，可以理解成“**协作**的**线程**”或“**协作**的**函数**”。

协程既可以用单线程实现，也可以用多线程实现。前者是一种特殊的**子例程**，后者是一种特殊的**线程**。

1. **协程与子例程的差异**

   传统的“子例程”（subroutine）采用堆栈式“后进先出”的执行方式，只有当调用的子函数完全执行完毕，才会结束执行父函数。

   协程与其不同，多个线程（单线程情况下，即多个函数）可以并行执行，但是只有一个线程（或函数）处于正在运行的状态，其他线程（或函数）都处于暂停态（suspended），线程（或函数）之间可以交换执行权。

   也就是说，一个线程（或函数）执行到一半，可以暂停执行，将执行权交给另一个线程（或函数），等到稍后收回执行权的时候，再恢复执行。这种可以并行执行、交换执行权的线程（或函数），就称为协程。

   从实现上看，在内存中，子例程只使用一个栈（stack），而协程是同时存在多个栈，但只有一个栈是在运行状态，也就是说，协程是以多占用内存为代价，实现多任务的并行。

2. **协程与普通线程的差异**

   不难看出，协程适合用于多任务运行的环境。在这个意义上，它与普通的线程很相似，都有自己的执行上下文、可以分享全局变量。它们的不同之处在于，**同一时间可以有多个线程处于运行状态，但是运行的协程只能有一个**，其他协程都处于暂停状态。此外，普通的线程是抢先式的，到底哪个线程优先得到资源，必须由运行环境决定，但是协程是合作式的，执行权由协程自己分配。

由于ECMAScript是单线程语言，只能保持一个调用栈。引入协程以后，每个任务可以保持自己的调用栈。这样做的最大好处，就是抛出错误的时候，可以找到原始的调用栈。不至于像异步操作的回调函数那样，一旦出错，原始的调用栈早就结束。

Generator函数是ECMAScript 6对协程的实现，但属于不完全实现。Generator函数被称为“半协程”（semi-coroutine），意思是**只有Generator函数的调用者**，才能将程序的执行权还给Generator函数。如果是完全执行的协程，任何函数都可以让暂停的协程继续执行。

如果将Generator函数当作协程，完全可以将多个需要互相协作的任务写成Generator函数，它们之间使用yield语句交换控制权。

## 应用

Generator可以暂停函数执行，返回任意表达式的值。这种特点使得Generator有多种应用场景

### 异步操作的同步化表达

Generator函数的暂停执行的效果，意味着可以把异步操作写在yield语句里面，等到调用next方法时再往后执行。这实际上等同于不需要写回调函数了，因为异步操作的后续操作可以放在yield语句下面，反正要等到调用next方法时再执行。所以，Generator函数的一个重要实际意义就是用来处理异步操作，改写回调函数。

```js
function* loadUI() {
  showLoadingScreen();
  yield loadUIDataAsynchronously();
  hideLoadingScreen();
}
var loader = loadUI();
// 加载UI
loader.next()

// 卸载UI
loader.next()
```

上面代码表示，第一次调用loadUI函数时，该函数不会执行，仅返回一个遍历器。下一次对该遍历器调用next方法，则会显示Loading界面，并且异步加载数据。等到数据加载完成，再一次使用next方法，则会隐藏Loading界面。可以看到，这种写法的好处是所有Loading界面的逻辑，都被封装在一个函数，按部就班非常清晰。

Ajax是典型的异步操作，通过Generator函数部署Ajax操作，可以用同步的方式表达。

```js
function* main() {
  var result = yield request("http://some.url");
  var resp = JSON.parse(result);
    console.log(resp.value);
}

function request(url) {
  makeAjaxCall(url, function(response){
    it.next(response);
  });
}

var it = main();
it.next();
```

上面代码的main函数，就是通过Ajax操作获取数据。可以看到，除了多了一个yield，它几乎与同步操作的写法完全一样。注意，makeAjaxCall函数中的next方法，必须加上response参数，因为yield语句构成的表达式，本身是没有值的，总是等于undefined。



### 控制流管理

如果有一个多步操作非常耗时，采用回调函数，可能会写成下面这样。

```js
step1(function (value1) {
  step2(value1, function(value2) {
    step3(value2, function(value3) {
      step4(value3, function(value4) {
        // Do something with value4
      });
    });
  });
});
```

采用Promise改写上面的代码。

```js
Promise.resolve(step1)
  .then(step2)
  .then(step3)
  .then(step4)
  .then(function (value4) {
    // Do something with value4
  }, function (error) {
    // Handle any error from step1 through step4
  })
  .done();
```

上面代码已经把回调函数，改成了直线执行的形式，但是加入了大量Promise的语法。Generator函数可以进一步改善代码运行流程。

```js
function* longRunningTask(value1) {
  try {
    var value2 = yield step1(value1);
    var value3 = yield step2(value2);
    var value4 = yield step3(value3);
    var value5 = yield step4(value4);
    // Do something with value4
  } catch (e) {
    // Handle any error from step1 through step4
  }
}
```

然后，使用一个函数，按次序自动执行所有步骤。

```js
//没看明白，task.value是什么？为啥还要调用自身？
function scheduler(task) {
  var taskObj = task.next(task.value);
  // 如果Generator函数未结束，就继续调用
  if (!taskObj.done) {
    task.value = taskObj.value
    scheduler(task);
  }
}
scheduler(longRunningTask(initialValue));
```

更多详细内容，请参见参考链接。

### 部署Iterator接口

利用Generator函数，可以在任意对象上部署Iterator接口。

更多详细内容，请参见参考链接。



### 作为数据结构

更多详细内容，请参见参考链接。