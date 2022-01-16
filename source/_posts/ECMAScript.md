---
title: ECMAScript
date: 2021-12-27 14:00:53
category:
    - 前端
tags:
    - javascript
    - ECMAScript
---

javascript 其实是 ECMAScript 的拓展语言

ECMAScript 只提供了最基本的语法

在浏览器中
js = ECMAScript + 浏览器提供的API，如：BOM、DOM
在 node 中
js = ECMAScript + node中提供的API，如：fs、net、etc.

人们平常说的 ES6 其实是泛指 包含ES2015及后续的所有新特性

1. var let const
   let 块级作用域，没有变量提升阶段，只在当前代码块生效
   const 在声明的时候就需要定义，一旦定义就不可更改（只适用普通数据类型，对于引用类型只要不改变指针，就可以对内部的数据进行更改）
2. 数组的解构
```js
const arr = [1, 2, 3, 4, 5]
const [first, two, ...args] = arr
console.log(first) // 1
console.log(two) // 2
console.log(args) // [3, 4, 5]
```
3. 对象的解构
```js
const obj = { a: 1, b: 2, c: 3 }
const { a, c } = obj
console.log(a, c) // 1 3
```
4. 模板字符串
```js
const s1 = '❤️'
const s2 = `I ${ s1 } U`
console.log(s2) // I ❤️ U
```
5. 带标签的模板字符串
```js
const name = 'zhangshan'
const gender = true
function myTagFunc (strings) {
    const sex = gender ? 'man' : 'woman'
    return strings[0] + name + strings[1] + sex + strings[2]
}
const result = myTagFunc`hey, ${name} is a ${gender}`
console.log(result)
```
6. 字符串的拓展方法
```js
const str = 'Good Good Study, Day Day Up.'
console.log(str.startsWith('Good')) // true
console.log(str.endsWith('.')) // true
console.log(str.includes('Study')) // true
```
7. 参数默认值
```js
function foo (enable = true) {
    console.log(enable)
}
foo() // true
foo(false) // false
```
8. 剩余参数
```js
function foo (a, b, ...args) {
    console.log(args)
}
foo() // []
foo(1, 2, 3, 4, 5, 6, 7) // [3, 4, 5, 6, 7]
```
9. 展开数组
```js
const arr = [1, 2, 3]
console.log(...arr) // 1 2 3
```
10. 箭头函数
```js
function foo (a, b) {
    return a + b
}
// =======箭头函数改造=======>
const foo = (a, b) => a + b
```
11. 箭头函数与this
箭头函数不会改变this的指向
```js
function foo () {
    const person = {
        name: 'TOM',
        sayHi: function () {
            console.log(`hi, my name is ${this.name}`)
            // hi, my name is TOM
        }
        sayHi: () => {
            console.log(`hi, my name is ${this.name}`)
            // hi, my name is undefined
        }
        sayHiAsync: function () {
            setTimeout(function() {
                console.log(this.name) // undefined
            }, 1000)
            setTimeout(() => {
                console.log(this.name) // TOM
            }, 1000)
        }
    }
}
```
12. 对象字面量的增强
```js
const bar = '123'
const obj = {
    foo: 345,
    // bar: bar,
    bar,
    // method: function () {
    //     console.log('xxx')
    // }
    method() {
        console.log('xxx')
    }
    [Math.random()]: 12345 // 计算属性名
}
```
13. Object.assign
```js
const obj1 = { a: 1, b: 2, c: 3 }
const obj2 = { a: 2, d: 4 }
const obj3 = { c: 4, e: 5 }
console.log(Object.assign(obj1, obj2, obj3)) // {a: 2, b: 2, c: 4, d: 4, e: 5}

function fn (obj) {
    const funcObj = Object.assign({}, obj)
    funcObj.name = 'func obj'
    console.log(funcObj)
}
const obj = { name: 'global obj' }
fn(obj) // { name: func obj }
console.log(obj) // { name: global obj }
```
14. Object.is
```js
Object.is(NaN, NaN) // true
NaN === NaN // false
```
15. Proxy 对比 defineProperty
```js
const person = {
    name: 'zhangshan',
    age: 18
}
const personProxy = new Proxy(person, {
    get (target, property) {
        return property in target ? target[property] : 'default'
    },
    set (target, property, value) {
        if (property === 'age') {
            if (!Number.isInteger(value)) {
                throw new TypeError(`${value} is not an int`)
            }
        }
        target[property] = value
    }
})
personProxy.age = '100' // TypeError: 100 is not an int
personProxy.age = 100
personProxy.gender = true

console.log(personProxy.name) // zhangshan
console.log(personProxy.xxx) // default
```
- Object.defineProperty 只能监听属性的读写
- Proxy 能够监视到更多的对象操作
```js
new Proxy(person, {
    // 方法     触发方式
    get() {}, // 读取某个属性
    set() {}, // 写入某个属性
    has // in 操作符
    deleteProperty // delete 操作符
    getPrototypeOf // Object.getPrototypeOf()
    setPrototypeOf // Object.setPrototypeOf()
    isExtensible // Object.isExtensible()
    preventExtensions // Object.preventExtensions()
    getOwnPropertyDescriptor // Object.getOwnPropertyDescriptor()
    defineProperty // Object.defineProperty()
    ownKeys // Object.getOwnPropertyNames()、Object.getOwnPropertySymbols()
    apply // 调用一个函数
    construct // 用 new 调用一个函数
})
```
- 可以监视数组
- Proxy 是以非侵入的方式监管了对象的读写

