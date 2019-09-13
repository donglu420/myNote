# ES 6 

## 一、简单讲一讲ES6的一些新特性

### 参考回答：

ES6在变量的声明和定义方面增加了let、const声明变量，有局部变量的概念，赋值中有比较吸引人的结构赋值，同时ES6对字符串、 数组、正则、对象、函数等拓展了一些方法，如字符串方面的模板字符串、函数方面的默认参数、对象方面属性的简洁表达方式，ES6也 引入了新的数据类型symbol，新的数据结构set和map,symbol可以通过typeof检测出来，为解决异步回调问题，引入了promise和 generator，还有最为吸引人了实现Class和模块，通过Class可以更好的面向对象编程，使用模块加载方便模块化编程，当然考虑到 浏览器兼容性，我们在实际开发中需要使用babel进行编译

重要的特性：

块级作用域：ES5只有全局作用域和函数作用域，块级作用域的好处是不再需要立即执行的函数表达式，循环体中的闭包不再有问题

rest参数：用于获取函数的多余参数，这样就不需要使用arguments对象了，

promise:一种异步编程的解决方案，比传统的解决方案回调函数和事件更合理强大

模块化：其模块功能主要有两个命令构成，export和import，export命令用于规定模块的对外接口，import命令用于输入其他模块提供的功能



## 二、Promise

> Promise 就是一个对象，用来传递异步操作的消息。有了 Promise 对象，就可以将异步操作以同步操作的流程表达出来。

### 1. Promise特点

- 承诺将来会执行

- 防止回调地狱 - Callback Hell

- 可以进行then的链式执行

- 区分数据请求和数据处理

- 三种状态

  - pending：等待中，或者进行中，表示还没有得到结果
  - resolved（fullfilled）：已经完成，表示得到了我们想要的结果，可以继续往下执行
  - rejected：也表示得到结果，但是由于结果并非我们所愿，因此拒绝执行

- 调用时会被传递两个参数：`resolve`和`reject`函数

- 具备

  ```
  then()
  ```

  方法，对于

  ```
  then()
  ```

  方法有以下简单的要求：

  1. 接收完成态、错误态的回调方法
  2. 可选地支持progress事件回调作为第三个方法
  3. 只接受function对象
  4. 返回Promise对象，以实现链式调用

### 2. 如何创建AJAX

一个Ajax建立要了解以下几点：

- XMLHttpRequest对象的工作流程
- 兼容性处理
- 事件的触发条件
- 事件的触发顺序

步骤：

1. 创建`XMLHttpRequest`对象，也就是创建一个**异步调用对象**。
2. 创建一个新的HTTP请求，并指定该HTTP请求的方法、URL及验证信息。
3. 设置响应HTTP请求状态变化的函数。
4. 发送HTTP请求。
5. 获取异步调用返回的数据。
6. 使用JavaScript和DOM实现局部刷新。

### 3. 利用Promise知识，用原生JS封装AJAX

```javascript
var url = '/请求的路径';
var params = {
    id: 'id=123',
    limit: 'limit=10'
};

// 封装一个get请求的方法
function getJSON(url) {
    return new Promise(function (resolve, reject) {
        var XHR = XMLHttpRequest ? new XMLHttpRequest() : new window.ActiveXObject('Microsoft.XMLHTTP');

        XHR.onreadystatechange = function () {
            //readyState属性表示请求/响应过程的当前活动阶段。
            if (XHR.readyState == 4) {
                if ((XHR.status >= 200 && XHR.status < 300) || XHR.status == 304) {
                    try {
                        //获取数据
                        var response = JSON.parse(XHR.responseText);
                        resolve(response);
                    } catch (e) {
                        reject(e);
                    }
                } else {
                    reject(new Error("Request was unsuccessful: " + XHR.statusText));
                }
            }
        }
        XHR.open('GET', url + '?' + params.join('&'), true);
        XHR.send(null);
    })
}

getJSON(url).then(resp => console.log(resp));
```

**readyState**
0 - 代表未初始化。 还没有调用 open 方法
1 - 代表正在加载。 open 方法已被调用，但 send 方法还没有被调用
2 - 代表已加载完毕。send 已被调用。请求已经开始
3 - 代表正在与服务器交互中。服务器正在解析响应内容
4 - 代表完成。响应发送完毕

> readyState的值等于3，就是“流”(streaming)，它是提升数据性能的强大工具。

### 4. Promise 语法

代码以实现异步操作img为例

#### 4.1 ES5 异步Callback实现

```javascript
function loadImg(src, callback, fail) {
    var img = document.createElement('img')
    img.onload = function () {
        callback(img)
    }
    img.onerror = function () {
        fail()
    }
    img.src = src
}
var src = 'https://img.mukewang.com/5be2bcb30001a46709360316.jpg'
loadImg(src, function (img) {
    console.log(img.width)
}, function () {
    console.log('failed')
})
```

#### 4.2 Promise实现

```javascript
function loadImg(src) {
    const promise = new Promise(function (resolve, resolve) {
        var img = document.createElement('img')
        img.onload = function () {
            resolve(img)
        }
        img.onerror = function () {
            reject()
        }
        img.src = src
    })
    return promise
}


var src = 'https://img.mukewang.com/5be2bcb30001a46709360316.jpg'
var result = loadImg(src)

result.then(function (img) {
    console.log(img.width)
}, function () {
    console.log('failed')
})
result.then(function (img) {
    console.log(img.height)
})
```

优点：便于集成、扩展
注意：

- new Promise 实例，必须要 return
- new Promise 时要传入函数，函数有 resolve reject 两个参数
- 成功时执行 resolve() 失败时执行 reject()

#### 4.3 重要语法

- Promise.all接收一个Promise对象组成的数组作为参数，当这个数组所有的Promise对象状态都变成resolved或者rejected的时候，它才会去调用then方法。

- Promise.race与Promise.all相似的是，Promise.race都是以一个Promise对象组成的数组作为参数，不同的是，只要当数组中的其中一个Promsie状态变成resolved或者rejected时，就可以调用.then方法了。

- 如何有效的将ajax的数据请求和数据处理分别放在不同的模块中进行管理

  - 将所有的数据请求这个动作放在同一个模块中统一管理

  ```javascript
  define(function(require) {
      var API = require('API');
  
      // 因为jQuery中的get方法也是通过Promise进行了封装，最终返回的是一个Promise对象，因此这样我们就可以将数据请求与数据处理放在不同的模块，即使用一个统一的模块来管理所有的数据请求
  
      // 获取当天的信息
      getDayInfo = function() {
          return $.get(API.dayInfo);
      }
  
      // 获取type信息
      getTypeInfo = function() {
          return $.get(API.typeInfo);
      };
  
      return {
          getDayInfo: getDayInfo,
          getTypeInfo: getTypeInfo
      }
  });
  ```

  可以增加过滤处理

  - 拿到数据并处理数据

  ```javascript
  // components/calendar.js
  define(function(require) {
      var request = require('request');
  
      // 拿到数据之后，需要处理的组件，可以根据数据渲染出需求想要的样式
  
      request.getTypeInfo()
          .then(function(resp) {
  
              // 拿到数据，并执行处理操作
              console.log(resp);
          })
  
      // 这样，我们就把请求数据，与处理数据分离开来，维护起来就更加方便了，代码结构也足够清晰
  })
  ```

## 三、箭头函数

### （1）与传统函数不同之处：

①没有this,super,arguments和new.target绑定，箭头函数中的这些值由外围最近一层非箭头函数决定。

②不能通过new关键字调用。它没有[[Construct]],所以不能用作构造函数。如用会报错

③没有原型（prototype）

④不可以改变this的绑定，函数内部的this值不可被改变，在函数的生命周期内始终保持一致。

⑤不支持arguments对象。所以只能通过命名参数和不定参数两种形式访问函数参数。

⑥不支持重复的命名函数。

### （2）箭头函数语法：

    let 函数名=（参数1，参数2，...）=>{ };

   当只有一个参数时，可直接写参数名。箭头后也可不跟{},直接跟表达式或要返回的值，对象字面量要用()括住。

   辨识方法：typeof instanceof

### （3）创建立即执行函数表达式：

        是javasscript函数比较流行的使用方式，可定义一个匿名函数并立即调用，自始至终不保存对该函数的引用。（想创建一个与其他程序隔离的作用域时，该模式比较方便）。

```
//es5
let person =function(name){
    return {
        getName:function(){
            return name;
        }
    };
}("Nicholas");
console.log(person.getName());//"Nicholas"

//es6:
let person =((name)=>{
    return {
        getName:function(){
            return name;
        }
    };
})("Nicholas");
consol.log(person.getName());//"Nicholas"
```

(3)ES6尾调用优化（尾调用系统的引擎优化）。

         尾调用指的是函数作为另个函数的最后一句被调用：

function doSomething(){
    return doSomethingElse();//尾调用
}
暂时也不太明白 ，书上是这么说：尾调用优化可帮函数保持一个更小的调用栈，从而减少内存的使用，避免栈溢出错误，当程序满足优化条件时，引擎会自动对其进行优化。

```javascript
//递归：
function factorial(n){
    if(n<=1){
        return 1;
    }else{

​```javascript
    //无法优化，必在返回后执行乘法操作。
    return n*factorial(n-1);
}
​```

}

//ES6引擎优化：

function factorial(n,p=1){
    if(n<=1){
        return 1*p;
    }else{
        let result=n*p;

​```
    //优化后
    return factorial(n-1，result);
}
​```

}


Array.from()\Array.of()

from是转换成数组，能够将非数组对象转换成数组，这样该对象就可以使用map reduce等方法
of()传入多少 就生成多少的数组

find():
const a = arry.find(b => b.names === '1')
findIndex和find一样但是是用于寻找index

some()如果数组里的某个参数满足我此时定义的条件，那么就返回true并且终止循环
every()则是得这个被检测的数组里所有的参数都查找一遍筛选出所有满足条件的才会结束(返回布尔值)
```

