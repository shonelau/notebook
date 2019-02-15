# ES6 Iterator & for ... of

[TOC]



>Iterator接口主要供for...of消费。

## Iterator（遍历器）的概念
JavaScript原有的表示“集合”的数据结构，主要是数组（Array）和对象（Object），ES6又添加了Map和Set。这样就有了四种数据集合。  

用户还可以组合使用它们，定义自己的数据结构，比如数组的成员是Map，Map的成员是对象。这样就需要一种统一的接口机制，来处理所有不同的数据结构。

遍历器（Iterator）就是这样一种机制。它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署Iterator接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。

Iterator的作用有三个：一是为各种数据结构，提供一个统一的、简便的访问接口；二是使得数据结构的成员能够按某种次序排列；三是ES6创造了一种新的遍历命令for...of循环，Iterator接口主要供for...of消费。

Iterator的遍历过程是这样的。
1. 创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象。
2. 第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员。
3. 第二次调用指针对象的next方法，指针就指向数据结构的第二个成员。
4. 不断调用指针对象的next方法，直到它指向数据结构的结束位置。

每一次调用next方法，都会返回数据结构的当前成员的信息。具体来说，就是返回一个包含value和done两个属性的对象。其中，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。

下面是一个模拟next方法返回值的例子。

```js
var it = makeIterator(['a', 'b']);

it.next() // { value: "a", done: false }
it.next() // { value: "b", done: false }
it.next() // { value: undefined, done: true }

function makeIterator(array) {
  var nextIndex = 0;
  return {
    next: function() {
      return nextIndex < array.length ?
        {value: array[nextIndex++], done: false} :
        {value: undefined, done: true};
    }
  };
}
```

在ES6中，有些数据结构原生具备Iterator接口（比如数组），即不用任何处理，就可以被for...of循环遍历，有些就不行（比如对象）。  

原因在于，这些数据结构原生部署了Symbol.iterator属性（详见下文），另外一些数据结构没有。凡是部署了Symbol.iterator属性的数据结构，就称为部署了遍历器接口。调用这个接口，就会返回一个遍历器对象。



## 数据结构默认Iterator接口

Iterator接口的目的，就是为所有数据结构，提供了一种统一的访问机制，即`for...of`循环（详见下文）。当使用`for...of`循环遍历某种数据结构时，该循环会自动去寻找Iterator接口。　　

ES6规定，默认的Iterator接口部署在数据结构的`Symbol.iterator`属性，或者说，一个数据结构只要具有`Symbol.iterator`属性，就可以认为是“可遍历的”（iterable）。

调用`Symbol.iterator`方法，就会得到当前数据结构默认的遍历器生成函数。

`Symbol.iterator`本身是一个**表达式**，返回Symbol对象的`iterator`属性，这是一个预定义好的、类型为Symbol的特殊值，所以要放在方括号内（请参考Symbol一章）。

在ES6中，有三类数据结构原生具备Iterator接口：

1. **数组**
2. **某些类似数组的对象**
3. **Set和Map结构**

除此之外，其他数据结构（主要是对象）的Iterator接口，都需要自己在`Symbol.iterator`属性上面部署，这样才会被`for...of`循环遍历（原型链上的对象具有该方法也可）。本质上，遍历器是一种线性处理，对于任何非线性的数据结构，部署遍历器接口，就等于部署一种线性转换。

```js
class RangeIterator {
  constructor(start, stop) {
    this.value = start;
    this.stop = stop;
  }

  [Symbol.iterator]() { return this; }

  next() {
    var value = this.value;
    if (value < this.stop) {
      this.value++;
      return {done: false, value: value};
    } else {
      return {done: true, value: undefined};
    }
  }
}

function range(start, stop) {
  return new RangeIterator(start, stop);
}

for (var value of range(0, 3)) {
  console.log(value);
}
```

有了遍历器接口，数据结构就可以用`for...of`循环遍历（详见下文），也可以使用`while`循环遍历。