16. Reflect
属于一个静态类 不可以使用 new 关键字
统一提供了一套用于操作对象的 API
```js
// Reflect.get
const obj = {
    foo: '123',
    bar: '456'
}
const proxy = new Proxy(obj, {
    get(target, property) {
        return Reflect.get(target, property)
    }
})
```
17. Promise
一种更优的异步编程解决方案，解决了传统异步编程中回调函数嵌套过深的问题

18. class 类
```js
class Person {
    constructor (name) {
        this.name = name
    }
    say () {
        console.log(`hi, my name is ${this.name}`)
    }
}
const p = new Person('tom')
p.say()
```
19. 静态方法 static
```js
class Person {
    constructor (name) {
        this.name = name
    }
    say () {
        console.log(`hi, my name is ${this.name}`)
    }
    static create (name) {
        return new Person(name)
    }
}
const tom = Person.create('tom')
tom.say()
```
20. 类的继承
```js
class Person {
    constructor (name) {
        this.name = name
    }
    say () {
        console.log(`hi, my name is ${this.name}`)
    }
}
class Student extends Person {
    constructor (name, number) {
        super(name)
        this.number = number
    }
    hello () {
        super.say()
        console.log(`my school number is ${this.number}`)
    }
}
const s = new Student('jack', '100')
s.hello()
```

21. Set
Set内部成员不存在重复
```js
const s = new Set()
s.add(1).add(2).add(3).add(1)
console.log(s) // Set { 1 2 3 }
console.log(s.size) // 3
s.has(3) // true
s.has(5) // false
s.delete(1) // true
s.clear() // 清空
const arr = [1, 2, 3, 4, 1, 2, 3]
// const result = Array.from(new Set(arr))
const result = [ ...new Set(arr) ]
console.log(result) // [1, 2, 3, 4]
```

22. Map
可以用任意类型作为键
```js
const m = new Map()
const tom = { name: 'tom' }
m.set(tom, 90)
console.log(m) // Map { { name: 'tom' } => 90 }
console.log(m.get(tom)) // 90

```

23. Symbol
新的数据类型
主要作用就是为对象添加独一无二的属性名
```js
Symbol() === Symbol() // false

const name = Symbol()
const person = {
    [name]: 'zs',
    say() {
        console.log(this[name])
    }
}

const s1 = Symbol.for('foo')
const s2 = Symbol.for('foo')
console.log(s1 === s2) // true
Symbol.for(true) === Symbol.for('true') // true 内部会自动转换成字符串
Symbol.iterator
Symbol.hasInstance
```
24. for...of 循环
原理是 Iterable

```js
const arr = [1, 2, 3, 4, 5]
for (const item of arr) {
    console.log(item)
    if (item > 100) {
        break // 可以终止循环
    }
}
```

