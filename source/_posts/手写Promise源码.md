---
title: 手写Promise源码
date: 2021-12-16 20:05:44
category:
    - 前端
tags:
    - javascript
    - Promise
    - 源码
---

从简单到复杂一步步实现自己的`Promise`

> 因为不好拆解，如果想直接看最终版，请跳转到文章末尾最后一个代码块


## 在手写源码的之前，首先需要知道`Promise`是什么，都做了哪些操作

1. `Promise`是一个类,在执行这个类的时候，需要传递一个执行器进去，执行器会立即执行

2. `Promise`中有三种状态，分别为 成功(`fulfilled`) 、失败(`rejected`)、等待(`pending`)。只能从等待到成功或者失败，且一旦状态确定就不可更改

3. `resolve`和`reject`函数是用来更改状态的

4. `then`方法做的事情就是判断状态，根据不同的状态调用不同的回调函数，`then`方法是被定义在原型对象上的

5. `then`成功和失败时都会有一个参数，来表示成功或失败后的值或原因

6. 同一个`promise`对象下面的`then`方法是可以被调用多次的

7. `then`方法是可以被链式调用的, 后面`then`方法的回调函数拿到值的是上一个`then`方法的回调函数的返回值

## Promise类核心逻辑实现

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor (executor) {
        executor(this.resolve, this.reject)
    }
    // promise 状态
    status = PENDING
    // 成功之后的值
    value = undefined
    // 失败之后的原因
    reason = undefined
    resolve = value => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为成功
        this.status = FULFILLED
        // 保存成功之后的值
        this.value = value
    }
    reject = reason => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为失败
        this.status = REJECTED
        // 保存失败后的原因
        this.reason = reason
    }
    then (successCallback, failCallback) {
        if (this.status === FULFILLED) {
            successCallback(this.value)
        } else if (this.status === REJECTED) {
            failCallback(this.reason)
        }
    }
}
module.exports = MyPromise
```
> 至此，我们实现了一个最简单版本的 Promise

```js
// 实现功能
const MyPromise = require('./MyPromise')
let promise = new MyPromise(function(resolve, reject){
    console.log('promise')
    resolve('success')
    reject('error')
})
promise.then(value => console.log(value), reason => console.log(reason))
// promise
// success
```

## 加入异步逻辑

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor (executor) {
        executor(this.resolve, this.reject)
    }
    // promise 状态
    status = PENDING
    // 成功之后的值
    value = undefined
    // 失败之后的原因
    reason = undefined
    // 成功之后的回调
    successCallback = undefined
    // 失败之后的回调
    failCallback = undefined
    resolve = value => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为成功
        this.status = FULFILLED
        // 保存成功之后的值
        this.value = value
        // 判断成功回调是否存在，存在则调用
        this.successCallback && this.successCallback(this.value)
    }
    reject = reason => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为失败
        this.status = REJECTED
        // 保存失败后的原因
        this.reason = reason
        // 判断失败回调是否存在，存在则调用
        this.failCallback && this.failCallback(this.reason)
    }
    then (successCallback, failCallback) {
        if (this.status === FULFILLED) {
            successCallback(this.value)
        } else if (this.status === REJECTED) {
            failCallback(this.reason)
        } else {
            // 状态是等待
            // 此时需要将callback存储起来
            this.successCallback = successCallback
            this.failCallback = failCallback

        }
    }
}
module.exports = MyPromise
```
```js
// 实现功能
const MyPromise = require('./MyPromise')
let promise = new MyPromise(function(resolve, reject){
    setTimeout(() => {
        console.log('promise')
        resolve('success')
    }, 2000)
    // reject('error')
})
promise.then(value => console.log(value), reason => console.log(reason))
// 2秒后输出 promise    success
```