## 四、类class

是es6的语法糖 是一个function但它不具备像函数那样的函数提升，不能在定义前调用它

static方法 静态方法，只能通过原型对象调用 不能通过实例来调用

Array.of() //可以

[1,2,3].of()//不可以



## 五、Genenator函数在Ajax的使用

```js
fucntion ajax (url) {
    axios.get(url).then((res) => {
        userGen.next(res.data);
    })
}
function* steps() {
    const users = yield ajax('htttp://xxxxxxx');//first
    const fisrtUsers = yield ajax(`http://xxxxxxx${users[0].login}`);//second
    const followers = yield ajax(firstUsers.follower_url);
}
const userGen = steps();
userGen.next();
```

## 六、实现 Promise.all 方法

在实现 Promise.all 方法之前，我们首先要知道 Promise.all 的功能和特点，因为在清楚了 Promise.all 功能和特点的情况下，我们才能进一步去写实现。

> Promise.all 功能

`Promise.all(iterable)` 返回一个新的 Promise 实例。此实例在 `iterable` 参数内所有的 `promise` 都 `fulfilled` 或者参数中不包含 `promise` 时，状态变成 `fulfilled`；如果参数中 `promise` 有一个失败`rejected`，此实例回调失败，失败原因的是第一个失败 `promise` 的返回结果。

```js
let p = Promise.all([p1, p2, p3]);
```

p的状态由 p1,p2,p3决定，分成以下；两种情况：

（1）只有p1、p2、p3的状态都变成 `fulfilled`，p的状态才会变成 `fulfilled`，此时p1、p2、p3的返回值组成一个数组，传递给p的回调函数。

（2）只要p1、p2、p3之中有一个被 `rejected`，p的状态就变成 `rejected`，此时第一个被reject的实例的返回值，会传递给p的回调函数。

> Promise.all 的特点

Promise.all 的返回值是一个 promise 实例

- 如果传入的参数为空的可迭代对象，`Promise.all` 会 **同步** 返回一个已完成状态的 `promise`
- 如果传入的参数中不包含任何 promise,`Promise.all` 会 **异步** 返回一个已完成状态的 `promise`
- 其它情况下，`Promise.all` 返回一个 **处理中（pending）** 状态的 `promise`.

> Promise.all 返回的 promise 的状态

- 如果传入的参数中的 promise 都变成完成状态，`Promise.all` 返回的 `promise` 异步地变为完成。
- 如果传入的参数中，有一个 `promise` 失败，`Promise.all` 异步地将失败的那个结果给失败状态的回调函数，而不管其它 `promise` 是否完成
- 在任何情况下，`Promise.all` 返回的 `promise` 的完成状态的结果都是一个数组

> Promise.all 实现

```js
Promise.all = function (promises) {
    //promises 是可迭代对象，省略参数合法性检查
    return new Promise((resolve, reject) => {
        //Array.from 将可迭代对象转换成数组
        promises = Array.from(promises);
        if (promises.length === 0) {
            resolve([]);
        } else {
            let result = [];
            let index = 0;
            for (let i = 0;  i < promises.length; i++ ) {
                //考虑到 i 可能是 thenable 对象也可能是普通值
                Promise.resolve(promises[i]).then(data => {
                    result[i] = data;
                    if (++index === promises.length) {
                        //所有的 promises 状态都是 fulfilled，promise.all返回的实例才变成 fulfilled 态
                        resolve(result);
                    }
                }, err => {
                    reject(err);
                    return;
                });
            }
        }
    });
}
```

### 可迭代对象有哪些特点

ES6 规定，默认的 `Iterator` 接口部署在数据结构的 `Symbol.iterator` 属性，换个角度，也可以认为，一个数据结构只要具有 `Symbol.iterator` 属性(`Symbol.iterator` 方法对应的是遍历器生成函数，返回的是一个遍历器对象)，那么就可以其认为是可迭代的。

## 七、可迭代对象的特点

- 具有 `Symbol.iterator` 属性，`Symbol.iterator()` 返回的是一个遍历器对象
- 可以使用 `for ... of` 进行循环
- 通过被 `Array.from` 转换为数组

```js
let arry = [1, 2, 3, 4];
let iter = arry[Symbol.iterator]();
console.log(iter.next()); //{ value: 1, done: false }
console.log(iter.next()); //{ value: 2, done: false }
console.log(iter.next()); //{ value: 3, done: false }
```

### 1.原生具有 `Iterator` 接口的数据结构：

- Array
- Map
- Set
- String
- TypedArray
- 函数的 arguments 对象
- NodeList 对象

# ES 5

## 一、JS的基本数据类型有哪些，基本数据类型和引用数据类型的区别，NaN是什么的缩写，JS的作用域类型，undefined==null返回的结果是什么，undefined与null的区别在哪，写一个函数判断变量类型

### 参考回答：

JS的基本数据类型有字符串，数字，布尔，数组，对象，Null，Undefined,基本数据类型是按值访问的，也就是说我们可以操作保存在变量中的实际的值，

基本数据类型和引用数据类型的区别如下：

基本数据类型的值是不可变的，任何方法都无法改变一个基本类型的值，当这个变量重新赋值后看起来变量的值是改变了，但是这里变量名只是指向变量的一个指针，所以改变的是指针的指向改变，该变量是不变的，但是引用类型可以改变

基本数据类型不可以添加属性和方法，但是引用类型可以

基本数据类型的赋值是简单赋值，如果从一个变量向另一个变量赋值基本类型的值，会在变量对象上创建一个新值，然后把该值复制到为新变量分配的位置上，引用数据类型的赋值是对象引用，

基本数据类型的比较是值的比较，引用类型的比较是引用的比较，比较对象的内存地址是否相同

基本数据类型是存放在栈区的，引用数据类型同事保存在栈区和堆区

NaN是JS中的特殊值，表示非数字，NaN不是数字，但是他的数据类型是数字，它不等于任何值，包括自身，在布尔运算时被当做false，NaN与任何数运算得到的结果都是NaN，党员算失败或者运算无法返回正确的数值的就会返回NaN，一些数学函数的运算结果也会出现NaN ,

JS的作用域类型：

一般认为的作用域是词法作用域，此外JS还提供了一些动态改变作用域的方法，常见的作用域类型有：

函数作用域，如果在函数内部我们给未定义的一个变量赋值，这个变量会转变成为一个全局变量，

块作用域：块作用域吧标识符限制在{}中，

改变函数作用域的方法：

eval（），这个方法接受一个字符串作为参数，并将其中的内容视为好像在书写时就存在于程序中这个位置的代码，

with关键字：通常被当做重复引用同一个对象的多个属性的快捷方式



undefined与null：目前null和undefined基本是同义的，只有一些细微的差别，null表示没有对象，undefined表示缺少值，就是此处应该有一个值但是还没有定义，因此undefined==null返回false

此外了解== 和===的区别：

在做==比较时。不同类型的数据会先转换成一致后在做比较，===中如果类型不一致就直接返回false，一致的才会比较

类型判断函数，使用typeof即可，首先判断是否为null，之后用typeof哦按段，如果是object的话，再用array.isarray判断是否为数组，如果是数字的话用isNaN判断是否是NaN即可
扩展学习：

JS采用的是词法作用域，也就是静态作用域，所以函数的作用域在函数定义的时候就决定了，

看如下例子：

```
var value = 1;
function foo() {
console.log(value);
}
function bar() {
var value = 2;
foo();
}
bar();
```

假设JavaScript采用静态作用域，让我们分析下执行过程：

执行foo 函数，先从 foo 函数内部查找是否有局部变量 value，如果没有，就根据书写的位置，查找上面一层的代码，也就是 value 等于 1，所以结果会打印 1。

假设JavaScript采用动态作用域，让我们分析下执行过程：

执行foo 函数，依然是从 foo 函数内部查找是否有局部变量 value。如果没有，就从调用函数的作用域，也就是 bar 函数内部查找 value 变量，所以结果会打印 2。

前面我们已经说了，JavaScript采用的是静态作用域，所以这个例子的结果是 1。



## 二、写一个newBind函数，完成bind的功能。

### 参考回答：

bind（）方法，创建一个新函数，当这个新函数被调用时，bind（）的第一个参数将作为它运行时的this，之后的一序列参数将会在传递的实参前传入作为它的参数

```javascript
`Function.prototype.bind``2` `= function (context) {``if (typeof this !== ``"function"``) {``throw new Error(``"Function.prototype.bind - what is trying to be bound is not callable"``);``}``var self = this;``var args = Array.prototype.slice.call(arguments, ``1``);``var fNOP = function () {};``var fbound = function () {``self.apply(this instanceof self ? this : context, args.concat(Array.prototype.slice.call(arguments)));``}``fNOP.prototype = this.prototype;``fbound.prototype = new fNOP();``return fbound;``}`
```

## 三、了解事件代理吗，这样做有什么好处

### 参考回答：

事件代理/事件委托：利用了事件冒泡，只指定一个事件处理程序，就可以管理某一类型的事件，

简而言之：事件代理就是说我们将事件添加到本来要添加的事件的父节点，将事件委托给父节点来触发处理函数，这通常会使用在大量的同级元素需要添加同一类事件的时候，比如一个动态的非常多的列表，需要为每个列表项都添加点击事件，这时就可以使用事件代理，通过判断e.target.nodeName来判断发生的具体元素，这样做的好处是减少事件绑定，同事动态的DOM结构任然可以监听，事件代理发生在冒泡阶段



## 四、跨域的几种方法（比较重要）

#### 跨域方法：

今天一共介绍七种常用跨域的方式，关于跨域大概可以分为 iframe 的跨域和纯粹的跨全域请求。

下面就先介绍三种跨全域的方法：

**1. JSONP：**

只要说到跨域，就必须聊到 JSONP，JSONP全称为：JSON with Padding，可用于解决主流浏览器的跨域数据访问的问题。

Web 页面上调用 js 文件不受浏览器同源策略的影响，所以通过 Script 便签可以进行跨域的请求：

1. 首先前端先设置好回调函数，并将其作为 url 的参数。
2. 服务端接收到请求后，通过该参数获得回调函数名，并将数据放在参数中将其返回
3. 收到结果后因为是 script 标签，所以浏览器会当做是脚本进行运行，从而达到跨域获取数据的目的。

实例：

后端逻辑：

```js
//server.js
const url = require('url');
	
