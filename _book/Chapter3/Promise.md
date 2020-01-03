# Promise

## 1.Promise的含义

Promise是异步编程的一种解决方案。

所谓Promise,简单说就是一个容器，里面保存着某个未来才会结束的事件(异步操作)。从语法上讲，Promise是一个对象，从它可以获取异步操作的消息。Promise提供统一的API，各种异步操作都可以用同样的方法进行处理。

Promise对象的特点：

1. 对象的状态不受外界影响；Promise对象代表一个异步操作，有三种状态：pending(进行中)、fulfilled(已成功)、rejected(已失败)。
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果；Promise对象的状态改变的两种可能：
   1. 从pending到fulfilled；
   2. 从pending到rejected；

Promise的缺点：

1. 无法取消Promise，一旦新建就会立即执行，无法中途取消；
2. 如果不设置回调函数，Promise内部的抛出的错误，不会反应到外部；
3. 当处于pending状态时，无法得知目前进展到哪一个阶段；

## 2.基本用法

ES6规定，Promise对象是一个构造函数，用来生成Promise实例。

```
const promise = new Promise(function(resolve,reject) {
	if(succeed){
		resolve(value);
	}else{
	 	reject(value);
	}
})
```

## 3.Promise.prototype.then()

Promise实例具有then方法，也就是说，then方法是定义在原型对象Promise.prototype上的。

作用：

为Promise实例添加状态改变时的回调函数。

then方法的参数：

第一个参数是resoloved状态的回调函数，第二个参数(可选)是rejected状态的回调函数。

then方法返回的是一个新的Promise(注意，不是原来那个Promise实例)。因此可以采用链式写法，即then方法再调用另一个then方法。

## 4.Promise.prototype.catch()

Promise.prototype.catch()方法是.then(null,rejection)或.then(undefined,rejection)的别名，用于指定发生错误的回调函数。

Promise对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个catch语句捕获。

Promise内部的错误不会影响到Promise外部的代码，通俗的说法就是"Promise会吃掉错误"。

## 5.Promise.prototype.finally()

finally方法用于指定不管Promise对象最后状态如何，都会执行的操作。该方法是ES2018引入标准的。

```
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```

finally方法的回调函数不接受任何参数，这意味着没有办法知道，前面的Promise状态到底是fulfilled还是rejected。这表明，finally方法里面的操作，应该是与状态无关的，不依赖于Promise的执行结果。

finally本质上是then方法的特例。

```
promise
.finally(() => {
  // 语句
});

// 等同于
promise
.then(
  result => {
    // 语句
    return result;
  },
  error => {
    // 语句
    throw error;
  }
);
```

finally实现：

```
Promise.prototype.finally = function (callback) {
  let P = this.constructor;
  return this.then(
    value  => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => { throw reason })
  );
};
```

## 6.Promise.all()

Promise.all()方法用于将多个Promise实例，包装成一个新的Promise实例。

```
const p = Promise.all([p1,p2,p3]); //p1,p2,p3都是Promise实例
```

Promise.all()方法的参数可以不是数组，但必须具有Iterator接口，且返回的每个成员都是Promise实例。

## 7.Promise.race()

Promise.race()方法同样是将多个Promise实例，包装成一个新的Promise实例。

```
const p = Promise.race([p1, p2, p3]);
```

## 8.Promise.allSettled()

Promise.allSettled()方法接收一组Promise实例作为参数，包装成一个新的Promise实例。只有等到这些参数实例都返回结果，不管是fulfilled还是rejected，包装实例才会结束。该方法由ES2020引入。

```
const promises = [
  fetch('/api-1'),
  fetch('/api-2'),
  fetch('/api-3'),
];

await Promise.allSettled(promises);
removeLoadingIndicator();
```

该方法返回的Promise实例，一旦结束，状态总是fulfilled,不会变成rejected。

Promise.allSettled()方法不关心异步操作的结果，只关心这些操作有没有结束，没有这个方法，想要确保所有的操作都结束，就很麻烦。

## 9.Promise.any()

