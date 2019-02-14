# ES6对象的扩展

## 属性的简洁表示法

```js
var birth = '2000/01/01';

var Person = {

  name: '张三',

  //等同于birth: birth
  birth,

  // 等同于hello: function ()...
  hello() { console.log('我的名字是', this.name); }

};
```

## 属性名表达式
JavaScript语言定义对象的属性，有两种方法。

```js
// 方法一
obj.foo = true;

// 方法二
obj['a' + 'bc'] = 123;
```
上面代码的方法一是直接用标识符作为属性名，方法二是用表达式作为属性名，这时要将表达式放在方括号之内。

ES6的例子　　
```js
var lastWord = 'last word';

var a = {
  'first word': 'hello',
  [lastWord]: 'world'
};

a['first word'] // "hello"
a[lastWord] // "world"
a['last word'] // "world"

let obj = {
  ['h'+'ello']() {
    return 'hi';
  }
};

obj.hello() // hi

```
> 注意，属性名表达式与简洁表示法，不能同时使用，会报错。

## 方法的name属性

如果对象的方法是一个Symbol值，那么name属性返回的是这个Symbol值的描述。
```js
const key1 = Symbol('description');
const key2 = Symbol();
let obj = {
  [key1]() {},
  [key2]() {},
};
obj[key1].name // "[description]"
obj[key2].name // ""

```

## Object.is()
ES5比较两个值是否相等，只有两个运算符：相等运算符（==）和严格相等运算符（===）。它们都有缺点，前者会自动转换数据类型，后者的NaN不等于自身，以及+0等于-0。JavaScript缺乏一种运算，在所有环境中，只要两个值是一样的，它们就应该相等。  

ES6提出“Same-value equality”（同值相等）算法，用来解决这个问题。Object.is就是部署这个算法的新方法。它用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致。

undefined和null无法转成对象。   
基本数据类型有6种：Undefined、Null、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object）。
这里新添加了一种：Symbol

## 属性的可枚举性
对象的每个属性都有一个描述对象（Descriptor），用来控制该属性的行为。Object.getOwnPropertyDescriptor方法可以获取该属性的描述对象。

```js
let obj = { foo: 123 };
Object.getOwnPropertyDescriptor(obj, 'foo')
//  {
//    value: 123,
//    writable: true,
//    enumerable: true,
//    configurable: true
//  }
```
描述对象的enumerable属性，称为”可枚举性“，如果该属性为false，就表示某些操作会忽略当前属性。  
引入enumerable的最初目的，就是让某些属性可以规避掉for...in操作。

## 属性的遍历
ES6一共有5种方法可以遍历对象的属性。

1. for...in
for...in循环遍历对象自身的和继承的可枚举属性（不含Symbol属性）。

2. Object.keys(obj)
Object.keys返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含Symbol属性）。

3. Object.getOwnPropertyNames(obj)
Object.getOwnPropertyNames返回一个数组，包含对象自身的所有属性（不含Symbol属性，但是包括不可枚举属性）。

4. Object.getOwnPropertySymbols(obj)
Object.getOwnPropertySymbols返回一个数组，包含对象自身的所有Symbol属性。

5. Reflect.ownKeys(obj)
Reflect.ownKeys返回一个数组，包含对象自身的所有属性，不管是属性名是Symbol或字符串，也不管是否可枚举。

以上的5种方法遍历对象的属性，都遵守同样的属性遍历的次序规则。

- 首先遍历所有属性名为数值的属性，按照数字排序。
- 其次遍历所有属性名为字符串的属性，按照生成时间排序。
- 最后遍历所有属性名为Symbol值的属性，按照生成时间排序。

## `__proto__`属性，Object.setPrototypeOf()，Object.getPrototypeOf()

### `__proto__`属性
1. prototype专属于构造函数，在使用构造函数new出来的对象中，使用`__proto__`表示。
2. prototype对象中包含的属性(包括函数属性)被使用构造函数构建的对象所共享。从某种意义上来说，prototype对象就是父对象。

`__proto__`属性（前后各两个下划线），用来读取或设置当前对象的prototype对象。

该属性没有写入ES6的正文，而是写入了附录，原因是`__proto__`前后的双下划线，说明它本质上是一个内部属性，而不是一个正式的对外的API，只是由于浏览器广泛支持，才被加入了ES6。标准明确规定，只有浏览器必须部署这个属性，其他运行环境不一定需要部署，而且新的代码最好认为这个属性是不存在的。


### Object.setPrototypeOf()
Object.setPrototypeOf方法的作用与`__proto__`相同，用来设置一个对象的prototype对象。它是ES6正式推荐的设置原型对象的方法。

```js
// 格式
Object.setPrototypeOf(object, prototype)

// 用法
var o = Object.setPrototypeOf({}, null);
```



