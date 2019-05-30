# 函数式 JavaScript：如何在数字之外使用 Array.reduce()

作者：James Sinclair  
时间：2019.5.22  
原文地址：`https://jrsinclair.com/articles/2019/functional-js-do-more-with-reduce/`

Reduce 方法就是 数组迭代器 的瑞士军刀，它真的很强大。它强大到，你可以用 Redude()
构建其他像.map(), .filter() 和.flatMap()等数组迭代器的方法。在这篇文章中，你可以看到一些 Reduce 可以做的更有意思的东西。但是如果你是一个数组迭代器的新手，你会对 reduce()感到困惑

> Reduce 是迄今为止发现的最通用的方法之一
> --Eric Elliott<sup>[1]</sup>

人们经常在超越基本的例子之后陷入麻烦。像加法和乘法很简单，但是你尝试一些稍微复杂的事情的时候，它就崩溃了。将 reduce 用在数字之外的类型上，就开始变得令人困惑了。

## 1 为什么 reduce()会给人们带来这么多麻烦？

我对此有一个理论。我认为有两个主要原因会导致这么多麻烦。首先，我们在教.reduce（）之前倾向于教人们学习.map（）和.filter（）。但.reduce（）的函数签名与它们是不同的。习惯于初始值的想法是一个非常重要的步骤。其次 reducer 函数也有不同的签名。它需要一个累加器值以及当前数组元素。所以学习.reduce（）可能很棘手，因为它与.map（）和.filter（）有很大的不同。而且这些也不能避免。但我认为还有另一个因素在起作用。  
第二点是关于我们如何教别人学习 .reduce() 函数,像下面教程给出的例子也很常见：

```js
function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

const sampleArray = [1, 2, 3, 4];

const sum = sampleArray.reduce(add, 0);
console.log('The sum total is:', sum);
// ⦘ The sum total is: 10

const product = sampleArray.reduce(multiply, 1);
console.log('The product total is:', product);
// ⦘ The product total is: 24
```

现在，我说这些不是为了羞辱谁。 MDN 文档使用这种示例。哎呀，我甚至自己这样做过。我们这样做是有充分理由的。像 add（）和 multiply（）这样的函数很容易理解。但不幸的是，它们有点过于简单。使用 add（），无论是 `b + a` 还是`a + b` 都无关紧要。同样适用于乘法。 `a * b` 与 `b * a` 相同。这就像你期望的那样。但麻烦的是，这使得更难以看到 reducer 函数发生了什么。  
reducer 函数是我们传递给.reduce（）的第一个参数。它的签名看起来像这样<sup>[2]</sup>:

```js
function myReducer(accumulator, arrayElement) {
  // Code to do something goes here
}
```