Promise.any()方法接收一组Promise实例作为参数，包装成一个新的Promise实例。只要参数实例有一个变成fulfilled状态，包装实例就会变成fulfilled状态；如果所有的参数实例都变成rejected状态，包装实例就会变成rejected状态。该方法目前是一个第三阶段的提案。

Promise.any()跟Promise.race()方法很像，只要一点不同，就是不会因为某个Promise变成reject特点状态而结束。

```
var resolved = Promise.resolve(42);
var rejected = Promise.reject(-1);
var alsoRejected = Promise.reject(Infinity);

Promise.any([resolved, rejected, alsoRejected]).then(function (result) {
  console.log(result); // 42
});

Promise.any([rejected, alsoRejected]).catch(function (results) {
  console.log(results); // [-1, Infinity]
});
```

## 10.Promise.resolve()

Promise.resolve()方法可以将现有对象转换为Promise对象。

```
const jsPromise = Promise.resolve($.ajax('/whatever.json'));
```

```
Promise.resolve('foo')
// 等价于
new Promise(resolve => resolve('foo'))
```

Promise.resolve方法的参数：

1. 参数是一个Promise实例；

   Promise.resolve将不做任何修改、原封不动返回这个实例。

2. 参数是一个thenable对象；

   thenable对象指的是具有then方法的对象

   ```
   let thenable = {
     then: function(resolve, reject) {
       resolve(17);
     }
   };
   ```

   Promise.resolve方法会将thenable对象转换为Promsie对象，然后就立即执行对象的then方法。

3. 参数不是具有then方法的对象，或根本就不是对象；

   如果参数是一个原始值，或者一个不具有then方法的对象，则Promise.resolve方法返回一个新的Promise对象，状态为resolved。

   ```
   const p = Promise.resolve('Hello');
   
   p.then(function (s){
     console.log(s)
   });
   // Hello
   ```

4. 不带有任何参数；

   Promise.resolve()方法允许调用时不带参数，直接返回一个resolved状态的Promise对象。

   ```
   const p = Promise.resolve();
   
   p.then(function () {
     // ...
   });
   ```

   注意：立即resolved()的Promise对象，是在本轮“事件循环”(event loop)的结束时执行，而不是在下一轮“事件循坏”的开始时。

   ```
   setTimeout(function () {
     console.log('three');
   }, 0);
   
   Promise.resolve().then(function () {
     console.log('two');
   });
   
   console.log('one');
   
   // one
   // two
   // three
   ```

##  11.Promise.reject()

Promise.reject(reason)方法也会返回一个新的Promise实例，该实例的状态为rejected。

```
const p = Promise.reject('出错了');
// 等同于
const p = new Promise((resolve, reject) => reject('出错了'))

p.then(null, function (s) {
  console.log(s)
});
// 出错了
```

## 12.应用

### 12.1加载图片

将图片的加载写成一个Promise，一旦加载完成，Promise的状态发生变化。

```
const preloadImage = function (path) {
  return new Promise(function (resolve, reject) {
    const image = new Image();
    image.onload  = resolve;
    image.onerror = reject;
    image.src = path;
  });
};
```

### 12.2Generator函数与Promise的结合

使用Generator函数管理流程，遇到异步操作的时候，通常返回一个Promise对象。

```
function getFoo () {
  return new Promise(function (resolve, reject){
    resolve('foo');
  });
}

const g = function* () {
  try {
    const foo = yield getFoo();
    console.log(foo);
  } catch (e) {
    console.log(e);
  }
};

function run (generator) {
  const it = generator();

  function go(result) {
    if (result.done) return result.value;

    return result.value.then(function (value) {
      return go(it.next(value));
    }, function (error) {
      return go(it.throw(error));
    });
  }

  go(it.next());
}

run(g);
```

## 13.Promise.try()

不知道或者不想区分，函数f是同步函数还是异步操作，但是想用Promise来处理。采用下面的写法：

```
Promise.resolve().then(f)
```

缺点：

当f是同步函数，那么在本轮事件循环的末端执行。

Promise.try()方法替代上面的写法。

```
const f = () => console.log('now');
Promise.try(f);
console.log('next');
// now
// next
```

事实上，Promise.try就是模拟try代码块，就像Promise.catch模拟的是catch代码块。