```js
var $iterator = ITERABLE[Symbol.iterator]();
var $result = $iterator.next();
while (!$result.done) {
  var x = $result.value;
  // ...
  $result = $iterator.next();
}
```

上面代码中，`ITERABLE`代表某种可遍历的数据结构，`$iterator`是它的遍历器对象。遍历器对象每次移动指针（`next`方法），都检查一下返回值的`done`属性，如果遍历还没结束，就移动遍历器对象的指针到下一步（`next`方法），不断循环。

其他更多实例，请参见参考链接

## 调用iterator接口的场合

有一些场合会默认调用Iterator接口（即`Symbol.iterator`方法）

### 解构赋值

对数组和Set结构进行解构赋值时，会默认调用`Symbol.iterator`方法。

```js
let set = new Set().add('a').add('b').add('c');

let [x,y] = set;
// x='a'; y='b'

let [first, ...rest] = set;
// first='a'; rest=['b','c'];
```

### 扩展运算符

扩展运算符（...）也会调用默认的iterator接口。扩展运算符就是用来生成数组的！

```js
// 例一
var str = 'hello';
[...str] //  ['h','e','l','l','o']

// 例二
let arr = ['b', 'c'];
['a', ...arr, 'd']
// ['a', 'b', 'c', 'd']
```

实际上，这提供了一种简便机制，可以将任何部署了Iterator接口的数据结构，转为数组。也就是说，只要某个数据结构部署了Iterator接口，就可以对它使用扩展运算符，将其转为数组。

```js
let arr = [...iterable];
```

### `yield*`

`yield*`后面跟的是一个可遍历的结构，它会调用该结构的遍历器接口。

```js
let generator = function* () {
  yield 1;
  yield* [2,3,4];
  yield 5;
};

var iterator = generator();

iterator.next() // { value: 1, done: false }
iterator.next() // { value: 2, done: false }
iterator.next() // { value: 3, done: false }
iterator.next() // { value: 4, done: false }
iterator.next() // { value: 5, done: false }
iterator.next() // { value: undefined, done: true }
```

### 其他场合

由于数组的遍历会调用遍历器接口，所以任何接受数组作为参数的场合，其实都调用了遍历器接口。下面是一些例子。

- for...of
- Array.from()
- Map(), Set(), WeakMap(), WeakSet()（比如`new Map([['a',1],['b',2]])`）
- Promise.all()
- Promise.race()

## 字符串的iterator接口

字符串是一个类似数组的对象，也原生具有Iterator接口。可以覆盖原生的`Symbol.iterator`方法，达到修改遍历器行为的目的。

```js
var str = new String("hi");

[...str] // ["h", "i"]

str[Symbol.iterator] = function() {
  return {
    next: function() {
      if (this._first) {
        this._first = false;
        return { value: "bye", done: false };
      } else {
        return { done: true };
      }
    },
    _first: true
  };
};

[...str] // ["bye"]
str // "hi"
```

上面代码中，字符串str的`Symbol.iterator`方法被修改了，所以扩展运算符（`...`）返回的值变成了`bye`，而字符串本身还是`hi`。

## iterator接口和Generator函数

`Symbol.iterator`方法的最简单实现，是使用Generator函数。

```js
var myIterable = {};

myIterable[Symbol.iterator] = function* () {
  yield 1;
  yield 2;
  yield 3;
};
[...myIterable] // [1, 2, 3]

// 或者采用下面的简洁写法

let obj = {
  * [Symbol.iterator]() {
    yield 'hello';
    yield 'world';
  }
};

for (let x of obj) {
  console.log(x);
}
// hello
// world
```

## 遍历器对象的return()和throw()

遍历器对象除了具有`next`方法，还可以具有`return`方法和`throw`方法。如果你自己写遍历器对象生成函数，那么`next`方法是必须部署的，`return`方法和`throw`方法是否部署是可选的。