## 实现 then 方法多次调用添加多个处理函数

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor (executor) {
        executor(this.resolve, this.reject)
    }
    // promise 状态
    status = PENDING
    // 成功之后的值
    value = undefined
    // 失败之后的原因
    reason = undefined
    // 成功之后的回调
    successCallback = []
    // 失败之后的回调
    failCallback = []
    resolve = value => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为成功
        this.status = FULFILLED
        // 保存成功之后的值
        this.value = value
        // 判断成功回调是否存在，存在则调用
        // this.successCallback && this.successCallback(this.value)
        while (this.successCallback.length) this.successCallback.shift()(this.value)
    }
    reject = reason => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为失败
        this.status = REJECTED
        // 保存失败后的原因
        this.reason = reason
        // 判断失败回调是否存在，存在则调用
        // this.failCallback && this.failCallback(this.reason)
        while (this.failCallback.length) this.failCallback.shift()(this.reason)
    }
    then (successCallback, failCallback) {
        if (this.status === FULFILLED) {
            successCallback(this.value)
        } else if (this.status === REJECTED) {
            failCallback(this.reason)
        } else {
            // 状态是等待
            // 此时需要将callback存储起来
            this.successCallback.push(successCallback)
            this.failCallback.push(failCallback)
        }
    }
}
module.exports = MyPromise
```
```js
// 实现功能
const MyPromise = require('./MyPromise')
let promise = new MyPromise(function(resolve, reject){
    setTimeout(() => {
        console.log('promise')
        resolve('success')
    }, 2000)
    // reject('error')
})
promise.then(value => console.log(value), reason => console.log(reason))
promise.then(value => console.log(value), reason => console.log(reason))
promise.then(value => console.log(value), reason => console.log(reason))
// 2秒后输出 promise success success success
```

## 实现then方法的链式调用

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor (executor) {
        executor(this.resolve, this.reject)
    }
    // promise 状态
    status = PENDING
    // 成功之后的值
    value = undefined
    // 失败之后的原因
    reason = undefined
    // 成功之后的回调
    successCallback = []
    // 失败之后的回调
    failCallback = []
    resolve = value => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为成功
        this.status = FULFILLED
        // 保存成功之后的值
        this.value = value
        // 判断成功回调是否存在，存在则调用
        // this.successCallback && this.successCallback(this.value)
        while (this.successCallback.length) this.successCallback.shift()(this.value)
    }
    reject = reason => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为失败
        this.status = REJECTED
        // 保存失败后的原因
        this.reason = reason
        // 判断失败回调是否存在，存在则调用
        // this.failCallback && this.failCallback(this.reason)
        while (this.failCallback.length) this.failCallback.shift()(this.reason)
    }
    then (successCallback, failCallback) {
        // then 方法返回的是一个 promise
        return new MyPromise((resolve, reject) => {
            // 判断状态
            if (this.status === FULFILLED) {
                let v = successCallback(this.value)
                // 判断 v 的值是普通值还是promise对象
                // 如果是普通值，直接调用 resolve
                // 如果是 promise 对象，查看promise对象返回的结果
                // 再根据promise对象返回的结果，决定调用resolve还是reject
                resolvePromise(v, resolve, reject)
            } else if (this.status === REJECTED) {
                failCallback(this.reason)
            } else {
                // 状态是等待
                // 此时需要将callback存储起来
                this.successCallback.push(successCallback)
                this.failCallback.push(failCallback)
            }
        })

    }
}

function resolvePromise (v, resolve, reject) {
    if (v instanceof MyPromise) {
        // v.then(value => resolve(value), reason => reject(reason))
        v.then(resolve, reject)
    } else {
        resolve(v)
    }
}

module.exports = MyPromise
```

```js
// 实现功能
const MyPromise = require('./MyPromise')
let promise = new MyPromise(function(resolve, reject){
    resolve('success')
})
function other() {
    return new MyPromise(resolve => {
        resolve('other')
    })
}
promise.then(value => {
    console.log(value)
    return other()
}).then(value => {
    console.log(value)
})
// success
// other
```

## *终版 (针对一些异常情况并没有做处理)

