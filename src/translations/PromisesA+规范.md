# Promises/A+ 规范

[原文地址](https://promisesaplus.com/)

一个开放的，可交互的由开发者制定的 `JavaScript Promise` 标准。

一个 `Promise` 代表一个异步操作的最终结果。与 `Promise` 交互的主要方式是通过它的 `then` 方法，该 `then` 方法注册了两个回
调函数，用来接收 `Promise` 的最终结果或者导致 `Promise` 不能 `fulfilled`（已完成） 的原因。

本规范详细的列出 `then` 方法的行为，为所有符合 `Promises/A+` 规范的 `Promise` 提供了一个可互操作的基础来定义 `then` 方法
。因此本规范是稳定的。尽管 `Promise/A+` 组织可能会偶尔地通过一些较小的且向后兼容的修订，来解决新发现的一些边界情况。如果
要进行大规模或者不兼容的更新，我们一定会经过仔细的考虑、讨论和测试。

从历史上来说，`Promises/A+` 规范实际上是把之前 `Promises/A`规范中的建议变成了标准：扩展了原有规范约定俗成的行为，并删减
了原规范的一些特例情况和有问题的部分

最后 `Promises/A+` 规范不设计如何创建、解决和拒绝 `promise`，而是专注于提供一个可交互操作的 `then` 方法。未来在其他相关
规范中可能会提及。

## 1 术语

1.1 “`promise`” 是一个拥有 `then` 方法的 对象或者函数，它的行为符合本规范  
1.2 “`thenable`” 是一个定义了 `then` 方法的对象和函数  
1.3 值（“`value`”） 是任何 `JavaScript` 的合法值 (包括 `undefined`, `thenable`, 或者 `promise`).  
1.4 异常（“`exception`”） 是使用 `throw` 抛出的一个值   
1.5 原因（“`reason`”）是一个 `promise` 的拒绝原因

## 2 要求

### 2.1 `Promise` 的状态

一个 `Promise` 的当前状态必须是以下三种状态中的一种：等待（`pending`），完成（`fulfilled`）和拒绝（`rejected`）

2.1.1 当一个`promise`处于等待状态：  
 &emsp;&emsp; 2.1.1.1 可以迁移到完成状态或者拒绝状态中的任意一个  
2.1.2 当一个 `promise` 处于完成状态  
 &emsp;&emsp; 2.1.2.1 一定不能再迁移到其他状态  
 &emsp;&emsp; 2.1.2.2 必须有一个值(`value`)，而且一定不能再改变  
2.1.3 当一个 `promise` 处于拒绝状态  
 &emsp;&emsp; 2.1.3.1 一定不能再迁移到其他状态  
 &emsp;&emsp; 2.1.3.2 必须有一个原因（`reason`），而且一定不能再改变

这里，“一定不能改变” 意味着 恒等 （例如 ===），但是并不意味着更深层次的不可变。

### 2.2 Then 方法

一个 `promise` 必须提供一个 `then` 方法，来访问它当前的最终结果或者原因一个 `promise` 的 `then` 方法接收 2 个参数：

```js
promise.then(onFulfilled, onRejected);
```

2.2.1 `onFulfilled` 和 `onRejected` 都是可选参数:  
 &emsp;&emsp; 2.2.1.1 如果 `onFulfilled` 不是函数，那它必须被忽略  
 &emsp;&emsp; 2.2.1.2 如果 `onRejected` 不是函数，那它必须被忽略  
2.2.2 如果 `onFulfilled` 是一个函数:  
 &emsp;&emsp; 2.2.2.1 它必须在 promise 状态变成完成（`fulfilled`）状态之后才会调用，并且`promise`的值是第一个参数  
 &emsp;&emsp; 2.2.2.2 它一定不能在 promise 是完成（`fulfilled`）状态之前被调用  
 &emsp;&emsp; 2.2.2.3 它一定不能被调用超过 1 次  
2.2.3 如果 `onRejected` 是一个函数  
 &emsp;&emsp; 2.2.3.1 它必须在 `promise`状态变成拒绝（`rejected`）之后才会调用，并且`promise`的原因是第一个参数  
 &emsp;&emsp; 2.2.3.2 它一定不能在 `promise` 是拒绝（`rejected`）状态之前被调用  
 &emsp;&emsp; 2.2.3.3 它一定不能被调用超过 1 次  
2.2.4 `onFulfilled` 和 `onRejected` 只有在执行上下文栈仅包含平台代码的时候才会被调用 [3.1].  
2.2.5 `onFulfilled` 和 `onRejected` 必须当作函数调用（例如，没有 `this`值） [3.2]  
2.2.6 同一个 `promise` 的 `then` 方法可以多次调用  
&emsp;&emsp; 2.2.6.1 当`promise`是完成（`fulfilled`）状态，所有相应的`onFulfilled` 回调必须按照他们初始调用`then`方法的
顺序执行  
&emsp;&emsp; 2.2.6.2 当`promise`是拒绝（`rejected`）状态，所有相应的`onFulfilled` 回调必须按照他们初始调用`then`方法的顺
序执行  
2.2.7 `then` 方法必须返回一个`promise`对象 [3.3].

```js
promise2 = promise1.then(onFulfilled, onRejected);
```

&emsp;&emsp; 2.2.7.1 如果`onFulfilled` 或者 `onRejected` 返回一个值 `x`，则运行下面的 `Promise` 解决过
程`[[Resolve]](promise2, x)`  
 &emsp;&emsp; 2.2.7.2 如果`onFulfilled` 或者`onRejected` 抛出一个异常 `e`, `promise2` 必须以 `e`为原因被拒绝  
 &emsp;&emsp; 2.2.7.3 如果 `onFulfilled` 不是函数，并且 `promise1` 是完成状态，那么`promise2`必须以`promise1`同样的值完成.  
 &emsp;&emsp; 2.2.7.4 如果 `onReject` 不是函数，并且`promise1` 是拒绝状态，那么`promise2`必须以`promise1`同样的值拒绝.

### 2.3 Promise 解决过程

Promise 的解决过程是一个抽象操作，需要一个 `promise` 和一个值来作为输入，我们将其表示为`[[Resolve]](promise, x)`。如果`x` 有 `then` 方法且看上去像一个 `Promise` ，那它会尝试让 `promise` 接收 `x` 的状态，否则就用 `x` 的值来完成（`fulfilled`） `promise`。

这种`thenable`的特性使得 `Promise` 的实现更具有通用性：只要其暴露出一个遵循 `Promise/A+` 协议的 `then` 方法即可；这同时也使遵循 `Promise/A+` 规范的实现可以与那些不太规范但可用的实现能良好共存。

要运行`[[Resolve]](promise, x)`，需要执行如下步骤：

2.3.1 如果 `promise` 和 `x` 指向同一个对象，那用`TypeError`为原因拒绝`promise`
2.3.2 如果 x 是一个`promise`，那就让 `promise` 接受`x`的状态 [3.4]:
   &emsp;&emsp; 2.3.2.1 如果`x`是`pending`状态，那`promise`必须保持`pending`状态直到`x`变成完成（fulfilled）或者拒绝（`rejected`）
   &emsp;&emsp; 2.3.2.2 如果`x`是完成态`fulfilled`, 那让`promise` 用同样的值（`value`）完成.
   &emsp;&emsp; 2.3.2.3 如果`x`是拒绝态`rejected`, 那让`promise` 用同样的原因（`reason`）拒绝
2.3.3 `x`为对象或者函数,
   &emsp;&emsp; 2.3.3.1 把 `x.then` 赋值给 `then`. [3.5]
   &emsp;&emsp; 2.3.3.2 如果检索属性 `x.then` 导致抛出了一个异常 `e`，用 `e` 作为原因拒绝 `promise`。
   &emsp;&emsp; 2.3.3.3 如果`then`是一个函数，用`x`作为`this`调用它，第一个参数是 `resolvePromise`, 第二个参数是`rejectPromise`:
      &emsp;&emsp; &emsp;&emsp; 2.3.3.3.1 如果`resolvePromise`被一个值`y`调用，执行`[[Resolve]](promise, y)`.
      &emsp;&emsp; &emsp;&emsp; 2.3.3.3.2 如果 `rejectPromise` 以原因 `r`为参数被调用，则以原因`r`拒绝 `promise`
      &emsp;&emsp; &emsp;&emsp; 2.3.3.3.3 如果 `resolvePromise` 和 `rejectPromise` 都被调用，或者被同一参数调用了多次，则优先采用第一次调用并剩下的调用都
         会被忽略
      &emsp;&emsp; &emsp;&emsp; 2.3.3.3.4 如果调用抛出异常 `e`
        &emsp;&emsp; &emsp;&emsp; &emsp;&emsp;  2.3.3.3.4.1 如果 `resolvePromise` 或者 `rejectPromise` 已经被调用过了，那忽略它
        &emsp;&emsp; &emsp;&emsp; &emsp;&emsp;  2.3.3.3.4.2 用异常`e`为原因拒绝`promise`
   &emsp;&emsp; 2.3.3.4 如果`then`不是一个函数，用`x`完成`promise`
2.3.4 如果`x`不是一个对象或者数组，用`x`完成`promise`

如果一个 `promise` 被一个循环的 thenable 链中的对象完成，而 `[[Resolve]](promise, thenable)`的递归性质又使得其被再次调用，根据上面的算法将会导致无限递归。规范中并没有强制要求处理这种情况，但也鼓励实现者检测这样的递归是否存在，若检测到存在则用一个可识别的 `TypeError` 为原因来拒绝 `promise`[3.6]。

## 3、注释

3.1 这里的平台代码是指引擎、环境以及 `promise` 的实施代码。在实践中，要确保 `onFulfilled` 和 `onRejected` 两个参数异步执行，并且应该在 `then`方法被调用的那一轮事件循环之后的新执行栈中执行。这可以用如 `setTimeout` 或 `setImmediate`这样的“宏任务”机制实现，或者用如`MutationObserver`或 `process.nextTick` 这样的“微任务”机制实现。由于 `promise` 的实施代码本身就是平台代码，故代码自身在处理在处理程序时可能已经包含一个任务调度队列  
3.2 严格模式下，它们中的`this`将会是`undefined`；在非严格模式，`this`将会是全局对象  
3.3 假如实现满足所有需求，可以允许 `promise2 === promise1`。每一个实现都应该记录是否能够产生`promise2 === promise1` 以及什么情况下会出现 `promise2 === promise1`  
3.4 总的来说，只有`x`来自于当前实现，才知道它是一个真正的`promise`。这条规则允许那些特例实现采用符合已知要求的`Promise`的状态  
3.5 这个程序首先存储`x.then`的引用，之后测试和调用那个引用，这样避免了多次访问`x.then`属性。这种预防措施确保了该属性的一致性，因为访问者属性的值可能在俩次检索之间发生变化  
3.6 实现不应该在`thenable`链的深度上做任意限制，并且假设超过那个任意限制将会无限递归。只有真正的循环才应该引发一个`TypeError`；如果遇到一个无限循环的`thenable`，永远执行递归是正确的行为  