`return`方法的使用场合是，如果`for...of`循环提前退出（通常是因为出错，或者有`break`语句或`continue`语句），就会调用`return`方法。如果一个对象在完成遍历前，需要清理或释放资源，就可以部署`return`方法。

```js
function readLinesSync(file) {
  return {
    next() {
      if (file.isAtEndOfFile()) {
        file.close();
        return { done: true };
      }
    },
    return() {
      file.close();
      return { done: true };
    },
  };
}

for (let line of readLinesSync(fileName)) {
  console.log(x);
  break;
}
```

我们让文件的遍历提前返回，这样就会触发执行`return`方法。

注意，`return`方法必须返回一个对象，这是Generator规格决定的。

`throw`方法主要是配合Generator函数使用，一般的遍历器对象用不到这个方法。

## for..of和for..in

一个数据结构只要部署了`Symbol.iterator`属性，就被视为具有iterator接口，就可以用`for...of`循环遍历它的成员。

### 数组

数组原生具备iterator接口，`for...of`循环本质上就是调用这个接口产生的遍历器，可以用下面的代码证明。

```js
const arr = ['red', 'green', 'blue'];
let iterator  = arr[Symbol.iterator]();

for(let v of arr) {
  console.log(v); // red green blue
}

for(let v of iterator) {
  console.log(v); // red green blue
}
```

上面代码的`for...of`循环的两种写法是等价的。

`for...of`循环可以代替数组实例的`forEach`方法。

```js
const arr = ['red', 'green', 'blue'];

arr.forEach(function (element, index) {
  console.log(element); // red green blue
  console.log(index);   // 0 1 2
});
```

JavaScript原有的`for...in`循环，只能获得对象的键名，不能直接获取键值。ES6提供`for...of`循环，允许遍历获得键值。

```js
var arr = ['a', 'b', 'c', 'd'];

for (let a in arr) {
  console.log(a); // 0 1 2 3
}

for (let a of arr) {
  console.log(a); // a b c d
}
```

上面代码表明，`for...in`循环读取键名，`for...of`循环读取键值。如果要通过`for...of`循环，获取数组的索引，可以借助数组实例的`entries`方法和`keys`方法，参见《数组的扩展》章节。

`for...of`循环调用遍历器接口，数组的遍历器接口只返回具有数字索引的属性。这一点跟`for...in`循环也不一样。如下例：

```js
let arr = [3, 5, 7];
arr.foo = 'hello';
//for in 返回全部数组的全部属性
for (let i in arr) {
  console.log(i); // "0", "1", "2", "foo"
}
//for of 只返回了数字索引属性，这是数组默认的[Sysmbol.iterator]属性方法决定的
for (let i of arr) {
  console.log(i); //  "3", "5", "7"
}
```

### set和map结构

Set和Map结构也原生具有Iterator接口，可以直接使用`for...of`循环。

```js
var engines = new Set(["Gecko", "Trident", "Webkit", "Webkit"]);
for (var e of engines) {
  console.log(e);
}
// Gecko
// Trident
// Webkit

var es6 = new Map();
es6.set("edition", 6);
es6.set("committee", "TC39");
es6.set("standard", "ECMA-262");
for (var [name, value] of es6) {
  console.log(name + ": " + value);
}
// edition: 6
// committee: TC39
// standard: ECMA-262
```

### 计算生成的数据结构

有些数据结构是在现有数据结构的基础上，计算生成的。比如，ES6的数组、Set、Map都部署了以下三个方法，调用后都返回遍历器对象。

- `entries()` 返回一个遍历器对象，用来遍历`[键名, 键值]`组成的数组。对于数组，键名就是索引值；对于Set，键名与键值相同。Map结构的iterator接口，默认就是调用entries方法。
- `keys()` 返回一个遍历器对象，用来遍历所有的键名。
- `values()` 返回一个遍历器对象，用来遍历所有的键值。

