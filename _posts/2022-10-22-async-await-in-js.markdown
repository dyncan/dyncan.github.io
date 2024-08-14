---
layout: post
title: "理解 JavaScript 中的 Async/Await "
subtitle: ""
date: 2022-10-22 12:00:00
author: "Peter Dong"
header-img: "img/post-bg-javascript-async-await.jpeg"
catalog: true
tags:
  - async
  - await
  - JavaScript
---

> async/await 是以更舒适的方式使用 promise 的一种特殊语法。

在 Salesforce 中，如果你有开发过 lwc 相关的组件，想必应该使用过 `Async/Await` 语法。其实在 JavaScript 中做异步开发时，我们通常会毫不犹豫的使用 `Async/Await`. 不管是并发还是串行，`Async/Await` 都能处理的很好，而且还保证了代码的可读性。本篇内容主要是我根据一些资料和官方文档来阐述对 `Async/Await` 语法的一些理解，如有叙述不对的地方，欢迎指正。

## 什么是 async/await?

JavaScript 中的 `async/await` 是 [AsyncFunction](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction) 特性中的关键字。目前为止，除了 IE 之外，常用浏览器和 Node.js (v7.6+) 都已经支持该特性。具体支持情况可以在[参考](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/AsyncFunction#browser_compatibility) 这里。

`async/await` 是一种建立在 Promise 之上的编写异步或非阻塞代码的新方法 (Generator 的语法糖), 普遍认为是 JS 异步操作的最优雅的解决方案。相对于 `Promise` 和回调，它的可读性和简洁度都更高。只要 function 标记为 `async`, 就表示函数里面可以写 await 的同步语法，而 await 顾名思义就是「等待」,它会确保一个 `promise` 都解決 ( `resolve` ) 或出错 ( `reject` ) 后才会进行下一步，当 async function 的内容全部执行结束，会返回一个 `promise`, 表示后续代码可以使用 `.then` 语法來连接，基本的代码就像下面这样：

```javascript
async function a(){
  await b();
  .....       // 等 b() 完成后才会执行
  await c();
  .....       // 等 c() 完成后才会执行
  await new Promise(resolve=>{
    .....
  });
  .....       // 上方的 promise 完成后才会执行
}

a();
a().then(()=>{
  .....       // 等 a() 完成后接著执行
});
```

### async 的作用？

我们先看如下代码的输出结果是什么：

```javascript
async function testAsync() {
    return "hello async";
}

const result = testAsync();
console.log(result); //Promise {<fulfilled>: 'hello async'}
```

所以，async 函数返回的是其实一个 Promise 对象。从[文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function#%E8%BF%94%E5%9B%9E%E5%80%BC)中也可以得到这个信息。async 函数会返回一个 Promise 对象。如果在函数中 return 一个直接量，async 会把这个直接量通过 Promise.resolve() 封装成 Promise 对象。

async 函数返回的是一个 Promise 对象，所以在最外层不用 await 获取其返回值的情况下，我们可以用原来的方式:then() 链来处理这个 Promise 对象，就像这样：

```javascript
testAsync().then(v => {
    console.log(v);    // 输出 hello async
});
```

### await 在等待什么呢？

根据 [MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/await) 上写的：

```javascript
[return_value] = await expression;
```

await 等待的是一个表达式，那么表达式，可以是一个常量 ,变量，promise, 函数等，换句话说，就是没有特殊限定。

```javascript
function getSomething() {
    return "something";
}

async function testAsync() {
    return Promise.resolve("hello async");
}

async function test() {
    const v1 = await getSomething();
    const v2 = await testAsync();
    console.log(v1, v2);
}

test(); // something hello async
```

### 为什么 await 关键词只能在 async 函数中用？ 

await 操作符等的是一个返回的结果，如果它等到的是一个 Promise 对象，它会阻塞后面的代码，等着 Promise 对象 resolve，然后得到 resolve 的值，作为 await 表达式的运算结果。这就是 await 必须用在 async 函数中的原因。async 函数调用不会造成阻塞，它内部所有的阻塞都被封装在一个 Promise 对象中异步执行。

### async/await 的优势

假设一个业务，分多个步骤完成，每个步骤都是异步的，而且依赖于上一个步骤的结果。我们先用 setTimeout 来模拟异步操作：

```javascript
function takeLongTime(n) {
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}
```

现在用 Promise 方式来实现这三个步骤的处理：

```javascript

function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => step2(time2))
        .then(time3 => step3(time3))
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

doIt();

//step1 with 300
// step2 with 500
// step3 with 700
// result is 900
// doIt: 1512.908935546875 ms
```

如果用 async/await 来实现呢，会是这样：

```javascript
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time2);
    const result = await step3(time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}

doIt();

// step1 with 300
// step2 with 500
// step3 with 700
// result is 900
// doIt: 1512.2080078125 ms
```

结果和之前的 Promise 实现是一样的，但是这个代码看起来清晰得多，几乎跟同步代码一样。

### 思考：为什么需要异步编程？

JavaScript 是单线程的，就是必须等待上一个任务执行完才能执行下一个任务，这种执行模式叫：`同步`.

但是随着业务复杂度的提升，很多情况下有处理高并发 (单位时间内极大的访问量) 和 I/O 密集场景 (ps: I/O 操作往往非常耗时，所以异步的关键在于解决 I/O 耗时问题), 如果采用同步编程，问题就来了，服务器处理一个 I/O 请求需要大量的时间，后面的请求都将排队，造成浏览器端的卡顿。`异步编程`能解决这个问题。




