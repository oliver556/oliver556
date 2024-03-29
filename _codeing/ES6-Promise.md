#  ES6 Promise

## 前言

鉴于自己对promise的理解还不是非常的深入及全面，特此在这里记录一下，结合各路大神的指点

---

## 什么是Promise

Promise 是异步编程的一种解决方案，其实是一个构造函数，自己身上有 `all`、`reject`、`resolve` 这几个方法，原型上有 `then`、`catch` 等方法。

Promise 对象有以下两个特点

（1）对象的状态不受外界影响。Promise 对象代表一个异步操作，有三种状态：`pending（进行中）`、`fulfilled（已成功）` 和 `rejected（已失败）`。只有异步操作的结果，可以决定当前是哪一种状态，任何其他操作都无法改变这个状态。这也是 Promise 这个名字的由来，它的英语意思就是“承诺”，表示其他手段无法改变。

（2）一旦改变状态，就不会再改变，任何时候都可以得到这个结果。Promise 对象的状态改变，只有两种可能：从 `pending` 变为 `fulfilled` 和从 `pending` 变为`rejected`。只要这两种情况发生，状态就凝固了，不会再变了，会一直保持这个结果，这时就成为 `resolved（已定形）`。如果改变已经发生了，你再对 Promise 对象添加回调函数，也会立即得到这个结果。这与事件（Event）完全不同，事件的特点是，如果你错过了它，再去监听，是得不到结果的。

下面先 new 一个 promise

```javascript
let p = new Promise(function(resolve, reject){
  // 做一些异步操作
  setTimeout(function(){
    console.log('执行完成Promise');
    resolve('要返回的数据可以任何数据例如接口返回数据');
  }, 2000);
})
```

刷新页面会发现控制台直接打出

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8h8syi2gyj30cd081jte.jpg)

其执行过程是：执行了一个异步操作，也就是setTimeout，2秒后，输出“执行完成”，并且调用 `resolve` 方法。

<div class="note warning"><p>注意！我只是new了一个对象，并没有调用它，我们传进去的函数就已经执行了，这是需要注意的一个细节。所以我们用 Promise 的时候一般是包在一个函数中，在需要的时候去运行这个函数，如：
</p></div>

```html
<div onClick={promiseClick}>开始异步请求</div>
```

```javascript
const promiseClick =()=>{
  console.log('点击方法被调用')
  let p = new Promise(function(resolve, reject){
  //做一些异步操作
  setTimeout(function(){
    console.log('执行完成Promise');
    resolve('要返回的数据可以任何数据例如接口返回数据');
    }, 2000);
  });
  return p
}
```

刷新页面的时候是没有任何反映的，但是点击后控制台打出

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8h8wkgu4xj308602et8o.jpg)

当放在函数里面的时候只有调用的时候才会被执行

**那么，接下里解决两个问题：**

1、为什么要放在函数里面

2、resolve是个什么鬼

我们包装好的函数最后，会 return 出 Promise 对象，也就是说，执行这个函数我们得到了一个 Promise 对象。接下来就可以用 Promise 对象上有 `then`、`catch` 方法了，这就是 Promise 的强大之处了，看下面的代码：

```javascript
promiseClick().then(function(data){
  console.log(data);
  //后面可以用传过来的数据做些其他操作
  //......
});
```

这样控制台输出

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8h8xduyp1j309701kwej.jpg)

先是方法被调用起床执行了 promise ,最后执行了 promise 的 `then` 方法，then方法是一个函数接受一个参数是接受 `resolve` 返回的数据这事就输出了‘要返回的数据可以任何数据例如接口返回数据’

这时候你应该有所领悟了，原来 `then` 里面的函数就跟我们平时的回调函数一个意思，能够在 `promiseClick` 这个异步任务执行完成之后被执行。这就是 Promise 的作用了，简单来讲，就是能把原来的回调写法分离出来，在异步操作执行完后，用链式调用的方式执行回调函数。

你可能会觉得在这个和写一个回调函数没有什么区别；那么，如果有多层回调该怎么办？如果 `callback` 也是一个异步操作，而且执行完后也需要有相应的回调函数，该怎么办呢？总不能再定义一个 callback2，然后给 callback 传进去吧。而 Promise 的优势在于，可以在 `then`方法中继续写 Promise 对象并返回，然后继续调用 `then`来进行回调操作。

所以：精髓在于：Promise 只是能够简化层层回调的写法，而实质上，Promise 的精髓是 `“状态”`，用维护状态、传递状态的方式来使得回调函数能够及时调用，它比传递 `callback` 函数要简单、灵活的多。所以 `使用Promise的正确场景是这样的`：

```javascript
promiseClick()
  .then(function(data){
      console.log(data);
      return runAsync2();
  })
  .then(function(data){
      console.log(data);
      return runAsync3();
  })
  .then(function(data){
      console.log(data);
  });
```

