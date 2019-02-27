# ES6 Class

## class的基本语法

### 概述

JavaScript语言的传统方法是通过构造函数，定义并生成新对象。下面是一个例子:

```js
function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function () {
  return '(' + this.x + ', ' + this.y + ')';
};

var p = new Point(1, 2);
```

ES6提供了更接近传统语言的写法，引入了Class（类）这个概念，作为对象的模板。通过`class`关键字，可以定义类。基本上，ES6的`class`可以看作只是一个**语法糖**，它的绝大部分功能，ES5都可以做到，新的`class`写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。

上例中的`constructor`方法，这就是构造方法，而`this`关键字则代表实例对象。也就是说，ES5的构造函数`Point`，对应ES6的`Point`类的构造方法。

定义“类”的方法的时候，前面不需要加上`function`这个关键字，直接把函数定义放进去了就可以了。另外，方法之间不需要逗号分隔，加了会报错。

***ES6的类，完全可以看作构造函数的另一种写法。***

```js
class Point {
  // ...
}
//prototype是函数的一个属性，并且是函数的原型对象。引用它的必然是函数
typeof Point // "function"
Point === Point.prototype.constructor // true
```

ES5以前构造函数的`prototype`属性，在ES6的“类”上面继续存在。事实上，类的所有方法都定义在类的。`prototype`属性上面。

```js
class Point {
  constructor(){
    // ...
  }

  toString(){
    // ...
  }

  toValue(){
    // ...
  }
}

// 等同于
//prototype是函数的一个属性，并且是函数的原型对象。引用它的必然是函数
Point.prototype = {
  toString(){},
  toValue(){}
};
```

在类的实例上面调用方法，其实就是调用原型上的方法。

```js
class B {}
let b = new B();

b.constructor === B.prototype.constructor // true
```

上面代码中，`b`是B类的实例，它的`constructor`方法就是B类原型的`constructor`方法。

由于类的方法都定义在`prototype`对象上面，所以类的新方法可以添加在`prototype`对象上面。`Object.assign`方法可以很方便地一次向类添加多个方法。

```js
class Point {
  constructor(){
    // ...
  }
}

Object.assign(Point.prototype, {
  toString(){},
  toValue(){}
});
```

`prototype`对象的`constructor`属性，直接指向“类”的本身，这与ES5的行为是一致的。

类的属性名，可以采用表达式。

```js
let methodName = "getArea";
class Square{
  constructor(length) {
    // ...
  }

  [methodName]() {
    // ...
  }
}
```

### constructor 方法

`constructor`方法是类的默认方法，通过`new`命令生成对象实例时，自动调用该方法。一个类必须有`constructor`方法，如果没有显式定义，一个空的`constructor`方法会被默认添加。

`constructor`方法**默认**返回实例对象（即`this`），完全可以指定返回**另外一个对象**。

```js
class Foo {
  constructor() {
    return Object.create(null);
  }
}

new Foo() instanceof Foo
// false
```

上面代码中，`constructor`函数返回一个全新的对象，结果导致实例对象不是`Foo`类的实例。

类的构造函数，不使用`new`是没法调用的，会报错。这是它跟普通构造函数的一个主要区别，后者不用`new`也可以执行。

### 类的实例对象

实例化类时，如果忘记加上`new`，像函数那样调用`Class`，将会报错。

```js
// 报错
var point = Point(2, 3);

// 正确
var point = new Point(2, 3);
```



与ES5一样，实例的属性除非显式定义在其本身（即定义在`this`对象上），否则都是定义在原型上（即定义在`class`上）。

```js
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }

}

var point = new Point(2, 3);

point.toString() // (2, 3)

point.hasOwnProperty('x') // true
point.hasOwnProperty('y') // true
point.hasOwnProperty('toString') // false
point.__proto__.hasOwnProperty('toString') // true
```

与ES5一样，类的所有实例共享一个原型对象。

```js
var p1 = new Point(2,3);
var p2 = new Point(3,2);

//__proto__和对象相关，prototype和对象的构建函数相关
console.log(p1.__proto__ === p2.__proto__)
//true
console.log(p1.__proto__ === Point.prototype)
//true
```