```js
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
    constructor(executor) {
        // 如果在执行执行器的时候报错了，需要捕获错误情况
        try {
            // 传入立即执行的执行器
            executor(this.resolve, this.reject)
        } catch (e) {
            this.reject(e)
        }
    }
    // promise 状态
    status = PENDING
    // 成功之后的值
    value = undefined
    // 失败之后的原因
    reason = undefined
    // 成功之后的回调
    successCallback = []
    // 失败之后的回调
    failCallback = []
    resolve = value => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为成功
        this.status = FULFILLED
        // 保存成功之后的值
        this.value = value
        // 判断成功回调是否存在，存在则调用
        // this.successCallback && this.successCallback(this.value)
        // 当使用 then 方法多次调用时，添加多个处理函数
        while (this.successCallback.length) this.successCallback.shift()()
    }
    reject = reason => {
        // 因为状态一旦确定就不可以更改，所以需要有一层判断
        if (this.status !== PENDING) return
        // 状态更改为失败
        this.status = REJECTED
        // 保存失败后的原因
        this.reason = reason
        // 判断失败回调是否存在，存在则调用
        // this.failCallback && this.failCallback(this.reason)
        // 当使用 then 方法多次调用时，添加多个处理函数
        while (this.failCallback.length) this.failCallback.shift()()
    }
    then(successCallback, failCallback) {
        // 实现 .then() 参数可选
        successCallback = successCallback ? successCallback : value => value
        failCallback = failCallback ? failCallback : reason => { throw reason }
        // then 方法返回的是一个 promise，实现链式调用
        let promise2 = new MyPromise((resolve, reject) => {
            // 判断状态
            // 判断当状态为成功时，调用成功回调
            if (this.status === FULFILLED) {
                setTimeout(() => {
                    // 如果 then 方法在执行过程中报错，需要捕获到这个错误，并提示出来
                    try {
                        let v = successCallback(this.value)
                        // 判断 v 的值是普通值还是promise对象
                        // 如果是普通值，直接调用 resolve
                        // 如果是 promise 对象，查看promise对象返回的结果
                        // 再根据promise对象返回的结果，决定调用resolve还是reject
                        resolvePromise(promise2, v, resolve, reject) // 此时primise2还获取不到，所以需要将此编程异步代码
                    } catch (e) {
                        reject(e)
                    }
                }, 0)
            } else if (this.status === REJECTED) {
                setTimeout(() => {
                    // 如果 then 方法在执行过程中报错，需要捕获到这个错误，并提示出来
                    try {
                        // 当状态为失败时，调用失败回调
                        let v = failCallback(this.reason)
                        // 判断 v 的值是普通值还是promise对象
                        // 如果是普通值，直接调用 resolve
                        // 如果是 promise 对象，查看promise对象返回的结果
                        // 再根据promise对象返回的结果，决定调用resolve还是reject
                        resolvePromise(promise2, v, resolve, reject) // 此时primise2还获取不到，所以需要将此编程异步代码
                    } catch (e) {
                        reject(e)
                    }
                }, 0)
            } else {
                // 状态是等待，意味着是异步操作的时候
                // 此时需要将callback存储起来
                this.successCallback.push(() => {
                    setTimeout(() => {
                        // 如果 then 方法在执行过程中报错，需要捕获到这个错误，并提示出来
                        try {
                            let v = successCallback(this.value)
                            // 判断 v 的值是普通值还是promise对象
                            // 如果是普通值，直接调用 resolve
                            // 如果是 promise 对象，查看promise对象返回的结果
                            // 再根据promise对象返回的结果，决定调用resolve还是reject
                            resolvePromise(promise2, v, resolve, reject) // 此时primise2还获取不到，所以需要将此编程异步代码
                        } catch (e) {
                            reject(e)
                        }
                    }, 0)
                })
                this.failCallback.push(() => {
                    setTimeout(() => {
                        // 如果 then 方法在执行过程中报错，需要捕获到这个错误，并提示出来
                        try {
                            // 当状态为失败时，调用失败回调
                            let v = failCallback(this.reason)
                            // 判断 v 的值是普通值还是promise对象
                            // 如果是普通值，直接调用 resolve
                            // 如果是 promise 对象，查看promise对象返回的结果
                            // 再根据promise对象返回的结果，决定调用resolve还是reject
                            resolvePromise(promise2, v, resolve, reject) // 此时primise2还获取不到，所以需要将此编程异步代码
                        } catch (e) {
                            reject(e)
                        }
                    }, 0)
                })
            }
        })
        return promise2
    }
    // 实现 catch 方法
    catch (failCallback) {
        // catch 方式其实就是 then 方法，只是第一个参数传入 undefined 就ok了
        return this.then(undefined, failCallback)
    }
    // 谁先 finally 方法
    finally(callback) {
        // 如果成功失败都会执行，所以不依赖状态
        return this.then(value => {
            return MyPromise.resolve(callback()).then(() => value)
            // callback()
            // return value
        }, reason => {
            return MyPromise.resolve(callback()).then(() => { throw reason })
            // callback()
            // throw reason
        })
    }
    // 实现 all 方法，接收一个数组参数
    static all(array) { // 基础版，未考虑异常情况，没有对传入数据做校验，默认传值是正确的
        let result = [] // 返回值
        let index = 0 // 索引
        return new MyPromise((resolve, reject) => {
            // 添加数据h桉树，key 即索引
            function addData(key, value) {
                result[key] = value // 将结果传入返回值中
                index++ // 每次添加都自增 1
                // 因为可能内部存在异步代码，for循环是立即执行完成，所以，需要在判断全部相等之后，再去返回 resolve
                if (index === array.length) { // 判断，只有当所有的全部添加完成之后，返回 resolve
                    resolve(result)
                }
            }
            // 循环遍历传入的数组参数
            for (let i = 0; i < array.length; i++) {
                let current = array[i] // 拿到当前的值
                if (current instanceof MyPromise) {
                    // promise 对象
                    current.then(value => { addData(i, value) }, error => reject(error))
                } else {
                    // 普通值
                    addData(i, array[i])
                }
            }
        })
    }
    // 实现静态 resolve 方法， 传入参数
    static resolve(value) {
        // 判断参数是否为 promise，如果是，直接返回
        if (value instanceof MyPromise) return value
        // 如果是一个普通值，则返回 promise，在 resolve中传入 value
        return new MyPromise(resolve => resolve(value))
    }
}

function resolvePromise(promise2, v, resolve, reject) {
    // 如果返回的primise同当前的promise是同一个promise，则报错并return 阻止程序继续向下执行
    if (promise2 === v) {
        return reject(new TypeError('Chaining cycle detected for promise #<Promise>'))
    }
    if (v instanceof MyPromise) {
        // v.then(value => resolve(value), reason => reject(reason))
        v.then(resolve, reject)
    } else {
        resolve(v)
    }
}

module.exports = MyPromise
```