这样能够按顺序，每隔两秒输出每个异步回调中的内容，在 runAsync2 中传给 resolve 的数据，能在接下来的 then 方法中拿到。

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8h8y4ewtwj308y09ojss.jpg)

（Ps：此处执行多次是因为研究该用法的时候我在一个 react 的 demo 中进行的，该页面多个元素改变导致页面多次渲染执行所致，正常页面只渲染一次的话就所有只会执行一次）

---

## reject的用法

以上是对 promise 的 `resolve` 用法进行了解释，相当于 resolve 是对 promise 成功时候的回调，它把 promise 的状态修改为`fullfiled`，  
那么，reject 就是失败的时候的回调，他把 promise 的状态修改为 `rejected` ，这样我们在 `then` 中就能捕捉到，然后执行“失败”情况的回调。

```javascript
function promiseClick(){
  let p = new Promise(function(resolve, reject){
    setTimeout(function(){
      var num = Math.ceil(Math.random()*20); //生成1-10的随机数
      console.log('随机数生成的值：',num)
      if(num<=10){
        resolve(num);
      }
      else{
        reject('数字太于10了即将执行失败回调');
      }
    }, 2000);
      })
      return p
    }

promiseClick().then(
  function(data){
    console.log('resolved成功回调');
    console.log('成功回调接受的值：',data);
  },
  function(reason, data){
    console.log('rejected失败回调');
    console.log('失败执行回调抛出失败原因：',reason);
  }
);
```

执行结果：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8h8ze5rx2j30c909wq46.jpg)

（PS：此处也是执行多次所以输出多次，执行多次的原因和上次原因一致）

以上代码：调用 promiseClick 方法执行，2秒后获取到一个随机数，如果小于10，我们算成功，调用 `resolve` 修改 Promise 的状态为 `fullfiled`。否则我们认为是“失败”了，调用 `reject` 并传递一个参数，作为失败的原因。并将状态改成 `rejected`

运行 promiseClick 并且在 then 中传了两个参数，这两个参数分别是两个函数，then 方法可以接受两个参数，第一个对应 resolve 的回调，第二个对应 reject 的回调。（也就是说 `then方法中接受两个回调`，一个成功的回调函数，一个失败的回调函数，并且能在回调函数中拿到成功的数据和失败的原因），所以我们能够分别拿到成功和失败传过来的数据就有以上的运行结果

---

## catch的用法

与 Promise 对象方法 then 方法并行的一个方法就是 `catch` ,与 `try`  `catch`类似，catch就是用来捕获异常的，也就是和then方法中接受的第二参数rejected的回调是一样的，如下：

```javascript
function promiseClick(){
  let p = new Promise(function(resolve, reject){
    setTimeout(function(){
      var num = Math.ceil(Math.random()*20); //生成1-10的随机数
      console.log('随机数生成的值：',num)
      if(num<=10){
        resolve(num);
      }
      else{
        reject('数字太于10了即将执行失败回调');
      }
    }, 2000);
      })
      return p
    }

promiseClick().then(
  function(data){
    console.log('resolved成功回调');
    console.log('成功回调接受的值：',data);
  }
)
.catch(function(reason, data){
  console.log('catch到rejected失败回调');
  console.log('catch失败执行回调抛出失败原因：',reason);
});
```

