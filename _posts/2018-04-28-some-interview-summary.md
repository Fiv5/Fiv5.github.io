---
layout: post
title: 一些面试题总结(编程题)
date: 2018-04-28 15:06:00
header-mask: 0.3
tags: 
    - 面试
    - javascript
---

从四月中旬决定离职, 到现在过去差不多半个月, 这段时间面试了几家公司, 记录一下遇到的一些印象比较深刻的编程题.

## 给定两个数组, 这两个数组已排好序, 要求编写一个函数, 返回新数组, 且已经排好序

这道题目乍一看, 心想: 不会这么简单吧? 直接一个 sort 不就完事了吗. 然后面试官说了下他的要求: 要利用两个数组已经是有序的特点, 不能直接暴力拼接然后重新排序. 好吧, 原来是考察数组的优化问题. 这道题当时我没想到思路, 然后面试官跟我概述了一下他的思路, 我便顺着他思路写了出来(算法还是要加强..)

大致思路是: 对两个数组分别建立一个索引, 并使用第三个数组进行对结果集的组装, 每次查找时找到当前索引下的两个数组下的值, 比较大小并 push 入结果数组, 重复这一过程. 有可能存在这么一种情况: 当数组一最大值都比数组二最小值小, 则只需要 arr1.concat(arr2)即可得到排好序后的结果数组. 当然, 为了一般的情况, 我就不考虑这种特殊情况了.

> Talk is cheap, show me the code.

```js
// 实验数组, 已排好序
const testArr1 = [1, 5, 6, 6, 9, 10, 13, 14, 19, 22, 22, 22, 31]
const testArr2 = [0, 0, 1, 2, 3, 5, 6, 7, 8, 9, 11, 23, 29]

// code ..
function mySort(arr1, arr2) {
  let idx1 = 0
  let idx2 = 0
  let rs = []

  while (true) {
    //   每次循环先判断有没有一个数组已经走完, 如果有则直接把另一个数组拼接到结果数组, void 0 === undefined, 这里不考虑数组中有undefined的情况, 只考虑数组已排序好
    if (arr1[idx1] === void 0) {
      rs = rs.concat(arr2.slice(idx2))
      break
    } else if (arr2[idx2] === void 0) {
      rs = rs.concat(arr1.slice(idx1))
      break
    }

    arr1[idx1] >= arr2[idx2] ? rs.push(arr2[idx2++]) : rs.push(arr1[idx1++])
  }

  return rs
}
```

## 给定两个 div, 比如 box 和 content, 实现将 box 能够拖拽到 content 区域, 当放到 content 区域内时, 改变 content 背景色. 要求不能使用 html5 拖拽 API

需求很简单, 就是实现拖拽并改变颜色.

下面是我的解法：

```html
  <style>
    #box {
      width: 200px;
      height: 200px;
      background: green;
      position: absolute;
      left: 50px;
      z-index: 10; // 这里的设置一个z-index防止box拖到content时捕捉不到鼠标流，也可以在js里控制。
    }

    #content {
      position: absolute;
      top: 20px;
      right: 0;
      width: 800px;
      height: 800px;
      border: 1px solid red;
    }
  </style>
  <div id="box"></div>
  <div id="content"></div>
```

```js
class DragBox {
  constructor(el, target) {
    this.el = typeof el === 'string' ? document.querySelector(el) : el
    this.target =
      typeof target === 'string' ? document.querySelector(target) : target
    this.diffX = null
    this.diffY = null
    this.drag = this.drag.bind(this)
    this.handler = this.handler.bind(this)

    this.el.addEventListener('mousedown', e => {
      this.diffX = e.clientX - this.el.offsetLeft
      this.diffY = e.clientY - this.el.offsetTop
      document.body.addEventListener('mousemove', this.drag)
    })
    this.el.addEventListener('mouseup', e => {
      document.body.removeEventListener('mousemove', this.drag)
      this.handler()
    })
  }

  drag(e) {
    this.el.style.top = e.clientY - this.diffY + 'px'
    this.el.style.left = e.clientX - this.diffX + 'px'
  }

  handler() {
    if (
      this.el.offsetLeft >= this.target.offsetLeft &&
      this.el.offsetTop >= this.target.offsetTop
    ) {
      this.target.style.background = 'pink'
    }
  }
}

const box = new DragBox(
  document.getElementById('box'),
  document.getElementById('content')
)
```

没什么技巧，按部就班地做就行了。需要注意的点就是箱子拖拽时应该是鼠标按下的状态，当鼠标的松开按键到时候应取消箱子的移动监听事件。

## 请实现一个函数 uniqueInOrder，接收一个由字母或数字组成的字符串，返回一个数组，要求将原字符串中相邻的相同字符缩减为 1 个字符，字母需要区分大小写。例如：
```js
uniqueInOrder('AAAABBBCCDAABBB') == ['A', 'B', 'C', 'D', 'A', 'B']
uniqueInOrder('ABBCcAD')         == ['A', 'B', 'C', 'c', 'A', 'D']
uniqueInOrder([1,2,2,3,3])       == [1,2,3]
```

稍微变化了一点的数组去重，要使得连续的字符去重为单个，容易想到通过比对当前游标元素和下一个元素，这里给出一个时间复杂度O(n)的解法

```js
const uniqueInOrder = (p) => {
  let arr = typeof p === 'string' ? p.split('') : p
  return arr.reduce((r, next) => {
    return r[r.length-1] === next ? r : r.concat(next)
  }, [])
}
```