```js
var p1 = new Point(2,3);
var p2 = new Point(3,2);

p1.__proto__.printName = function () { return 'Oops' };

p1.printName() // "Oops"
p2.printName() // "Oops"

var p3 = new Point(4,2);
p3.printName() // "Oops"
```

使用实例的`__proto__`属性改写原型，必须相当谨慎，不推荐使用，因为这会改变Class的原始定义，影响到所有实例。

### 不存在变量提升

Class不存在变量提升（hoist），这一点与ES5完全不同

```js
new Foo(); // ReferenceError
class Foo {}
```

### Class表达式

与函数一样，类也可以使用表达式的形式定义

```js
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
let inst = new MyClass();
inst.getClassName() // Me
Me.name // ReferenceError: Me is not defined
```

上面代码使用表达式定义了一个类。需要注意的是，这个类的名字是`MyClass`而不是`Me`，`Me`只在Class的内部代码可用，指代当前类。(其根源就是class其实是构建函数的语法糖)

`Me`只在Class内部有定义。

如果类的内部没用到的话，可以省略`Me`，也就是可以写成下面的形式。

```js
const MyClass = class { /* ... */ };
```

采用Class表达式，可以写出立即执行的Class。

```js
let person = new class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}('张三');

person.sayName(); // "张三"
```

### 私有方法

私有方法是常见需求，但ES6**不提供**，只能通过变通方法模拟实现。

一种做法是在命名上加以区别。

```js
class Widget {

  // 公有方法
  foo (baz) {
    this._bar(baz);
  }

  // 私有方法
  _bar(baz) {
    return this.snaf = baz;
  }

  // ...
}
```

`_bar`方法前面的下划线，表示这是一个只限于内部使用的私有方法。但是，这种命名是不保险的，在类的外部，还是可以调用到这个方法。

另一种方法就是索性将私有方法移出模块，因为模块内部的所有方法都是对外可见的。

上面代码中，`foo`是公有方法，内部调用了`bar.call(this, baz)`。这使得`bar`实际上成为了当前模块的私有方法。

还有一种方法是利用`Symbol`值的唯一性，将私有方法的名字命名为一个`Symbol`值。

```javascript
const bar = Symbol('bar');
const snaf = Symbol('snaf');

export default class myClass{

  // 公有方法
  foo(baz) {
    this[bar](baz);
  }

  // 私有方法
  [bar](baz) {
    return this[snaf] = baz;
  }

  // ...
};
```

`bar`和`snaf`都是`Symbol`值，导致第三方无法获取到它们，因此达到了私有方法和私有属性的效果。

### 严格模式

类和模块的内部，默认就是严格模式，所以不需要使用`use strict`指定运行模式。只要你的代码写在类或模块之中，就只有严格模式可用。

考虑到未来所有的代码，其实都是运行在模块之中，所以ES6实际上把整个语言升级到了严格模式。

### name属性

由于本质上，ES6的类只是ES5的构造函数的一层包装，所以函数的许多特性都被`Class`继承，包括`name`属性。

```JS
class Point {}
Point.name // "Point"
```

`name`属性总是返回紧跟在`class`关键字后面的类名。

## Class的继承

### 基本用法

Class之间可以通过`extends`关键字实现继承，这比ES5的通过修改原型链实现继承，要清晰和方便很多。

```js
class ColorPoint extends Point {}
```

我们在`ColorPoint`内部加上代码。

```js
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}
```

子类**必须**在`constructor`方法中调用`super`方法，否则新建实例时会报错。这是因为子类没有自己的`this`对象，而是继承父类的`this`对象，然后对其进行加工。如果不调用`super`方法，子类就得不到`this`对象。

```js
class ColorPoint extends Point {
  constructor() {
  }
}

let cp = new ColorPoint(); // ReferenceError
```



ES5的继承，实质是先创造子类的实例对象`this`，然后再将父类的方法添加到`this`上面（`Parent.apply(this)`）。ES6的继承机制完全不同，实质是先创造父类的实例对象`this`（所以必须先调用`super`方法），然后再用子类的构造函数修改`this`。

如果子类没有显式定义`constructor`方法，这个方法会被默认添加，代码如下。也就是说，不管有没有显式定义，任何一个子类都有`constructor`方法。

