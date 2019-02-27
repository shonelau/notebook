# ES6 Promise对象

## Promise的含义

Promise是**异步**编程的一种解决方案，比传统的解决方案，回调函数和事件，更合理和更强大。它由社区最早提出和实现，ES6正式将其写进了语言标准，统一了用法，原生提供了`Promise`对象。

所谓`Promise`，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的**结果**。

从语法上说，*Promise是一个对象*，从它可以获取异步操作的消息。Promise提供统一的API，各种异步操作都可以用同样的方法进行处理。

1. 对象的状态不受外界影响。`Promise`对象代表一个异步操作，有三种状态：`Pending`（进行中）、`Resolved`（已完成，又称Fulfilled）和`Rejected`（已失败）。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是`Promise`这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。`Promise`对象的状态改变，只有两种可能：从`Pending`变为`Resolved`和从`Pending`变为`Rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果。就算改变已经发生了，你再对`Promise`对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

`Promise`也有一些缺点。

1. 无法取消`Promise`，一旦新建它就会立即执行，无法中途取消。
2. 如果不设置回调函数，`Promise`内部抛出的错误，不会反应到外部。
3. 当处于`Pending`状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成,也即无法细节控制）。

## 基本用法

ES6规定，Promise对象是一个构造函数，用来生成Promise实例。下面代码创造了一个Promise实例。

```js
var promise = new Promise(function(resolve, reject) {
  // ... some code

  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
```

两个参数分别是`resolve`和`reject`。它们是两个函数，由JavaScript引擎提供，不用自己部署。

`resolve`函数的作用是，将Promise对象的状态从“未完成”变为“成功”（即从Pending变为Resolved），在异步操作成功时调用，并将异步操作的结果，作为参数传递出去；

`reject`函数的作用是，将Promise对象的状态从“未完成”变为“失败”（即从Pending变为Rejected），在异步操作失败时调用，并将异步操作报出的错误，作为参数传递出去。

Promise实例生成以后，可以用`then`方法分别指定`Resolved`状态和`Reject`状态的回调函数。

```js
//注意下面是promise,小写p, 代表是Promise的一个实例对象
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

`then`方法可以接受两个回调函数作为参数。第一个回调函数是Promise对象的状态变为Resolved时调用，第二个回调函数是Promise对象的状态变为Reject时调用。其中，第二个函数是可选的，不一定要提供。这两个函数都接受Promise对象传出的值作为参数。

### 例子1

下面是一个Promise对象的简单例子。

```js
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
    //setTimeOut(func,time,func_para)
  });
}

timeout(100).then((value) => {
  console.log(value);
});
```

上面代码中，`timeout`方法返回一个Promise实例，表示一段时间以后才会发生的结果。过了指定的时间（`ms`参数）以后，Promise实例的状态变为Resolved，就会触发`then`方法绑定的回调函数。

### 例子2

Promise新建后就会立即执行。

```js
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('Resolved.');
});

console.log('Hi!');

// Promise
// Hi!
// Resolved
```

上面代码中，Promise新建后立即执行，所以首先输出的是“Promise”。然后，`then`方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行，所以“Resolved”最后输出。

### 例子3

下面是异步加载图片的例子。

```js
function loadImageAsync(url) {
  return new Promise(function(resolve, reject) {
    var image = new Image();

    image.onload = function() {
      resolve(image);
    };

    image.onerror = function() {
      reject(new Error('Could not load image at ' + url));
    };

    image.src = url;
  });
}
```

上面代码中，使用Promise包装了一个图片加载的异步操作。如果加载成功，就调用`resolve`方法，否则就调用`reject`方法。

### 例子4

下面是一个用Promise对象实现的Ajax操作的例子。

```js
var getJSON = function(url) {
  var promise = new Promise(function(resolve, reject){
    var client = new XMLHttpRequest();
    client.open("GET", url);
    client.onreadystatechange = handler;
    client.responseType = "json";
    client.setRequestHeader("Accept", "application/json");
    client.send();

    function handler() {
      if (this.readyState !== 4) {
        return;
      }
      if (this.status === 200) {
        resolve(this.response);
      } else {
        reject(new Error(this.statusText));
      }
    };
  });

  return promise;
};

getJSON("/posts.json").then(function(json) {
  console.log('Contents: ' + json);
}, function(error) {
  console.error('出错了', error);
});
```

如果调用`resolve`函数和`reject`函数时带有参数，那么它们的参数会被传递给回调函数。`reject`函数的参数通常是Error对象的实例，表示抛出的错误；`resolve`函数的参数除了正常的值以外，还可能是另一个Promise实例，表示异步操作的结果有可能是一个值，也有可能是另一个异步操作，比如像下面这样。

```js
var p1 = new Promise(function (resolve, reject) {
  // ...
});

var p2 = new Promise(function (resolve, reject) {
  // ...
  resolve(p1);
})
```

上面代码中，`p1`和`p2`都是Promise的实例，但是`p2`的`resolve`方法将`p1`作为参数，即一个异步操作的结果是返回另一个异步操作。

注意，这时`p1`的状态就会传递给`p2`，也就是说，`p1`的状态决定了`p2`的状态（promise对象只有三个状态pending,resolved,rejected）。

如果`p1`的状态是`Pending`，那么`p2`的回调函数(then指定的)就会等待`p1`的状态改变；如果`p1`的状态已经是`Resolved`或者`Rejected`，那么`p2`的回调函数将会立刻执行。

```js
var p1 = new Promise(function (resolve, reject) {
   setTimeout(() => reject(new Error('fail in 10 secs')), 10000)
})

var p2 = new Promise(function (resolve, reject) {
   setTimeout(() => {console.log("set p1 in 5 secs");resolve(p1)}, 5000)
})

p2
  .then(result => {console.log(result);console.log("then is never coming!");})
  .catch(error => console.log(error))
```

```js
var p1 = new Promise(function (resolve, reject) {
  //10秒后p1状态转为resolved
    setTimeout(() => resolve("this is p1 in 10secs"), 10000)
 })

var p2 = new Promise(function (resolve, reject) {
    ////5秒后,log,且根据p1的状态进行转换。
  setTimeout(() => {console.log("set p1 in 5 secs");resolve(p1)}, 5000)
})

p2
  .then(result => {console.log(result);console.log("then is coming! in 10secs");})
  .catch(error => console.log(error))

```

resolve和reject 方法对状态进行翻转，然后then和catch根据状态，进行对应的回调处理。

比如进行如下测试：

```js
var p1 = new Promise(function (resolve, reject) {
  //5秒后p1状态转为resolved
  setTimeout(() => resolve("this is p1 in 10secs"), 5000)
 })
```

隔15秒后，运行

```js
var p2 = new Promise(function (resolve, reject) {
  //2秒后,log,且根据p1的状态进行转换。
  setTimeout(() => {console.log("set p1 in 5 secs");resolve(p1)}, 2000)
})
//15+2秒后，p1状态已改变，立即进行回调处理，log,
p2
  .then(
    result => {
    	console.log(result);
    	console.log("then is coming! in 10secs");}
  ).catch(error => console.log(error))
```

### Promise.prototype.then()

romise实例具有`then`方法，也就是说，`then`方法是定义在原型对象Promise.prototype上的。它的作用是为Promise实例添加状态改变时的回调函数。前面说过，`then`方法的第一个参数是Resolved状态的回调函数，第二个参数（可选）是Rejected状态的回调函数。

<u>`then`方法返回的是一个**新的**Promise实例（注意，不是原来那个Promise实例）。</u>

因此可以采用链式写法，即`then`方法后面再调用另一个`then`方法。

```js
getJSON("/posts.json").then(function(json) {
  return json.post;
}).then(function(post) {
  // ...
});
```

上面的代码使用`then`方法，依次指定了两个回调函数。第一个回调函数完成以后，会将返回结果作为参数，传入第二个回调函数。

采用链式的`then`，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个新的Promise对象（即有异步操作），这时后一个回调函数，就会等待该Promise对象的状态发生变化，才会被调用。

```js
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function funcA(comments) {
  console.log("Resolved: ", comments);
}, function funcB(err){
  console.log("Rejected: ", err);
});
```

上面代码中，第一个`then`方法指定的回调函数，返回的是另一个Promise对象。这时，第二个`then`方法指定的回调函数，就会等待这个新的Promise对象状态发生变化。如果变为Resolved，就调用`funcA`，如果状态变为Rejected，就调用`funcB`。

如果采用箭头函数，上面的代码可以写得更简洁。

```javascript
getJSON("/post/1.json").then(
  post => getJSON(post.commentURL)
).then(
  comments => console.log("Resolved: ", comments),
  err => console.log("Rejected: ", err)
);
```

### Promise.prototype.catch()

`Promise.prototype.catch`方法是`.then(null, rejection)`的别名，用于指定发生错误时的回调函数。

```js
getJSON("/posts.json").then(function(posts) {
  // ...
}).catch(function(error) {
  // 处理 getJSON 和 前一个回调函数运行时发生的错误
  console.log('发生错误！', error);
});
```

```js
/ 写法一
var promise = new Promise(function(resolve, reject) {
  try {
    throw new Error('test');
  } catch(e) {
    reject(e);
  }
});
promise.catch(function(error) {
  console.log(error);
});

// 写法二
var promise = new Promise(function(resolve, reject) {
  reject(new Error('test'));
});
promise.catch(function(error) {
  console.log(error);
});
```

比较上面两种写法，可以发现`reject`方法的作用，等同于抛出错误。

如果Promise状态已经变成`Resolved`，再抛出错误是无效的。

```js
var promise = new Promise(function(resolve, reject) {
  resolve('ok');
  throw new Error('test');
});
promise
  .then(function(value) { console.log(value) })
  .catch(function(error) { console.log(error) });
// ok
```

上面代码中，Promise在`resolve`语句后面，再抛出错误，不会被捕获，等于没有抛出。

Promise对象的错误具有**“冒泡”**性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个`catch`语句捕获。

```js
getJSON("/post/1.json").then(function(post) {
  return getJSON(post.commentURL);
}).then(function(comments) {
  // some code
}).catch(function(error) {
  // 处理前面三个Promise产生的错误
});
```

上面代码中，一共有三个Promise对象：一个由`getJSON`产生，两个由`then`产生。它们之中任何一个抛出的错误，都会被最后一个`catch`捕获。

一般来说，不要在`then`方法里面定义Reject状态的回调函数（即`then`的第二个参数），总是使用`catch`方法。

更多细节讨论，参考相关连接

### Promise.all()

`Promise.all`方法用于将多个Promise实例，包装成一个新的Promise实例。

```js
var p = Promise.all([p1, p2, p3]);
```

上面代码中，`Promise.all`方法接受一个数组作为参数，`p1`、`p2`、`p3`都是Promise对象的实例，如果不是，就会先调用下面讲到的`Promise.resolve`方法，将参数转为Promise实例，再进一步处理。

`Promise.all`方法的参数可以不是数组，但必须具有**Iterator**接口，且返回的每个成员都是Promise实例。

`p`的状态由`p1`、`p2`、`p3`决定，分成两种情况。

1. 只有`p1`、`p2`、`p3`的状态都变成`fulfilled`，`p`的状态才会变成`fulfilled`，此时`p1`、`p2`、`p3`的返回值组成一个数组，传递给`p`的回调函数。

2. 只要`p1`、`p2`、`p3`之中有一个被`rejected`，`p`的状态就变成`rejected`，此时第一个被`reject`的实例的返回值，会传递给`p`的回调函数。

下面是一个具体的例子。

```js
// 生成一个Promise对象的数组
var promises = [2, 3, 5, 7, 11, 13].map(function (id) {
  return getJSON("/post/" + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});
```

上面代码中，`promises`是包含6个Promise实例的数组，只有这6个实例的状态都变成`fulfilled`，或者其中有一个变为`rejected`，才会调用`Promise.all`方法后面的回调函数。

下面是另一个例子。

```js
const databasePromise = connectDatabase();

//异步查询书籍
const booksPromise = databaseProimse
  .then(findAllBooks);
//异步查询用户
const userPromise = databasePromise
  .then(getCurrentUser);

//书籍和用户都返回时，才返回用户对书籍的评论
Promise.all([
  booksPromise,
  userPromise
])
.then(([books, user]) => pickTopRecommentations(books, user));
```



### Promise.race()

`Promise.race`方法同样是将多个Promise实例，包装成一个新的Promise实例。

```js
var p = Promise.race([p1,p2,p3]);
```

上面代码中，只要`p1`、`p2`、`p3`之中有一个实例率先改变状态，`p`的状态就跟着改变。那个率先改变的Promise实例的返回值，就传递给`p`的回调函数。

`Promise.race`方法的参数与`Promise.all`方法一样，如果不是Promise实例，就会先调用下面讲到的`Promise.resolve`方法，将参数转为Promise实例，再进一步处理。
下面是一个例子，如果指定时间内没有获得结果，就将Promise的状态变为`reject`，否则变为`resolve`

```js
var p = Promise.race([
  fetch('/resource-that-may-take-a-while'),
  new Promise(function (resolve, reject) {
    setTimeout(() => reject(new Error('request timeout')), 5000)
  })
])
p.then(response => console.log(response))
p.catch(error => console.log(error))
```



### Promise.resolve()

有时需要将现有对象转为Promise对象，`Promise.resolve`方法就起到这个作用。

```js
var jsPromise = Promise.resolve($.ajax('/whatever.json'));
```

上面代码将jQuery生成的`deferred`对象，转为一个新的Promise对象。

`Promise.resolve`等价于下面的写法。

```js
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

`Promise.resolve`方法的参数分成四种情况。

#### 参数是一个promise实例

如果参数是Promise实例，那么`Promise.resolve`将不做任何修改、原封不动地返回这个实例。

#### 参数是一个thenable对象

所谓`thenable`对象指的是具有`then`方法的对象，比如下面这个对象。

```js
let thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};
```

`Promise.resolve`方法会将这个对象转为Promise对象，然后就立即执行`thenable`对象的`then`方法

```js
var thenable = {
  then: function(resolve, reject) {
    resolve(42);
  }
};

var p1 = Promise.resolve(thenable);
p1.then(function(value) {
  console.log(value);  // 42
});
```

上面代码中，在执行Promise.resolve(thenable)后，立即自动调用`thenable`对象的`then`方法，对象`p1`的状态就变为`resolved`，从而立即执行最后那个`then`方法指定的回调函数，输出42。





### Promise.reject()

## 两个有用的附加方法

### done()

### finally()



## 应用