执行结果：![ ](https://tva1.sinaimg.cn/large/006y8mN6gy1g8hdk00d9yj30d905vaaw.jpg)

效果和写在 then 的第二个参数里面一样。它将大于10的情况下的失败回调的原因输出，但是，它还有另外一个作用：在执行 resolve 的回调（也就是上面then中的第一个参数）时，如果抛出异常了（代码出错了），那么并不会报错卡死js，而是会进到这个 catch 方法中。如下：

```javascript
function promiseClick(){
  let p = new Promise(function(resolve, reject){
    setTimeout(function(){
      var num = Math.ceil(Math.random()*20); //生成1-10的随机数
      console.log('随机数生成的值：',num)
      if(num<=10){
        resolve(num);
      }
      else{
        reject('数字太于10了即将执行失败回调');
      }
    }, 2000);
      })
      return p
    }

promiseClick().then(
  function(data){
    console.log('resolved成功回调');
    console.log('成功回调接受的值：',data);
    console.log(noData);
  }
)
.catch(function(reason, data){
  console.log('catch到rejected失败回调');
  console.log('catch失败执行回调抛出失败原因：',reason);
});
```

执行结果：![ ](https://tva1.sinaimg.cn/large/006y8mN6gy1g8hdl55rozj30ez01ft8v.jpg)

在 resolve 的回调中，我们 console.log(noData) ;而 noData 这个变量是没有被定义的。如果我们不用 Promise，代码运行到这里就直接在控制台报错了，不往下运行了。但是在这里，会得到上图的结果，也就是说进到 catch 方法里面去了，而且把错误原因传到了 reason 参数中。即便是有错误的代码也不会报错了

---

## all的用法

与 then 同级的另一个方法，all 方法，该方法提供了并行执行异步操作的能力，并且在所有异步操作执行完后并且执行结果都是成功的时候才执行回调。

将上述方法复制两份并重命名 `promiseClick3()`, `promiseClick2()`, `promiseClick1()`，如下

```javascript
function promiseClick1(){
  let p = new Promise(function(resolve, reject){
    setTimeout(function(){
      var num = Math.ceil(Math.random()*20); //生成1-10的随机数
      console.log('随机数生成的值：',num)
      if(num<=10){
        resolve(num);
      }
      else{
        reject('数字太于10了即将执行失败回调');
      }
    }, 2000);
      })
      return p
    }
    function promiseClick2(){
  let p = new Promise(function(resolve, reject){
    setTimeout(function(){
      var num = Math.ceil(Math.random()*20); //生成1-10的随机数
      console.log('随机数生成的值：',num)
      if(num<=10){
        resolve(num);
      }
      else{
        reject('数字太于10了即将执行失败回调');
      }
    }, 2000);
      })
      return p
    }
    function promiseClick3(){
  let p = new Promise(function(resolve, reject){
    setTimeout(function(){
      var num = Math.ceil(Math.random()*20); //生成1-10的随机数
      console.log('随机数生成的值：',num)
      if(num<=10){
        resolve(num);
      }
      else{
        reject('数字太于10了即将执行失败回调');
      }
    }, 2000);
      })
      return p
    }

Promise
  .all([promiseClick3(), promiseClick2(), promiseClick1()])
  .then(function(results){
    console.log(results);
  });
```

`Promise.all` 来执行，all 接收一个数组参数，这组参数为需要执行异步操作的所有方法，里面的值最终都算返回 Promise 对象。这样，三个异步操作的并行执行的，等到它们都执行完后才会进到 then 里面。那么，三个异步操作返回的数据哪里去了呢？都在 then 里面，all 会把所有异步操作的结果放进一个数组中传给 then，然后再执行 then 方法的成功回调将结果接收，结果如下：（分别执行得到结果，all 统一执行完三个函数并将值存在一个数组里面返回给 then 进行回调输出）：

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8hdmm224ij305r027dfv.jpg)

这样以后就可以用 all 并行执行多个异步操作，并且在一个回调中处理所有的返回数据，比如你需要提前准备好所有数据才渲染页面的时候就可以使用 all,执行多个异步操作将所有的数据处理好，再去渲染

---

## race的用法

`all是等所有的异步操作都执行完了再执行then方法`，那么 `race` 方法就是相反的，`谁先执行完成就先执行回调`。

我们将上面的方法延迟分别改成234秒

```js
function promiseClick1(){
  let p = new Promise(function(resolve, reject){
    setTimeout(function(){
      var num = Math.ceil(Math.random()*20); //生成1-10的随机数
      console.log('2s随机数生成的值：',num)
      if(num<=10){
        resolve(num);
      }
      else{
        reject('2s数字太于10了即将执行失败回调');
      }
    }, 2000);
      })
      return p
    }
    function promiseClick2(){
  let p = new Promise(function(resolve, reject){
    setTimeout(function(){
      var num = Math.ceil(Math.random()*20); //生成1-10的随机数
      console.log('3s随机数生成的值：',num)
      if(num<=10){
        resolve(num);
      }
      else{
        reject('3s数字太于10了即将执行失败回调');
      }
    }, 3000);
      })
      return p
    }
    function promiseClick3(){
  let p = new Promise(function(resolve, reject){
    setTimeout(function(){
      var num = Math.ceil(Math.random()*20); //生成1-10的随机数
      console.log('4s随机数生成的值：',num)
      if(num<=10){
        resolve(num);
      }
      else{
        reject('4s数字太于10了即将执行失败回调');
      }
    }, 4000);
      })
      return p
    }

Promise
  .race([promiseClick3(), promiseClick2(), promiseClick1()])
  .then(function(results){
    console.log(results);
  },function(reason){
    console.log(reason);
  });
```

当 2s 后 promiseClick1 执行完成后就已经进入到了 then 里面回调，在 then 里面的回调开始执行时， promiseClick2() 和  promiseClick3() 并没有停止，仍旧再执行。于是再过3秒后，输出了他们各自的回调值

![](https://tva1.sinaimg.cn/large/006y8mN6gy1g8hdng7040j306s06ndgd.jpg)

race 的使用比如可以使用在一个请求在10s内请求成功的话就走then方法，如果10s内没有请求成功的话进入 reject 回调执行另一个操作。
