---
layout: post
title: redux中间件学习
subtitle: "\"初探函数式编程\""
date: 2018-05-24 14:39:00
tags: 
    - react
    - javascript
---

在使用 react 进行开发的过程中, 我们经常会与[高阶组件][hoc]进行打交道。react 中的高阶组件非常强大, 许多 react 的相关库都利用了高阶组件的特性, 如最著名的[react-redux][react-redux]、[react-router][react-router]等, 都借助高阶组件实现许多强大的功能诸如: 属性代理、反向继承等。这些听起来高大上的名词隐隐有种武功秘籍的感觉, 高阶组件当中包含了一种函数式编程的思想。

直接学习 react 的高阶组件对我这种 fish 来讲未免稍微有点晦涩, 因此我决定先从 redux 的中间件开始学习。关于 redux 中间件, 推荐可以看一下[阮一峰][ryf]的博客,浅显易懂, 一目了然.

## applyMiddleWare

redux 提供我们一个调用插件的方法`applyMiddleWare`, 我觉得这是一个非常好的学习入口, 下面看看源码中是怎么实现的.

```js
import compose from './compose'

/**
 * 为了减少篇幅, 这里省略了注释, 但强烈推荐阅读这些注释, 这些注释完整地表达了redux中间件的一个思想
 * @param {...Function} middlewares 接收的参数是一个个的我们所要调用的中间件插件函数, 方法内部对它们进行了解构得到一个`middlewares`数组
 * @returns {Function} A store enhancer applying the middleware.
 */
export default function applyMiddleware(...middlewares) {
  //  applyMiddleware返回一个新的函数, 参数createStore就是redux里的createStore方法, 在createStore方法里, 会根据applyMiddleware是否传入对store做处理, 这里不讨论。
  return createStore => (...args) => {
    const store = createStore(...args)
    // 这里的dispatch是私有的, 如果dispatch没有被下面的代码改写掉, 调用时会抛出一个错误
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }
    // 在中间件中, redux暴露getState和dispatch供我们使用
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args),
    }
    // middleware方法也是一个柯里化函数
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    // 对私有dispatch通过外部接口store.dispatch进行改造
    dispatch = compose(...chain)(store.dispatch)

    // 把我们私有的dispatch暴露给外部调用, 通过middleware, 我们现在使用的dispatch已经是改造过后的dispatch了
    return {
      ...store,
      dispatch,
    }
  }
}
```

applyMiddleware 方法中有个很有意思的方法 `compose` , 我们去看看 compose 里面做了什么

```js
/**
 * Composes single-argument functions from right to left. The rightmost
 * function can take multiple arguments as it provides the signature for
 * the resulting composite function.
 *
 * @param {...Function} funcs The functions to compose.
 * @returns {Function} A function obtained by composing the argument functions
 * from right to left. For example, compose(f, g, h) is identical to doing
 * (...args) => f(g(h(...args))).
 */

export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```

代码非常精炼简洁, 注释里也写得很清楚, compose 做的事情就是帮我们把多个单参数函数从右往左进行组合, 比如 compose(f, g, h)(arg1, arg2)可以得到
f(g(h(arg1, arg2)))。通过 applyMiddleware 方法可以得知, 主要是处理中间件数组的。

到了这里我们还是不知道中间件是怎么处理我们的 dispatch 的, 接下来我们看看一个著名的中间件[redux-thunk][redux-thunk], redux-thunk 这个中间件也是精练的夸张, 核心代码只有 12 行, 它只做了一件事: 让我们的 dispatch 可以接收一个函数作为 action, 要知道普通 dispatch 都是接收**一个普通对象**作为参数的, 这就导致了我们的 dispatch 一定是一个同步操作, 如果 dispatch 可以接收一个函数, 那函数能做的事情可就多了, 这也是 redux 异步的方式之一。让我们看看中间件里到底做了哪些操作。

```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument)
    }

    return next(action)
  }
}

const thunk = createThunkMiddleware()
thunk.withExtraArgument = createThunkMiddleware

export default thunk
```

我们对照`applyMiddleware`方法, 问题一下子清晰了许多: 原来我们的中间件先通过第一次 map 调用得到参数为 next 的函数数组 chain, 然后 compose 方法巧妙的将函数进行组合，并且通过 compose 我们可以将 store.dispatch 原方法传入，在 compose 中，这个方法会从右到左依次进行包裹并传给下一个中间件，这个过程并没有改变 store.dispatch 方法。在 redux 官网文档中，作者为了深入浅出介绍 middleware 的实现思路，以最简版的实现开始（每经历一个中间件，就破坏性地覆盖 store.dispatch 方法），层层深入到最终版，详细看[这里][middleware]。

## 总结
redux中间的核心思想是通过对dispatch的包装和处理，使得action -> reducer 方式变成了 action -> middlewawre -> reducer。对dispatch进行处理就要考虑我们应该以什么样的方式进行对dispatch方法的层层包装。redux的想法很巧妙，我们不直接修改dispatch方法，而是对中间件进行柯里化。比如我们传入中间件ABC：applyMiddleware([A, B, C])，在通过compose处理后便是composedABC，于是先是进入A中间件，此时next就是 composedBC(action), 在执行next时就会进入到B中间件，以此类推直到最后一个中间件C，此时在C中的next便是之前调用compose(...chain)(store.dispatch)的dispatch方法，于是就调用了dispatch(action)。需要值得注意的是，有许多中间件内部并不会直接`return next(action)`，更常见的做法是：
```js
function ASimpleMiddleware ({getState, dispatch}) => next => action => {
  // ...some code
  let returnValue = next(action)
  // ...others code
  return returnValue
}
```
这样的话我们在处理完最后一个中间件C得到返回值的时候，代码继续执行，于是返回到上一个中间B，以此类推直到回到第一个中间件。如果接触过koa的话是不是觉得这和koa当中的洋葱模型非常的相似！至此，中间件的整个过程就算结束了。这么捋一遍收获还是蛮多的，感觉对redux理解又加深了一点点，同时对redux作者的敬佩之情更深了！！

<!-- Link -->

[HOC](https://reactjs.org/docs/higher-order-components.html)
[react-redux](https://redux.js.org/basics/usage-with-react)
[react-router](https://reacttraining.com/react-router/)
[ryf](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html)
[redux-thunk](https://github.com/reduxjs/redux-thunk)
[middleware](https://redux.js.org/advanced/middleware)
