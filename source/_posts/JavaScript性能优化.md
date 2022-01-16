---
title: JavaScript性能优化
date: 2021-12-31 16:09:50
category:
    - 前端
tags:
    - javascript
    - 性能优化
---

## 内存管理

内存为什么需要管理
```js
function fn () {
    list = []
    list[10000] = 'lg is a coder'
}
fn()
```

内存管理介绍
- 内存：由可读写单元组成，表示一片可操作空间
- 管理：人为的去操作一片空间的申请、使用和释放
- 内存管理：开发者主动申请空间、使用空间、释放空间
- 管理流程：申请 -- 使用 -- 释放

js中的内存管理
- 申请内存空间
```js
let obj = {}
```
- 使用内存空间
```js
obj.name = 'xxx'
```
- 释放内存空间
```js
obj = null
```

垃圾回收机制
```js
function objGroup(obj1, obj2) {
    obj1.next = obj2
    obj2.prev = obj1
    return {
        o1: obj1,
        o2: obj2
    }
}
let obj = objGroup({ name: 'obj1' }, { name: 'obj2' })
console.log(obj1)
// {
//     o1: { name: 'obj1', next: { name: 'obj2', prev: [Circular] } },
//     o2: { name: 'obj2', prev: { name: 'obj1', next: [Circular] } },
// }
```