25. 可迭代接口 Iterable
> 对象方法不可以用 for...of 循环，但因为 for...of 循环的原理是使用 iterable ，所以我们可以给对象内部手写一个 iterable 让对象也同样可以使用 for...of 去进行遍历

26. 实现可迭代接口
```js
const set = new Set(['foo', 'bar', 'abc'])
const iterator = set[Symbol.iterator]()

console.log(iterator.next()) // { value: 'foo', done: false }
console.log(iterator.next()) // { value: 'bar', done: false }
console.log(iterator.next()) // { value: 'abc', done: false }
console.log(iterator.next()) // { value: undefined, done: true }
console.log(iterator.next()) // { value: undefined, done: true }

const obj = {
    store: ['foo', 'bar', 'abc'],
    [Symbol.iterator]: function () {
        let index = 0
        const self = this
        return {
            next: function(){
                const result = {
                    value: self.store[index],
                    done: index >= self.store.length
                }
                index++
                return result
            }
        }
    }
}
for (const item of obj) {
    console.log(item)
}
// foo
// bar
// abc
```
27. 迭代器模式
```js
const todos = {
    life: ['吃饭', '睡觉', '打豆豆'],
    learn: ['js', 'node', 'html', 'css'],
    work: ['跳舞', '唱歌'],
    each: function (callback) {
        const all = [].concat(this.life, this.learn, this.work)
        for (const item of all) {
            callback(item)
        }
    },
    [Symbol.iterator]: function() {
        const all = [...this.life, ...this.learn, ...this.work]
        let index = 0
        return {
            next: function () {
                return {
                    value: all[index],
                    done: index ++ >= all.length
                }
            }
        }
    }
}
// for (const item of todos.life) {
//     console.log(item)
// }
// for (const item of todos.learn) {
//     console.log(item)
// }
// for (const item of todos.work) {
//     console.log(item)
// }
// todos.each(function (item) {
//     console.log(item)
// })
for (const item of todos) {
    console.log(item)
}
```

28. 生成器函数 Generator
```js
function * foo () {
    console.log(1)
    yield 100
    console.log(2)
    yield 200
    console.log(3)
    yield 300
}
const generator = foo()

console.log(generator.next()) // 1  { value: 100, done: false }
console.log(generator.next()) // 2  { value: 200, done: false }
console.log(generator.next()) // 3  { value: 300, done: false }
console.log(generator.next()) // { value: undefined, done: true }

// 模拟场景
function * createIdMaker () {
    let id = 1
    whild(true) {
        yield id ++
    }
}
const idMaker = createIdMaker()
console.log(idMaker.next().value) // 1
console.log(idMaker.next().value) // 2
console.log(idMaker.next().value) // 3
console.log(idMaker.next().value) // 4

// 使用 Generator 函数实现 iterator 方法
const todos = {
    life: ['吃饭', '睡觉', '打豆豆'],
    learn: ['js', 'node', 'html', 'css'],
    work: ['跳舞', '唱歌'],
    [Symbol.iterator]: function * () {
        const all = [...this.life, ...this.learn, ...this.work]
        for (const item of all) {
            yield item
        }
    }
}
for (const item of todos) {
    console.log(item)
}
```

29. ES Modules

30. ECMAScript2016
- includes方法
- 指数运算符
    ```js
    Math.pow(2, 10) // 1024
    2 ** 10 // 1024
    ```
31. ES2017
- Object.values
    ```js
    const obj = {
        name: 'zs',
        age: 13
    }
    console.log(Object.values(obj)) // ['zs', 13]
    ```
- Object.entries
    ```js
    const obj = {
        name: 'zs',
        age: 13
    }
    console.log(Object.entries(obj)) // [ [ name, 'zs' ], [ age, 13 ] ]
    for (const [key, value] of Object.entries(obj)) {
        console.log(key, value)
    }
    // name zs
    // age 13
    console.log(new Map(Object.entries(obj))) // Map { 'name' => 'zs', 'age' => 13 }
    ```
- Object.getOwnPropertyDescriptors
- String.protptype.padStart / String.protptype.padEnd
- 在函数参数中添加尾逗号