```js
constructor(...args) {
  super(...args);
}
```

另一个需要注意的地方是，在子类的构造函数中，只有调用`super`之后，才可以使用`this`关键字，否则会报错。这是因为子类实例的构建，是基于对父类实例加工，只有`super`方法才能返回父类实例。

```js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    this.color = color; // ReferenceError
    super(x, y);
    this.color = color; // 正确
  }
}
```

```js
let cp = new ColorPoint(25, 8, 'green');

cp instanceof ColorPoint // true
cp instanceof Point // true
```

上面代码中，实例对象`cp`同时是`ColorPoint`和`Point`两个类的实例，这与ES5的行为完全一致。

### 类的prototype属性和`__proto__`属性

大多数浏览器的ES5实现之中，每一个对象都有`__proto__`属性，指向对应的构造函数的prototype属性。Class作为构造函数的语法糖，同时有prototype属性和`__proto__`属性，因此同时存在两条继承链(类的继承链和对象的继承链)。

```js
class A {
}

class B extends A {
}

B.__proto__ === A // true (类B继承类A的静态属性)
B.prototype.__proto__ === A.prototype // true　（类B的实例继承类A的实例的属性）

//class A, class  B完成了以下任务
//1. 生成构造函数A, B, 
//2. 为把class A{},class B{}中定义的方法放到对应的prototype对象中。
//3. 把B.prototype.__proto__指向A.prototype, B的实例集成A的实例属性，组成原型链（链１）
//4. 把B.__proto__指向A, 类B继承类A的静态属性（链2）
```



关于 `__proto__`,prototype, constructor, 原型链：

```js
function A(a){
		this.a = a;
	}
console.log(A.prototype); //A {}
var aa = new A("123");
console.log(aa.__proto__); //A {}
console.log(aa.__proto__===A.prototype); //true，aa是A的实例对象
console.log(aa.constructor.prototype===A.prototype); //true
console.log(aa.constructor); //true
console.log(A.prototype.__proto__===Object.prototype ) //true，　通过__protoo__属性，形成原型链
console.log(A.__proto__===Function.prototype)　//true, A是Function的实例对象
//A.prototype: 是A类的可继承属性对象，　只有prototype中的属性，才是可继承的属性。
//aa.__proto__： aa实例指向其类A的可继承属性对象，继承树上的对象通过__proto__属性形成原型链
//aa.constructor:是A类，或A函数，或构建器A。
//之所以容易混淆，是因为JS里严格说，没有类，只有constructor, 两个东西是一个事情，有时又分开表述
```

### Extends的继承目标

`extends`关键字后面可以跟多种类型的值。

```js
class B extends A {
}
```

上面代码的`A`，只要是一个有`prototype`属性的函数，就能被`B`继承。由于函数都有`prototype`属性（除了`Function.prototype`函数），因此`A`可以是任意函数。

下面，讨论三种特殊情况。

第一种特殊情况，子类继承Object类:

```js
class A extends Object {
}

A.__proto__ === Object // true
A.prototype.__proto__ === Object.prototype // true
```

这种情况下，`A`其实就是构造函数`Object`的复制，`A`的实例就是`Object`的实例。

第二种特殊情况，不存在任何继承。

```javascript
class A {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === Object.prototype // true
```

A作为一个基类（即不存在任何继承），就是一个普通函数，所以直接继承`Funciton.prototype`。但是，`A`调用后返回一个空对象（即`Object`实例），所以`A.prototype.__proto__`指向构造函数（`Object`）的`prototype`属性。

第三种特殊情况，子类继承`null`。

```js
class A extends null {
}

A.__proto__ === Function.prototype // true
A.prototype.__proto__ === undefined // true
```

### Object.getPrototypeOf()

`Object.getPrototypeOf`方法可以用来从子类上获取父类。

```js
Object.getPrototypeOf(ColorPoint) === Point
// true
```

可以使用这个方法判断，一个类是否继承了另一个类。

### super关键字

`super`这个关键字，有两种用法，含义不同。

