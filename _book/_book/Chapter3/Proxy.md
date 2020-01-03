# Proxy

## 1. Proxy实例的方法

1. get();
2. set();
3. apply();
4. has();
5. construct();
6. deleteProperty();
7. defineProperty();
8. getOwnpropertyDescriptor();
9. getPrototypeOf();
10. isExtensible();
11. ownKeys();
12. prevebtExtensions();
13. setPrototypeOf();

## 2. Proxy的应用场景

1. 拦截读取操作；

   ```
   var person = {
     name: "张三"
   };
   
   var proxy = new Proxy(person, {
     get: function(target, propKey) {
       if (propKey in target) {
         return target[propKey];
       } else {
         throw new ReferenceError("Prop name \"" + propKey + "\" does not exist.");
       }
     }
   });
   
   proxy.name // "张三"
   proxy.age // 抛出一个错误
   ```

   如何访问目标对象不存在的属性，会抛出一个错误。如果没有拦截函数，访问不存在的属性。只会返回undefined。

2. 使用`get`拦截，实现数组读取负数的索引；

   ```
   function createArray(...elements) {
     let handler = {
       get(target, propKey, receiver) {
         let index = Number(propKey);
         if (index < 0) {
           propKey = String(target.length + index);
         }
         return Reflect.get(target, propKey, receiver);
       }
     };
   
     let target = [];
     target.push(...elements);
     return new Proxy(target, handler);
   }
   
   let arr = createArray('a', 'b', 'c');
   arr[-1] // c
   ```

   数组的位置参数是-1，就会输出数组的倒数第一个成员。

3.  利用 Proxy，可以将读取属性的操作（`get`），转变为执行某个函数，从而实现属性的链式操作;

   ```
   var pipe = (function () {
     return function (value) {
       var funcStack = [];
       var oproxy = new Proxy({} , {
         get : function (pipeObject, fnName) {
           if (fnName === 'get') {
             return funcStack.reduce(function (val, fn) {
               return fn(val);
             },value);
           }
           funcStack.push(window[fnName]);
           return oproxy;
         }
       });
   
       return oproxy;
     }
   }());
   
   var double = n => n * 2;
   var pow    = n => n * n;
   var reverseInt = n => n.toString().split("").reverse().join("") | 0;
   
   pipe(3).double.pow.reverseInt.get; // 63
   ```

   设置Proxy后，达到将函数名链式使用的效果。

4.  利用`get`拦截，实现一个生成各种 DOM 节点的通用函数`dom`；

   ```
   const dom = new Proxy({}, {
     get(target, property) {
       return function(attrs = {}, ...children) {
         const el = document.createElement(property);
         for (let prop of Object.keys(attrs)) {
           el.setAttribute(prop, attrs[prop]);
         }
         for (let child of children) {
           if (typeof child === 'string') {
             child = document.createTextNode(child);
           }
           el.appendChild(child);
         }
         return el;
       }
     }
   });
   
   const el = dom.div({},
     'Hello, my name is ',
     dom.a({href: '//example.com'}, 'Mark'),
     '. I like:',
     dom.ul({},
       dom.li({}, 'The web'),
       dom.li({}, 'Food'),
       dom.li({}, '…actually that\'s it')
     )
   );
   
   document.body.appendChild(el);
   ```

5.  我们会在对象上面设置内部属性，属性名的第一个字符使用下划线开头，表示这些属性不应该被外部使用。结合`get`和`set`方法，就可以做到防止这些内部属性被外部读写 ;

   ```
   const handler = {
     get (target, key) {
       invariant(key, 'get');
       return target[key];
     },
     set (target, key, value) {
       invariant(key, 'set');
       target[key] = value;
       return true;
     }
   };
   function invariant (key, action) {
     if (key[0] === '_') {
       throw new Error(`Invalid attempt to ${action} private "${key}" property`);
     }
   }
   const target = {};
   const proxy = new Proxy(target, handler);
   proxy._prop
   // Error: Invalid attempt to get private "_prop" property
   proxy._prop = 'c'
   // Error: Invalid attempt to set private "_prop" property
   ```

    只要读写的属性名的第一个字符是下划线，一律抛错，从而达到禁止读写内部属性的目的。 

## 3. Proxy.revocable()

 `Proxy.revocable`方法返回一个可取消的 Proxy 实例。 

```
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```

 `Proxy.revocable`的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问 。

## 4. 实例：Web服务的客户端

 Proxy 对象可以拦截目标对象的任意属性，这使得它很合适用来写 Web 服务的客户端。 