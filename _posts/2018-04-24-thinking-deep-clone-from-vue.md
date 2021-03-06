---
layout: post
title: 从Vuex中思考深拷贝
subtitle: "\"其实就是抄源码\""
date: 2018-04-24 21:12:00
tags: 
    - 源码
    - Vuex
    - javascript
---

# Vuex 当中的深拷贝

深拷贝话题在很多场景都能看到，这里只讨论关于plain object（一般的对象或数组）的深拷贝

## 难点

对象深拷贝的一个难点在于对象的循环结构（circular structure）如何处理。
类似：
```js
const obj = {}
obj.o = obj
```
如果一个对象不是一个循环结构, 我们可以通过最简单的序列与反序列化来切断对象的引用关系：
```js
const obj = {
  name: 'a simple object',
  others: '...'
}

const dCloneObj = JSON.parse(JSON.stringify(obj))

dCloneObj.name = 'a new object'
obj.name //a simple object
```
但是， 当对象当中包含某个父级的引用，问题就变得没那么简单了

```js
const obj = {}
obj.o = obj
const dCloneObj = JSON.parse(JSON.stringify(obj)) //Uncaught TypeError: Converting circular structure to JSON
```
如果我们用递归的方式，通过新建一个对象按照key名以按值传递方式进行处理（这也是浅拷贝的一种常见实现方式），则会陷入无限循环而导致堆栈溢出。

由此可见， 我们需要一种方式去判断当前对象是否是一个循环引用的结构，当我们递归时发现当前需要的拷贝的对象在之前的某次循环时已经存在了，这时我们就需要将其的克隆值赋给对应的value，以避免无限循环。

## Vuex中的实现
在vuex中，作者自己实现了一套对象的深拷贝，直接看代码：
```js
/**
 * 获得能通过第二个函数参数的值
 *
 * @param {Array} list
 * @param {Function} f
 * @return {*}
 */
export function find (list, f) {
  const { length } = list
  let index = 0
  let value
  while (++index < length) {
    value = list[index]
    if (f(value, index, list)) {
      return value
    }
  }
}

/**
 * 这个方法可以将所有的嵌套对象和拷贝值缓存起来。
 * 如果检测到循环结构就将其拷贝值赋给相应的key以避免无限循环
 *
 * @param {*} obj
 * @param {Array<Object>} cache
 * @return {*}
 */
export function deepCopy (obj, cache = []) {
  // 首先判断对象是不是一个object类型，包括对象和数组，需要排除null。如果都不符合说明是一个基本类型值， 可以直接赋值
  if (obj === null || typeof obj !== 'object') {
    return obj
  }

  // 这里我们检查缓存中保存的所有对象有没有就是当前我们需要进行深拷贝的对象，如果有说明是一个循环结构，直接将它的拷贝值返回即可
  const hit = find(cache, c => c.original === obj)
  if (hit) {
    return hit.copy
  }

  // 使用一个全新的对象进行接收拷贝
  const copy = Array.isArray(obj) ? [] : {}
  // 把当前值作为原值和拷贝后的值塞入缓存队列，在后续的递归中，cache保存着整个过程的状态
  cache.push({
    original: obj,
    copy
  })

  // 按key遍历，并递归这个过程
  Object.keys(obj).forEach(key => {
    copy[key] = deepCopy(obj[key], cache)
  })

  return copy
}
```

Vuex当中的深拷贝整个方法核心就是通过一个cache数组用来记录历次遍历的值， 每次递归，cache当中都存一次原对象和原对象的浅拷贝层（之所以是层是因为要对其深层次进行递归）。

像Vue、React这类当红框架中，我们可以看到许多类似的工具方法，多思考这类代码有助于代码质量的提升，平时也可以自己实现一个。很多时候代码看起来不难，但真到了动手做的时候才发现有许多需要注意的地方。还是需要勤加练习！
