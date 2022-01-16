---
title: JavaScript异步编程
date: 2021-12-02 20:12:38
category:
    - 前端
tags:
    - javascript
    - 异步编程
---

## JS异步编程的介绍
首先说说为什么会有`JS`异步编程
众所周知，`JS`是单线程模式工作的，因为`JS`是运行在浏览器端的脚本语言，目的是为了实现页面的交互，那么因为页面交互主要是DOM操作，从而决定了它必须使用单线程模式工作，否则就会出现特别复杂的线程同步问题。
这种模式优点是更安全、更简单，缺点就是如果在代码执行中遇到一个特别耗时的操作，那么后面的代码就必须等待这个操作结束以后才可以执行。
所以，为了解决像这样的情况，`JS`将任务的执行模式分成了两种，分别是**同步模式(Synchronous)**和**异步模式(Asynchronous)**

### 同步模式和异步模式
- 同步模式
至上而下依次执行，代码可读性高
```js
console.log('global begin')
function bar () {
    console.log('bar')
}
function foo() {
    console.log('foo')
    bar()
}
foo()
console.log('global end')

// global begin
// foo
// bar
// global end
```
- 异步模式
通过事件循环和消息队列实现的
```js
console.log('g1 begin')
setTimeout(function timer1() {
    console.log('timer1')
}, 1800) // 这里如果把时间改为 >= 2000 则结果 timer1 会在 timer3 之后打印

setTimeout(function timer2() {
    console.log('timer2')
    setTimeout(function timer3() {
        console.log('timer3')
    }, 1000)
}, 1000)
console.log('g1 end')

// g1 begin
// g1 end
// timer2
// timer1
// timer3
```

### Promise
状态只有三种 PENDING FULFILLED REJECTED
单向转换，且状态不可再更改
pending ---> fulfilled
pending ---> rejected
```js
new Promise((resolve, reject) => {
    resolve('success')
    // reject('error')
})
```
- 链式调用
```js
let promise = new Promise((resolve, reject) => {
    resolve('success')
})
promise
    .then(v => {
        console.log(v) // success
        return '2'
    })
    .then(v => console.log(v)) // 2
    .then()
    .then()
```
- 静态方法
```js
// 直接成功
Promise.resolve()
// 直接失败
Promise.reject()
```
- Promise 并行
```js
// all() 和 race() 的区别
Promise.all() // 是等待所有的异步任务都执行结束以后才会结束
Promise.race() // 当有一个任务结束的时候就结束
```

### 宏任务、微任务
目前大部分异步调用都是作为宏任务执行的, 如：setTimeout...

作为微任务处理的有：

Promise 和 MutationObserver 和 nodeJs中的 process.nextTick

### Generator 生成器函数
```js
// 带有 * 的函数为生成器函数
function * foo() {
    console.log('start')
    // yield 会暂停执行，但不会结束，当下次再调用 next() 的时候会继续往下执行
    try {
        let res = yield 'foo' 
        console.log(res)
    } catch (e) {
        console.log(e)
    }
}
// 生成器函数调用不会立即执行
const generator = foo()
// 当手动调用 next() 方法时才会执行
// 如果在next()中传入参数，则会作为 yield 语句的返回值
let result = generator.next('bar')
// result返回值 为一个对象，存在两个参数： value 和 done
// value 为 生成器函数 yield 的返回值
// done 为判断当前生成器是否全部执行完成 值为： true or false
// 当调用 throw 方法抛出异常时，可以在函数中通过 try catch 接收
generator.throw(new Error('Generator Error'))
```

### `Async` / `Await` 语法糖
语言层面的异步编程标准
同`Generator`功能一样，只是把 `*` 换成 `async`, `yield`换成`await`
```js
async function () {
    let res = await xxx
    console.log(res)
}
```

#### 全局捕获异常*
- unhandledrejection
```js
// 在浏览器上
window.addEventListener('unhandledrejection', event => {
    const { reason, promise } = event
    console.log(reason, promise)
    // reason => Promise 失败原因，一般是一个错误对象
    // promise => 出现异常的 Promise 对象
    event.preventDefault()
}, false)
// 在 node 上
process.on('unhandledRejection', (reason, promise) => {
    console.log(reason, promise)
    // reason => Promise 失败原因，一般是一个错误对象
    // promise => 出现异常的 Promise 对象
})
```
