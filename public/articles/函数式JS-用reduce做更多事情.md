# 函数式 JavaScript：如何在数字之外使用 Array.reduce()

作者：James Sinclair  
时间：2019.5.22  
原文地址：`https://jrsinclair.com/articles/2019/functional-js-do-more-with-reduce/`

Reduce 方法就是 数组迭代器 的瑞士军刀，它真的很强大。它强大到，你可以用 Redude()
构建其他像.map(), .filter() 和.flatMap()等数组迭代器的方法。在这篇文章中，你可以看到一些 Reduce 可以做的更有意思的东西。但是如果你是一个数组迭代器的新手，你会对 reduce()感到困惑

> Reduce 是迄今为止发现的最通用的方法之一
> --Eric Elliott

人们经常在超越基本的例子之后陷入麻烦。像加法和乘法很简单，但是你尝试一些稍微复杂的事情的时候，它就崩溃了。将 reduce 用在数字之外的类型上，就开始变得令人困惑了。

## 1 为什么 reduce()会给人们带来这么多麻烦？
