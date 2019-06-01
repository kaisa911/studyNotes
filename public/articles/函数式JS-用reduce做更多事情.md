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

我对此有一个理论。我认为有两个主要原因会导致这么多麻烦。首先，我们在教.reduce()之前倾向于教人们学习.map()和.filter()。但.reduce()的函数签名与它们是不同的。习惯于初始值的想法是一个非常重要的步骤。其次 reducer 函数也有不同的签名。它需要一个累加器值以及当前数组元素。所以学习.reduce()可能很棘手，因为它与.map()和.filter()有很大的不同。而且这些也不能避免。但我认为还有另一个因素在起作用。  
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

当然，我说这些不是为了让谁感到羞愧。 MDN 文档使用这种示例。哎呀，我甚至自己这样做过。我们这样做是有充分理由的。像 add()和 multiply()这样的函数很容易理解。但不幸的是，它们有点过于简单。使用 add()，无论是 `b + a` 还是`a + b` 都无关紧要。同样适用于乘法。 `a * b` 与 `b * a` 相同。这就像你期望的那样。但麻烦的是，这使得更难以看到 reducer 函数发生了什么。  
reducer 函数是我们传递给.reduce()的第一个参数。它的签名看起来像这样<sup>[2]</sup>:

```js
function myReducer(accumulator, arrayElement) {
  // Code to do something goes here
}
```

累加器代表了一个类似'进位'值。它包含上次调用 reducer 函数时返回的内容。如果 reducer 函数还没有被调用，则它就是初始值。因此，当我们将 add()作为 reducer 传递时，累加器将映射到 `a + b` 的一部分。恰好包含了数组之前所有值的的总和。乘法也是如此。 `a * b` 中的 a 参数包含运行的之前的乘积。向人们展示这一点并没有错。但是，它掩盖了一个.reduce()最有趣的功能。

.reduce()的强大功能来自于 accumulator 和 arrayElement 不必是同一类型的事实。对于加法和乘法，a 和 b 都是数字。他们是同一类型。但我们没必要这样来使用.reduce()。累加器可以是与数组元素完全不同的东西。

例如，我们的累加器可能是一个字符串，而我们的数组包含数字:

```js
function fizzBuzzReducer(acc, element) {
  if (element % 15 === 0) return `${acc}Fizz Buzz\n`;
  if (element % 5 === 0) return `${acc}Fizz\n`;
  if (element % 3 === 0) return `${acc}Buzz\n`;
  return `${acc}${element}\n`;
}

const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15];

console.log(nums.reduce(fizzBuzzReducer, ''));
```

这只是一个说明问题的例子。如果我们使用字符串，我们可以使用.map()和.join()组合来实现相同的功能。但.reduce()不仅仅对字符串有用。累加器值不必是简单类型（如数字或字符串）。它可以是结构化类型，如数组或普通的'JavaScript 对象（PO​​JO）。这让我们可以做一些非常有趣的事情，我们马上就会看到。

## 2 我们可以用 reduce 做一些有趣的事情

那么，我们可以做些什么有趣的事情呢？我在这里列出了五个不涉及将数字相加的内容：  
1、将数组转换为对象;  
2、展开更大的阵列;  
3、在一次遍历中进行两次计算;  
4、将映射和过滤组合成一个通道;  
5、按顺序运行异步函数