### 类似数组的对象
类似数组的对象包括好几类。下面是for...of循环用于**字符串**、**DOM NodeList对象**、**arguments对象**的例子。
```js
// 字符串
let str = "hello";

for (let s of str) {
  console.log(s); // h e l l o
}

// DOM NodeList对象
let paras = document.querySelectorAll("p");

for (let p of paras) {
  p.classList.add("test");
}

// arguments对象
function printArgs() {
  for (let x of arguments) {
    console.log(x);
  }
}
printArgs('a', 'b');
// 'a'
// 'b'
```

对于字符串来说，for...of循环还有一个特点，就是会**正确识别32位UTF-16字符**。
```js
for (let x of 'a\uD83D\uDC0A') {
  console.log(x);
}
// 'a'
// '\uD83D\uDC0A'
```

并不是所有类似数组的对象都具有iterator接口，一个简便的解决方法，就是使用Array.from方法将其转为数组。
```js
let arrayLike = { length: 2, 0: 'a', 1: 'b' };

// 报错
for (let x of arrayLike) {
  console.log(x);
}

// 正确
for (let x of Array.from(arrayLike)) {
  console.log(x);
}
```
### 对象
对于普通的对象，for...of结构不能直接使用，会报错，必须部署了iterator接口后才能使用。但是，这样情况下，for...in循环依然可以用来遍历键名。  
```js
var es6 = {
  edition: 6,
  committee: "TC39",
  standard: "ECMA-262"
};

for (e in es6) {
  console.log(e);
}
// edition
// committee
// standard

for (e of es6) {
  console.log(e);
}
// TypeError: es6 is not iterable
```

一种解决方法是，使用Object.keys方法将对象的键名生成一个数组，然后遍历这个数组。
```js
for (var key of Object.keys(someObject)) {
  console.log(key + ": " + someObject[key]);
}
```
在对象上部署iterator接口的代码，参见本章前面部分。一个方便的方法是将数组的Symbol.iterator属性，直接赋值给其他对象的Symbol.iterator属性。比如，想要让for...of环遍历jQuery对象，只要加上下面这一行就可以了。

```js
jQuery.prototype[Symbol.iterator] =
  Array.prototype[Symbol.iterator];
```
另一个方法是使用Generator函数将对象重新包装一下
```js
function* entries(obj) {
  for (let key of Object.keys(obj)) {
    yield [key, obj[key]];
  }
}

for (let [key, value] of entries(obj)) {
  console.log(key, "->", value);
}
// a -> 1
// b -> 2
// c -> 3
```
### 与其他遍历语法的比较(重点)
以数组为例，JavaScript提供多种遍历语法。最原始的写法就是for循环。
```js
for (var index = 0; index < myArray.length; index++) {
  console.log(myArray[index]);
}
```
这种写法比较麻烦，因此数组提供内置的forEach方法
```js
myArray.forEach(function (value) {
  console.log(value);
});
```
这种写法的问题在于，无法中途跳出forEach循环，break命令或return命令都不能奏效。

for...in循环可以遍历数组的键名。

```js
for (var index in myArray) {
  console.log(myArray[index]);
}
```
for...in循环有几个缺点。

- 数组的键名是数字，但是for...in循环是以**字符串作为键名**“0”、“1”、“2”等等。
- for...in循环不仅遍历数字键名，还会遍历手动添加的其他键，甚至包括原型链上的键。
- 某些情况下，for...in循环会以任意顺序遍历键名。
总之，for...in循环主要是为遍历对象而设计的，不适用于遍历数组。

for...of循环相比上面几种做法，有一些显著的优点。
- 有着同for...in一样的简洁语法，但是没有for...in那些缺点。
- 不同用于forEach方法，它可以与break、continue和return配合使用。
- 提供了遍历所有数据结构的统一操作接口。

下面是一个使用break语句，跳出for...of循环的例子。
```js
for (var n of fibonacci) {
  if (n > 1000)
    break;
  console.log(n);
}
```











