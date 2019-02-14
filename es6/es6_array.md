# ES6数组扩展  
[TOC]

## Array.from

Array.from 方法用于将以下两类对象转化为真正的数组
* 类似数组的对象(Array like object，拥有length属性的都是)
* 可遍历的对象(实现了iterator接口)



**扩展运算符** (...para) 也可以将某些数据结构转为数组，其背后实现是扩展运算符调用的是遍历器接口（Symbol.iterator），如果一个对象没有部署这个接口，就无法转换。  

```js
// arguments对象
function foo() {
  var args = [...arguments];
}

// NodeList对象
[...document.querySelectorAll('div')]
```

类数组对象的本质只有一点：必需拥有length属性
```js
Array.from({ length: 3 });
// [ undefined, undefined, undefined ]
```

Array.from还可以接受第二个参数，作用类似于数组的map方法，用来对每个元素进行处理，将处理后的值放入返回的数组。　　
```js
Array.from(arrayLike, x => x * x);
// 等同于
Array.from(arrayLike).map(x => x * x);

Array.from([1, 2, 3], (x) => x * x)
// [1, 4, 9]
```

如果map函数里面用到了this关键字，还可以传入Array.from的第三个参数，用来绑定this。  
Array.from()可以将各种值转为真正的数组，并且还提供map功能。这实际上意味着，只要有一个原始的数据结构，你就可以先对它的值进行处理，然后转成规范的数组结构，进而就可以使用数量众多的数组方法。

Array.from()的另一个应用是，将**字符串转为数组**，然后返回字符串的长度。因为它能++正确处理各种Unicode字符++，可以避免JavaScript将大于\uFFFF的Unicode字符，算作两个字符的bug。

## Array.of

Array.of方法用于将一组值，转换为数组
```js
Array.of(3, 11, 8) // [3,11,8]
Array.of(3) // [3]
Array.of(3).length // 1
```
这个方法的主要目的，是弥补数组构造函数Array()的不足。因为参数个数的不同，会导致Array()的行为有差异。
```js
Array() // []
Array(3) // [, , ,]
Array(3, 11, 8) // [3, 11, 8]
```

Array.of的行为很统一
```js
Array.of() // []
Array.of(undefined) // [undefined]
Array.of(1) // [1]
Array.of(1, 2) // [1, 2]
```

## ES6提供三个新的方法——entries()，keys()和values()——用于遍历数组。
它们都返回一个遍历器对象（详见《Iterator》一章），可以用for...of循环进行遍历，唯一的区别是keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历。

```js
for (let index of ['a', 'b'].keys()) {
  console.log(index);
}
// 0
// 1

for (let elem of ['a', 'b'].values()) {
  console.log(elem);
}
// 'a'
// 'b'

for (let [index, elem] of ['a', 'b'].entries()) {
  console.log(index, elem);
}
// 0 "a"
// 1 "b"
```

## 数组的空位
数组的空位指，数组的某一个位置没有任何值。比如，Array构造函数返回的数组都是空位。
```js
Array(3) // [, , ,]
```
ES5对空位的处理，已经很不一致了，大多数情况下会忽略空位。

* forEach(), filter(), every() 和some()都会跳过空位。
* map()会跳过空位，但会保留这个值
* join()和toString()会将空位视为undefined，而undefined和null会被处理成空字符串。

ES6则是明确将空位转为undefined。  

由于空位的处理规则非常不统一，所以建议避免出现空位。