1. 作为函数调用时（即`super(...args)`），`super`代表父类的构造函数。
2. 作为对象调用时（即`super.prop`或`super.method()`），`super`代表父类。注意，此时`super` **即**可以引用父类实例的属性和方法，**也**可以引用父类的静态方法。

```js
class B extends A {
  get m() {
    return this._p * super._p;
  }
  set m() {
    throw new Error('该属性只读');
  }
}
```

上面代码中，子类通过`super`关键字，调用父类实例的`_p`属性。

### 实例的`__proto__`属性

子类实例的`__proto__`属性的`__proto__`属性，指向父类实例的`__proto__`属性。也就是说，子类的原型的原型，是父类的原型。

简单的说就是：子类实例的爷爷类是父类实例的父类。

### 原生构造函数的继承

原生构造函数是指语言内置的构造函数，通常用来生成数据结构。ECMAScript的原生构造函数大致有下面这些。

- Boolean()
- Number()
- String()
- Array()
- Date()
- Function()
- RegExp()
- Error()
- Object()

以前，这些原生构造函数是无法继承的，比如，不能自己定义一个`Array`的子类。

```js
function MyArray() {
  Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
  constructor: {
    value: MyArray,
    writable: true,
    configurable: true,
    enumerable: true
  }
});

```

上面代码定义了一个继承Array的`MyArray`类。但是，这个类的行为与`Array`完全不一致。

```js
var colors = new MyArray();
colors[0] = "red";
colors.length  // 0

colors.length = 0;
colors[0]  // "red"
```

之所以会发生这种情况，是因为子类无法获得原生构造函数的内部属性，通过`Array.apply()`或者分配给原型对象都不行。

原生构造函数会**忽略**`apply`方法传入的`this`，也就是说，原生构造函数的`this`无法绑定，导致拿不到内部属性。

**ES6允许继承原生构造函数定义子类**，因为ES6是先新建父类的实例对象`this`，然后再用子类的构造函数修饰`this`，使得父类的所有行为都可以继承。下面是一个继承`Array`的例子。

## Class的取值函数（getter）和存值函数（setter）

与ES5一样，在Class内部可以使用`get`和`set`关键字，

对某个属性设置存值函数和取值函数，就可以**拦截**该属性的存取行为。

存值函数和取值函数是设置在属性的descriptor对象上的。

## Class的Generator方法

如果某个方法之前加上星号（`*`），就表示该方法是一个Generator函数。

```js
class Foo {
  constructor(...args) {
    this.args = args;
  }
  * [Symbol.iterator]() {
    for (let arg of this.args) {
      yield arg;
    }
  }
}

for (let x of new Foo('hello', 'world')) {
  console.log(x);
}
```

上面代码中，Foo类的Symbol.iterator方法前有一个星号，表示该方法是一个Generator函数。Symbol.iterator方法返回一个Foo类的默认遍历器，for...of循环会自动调用这个遍历器。



## Class的静态方法

类相当于实例的原型，所有在类中定义的方法，都会被实例继承。

如果在一个方法前，加上`static`关键字，就表示该方法不会被实例继承，而是直接通过类来调用，这就称为“静态方法”。

上面代码中，`Foo`类的`classMethod`方法前有`static`关键字，表明该方法是一个静态方法，可以直接在`Foo`类上调用（`Foo.classMethod()`），而不是在`Foo`类的实例上调用。如果在实例上调用静态方法，会抛出一个错误，表示不存在该方法。

父类的静态方法，可以被子类继承。

```js
class Foo {
  static classMethod() {
    return 'hello';
  }
}

class Bar extends Foo {
}

Bar.classMethod(); // 'hello'
```

静态方法也是可以从`super`对象上调用的。



## Class的静态属性和实例属性

静态属性指的是Class本身的属性，即`Class.propname`，而不是定义在实例对象（`this`）上的属性。

```js
class Foo {
}

Foo.prop = 1;
Foo.prop // 1
```

上面的写法为`Foo`类定义了一个静态属性`prop`。

目前，只有这种写法可行，因为ES6明确规定，Class内部只有静态方法，没有静态属性。ES7有一个静态属性的[提案](https://github.com/jeffmo/es-class-properties)，目前Babel转码器支持。

## new.target属性

## Mixin模式的实现