require('http').createServer((req, res) => {

    const data = {
	x: 10
    };
    const callback = url.parse(req.url, true).query.callback;
    res.writeHead(200);
    res.end(`${callback}(${JSON.stringify(data)})`);

}).listen(3000, '127.0.0.1');

console.log('启动服务，监听 127.0.0.1:3000');
```

通过 node server.js 启动服务，监听端口 3000，这样服务端就建立起来了

前端页面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>index.html</title>
</head>
<body>
    <script>
	function jsonpCallback(data) {
	    alert('获得 X 数据:' + data.x);
	}
    </script>
    <script src="http://127.0.0.1:3000?callback=jsonpCallback"></script>
</body>
</html>
```

逻辑已经写好了，那如何来模拟一个跨域的场景呢？

这里我们通过端口号的不同来模拟跨域的场景，通过 [http://127.0.0.1:8080](https://link.zhihu.com/?target=http%3A//127.0.0.1%3A8080) 端口来访问页面。先通过 npm 下载 http-server 模块：

```text
nom install -g http-server
```

并且在页面同目录下输入：

```text
http-server
```

![img](https://pic1.zhimg.com/80/v2-9c735edccc5b298d66ea26560866324c_hd.png)

这样就可以通过端口 8080 访问 index.html 刚才那个页面了，相当于是开启两个监听不同端口的 http 服务器，通过页面中的请求来模拟跨域的场景。打开浏览器，访问 [http://127.0.0.1:8080](https://link.zhihu.com/?target=http%3A//127.0.0.1%3A8080) 就可以看到从 [http://127.0.0.1:3000](https://link.zhihu.com/?target=http%3A//127.0.0.1%3A3000) 获取到的数据了。



![img](https://pic4.zhimg.com/80/v2-f69328f1751471696f52c4ee0f34d723_hd.png)

至此，通过 JSONP 跨域获取数据已经成功了，但是通过这种事方式也存在着一定的优缺点：



优点：

1. 它不像XMLHttpRequest 对象实现 Ajax 请求那样受到同源策略的限制
2. 兼容性很好，在古老的浏览器也能很好的运行
3. 不需要 XMLHttpRequest 或 ActiveX 的支持；并且在请求完毕后可以通过调用 callback 的方式回传结果。

缺点：

1. 它支持 GET 请求而不支持 POST 等其它类行的 HTTP 请求。
2. 它只支持跨域 HTTP 请求这种情况，不能解决不同域的两个页面或 iframe 之间进行数据通信的问题

**2. CORS：**

CORS 是一个 W3C 标准，全称是"跨域资源共享"（Cross-origin resource sharing）它允许浏览器向跨源服务器，发出 XMLHttpRequest 请求，从而克服了 ajax 只能同源使用的限制。

CORS 需要浏览器和服务器同时支持才可以生效，对于开发者来说，CORS 通信与同源的 ajax 通信没有差别，代码完全一样。浏览器一旦发现 ajax 请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

因此，实现 CORS 通信的关键是服务器。只要服务器实现了 CORS 接口，就可以跨源通信。

首先前端先创建一个 index.html 页面：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CORS</title>
</head>
<body>
    <script>
	const xhr = new XMLHttpRequest();
	xhr.open('GET', 'http://127.0.0.1:3000', true);
	xhr.onreadystatechange = function() {
	    if(xhr.readyState === 4 && xhr.status === 200) {
	        alert(xhr.responseText);
	    }
	}
	xhr.send(null);
    </script>
</body>
</html>
```

这似乎跟一次正常的异步 ajax 请求没有什么区别，关键是在服务端收到请求后的处理：

```js
require('http').createServer((req, res) => {

    res.writeHead(200, {
	'Access-Control-Allow-Origin': 'http://localhost:8080'
    });
    res.end('这是你要的数据：1111');

}).listen(3000, '127.0.0.1');

console.log('启动服务，监听 127.0.0.1:3000');
```

关键是在于设置相应头中的 Access-Control-Allow-Origin，该值要与请求头中 Origin 一致才能生效，否则将跨域失败。

接下来再次开启两个 http 服务器进程：



![img](https://pic3.zhimg.com/80/v2-4a05419572580bc4fd8f48c00dc90e5a_hd.png)

打开浏览器访问 [http://localhost:8080](https://link.zhihu.com/?target=http%3A//localhost%3A8080)





![img](https://pic1.zhimg.com/80/v2-d03ba54029b004a44b84891b6d62c84c_hd.png)

成功的关键在于 Access-Control-Allow-Origin 是否包含请求页面的域名，如果不包含的话，浏览器将认为这是一次失败的异步请求，将会调用 xhr.onerror 中的函数。



CORS 的优缺点：

1. 使用简单方便，更为安全
2. 支持 POST 请求方式
3. CORS 是一种新型的跨域问题的解决方案，存在兼容问题，仅支持 IE 10 以上



这里只是对 CORS 做一个简单的介绍，如果想更详细地了解其原理的话，可以看看下面这篇文章：

[跨域资源共享 CORS 详解 - 阮一峰的网络日志](https://link.zhihu.com/?target=http%3A//www.ruanyifeng.com/blog/2016/04/cors.html)

**3. Server Proxy:**

服务器代理，顾名思义，当你需要有跨域的请求操作时发送请求给后端，让后端帮你代为请求，然后最后将获取的结果发送给你。

假设有这样的一个场景，你的页面需要获取 [CNode：Node.js专业中文社区](https://link.zhihu.com/?target=https%3A//cnodejs.org/api) 论坛上一些数据，如通过 [https://cnodejs.org/api/v1/topics](https://link.zhihu.com/?target=https%3A//cnodejs.org/api/v1)，当时因为不同域，所以你可以将请求后端，让其对该请求代为转发。

代码如下：

```js
const url = require('url');
const http = require('http');
const https = require('https');

const server = http.createServer((req, res) => {
    const path = url.parse(req.url).path.slice(1);
    if(path === 'topics') {
	https.get('https://cnodejs.org/api/v1/topics', (resp) => {
	    let data = "";
	    resp.on('data', chunk => {
		data += chunk;
	    });
	    resp.on('end', () => {
		res.writeHead(200, {
		    'Content-Type': 'application/json; charset=utf-8'
		});
		res.end(data);
	    });
	})		
    }
}).listen(3000, '127.0.0.1');

console.log('启动服务，监听 127.0.0.1:3000');
```

通过代码你可以看出，当你访问 [http://127.0.0.1:3000](https://link.zhihu.com/?target=http%3A//127.0.0.1%3A3000) 的时候，服务器收到请求，会代你发送请求 [https://cnodejs.org/api/v1/topics](https://link.zhihu.com/?target=https%3A//cnodejs.org/api/v1) 最后将获取到的数据发送给浏览器。

同样地开启服务:



![img](https://pic3.zhimg.com/80/v2-6678071b4e880188f38e1e9b0376799e_hd.png)

打开浏览器访问 [http://localhost:3000/topics](https://link.zhihu.com/?target=http%3A//localhost%3A3000/topics)



![img](https://pic4.zhimg.com/80/v2-0733a4f4bfda4f54c5ba3ae6f2e21453_hd.png)

跨域请求成功。

纯粹的跨全域请求的方式已经介绍完了，另外介绍四种通过 iframe 跨域与其它页面通信的方式。

**4. location.hash:**

在 url 中，[http://www.baidu.com#helloworld](https://link.zhihu.com/?target=http%3A//www.baidu.com%23helloworld) 的 "#helloworld" 就是 location.hash，改变 hash 值不会导致页面刷新，所以可以利用 hash 值来进行数据的传递，当然数据量是有限的。

假设 localhost:8080 下有文件 cs1.html 要和 localhost:8081 下的 cs2.html 传递消息，cs1.html 首先创建一个隐藏的 iframe，iframe 的 src 指向 localhost:8081/cs2.html，这时的 hash 值就可以做参数传递。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CS1</title>
</head>
<body>
    <script>
	// http://localhost:8080/cs1.html
	let ifr = document.createElement('iframe');
	ifr.style.display = 'none';
	ifr.src = "http://localhost:8081/cs2.html#data";
	document.body.appendChild(ifr);
		
	function checkHash() {
	    try {
		let data = location.hash ? location.hash.substring(1) : '';
		console.log('获得到的数据是：', data);
	    }catch(e) {

	    }
	}
	window.addEventListener('hashchange', function(e) {
	    console.log('获得的数据是：', location.hash.substring(1));
        });
    </script>
</body>
</html>
```

cs2.html 收到消息后通过 parent.location.hash 值来修改 cs1.html 的 hash 值，从而达到数据传递。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>CS2</title>
</head>
<body>
    <script>
    // http://locahost:8081/cs2.html
    switch(location.hash) {
        case "#data":
	    callback();
	    break;
    }
    function callback() {
	const data = "some number: 1111"
	try {
	    parent.location.hash = data;
	}catch(e) {
	    // ie, chrome 下的安全机制无法修改 parent.location.hash
	    // 所以要利用一个中间的代理 iframe 
	    var ifrproxy = document.createElement('iframe');
		ifrproxy.style.display = 'none';
		ifrproxy.src = 'http://localhost:8080/cs3.html#' + data;     // 该文件在请求域名的域下
		document.body.appendChild(ifrproxy);
	    }
       }
    </script>
</body>
</html>
```

由于两个页面不在同一个域下IE、Chrome不允许修改parent.location.hash的值，所以要借助于 localhost:8080 域名下的一个代理 iframe 的 cs3.html 页面

```html
<script>
    parent.parent.location.hash = self.location.hash.substring(1);
</script>
```

之后老规矩，开启两个 http 服务器：



![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='2558' height='278'></svg>)

这里为了图方便，将 cs1,2,3 都放在同个文件夹下，实际情况的话 cs1.html 和 cs3.html 要与 cs2.html 分别放在不同的服务器才对。



之后打开浏览器访问 localhost:8080/cs1.html，注意不是 8081，就可以看到获取到的数据了，此时页面的 hash 值也已经改变。

![img](https://pic2.zhimg.com/80/v2-ace462fea3cd4f0d34e7448a7749db29_hd.png)

当然这种方法存在着诸多的缺点：

1. 数据直接暴露在了 url 中
2. 数据容量和类型都有限等等

**5. window.name：**

window.name（一般在 js 代码里出现）的值不是一个普通的全局变量，而是当前窗口的名字，这里要注意的是每个 iframe 都有包裹它的 window，而这个 window 是top window 的子窗口，而它自然也有 window.name 的属性，window.name 属性的神奇之处在于 name 值在不同的页面（甚至不同域名）加载后依旧存在（如果没修改则值不会变化），并且可以支持非常长的 name 值（2MB）。

举个简单的例子：

你在某个页面的控制台输入：

```text
window.name = "Hello World";
window.location = "http://www.baidu.com";
```

页面跳转到了百度首页，但是 window.name 却被保存了下来，还是 Hello World，跨域解决方案似乎可以呼之欲出了：

首先创建 a.html 文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>a.html</title>
</head>
<body>
    <script>
	let data = '';
	const ifr = document.createElement('iframe');
	ifr.src = "http://localhost:8081/b.html";
	ifr.style.display = 'none';
	document.body.appendChild(ifr);
	ifr.onload = function() {
	    ifr.onload = function() {
	        data = ifr.contentWindow.name;
		console.log('收到数据:', data);
	    }
	    ifr.src = "http://localhost:8080/c.html";
	}
    </script>
</body>
</html>
```

之后在创建 b.html 文件：

```html
<script>
   window.name = "你想要的数据!";
</script>
```

[http://localhost:8080/a.html](https://link.zhihu.com/?target=http%3A//localhost%3A8080/a.html) 在请求远端服务器 [http://localhost:8081/b.html](https://link.zhihu.com/?target=http%3A//localhost%3A8081/b.html) 的数据，我们可以在该页面下新建一个 iframe，该 iframe 的 src 属性指向服务器地址，(利用 iframe 标签的跨域能力)，服务器文件 b.html 设置好 window.name 的值。

但是由于 a.html 页面和该页面 iframe 的 src 如果不同源的话，则无法操作 iframe 里的任何东西，所以就取不到 iframe 的 name 值，所以我们需要在 b.html 加载完后重新换个 src 去指向一个同源的 html 文件，或者设置成 'about:blank;' 都行，这时候我只要在 a.html 相同目录下新建一个 c.html 的空页面即可。如果不重新指向 src 的话直接获取的 window.name 的话会报错：



![img](https://pic2.zhimg.com/80/v2-980cdf011dd25656023a2fe13fb561f9_hd.png)

老规矩，打开两个 http 服务器：





![img](https://pic4.zhimg.com/80/v2-5c6d46dd77d4ee66746c6ca9427b6707_hd.png)

打开浏览器就可以看到结果：





![img](https://pic4.zhimg.com/80/v2-e8881e591798e483c0ebd49d23d8cf0b_hd.png)

**6. postMessage:**



postMessage 是 HTML5 新增加的一项功能，跨文档消息传输(Cross Document Messaging)，目前：Chrome 2.0+、Internet Explorer 8.0+, Firefox 3.0+, Opera 9.6+, 和 Safari 4.0+ 都支持这项功能，使用起来也特别简单。

首先创建 a.html 文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>a.html</title>
</head>
<body>
    <iframe src="http://localhost:8081/b.html" style='display: none;'></iframe>
    <script>
	window.onload = function() {
	    let targetOrigin = 'http://localhost:8081';
	    window.frames[0].postMessage('我要给你发消息了!', targetOrigin);
	}
	window.addEventListener('message', function(e) {
	    console.log('a.html 接收到的消息:', e.data);
	});
    </script>
</body>
</html>
```

创建一个 iframe，使用 iframe 的一个方法 postMessage 可以想 [http://localhost:8081/b.html](https://link.zhihu.com/?target=http%3A//localhost%3A8081/b.html) 发送消息，然后监听 message，可以获得其他文档发来的消息。

同样的 b.html 文件：

```html
<script>
    window.addEventListener('message', function(e) {
        if(e.source != window.parent) {
	    return;
        }
        let data = e.data;
        console.log('b.html 接收到的消息:', data);
        parent.postMessage('我已经接收到消息了!', e.origin);
    });
</script>
```

同样的开启 http 服务器：

![img](https://pic1.zhimg.com/80/v2-1f8bd3c21cf92920c5ea4fe0996cdf78_hd.png)

打开浏览器同样可以看到：

![img](https://pic1.zhimg.com/80/v2-e7782ce8ef749c62c59725bac6c85140_hd.png)

对 postMessage 感兴趣的详细内容可以看看教程：

[PostMessage_百度百科](https://link.zhihu.com/?target=http%3A//baike.baidu.com/link%3Furl%3Dphr8dWERsuj7ll5ClY1L-n1grmQRbIQZMp57y31NH_O_goLJJQMycoGeDjCGHtF2LEQ6_pL7tmz3ppcUqSL2D5R2uBG0TMoqJC9uUWhnN4i)

[Window.postMessage()](https://link.zhihu.com/?target=https%3A//developer.mozilla.org/en-US/docs/Web/API/Window/postMessage)

**7. document.domain：**

对于主域相同而子域不同的情况下，可以通过设置 document.domain 的办法来解决，具体做法是可以在 [http://www.example.com/a.html](https://link.zhihu.com/?target=http%3A//www.example.com/a.html) 和 [http://sub.example.com/b.html](https://link.zhihu.com/?target=http%3A//sub.example.com/b.html) 两个文件分别加上 document.domain = "a.com"；然后通过 a.html 文件创建一个 iframe，去控制 iframe 的 window，从而进行交互，当然这种方法只能解决主域相同而二级域名不同的情况，如果你异想天开的把 script.example.com 的 domain 设为 qq.com 显然是没用的，那么如何测试呢？

测试的方式稍微复杂点，需要安装 nginx 做域名映射，如果你电脑没有安装 nginx，请先去安装一下: [nginx news](https://link.zhihu.com/?target=http%3A//nginx.org/)

先创建一个 a.html 文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>a.html</title>
</head>
<body>
    <script>
	document.domain = 'example.com';
	let ifr = document.createElement('iframe');
	ifr.src = 'http://sub.example.com/b.html';
	ifr.style.display = 'none';
	document.body.append(ifr);
	ifr.onload = function() {
	    let win = ifr.contentWindow;
	    alert(win.data);
	}
    </script>
</body>
</html>
```

在创建一个 b.html 文件：

```html
<script>
    document.domain = 'example.com';
    window.data = '传送的数据：1111';
</script>
```

之后打开 http 服务器：



![img](https://pic4.zhimg.com/80/v2-4f97906d3e488442e0f9613f4c502d13_hd.png)

这时候只是开启了两个 http 服务器，还需要通过 nginx 做域名映射，将 [Example Domain](https://link.zhihu.com/?target=http%3A//www.example.com)



打开操作系统下的 hosts 文件：mac 是位于 /etc/hosts 文件，并添加：

```text
127.0.0.1 www.example.com
127.0.0.1 sub.example.com
```

这样在浏览器打开这两个网址后就会访问本地的服务器。

之后打开 nginx 的配置文件：/usr/local/etc/nginx/nginx.conf，并在 http 模块里添加：



![img](https://pic3.zhimg.com/80/v2-502dff781dc5d8d4cfe1058b5c1090d2_hd.png)

上面代码的意思是：如果访问本地的域名是 [Example Domain](https://link.zhihu.com/?target=http%3A//www.example.com)



所以我们这时候在打开浏览器访问 [Example Domain](https://link.zhihu.com/?target=http%3A//www.example.com) 的时候其实访问的就是本地服务器 localhost:8080。

最后打开浏览器访问 [http://www.example.com/a.html](https://link.zhihu.com/?target=http%3A//www.example.com/a.html) 就可以看到结果：



![img](https://pic1.zhimg.com/80/v2-a9ba4c4107030e3372875344c97e5a30_hd.png)

**8. flash：**



这种方式我没有尝试过，不好往下定论，感兴趣的话可以上网搜看看教程。

## 总结：

前面八种跨域方式我已经全部讲完，其实讲道理，常用的也就是前三种方式，后面四种更多时候是一些小技巧，虽然在工作中不一定会用到，但是如果你在面试过程中能够提到这些跨域的技巧，无疑在面试官的心中是一个加分项。

上面阐述方法的时候可能有些讲的不明白，希望在阅读的过程中建议你跟着我敲代码，当你打开浏览器看到结果的时候，你也就能掌握到这种方法。

## 五、async/await

### 1.解决了什么问题

在async/await之前，我们有三种方式写异步代码

1. 嵌套回调
2. 以Promise为主的链式回调
3. 使用Generators

但是，这三种写起来都不够优雅，ES7做了优化改进，async/await应运而生，async/await相比较Promise 对象then 函数的嵌套，与 Generator 执行的繁琐（需要借助co才能自动执行，否则得手动调用next() ）， Async/Await 可以让你轻松写出同步风格的代码同时又拥有异步机制，更加简洁，逻辑更加清晰。

### 2.async/await特点

1. async/await更加语义化，async 是“异步”的简写，async function 用于申明一个 function 是异步的； await，可以认为是async wait的简写， 用于等待一个异步方法执行完成；
2. async/await是一个用同步思维解决异步问题的方案（等结果出来之后，代码才会继续往下执行）
3. 可以通过多层 async function 的同步写法代替传统的callback嵌套

### 3.async function语法

- 自动将常规函数转换成Promise，返回值也是一个Promise对象
- 只有async函数内部的异步操作执行完，才会执行then方法指定的回调函数
- 异步函数内部可以使用await

```jsx
async function name([param[, param[, ... param]]]) { statements }
name: 函数名称。
param:  要传递给函数的参数的名称
statements: 函数体语句。
返回值: 返回的Promise对象会以async function的返回值进行解析，或者以该函数抛出的异常进行回绝。
```



![img](https://upload-images.jianshu.io/upload_images/6522842-b2b6a9432cdd0996.png?imageMogr2/auto-orient/strip|imageView2/2/w/898/format/webp)



### 4.await语法

- await 放置在Promise调用之前，await 强制后面点代码等待，直到Promise对象resolve，得到resolve的值作为await表达式的运算结果
- await只能在async函数内部使用,用在普通函数里就会报错

```jsx
[return_value] = await expression;

expression:  一个 Promise  对象或者任何要等待的值。

返回值:返回 Promise 对象的处理结果。如果等待的不是 Promise 对象，则返回该值本身。
```



![img](https://upload-images.jianshu.io/upload_images/6522842-6e6b1cefa95688c9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



### 5.错误处理

在async函数里，无论是Promise reject的数据还是逻辑报错，都会被默默吞掉,所以最好把await放入try{}catch{}中，catch能够捕捉到Promise对象rejected的数据或者抛出的异常

```jsx
function timeout(ms) {

  return new Promise((resolve, reject) => {

    setTimeout(() => {reject('error')}, ms);  //reject模拟出错，返回error

  });

}

async function asyncPrint(ms) {

  try {

     console.log('start');

     await timeout(ms);  //这里返回了错误

     console.log('end');  //所以这句代码不会被执行了

  } catch(err) {

     console.log(err); //这里捕捉到错误error

  }

}

asyncPrint(1000);
```

如果不用try/catch的话，也可以像下面这样处理错误（因为async函数执行后返回一个promise）

```jsx
function timeout(ms) {

  return new Promise((resolve, reject) => {

    setTimeout(() => {reject('error')}, ms);  //reject模拟出错，返回error

  });

}

async function asyncPrint(ms) {

  console.log('start');

  await timeout(ms)

  console.log('end');  //这句代码不会被执行了

}

asyncPrint(1000).catch(err => {

    console.log(err)； // 从这里捕捉到错误

});
```

如果你不想让错误中断后面代码的执行，可以提前截留住错误，像下面

```jsx
function timeout(ms) {

  return new Promise((resolve, reject) => {

    setTimeout(() => {

        reject('error')

    }, ms);  //reject模拟出错，返回error

  });

}

async function asyncPrint(ms) {

  console.log('start');

  await timeout(ms).catch(err => {  // 注意要用catch

console.log(err) 

  })

  console.log('end');  //这句代码会被执行

}

asyncPrint(1000);
```

### 6.使用场景

**多个await命令的异步操作，如果不存在依赖关系（后面的await不依赖前一个await返回的结果），用Promise.all()让它们同时触发**

```jsx
function test1 () {
    return new Promise((resolve, reject) => {

        setTimeout(() => {

            resolve(1)

        }, 1000)

    })

}

function test2 () {

    return new Promise((resolve, reject) => {

        setTimeout(() => {

            resolve(2)

        }, 2000)

    })

}

async function exc1 () {

    console.log('exc1 start:',Date.now())

    let res1 = await test1();

    let res2 = await test2(); // 不依赖 res1 的值

    console.log('exc1 end:', Date.now())

}

async function exc2 () {

    console.log('exc2 start:',Date.now())

    let [res1, res2] = await Promise.all([test1(), test2()])

    console.log('exc2 end:', Date.now())

}

exc1();

exc2();
```

exc1 的两个并列await的写法，比较耗时，只有test1执行完了才会执行test2

你可以在浏览器的Console里尝试一下，会发现exc2的用Promise.all执行更快一些



![img](https://upload-images.jianshu.io/upload_images/6522842-4301316d13397359.png?imageMogr2/auto-orient/strip|imageView2/2/w/524/format/webp)

image.png

### 7.兼容性



![img](https://upload-images.jianshu.io/upload_images/6522842-88080cdd92f0dafe.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image.png

**在自己的项目中使用**

通过 babel 来使用。

只需要设置 presets 为 stage-3 即可。

安装依赖：

```
npm install babel-preset-es2015 babel-preset-stage-3 babel-runtime babel-plugin-transform-runtime
```

修改.babelrc:

```bash
 "presets": ["es2015", "stage-3"],

 "plugins": ["transform-runtime"]
```

## 六、实用的数组去重的方法

### 1.**Array.sort()**

首先使用 sort() 将数组进行排序

然后比较相邻元素是否相等，从而排除重复项

```javascript
function distinct(a, b) {
    let arr = a.concat(b)
    arr = arr.sort()
    let result = [arr[0]]

    for (let i=1, len=arr.length; i<len; i++) {
        arr[i] !== arr[i-1] && result.push(arr[i])
    }
    return result
}
```

这种方法只做了一次排序和一次循环，所以效率会比上面的方法都要高

 ![img](https://img2018.cnblogs.com/blog/1059788/201809/1059788-20180920145432043-1219364650.png)

### 2.**new Set()**

ES6 新增了 [Set](http://es6.ruanyifeng.com/#docs/set-map) 这一数据结构，类似于数组，但 **Set 的成员具有唯一性**

基于这一特性，就非常适合用来做数组去重了

```
function distinct(a, b) {
    return Array.from(new Set([...a, ...b]))
}
```

### 3.for...of + Object（最快）

这个方法我只在一些文章里见过，实际工作中倒没怎么用

首先创建一个空对象，然后用 for 循环遍历

利用**对象的属性不会重复**这一特性，校验数组元素是否重复



```javascript
function distinct(a, b) {
    let arr = a.concat(b)
    let result = []
    let obj = {}

    for (let i of arr) {
        if (!obj[i]) {
            result.push(i)
            obj[i] = 1
        }
    }

    return result
}
```

### 4.利用Map数据结构去重

```javascript
function arrayNonRepeatfy(arr) {
  let map = new Map();
  let array = new Array();  // 数组用于返回结果
  for (let i = 0; i < arr.length; i++) {
    if(map .has(arr[i])) {  // 如果有该key值
      map .set(arr[i], true); 
    } else { 
      map .set(arr[i], false);   // 如果没有该key值
      array .push(arr[i]);
    }
  } 
  return array ;
}
 var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
    console.log(unique(arr))
//[1, "a", "true", true, 15, false, 1, {…}, null, NaN, NaN, "NaN", 0, "a", {…}, undefined]
```

创建一个空Map数据结构，遍历需要去重的数组，把数组的每一个元素作为key存到Map中。由于Map中不会出现相同的key值，所以最终得到的就是去重后的结果。

### 5.利用indexOf去重



```javascript
function unique(arr) {
    if (!Array.isArray(arr)) {
        console.log('type error!')
        return
    }
    var array = [];
    for (var i = 0; i < arr.length; i++) {
        if (array .indexOf(arr[i]) === -1) {
            array .push(arr[i])
        }
    }
    return array;
}
var arr = [1,1,'true','true',true,true,15,15,false,false, undefined,undefined, null,null, NaN, NaN,'NaN', 0, 0, 'a', 'a',{},{}];
console.log(unique(arr))
   // [1, "true", true, 15, false, undefined, null, NaN, NaN, "NaN", 0, "a", {…}, {…}]  //NaN、{}没有去重
```

新建一个空的结果数组，for 循环原数组，判断结果数组是否存在当前元素，如果有相同的值则跳过，不相同则push进数组。

## 七、如何判断两个对象相等？

判断对象相等的步骤： 

1. 先判断俩者是不是对象
2. 是对象后俩者长度是否一致
3. 判断俩个对象的所有key值是否相等相同
4. 判断俩个对象的相应的key对应的值是否相同
5. 来一个递归判断里面的对象循环1-4步骤

```javascript
function diff(obj1,obj2){
        var o1 = obj1 instanceof Object;
        var o2 = obj2 instanceof Object;
        // 判断是不是对象
        if (!o1 || !o2) {
            return obj1 === obj2;
        }
 
        //Object.keys() 返回一个由对象的自身可枚举属性(key值)组成的数组,
        //例如：数组返回下表：let arr = ["a", "b", "c"];console.log(Object.keys(arr))->0,1,2;
        if (Object.keys(obj1).length !== Object.keys(obj2).length) {
            return false;
        }
 
        for (var o in obj1) {
            var t1 = obj1[o] instanceof Object;
            var t2 = obj2[o] instanceof Object;
            if (t1 && t2) {
                return diff(obj1[o], obj2[o]);
            } else if (obj1[o] !== obj2[o]) {
                return false;
            }
        }
        return true;
    }
```

##  八、 new的实现原理是什么？

`new` 的实现原理:

1. 创建一个空对象，构造函数中的this指向这个空对象
2. 这个新对象被执行 [[原型]] 连接
3. 执行构造函数方法，属性和方法被添加到this引用的对象中
4. 如果构造函数中没有返回其它对象，那么返回this，即创建的这个的新对象，否则，返回构造函数中返回的对象。

```js
function _new() {
    let target = {}; //创建的新对象
    //第一个参数是构造函数
    let [constructor, ...args] = [...arguments];
    //执行[[原型]]连接;target 是 constructor 的实例
    target.__proto__ = constructor.prototype;
    //执行构造函数，将属性或方法添加到创建的空对象上
    let result = constructor.apply(target, args);
    if (result && (typeof (result) == "object" || typeof (result) == "function")) {
        //如果构造函数执行的结构返回的是一个对象，那么返回这个对象
        return result;
    }
    //如果构造函数返回的不是一个对象，返回创建的新对象
    return target;
}
```

## 九、 如何正确判断this的指向？

如果用一句话说明 this 的指向，那么即是: 谁调用它，this 就指向谁。

但是仅通过这句话，我们很多时候并不能准确判断 this 的指向。因此我们需要借助一些规则去帮助自己：

this 的指向可以按照以下顺序判断:

### 1.全局环境中的 this

浏览器环境：无论是否在严格模式下，在全局执行环境中（在任何函数体外部）this 都指向全局对象 `window`;

node 环境：无论是否在严格模式下，在全局执行环境中（在任何函数体外部），this 都是空对象 `{}`;

### 2.是否是 `new` 绑定

如果是 `new` 绑定，并且构造函数中没有返回 function 或者是 object，那么 this 指向这个新对象。如下:

> 构造函数返回值不是 function 或 object。`new Super()` 返回的是 this 对象。

```js
function Super(age) {
    this.age = age;
}

let instance = new Super('26');
console.log(instance.age); //26
```

> 构造函数返回值是 function 或 object，`new Super()`是返回的是Super种返回的对象。

```js
function Super(age) {
    this.age = age;
    let obj = {a: '2'};
    return obj;
}

let instance = new Super('hello'); 
console.log(instance);//{ a: '2' }
console.log(instance.age); //undefined
```

### 3.函数是否通过 call,apply 调用，或者使用了 bind 绑定，如果是，那么this绑定的就是指定的对象【归结为显式绑定】。

```js
function info(){
    console.log(this.age);
}
var person = {
    age: 20,
    info
}
var age = 28;
var info = person.info;
info.call(person);   //20
info.apply(person);  //20
info.bind(person)(); //20
```

这里同样需要注意一种**特殊**情况，如果 call,apply 或者 bind 传入的第一个参数值是 `undefined` 或者 `null`，严格模式下 this 的值为传入的值 null /undefined。非严格模式下，实际应用的默认绑定规则，this 指向全局对象(node环境为global，浏览器环境为window)

```js
function info(){
    //node环境中:非严格模式 global，严格模式为null
    //浏览器环境中:非严格模式 window，严格模式为null
    console.log(this);
    console.log(this.age);
}
var person = {
    age: 20,
    info
}
var age = 28;
var info = person.info;
//严格模式抛出错误；
//非严格模式，node下输出undefined（因为全局的age不会挂在 global 上）
//非严格模式。浏览器环境下输出 28（因为全局的age会挂在 window 上）
info.call(null);
```

### 4.隐式绑定，函数的调用是在某个对象上触发的，即调用位置上存在上下文对象。典型的隐式调用为: `xxx.fn()`

```js
function info(){
    console.log(this.age);
}
var person = {
    age: 20,
    info
}
var age = 28;
person.info(); //20;执行的是隐式绑定
```

### 5.默认绑定，在不能应用其它绑定规则时使用的默认规则，通常是独立函数调用。

非严格模式： node环境，执行全局对象 global，浏览器环境，执行全局对象 window。

严格模式：执行 undefined

```js
function info(){
    console.log(this.age);
}
var age = 28;
//严格模式；抛错
//非严格模式，node下输出 undefined（因为全局的age不会挂在 global 上）
//非严格模式。浏览器环境下输出 28（因为全局的age会挂在 window 上）
//严格模式抛出，因为 this 此时是 undefined
info();
```

### 6.箭头函数的情况：

箭头函数没有自己的this，继承外层上下文绑定的this。

```js
let obj = {
    age: 20,
    info: function() {
        return () => {
            console.log(this.age); //this继承的是外层上下文绑定的this
        }
    }
}

let person = {age: 28};
let info = obj.info();
info(); //20

let info2 = obj.info.call(person);
info2(); //28
```

## 十、 深拷贝和浅拷贝的区别是什么？实现一个深拷贝

深拷贝和浅拷贝是针对复杂数据类型来说的，浅拷贝只拷贝一层，而深拷贝是层层拷贝。

### 1.深拷贝

> 深拷贝复制变量值，对于非基本类型的变量，则递归至基本类型变量后，再复制。 深拷贝后的对象与原来的对象是完全隔离的，互不影响，对一个对象的修改并不会影响另一个对象。

### 2.浅拷贝

> 浅拷贝是会将对象的每个属性进行依次复制，但是当对象的属性值是引用类型时，实质复制的是其引用，当引用指向的值改变时也会跟着变化。

可以使用 `for in`、 `Object.assign`、 扩展运算符 `...` 、`Array.prototype.slice()`、`Array.prototype.concat()` 等，例如:

```js
let obj = {
    name: 'Yvette',
    age: 18,
    hobbies: ['reading', 'photography']
}
let obj2 = Object.assign({}, obj);
let obj3 = {...obj};

obj.name = 'Jack';
obj.hobbies.push('coding');
console.log(obj);//{ name: 'Jack', age: 18,hobbies: [ 'reading', 'photography', 'coding' ] }
console.log(obj2);//{ name: 'Yvette', age: 18,hobbies: [ 'reading', 'photography', 'coding' ] }
console.log(obj3);//{ name: 'Yvette', age: 18,hobbies: [ 'reading', 'photography', 'coding' ] }
```

可以看出浅拷贝只最第一层属性进行了拷贝，当第一层的属性值是基本数据类型时，新的对象和原对象互不影响，但是如果第一层的属性值是复杂数据类型，那么新对象和原对象的属性值其指向的是同一块内存地址。

### 3.深拷贝实现

> 1.深拷贝最简单的实现是: `JSON.parse(JSON.stringify(obj))`

`JSON.parse(JSON.stringify(obj))` 是最简单的实现方式，但是有一些缺陷：

1. 对象的属性值是函数时，无法拷贝。
2. 原型链上的属性无法拷贝
3. 不能正确的处理 Date 类型的数据
4. 不能处理 RegExp
5. 会忽略 symbol
6. 会忽略 undefined

> 2.实现一个 deepClone 函数

1. 如果是基本数据类型，直接返回
2. 如果是 `RegExp` 或者 `Date` 类型，返回对应类型
3. 如果是复杂数据类型，递归。
4. 考虑循环引用的问题

```js
function deepClone(obj, hash = new WeakMap()) { //递归拷贝
    if (obj instanceof RegExp) return new RegExp(obj);
    if (obj instanceof Date) return new Date(obj);
    if (obj === null || typeof obj !== 'object') {
        //如果不是复杂数据类型，直接返回
        return obj;
    }
    if (hash.has(obj)) {
        return hash.get(obj);
    }
    /**
     * 如果obj是数组，那么 obj.constructor 是 [Function: Array]
     * 如果obj是对象，那么 obj.constructor 是 [Function: Object]
     */
    let t = new obj.constructor();
    hash.set(obj, t);
    for (let key in obj) {
        //递归
        if (obj.hasOwnProperty(key)) {//是否是自身的属性
            t[key] = deepClone(obj[key], hash);
        }
    }
    return t;
}
```

## 十一、call/apply 的实现原理是什么？

call 和 apply 的功能相同，都是改变 `this` 的执行，并立即执行函数。区别在于传参方式不同。

- `func.call(thisArg, arg1, arg2, ...)`：第一个参数是 `this` 指向的对象，其它参数依次传入。
- `func.apply(thisArg, [argsArray])`：第一个参数是 `this` 指向的对象，第二个参数是数组或类数组。

一起思考一下，如何模拟实现 `call` ？

首先，我们知道，函数都可以调用 `call`，说明 `call` 是函数原型上的方法，所有的实例都可以调用。即: `Function.prototype.call`。

- 在 `call` 方法中获取调用`call()`函数
- 如果第一个参数没有传入，那么默认指向 `window / global`(非严格模式)
- 传入 `call` 的第一个参数是 this 指向的对象，根据隐式绑定的规则，我们知道 `obj.foo()`, `foo()` 中的 `this` 指向 `obj`;因此我们可以这样调用函数 `thisArgs.func(...args)`
- 返回执行结果

```js
Function.prototype.call = function() {
    let [thisArg, ...args] = [...arguments];
    if (!thisArg) {
        //context为null或者是undefined
        thisArg = typeof window === 'undefined' ? global : window;
    }
    //this的指向的是当前函数 func (func.call)
    thisArg.func = this;
    //执行函数
    let result = thisArg.func(...args);
    delete thisArg.func; //thisArg上并没有 func 属性，因此需要移除
    return result;
}
```

bind 的实现思路和 `call` 一致，仅参数处理略有差别。如下：

```js
Function.prototype.apply = function(thisArg, rest) {
    let result; //函数返回结果
    if (!thisArg) {
        //context为null或者是undefined
        thisArg = typeof window === 'undefined' ? global : window;
    }
    //this的指向的是当前函数 func (func.call)
    thisArg.func = this;
    if(!rest) {
        //第二个参数为 null / undefined 
        result = thisArg.func();
    }else {
        result = thisArg.func(...rest);
    }
    delete thisArg.func; //thisArg上并没有 func 属性，因此需要移除
    return result;
}
```

## 十二、 柯里化函数实现

在开始之前，我们首先需要搞清楚函数柯里化的概念。

函数柯里化是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。

```js
const curry = (fn, ...args) =>
    args.length < fn.length
        //参数长度不足时，重新柯里化该函数，等待接受新参数
        ? (...arguments) => curry(fn, ...args, ...arguments)
        //参数长度满足时，执行函数
        : fn(...args);
function sumFn(a, b, c) {
    return a + b + c;
}
var sum = curry(sumFn);
console.log(sum(2)(3)(5));//10
console.log(sum(2, 3, 5));//10
console.log(sum(2)(3, 5));//10
console.log(sum(2, 3)(5));//10
```

> 函数柯里化的主要作用：

- 参数复用
- 提前返回 – 返回接受余下的参数且返回结果的新函数
- 延迟执行 – 返回新函数，等待执行

## 十三、如何让 (a == 1 && a == 2 && a == 3) 的值为true？

### 1.利用隐式类型转换



`==` 操作符在左右数据类型不一致时，会先进行隐式转换。

`a == 1 && a == 2 && a == 3` 的值意味着其不可能是基本数据类型。因为如果 a 是 null 或者是 undefined bool类型，都不可能返回true。

因此可以推测 a 是复杂数据类型，JS 中复杂数据类型只有 `object`，回忆一下，Object 转换为原始类型会调用什么方法？

- 如果部署了 `[Symbol.toPrimitive]` 接口，那么调用此接口，若返回的不是基本数据类型，抛出错误。
- 如果没有部署 `[Symbol.toPrimitive]` 接口，那么根据要转换的类型，先调用 `valueOf` / `toString`

1. 非Date类型对象，`hint` 是 `default` 时，调用顺序为：`valueOf` >>> `toString`，即`valueOf` 返回的不是基本数据类型，才会继续调用 `valueOf`，如果`toString` 返回的还不是基本数据类型，那么抛出错误。
2. 如果 `hint` 是 `string`(Date对象的hint默认是string) ，调用顺序为：`toString` >>> `valueOf`，即`toString` 返回的不是基本数据类型，才会继续调用 `valueOf`，如果`valueOf` 返回的还不是基本数据类型，那么抛出错误。
3. 如果 `hint` 是 `number`，调用顺序为： `valueOf` >>> `toString`



```js
//部署 [Symbol.toPrimitive] / valueOf/ toString 皆可
//一次返回1，2，3 即可。
let a = {
    [Symbol.toPrimitive]: (function(hint) {
            let i = 1;
            //闭包的特性之一：i 不会被回收
            return function() {
                return i++;
            }
    })()
}
```

### 2.利用数据劫持(Proxy/Object.definedProperty)



```js
let i = 1;
let a = new Proxy({}, {
    i: 1,
    get: function () {
        return () => this.i++;
    }
});
```

### 3.数组的 `toString` 接口默认调用数组的 `join` 方法，重新 `join` 方法



```js
let a = [1, 2, 3];
a.join = a.shift;
```

## 十四、异步加载JS脚本的方式有哪些？

> `<script>` 标签中增加 `async`(html5) 或者 `defer`(html4) 属性,脚本就会异步加载。

```js
<script src="../XXX.js" defer></script>
```

`defer` 和 `async` 的区别在于：

- `defer` 要等到整个页面在内存中正常渲染结束（DOM 结构完全生成，以及其他脚本执行完成），在window.onload 之前执行；
- `async` 一旦下载完，渲染引擎就会中断渲染，执行这个脚本以后，再继续渲染。
- 如果有多个 `defer` 脚本，会按照它们在页面出现的顺序加载
- 多个 `async` 脚本不能保证加载顺序

> 动态创建 `script` 标签

动态创建的 `script` ，设置 `src` 并不会开始下载，而是要添加到文档中，JS文件才会开始下载。

```js
let script = document.createElement('script');
script.src = 'XXX.js';
// 添加到html文件中才会开始下载
document.body.append(script);
```

> XHR 异步加载JS

```js
let xhr = new XMLHttpRequest();
xhr.open("get", "js/xxx.js",true);
xhr.send();
xhr.onreadystatechange = function() {
    if (xhr.readyState == 4 && xhr.status == 200) {
        eval(xhr.responseText);
    }
}
```

## 十五、 ES5有几种方式可以实现继承？分别有哪些优缺点？

ES5 有 6 种方式可以实现继承，分别为：

### 1. 原型链继承

原型链继承的基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

```js
function SuperType() {
    this.name = 'Yvette';
    this.colors = ['pink', 'blue', 'green'];
}
SuperType.prototype.getName = function () {
    return this.name;
}
function SubType() {
    this.age = 22;
}
SubType.prototype = new SuperType();
SubType.prototype.getAge = function() {
    return this.age;
}
SubType.prototype.constructor = SubType;
let instance1 = new SubType();
instance1.colors.push('yellow');
console.log(instance1.getName()); //'Yvette'
console.log(instance1.colors);//[ 'pink', 'blue', 'green', 'yellow' ]

let instance2 = new SubType();
console.log(instance2.colors);//[ 'pink', 'blue', 'green', 'yellow' ]
```

> 缺点：

1. 通过原型来实现继承时，原型会变成另一个类型的实例，原先的实例属性变成了现在的原型属性，该原型的引用类型属性会被所有的实例共享。
2. 在创建子类型的实例时，没有办法在不影响所有对象实例的情况下给超类型的构造函数中传递参数。

### 2. 借用构造函数

**借用构造函数**的技术，其基本思想为:

在子类型的构造函数中调用超类型构造函数。

```js
function SuperType(name) {
    this.name = name;
    this.colors = ['pink', 'blue', 'green'];
}
function SubType(name) {
    SuperType.call(this, name);
}
let instance1 = new SubType('Yvette');
instance1.colors.push('yellow');
console.log(instance1.colors);//['pink', 'blue', 'green', yellow]

let instance2 = new SubType('Jack');
console.log(instance2.colors); //['pink', 'blue', 'green']
```

> 优点:

1. 可以向超类传递参数
2. 解决了原型中包含引用类型值被所有实例共享的问题

> 缺点:

1. 方法都在构造函数中定义，函数复用无从谈起，另外超类型原型中定义的方法对于子类型而言都是不可见的。

### 3. 组合继承(原型链 + 借用构造函数)

组合继承指的是将原型链和借用构造函数技术组合到一块，从而发挥二者之长的一种继承模式。基本思路：

使用原型链实现对原型属性和方法的继承，通过借用构造函数来实现对实例属性的继承，既通过在原型上定义方法来实现了函数复用，又保证了每个实例都有自己的属性。

```js
function SuperType(name) {
    this.name = name;
    this.colors = ['pink', 'blue', 'green'];
}
SuperType.prototype.sayName = function () {
    console.log(this.name);
}
function SuberType(name, age) {
    SuperType.call(this, name);
    this.age = age;
}
SuberType.prototype = new SuperType();
SuberType.prototype.constructor = SuberType;
SuberType.prototype.sayAge = function () {
    console.log(this.age);
}
let instance1 = new SuberType('Yvette', 20);
instance1.colors.push('yellow');
console.log(instance1.colors); //[ 'pink', 'blue', 'green', 'yellow' ]
instance1.sayName(); //Yvette

let instance2 = new SuberType('Jack', 22);
console.log(instance2.colors); //[ 'pink', 'blue', 'green' ]
instance2.sayName();//Jack
```

> 缺点:

- 无论什么情况下，都会调用两次超类型构造函数：一次是在创建子类型原型的时候，另一次是在子类型构造函数内部。

> 优点:

- 可以向超类传递参数
- 每个实例都有自己的属性
- 实现了函数复用

### 4. 原型式继承

原型继承的基本思想：

借助原型可以基于已有的对象创建新对象，同时还不必因此创建自定义类型。

```js
function object(o) {
    function F() { }
    F.prototype = o;
    return new F();
}
```

在 `object()` 函数内部，先穿甲一个临时性的构造函数，然后将传入的对象作为这个构造函数的原型，最后返回了这个临时类型的一个新实例，从本质上讲，`object()` 对传入的对象执行了一次浅拷贝。

ECMAScript5通过新增 `Object.create()`方法规范了原型式继承。这个方法接收两个参数：一个用作新对象原型的对象和（可选的）一个为新对象定义额外属性的对象(可以覆盖原型对象上的同名属性)，在传入一个参数的情况下，`Object.create()` 和 `object()` 方法的行为相同。

```js
var person = {
    name: 'Yvette',
    hobbies: ['reading', 'photography']
}
var person1 = Object.create(person);
person1.name = 'Jack';
person1.hobbies.push('coding');
var person2 = Object.create(person);
person2.name = 'Echo';
person2.hobbies.push('running');
console.log(person.hobbies);//[ 'reading', 'photography', 'coding', 'running' ]
console.log(person1.hobbies);//[ 'reading', 'photography', 'coding', 'running' ]
```

在没有必要创建构造函数，仅让一个对象与另一个对象保持相似的情况下，原型式继承是可以胜任的。

> 缺点:

同原型链实现继承一样，包含引用类型值的属性会被所有实例共享。

### 5. 寄生式继承

寄生式继承是与原型式继承紧密相关的一种思路。寄生式继承的思路与寄生构造函数和工厂模式类似，即创建一个仅用于封装继承过程的函数，该函数在内部已某种方式来增强对象，最后再像真地是它做了所有工作一样返回对象。

```js
function createAnother(original) {
    var clone = object(original);//通过调用函数创建一个新对象
    clone.sayHi = function () {//以某种方式增强这个对象
        console.log('hi');
    };
    return clone;//返回这个对象
}
var person = {
    name: 'Yvette',
    hobbies: ['reading', 'photography']
};

var person2 = createAnother(person);
person2.sayHi(); //hi
```

基于 `person` 返回了一个新对象 -—— `person2`，新对象不仅具有 `person` 的所有属性和方法，而且还有自己的 `sayHi()`方法。在考虑对象而不是自定义类型和构造函数的情况下，寄生式继承也是一种有用的模式。

> 缺点：

- 使用寄生式继承来为对象添加函数，会由于不能做到函数复用而效率低下。
- 同原型链实现继承一样，包含引用类型值的属性会被所有实例共享。

### 6. 寄生组合式继承

所谓寄生组合式继承，即通过借用构造函数来继承属性，通过原型链的混成形式来继承方法，基本思路：

不必为了指定子类型的原型而调用超类型的构造函数，我们需要的仅是超类型原型的一个副本，本质上就是使用寄生式继承来继承超类型的原型，然后再将结果指定给子类型的原型。寄生组合式继承的基本模式如下所示：

```js
function inheritPrototype(subType, superType) {
    var prototype = object(superType.prototype); //创建对象
    prototype.constructor = subType;//增强对象
    subType.prototype = prototype;//指定对象
}
```

- 第一步：创建超类型原型的一个副本
- 第二步：为创建的副本添加 `constructor` 属性
- 第三步：将新创建的对象赋值给子类型的原型

至此，我们就可以通过调用 `inheritPrototype` 来替换为子类型原型赋值的语句：

```js
function SuperType(name) {
    this.name = name;
    this.colors = ['pink', 'blue', 'green'];
}
//...code
function SuberType(name, age) {
    SuperType.call(this, name);
    this.age = age;
}
SuberType.prototype = new SuperType();
inheritPrototype(SuberType, SuperType);
//...code
```

> 优点:

只调用了一次超类构造函数，效率更高。避免在`SuberType.prototype`上面创建不必要的、多余的属性，与其同时，原型链还能保持不变。

因此寄生组合继承是引用类型最理性的继承范式。

## 十六、说一说你对JS执行上下文栈和作用域链的理解？

在开始说明JS上下文栈和作用域之前，我们先说明下JS上下文以及作用域的概念。

### 1.JS执行上下文

执行上下文就是当前 JavaScript 代码被解析和执行时所在环境的抽象概念， JavaScript 中运行任何的代码都是在执行上下文中运行。

> 执行上下文类型分为：

- 全局执行上下文
- 函数执行上下文

执行上下文创建过程中，需要做以下几件事:

1. 创建变量对象：首先初始化函数的参数arguments，提升函数声明和变量声明。
2. 创建作用域链（Scope Chain）：在执行期上下文的创建阶段，作用域链是在变量对象之后创建的。
3. 确定this的值，即 ResolveThisBinding

### 2.作用域

**作用域**负责收集和维护由所有声明的标识符（变量）组成的一系列查询，并实施一套非常严格的规则，确定当前执行的代码对这些标识符的访问权限。—— 摘录自《你不知道的JavaScript》(上卷)

作用域有两种工作模型：词法作用域和动态作用域，JS采用的是**词法作用域**工作模型，词法作用域意味着作用域是由书写代码时变量和函数声明的位置决定的。(`with` 和 `eval` 能够修改词法作用域，但是不推荐使用，对此不做特别说明)

> 作用域分为：

- 全局作用域
- 函数作用域
- 块级作用域

### 3.JS执行上下文栈(后面简称执行栈)

执行栈，也叫做调用栈，具有 **LIFO** (后进先出) 结构，用于存储在代码执行期间创建的所有执行上下文。

> 规则如下：

- 首次运行JavaScript代码的时候,会创建一个全局执行的上下文并Push到当前的执行栈中，每当发生函数调用，引擎都会为该函数创建一个新的函数执行上下文并Push当前执行栈的栈顶。
- 当栈顶的函数运行完成后，其对应的函数执行上下文将会从执行栈中Pop出，上下文的控制权将移动到当前执行栈的下一个执行上下文。

以一段代码具体说明：

```js
function fun3() {
    console.log('fun3')
}

function fun2() {
    fun3();
}

function fun1() {
    fun2();
}

fun1();
```

`Global Execution Context` (即全局执行上下文)首先入栈，过程如下：



![img](https://pic3.zhimg.com/80/v2-c878ea1e22663992087af3ec55929b3e_hd.jpg)



伪代码:

```java
//全局执行上下文首先入栈
ECStack.push(globalContext);

//执行fun1();
ECStack.push(<fun1> functionContext);

//fun1中又调用了fun2;
ECStack.push(<fun2> functionContext);

//fun2中又调用了fun3;
ECStack.push(<fun3> functionContext);

//fun3执行完毕
ECStack.pop();

//fun2执行完毕
ECStack.pop();

//fun1执行完毕
ECStack.pop();

//javascript继续顺序执行下面的代码，但ECStack底部始终有一个 全局上下文（globalContext）;
```

### 4.作用域链

作用域链就是从当前作用域开始一层一层向上寻找某个变量，直到找到全局作用域还是没找到，就宣布放弃。这种一层一层的关系，就是作用域链。

如：

```js
var a = 10;
function fn1() {
    var b = 20;
    console.log(fn2)
    function fn2() {
        a = 20
    }
    return fn2;
}
fn1()();
```

fn2作用域链 = [fn2作用域, fn1作用域，全局作用域]



![img](https://pic4.zhimg.com/80/v2-39a2d2b72b55d09a1879423c840bbe63_hd.jpg)

## 十七、什么是闭包？闭包的作用是什么？

### 1.闭包的定义

《JavaScript高级程序设计》:

> 闭包是指有权访问另一个函数作用域中的变量的函数

《JavaScript权威指南》：

> 从技术的角度讲，所有的JavaScript函数都是闭包：它们都是对象，它们都关联到作用域链。

《你不知道的JavaScript》

> 当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。

### 2.创建一个闭包

```js
function foo() {
    var a = 2;
    return function fn() {
        console.log(a);
    }
}
let func = foo();
func(); //输出2
```

闭包使得函数可以继续访问定义时的词法作用域。拜 fn 所赐，在 foo() 执行后，foo 内部作用域不会被销毁。

### 3.闭包的作用

1. 能够访问函数定义时所在的词法作用域(阻止其被回收)。
2. 私有化变量

```js
function base() {
    let x = 10; //私有变量
    return {
        getX: function() {
            return x;
        }
    }
}
let obj = base();
console.log(obj.getX()); //10
```

1. 模拟块级作用域

```js
var a = [];
for (var i = 0; i < 10; i++) {
    a[i] = (function(j){
        return function () {
            console.log(j);
        }
    })(i);
}
a[6](); // 6
```

1. 创建模块

```js
function coolModule() {
    let name = 'Yvette';
    let age = 20;
    function sayName() {
        console.log(name);
    }
    function sayAge() {
        console.log(age);
    }
    return {
        sayName,
        sayAge
    }
}
let info = coolModule();
info.sayName(); //'Yvette'
```

模块模式具有两个必备的条件(来自《你不知道的JavaScript》)

- 必须有外部的封闭函数，该函数必须至少被调用一次(每次调用都会创建一个新的模块实例)
- 封闭函数必须返回至少**一个**内部函数，这样内部函数才能在私有作用域中形成闭包，并且可以访问或者修改私有的状态。

  

## 十八、请实现一个 flattenDeep 函数，把嵌套的数组扁平化

例如:

```js
flattenDeep([1, [2, [3, [4]], 5]]); //[1, 2, 3, 4, 5]
```

> 利用 Array.prototype.flat

ES6 为数组实例新增了 `flat` 方法，用于将嵌套的数组“拉平”，变成一维的数组。该方法返回一个新数组，对原数组没有影响。

`flat` 默认只会 “拉平” 一层，如果想要 “拉平” 多层的嵌套数组，需要给 `flat` 传递一个整数，表示想要拉平的层数。

```js
function flattenDeep(arr, deepLength) {
    return arr.flat(deepLength);
}
console.log(flattenDeep([1, [2, [3, [4]], 5]], 3));
```

当传递的整数大于数组嵌套的层数时，会将数组拉平为一维数组，JS能表示的最大数字为 `Math.pow(2, 53) - 1`，因此我们可以这样定义 `flattenDeep` 函数

```js
function flattenDeep(arr) {
    //当然，大多时候我们并不会有这么多层级的嵌套
    return arr.flat(Math.pow(2,53) - 1); 
}
console.log(flattenDeep([1, [2, [3, [4]], 5]]));
```

> 利用 reduce 和 concat

```js
function flattenDeep(arr){
    return arr.reduce((acc, val) => Array.isArray(val) ? acc.concat(flattenDeep(val)) : acc.concat(val), []);
}
console.log(flattenDeep([1, [2, [3, [4]], 5]]));
```

> 使用 stack 无限反嵌套多层嵌套数组

```js
function flattenDeep(input) {
    const stack = [...input];
    const res = [];
    while (stack.length) {
        // 使用 pop 从 stack 中取出并移除值
        const next = stack.pop();
        if (Array.isArray(next)) {
            // 使用 push 送回内层数组中的元素，不会改动原始输入 original input
            stack.push(...next);
        } else {
            res.push(next);
        }
    }
    // 使用 reverse 恢复原数组的顺序
    return res.reverse();
}
console.log(flattenDeep([1, [2, [3, [4]], 5]]));
```

## 十九、请实现一个 uniq 函数，实现数组去重

例如:

```js
uniq([1, 2, 3, 5, 3, 2]);//[1, 2, 3, 5]
```

> 法1: 利用ES6新增数据类型 `Set`

`Set`类似于数组，但是成员的值都是唯一的，没有重复的值。

```js
function uniq(arry) {
    return [...new Set(arry)];
}
```

> 法2: 利用 `indexOf`

```js
function uniq(arry) {
    var result = [];
    for (var i = 0; i < arry.length; i++) {
        if (result.indexOf(arry[i]) === -1) {
            //如 result 中没有 arry[i],则添加到数组中
            result.push(arry[i])
        }
    }
    return result;
}
```

> 法3: 利用 `includes`

```js
function uniq(arry) {
    var result = [];
    for (var i = 0; i < arry.length; i++) {
        if (!result.includes(arry[i])) {
            //如 result 中没有 arry[i],则添加到数组中
            result.push(arry[i])
        }
    }
    return result;
}
```

> 法4：利用 `reduce`

```js
function uniq(arry) {
    return arry.reduce((prev, cur) => prev.includes(cur) ? prev : [...prev, cur], []);
}
```

> 法5：利用 `Map`

```js
function uniq(arry) {
    let map = new Map();
    let result = new Array();
    for (let i = 0; i < arry.length; i++) {
        if (map.has(arry[i])) {
            map.set(arry[i], true);
        } else {
            map.set(arry[i], false);
            result.push(arry[i]);
        }
    }
    return result;
}
```