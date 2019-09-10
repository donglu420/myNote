# ES 6 

promise是为了解决回调地狱=>因为发送Ajax请求不确定性所以传统的请求方式必须是回调函数 但是多了就会地狱

## Promise

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

## 箭头函数

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

## 类class

是es6的语法糖 是一个function但它不具备像函数那样的函数提升，不能在定义前调用它

static方法 静态方法，只能通过原型对象调用 不能通过实例来调用

Array.of() //可以

[1,2,3].of()//不可以



## Genenator函数在Ajax的使用

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

