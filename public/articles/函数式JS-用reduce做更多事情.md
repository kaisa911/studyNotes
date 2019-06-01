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
2、展开更大的数组;  
3、在一次遍历中进行两次计算;  
4、将 map 和 filter 组合在一次操作;  
5、按顺序运行异步函数

### 2.1 将数组转换为对象

我们可以使用.reduce()将数组转换为 POJO。如果您需要进行某种查找，这可能很方便。例如，假设我们有一个人员列表：

```js
const peopleArr = [
  {
    username: 'glestrade',
    displayname: 'Inspector Lestrade',
    email: 'glestrade@met.police.uk',
    authHash: 'bdbf9920f42242defd9a7f76451f4f1d',
    lastSeen: '2019-05-13T11:07:22+00:00'
  },
  {
    username: 'mholmes',
    displayname: 'Mycroft Holmes',
    email: 'mholmes@gov.uk',
    authHash: 'b4d04ad5c4c6483cfea030ff4e7c70bc',
    lastSeen: '2019-05-10T11:21:36+00:00'
  },
  {
    username: 'iadler',
    displayname: 'Irene Adler',
    email: null,
    authHash: '319d55944f13760af0a07bf24bd1de28',
    lastSeen: '2019-05-17T11:12:12+00:00'
  }
];
```

在某些情况下，通过用户名查找用户详细信息可能会很方便。为了使这更容易，我们可以将数组转换为对象。它可能看起来像这样：<sup>[3]</sup>

```js
function keyByUsernameReducer(acc, person) {
  return { ...acc, [person.username]: person };
}
const peopleObj = peopleArr.reduce(keyByUsernameReducer, {});
console.log(peopleObj);
// ⦘ {
//     "glestrade": {
//         "username":    "glestrade",
//         "displayname": "Inspector Lestrade",
//         "email":       "glestrade@met.police.uk",
//         "authHash":    "bdbf9920f42242defd9a7f76451f4f1d",
//          "lastSeen":    "2019-05-13T11:07:22+00:00"
//     },
//     "mholmes": {
//         "username":    "mholmes",
//         "displayname": "Mycroft Holmes",
//         "email":       "mholmes@gov.uk",
//         "authHash":    "b4d04ad5c4c6483cfea030ff4e7c70bc",
//          "lastSeen":    "2019-05-10T11:21:36+00:00"
//     },
//     "iadler":{
//         "username":    "iadler",
//         "displayname": "Irene Adler",
//         "email":       null,
//         "authHash":    "319d55944f13760af0a07bf24bd1de28",
//          "lastSeen":    "2019-05-17T11:12:12+00:00"
//     }
// }
```

在这个版本中，我将用户名留作了对象的一部分。但是通过一个小的调整你可以删除它（如果需要的话）。

### 2.2 把小数组展开成大的数组

通常，我们将.reduce()视为获取许多事物的列表并将其减少到单个值。但是没有理由单个值不能成为一个数组。并且也没有规则说数组必须小于原始数组。因此，我们可以使用.reduce()将短数组转换为更长的数组。

您可以很方便在从文本文件中读取数据，这是一个例子。想象一下，我们已经将一堆纯文本行读入数组中。我们想用逗号分隔每一行，并有一个大的名单。

```js
const fileLines = [
  'Inspector Algar,Inspector Bardle,Mr. Barker,Inspector Barton',
  'Inspector Baynes,Inspector Bradstreet,Inspector Sam Brown',
  'Monsieur Dubugue,Birdy Edwards,Inspector Forbes,Inspector Forrester',
  'Inspector Gregory,Inspector Tobias Gregson,Inspector Hill',
  'Inspector Stanley Hopkins,Inspector Athelney Jones'
];

function splitLineReducer(acc, line) {
  return acc.concat(line.split(/,/g));
}
const investigators = fileLines.reduce(splitLineReducer, []);
console.log(investigators);
// ⦘ [
//   "Inspector Algar",
//   "Inspector Bardle",
//   "Mr. Barker",
//   "Inspector Barton",
//   "Inspector Baynes",
//   "Inspector Bradstreet",
//   "Inspector Sam Brown",
//   "Monsieur Dubugue",
//   "Birdy Edwards",
//   "Inspector Forbes",
//   "Inspector Forrester",
//   "Inspector Gregory",
//   "Inspector Tobias Gregson",
//   "Inspector Hill",
//   "Inspector Stanley Hopkins",
//   "Inspector Athelney Jones"
// ]
```

我们从长度为 5 的数组开始，最后得到一个长度为 16 的数组。

你可能会看到我写的 [_Civilised Guide to JavaScript Array Methods_](https://jrsinclair.com/javascript-array-methods-cheat-sheet)。如果你正在关注这个，你可能已经注意到我推荐.flatMap()用于这种情况。所以，也许这个并不算数。但是，您可能还注意到.flatMap()在 Internet Explorer 或 Edge 中不可用。因此，我们可以使用.reduce()来创建我们自己的 flatMap()函数。

```js
function flatMap(f, arr) {
  const reducer = (acc, item) => acc.concat(f(item));
  return arr.reduce(reducer, []);
}

const investigators = flatMap(x => x.split(','), fileLines);
console.log(investigators);
```

所以，.reduce()可以帮助我们从短数组获取长数组。但它也可以在缺少的数组方法的时候代替那些方法。

### 2.3 在一次遍历中进行两次计算

有时我们需要根据单个数组进行两次计算。例如，我们可能想要计算数字列表的最大值和最小值。我们可以这样做：

```js
const readings = [0.3, 1.2, 3.4, 0.2, 3.2, 5.5, 0.4];
const maxReading = readings.reduce((x, y) => Math.max(x, y), Number.MIN_VALUE);
const minReading = readings.reduce((x, y) => Math.min(x, y), Number.MAX_VALUE);
console.log({ minReading, maxReading });
// ⦘ {minReading: 0.2, maxReading: 5.5}
```

这需要遍历我们的数组两次。但是，有时我们可能不想这样做。由于.reduce()允许我们返回任何我们想要的类型，因此我们不必返回数字。我们可以将两个值编码到一个对象中。然后我们可以在每次迭代时进行两次计算，并且只遍历数组一次：

```js
const readings = [0.3, 1.2, 3.4, 0.2, 3.2, 5.5, 0.4];
function minMaxReducer(acc, reading) {
  return {
    minReading: Math.min(acc.minReading, reading),
    maxReading: Math.max(acc.maxReading, reading)
  };
}
const initMinMax = {
  minReading: Number.MAX_VALUE,
  maxReading: Number.MIN_VALUE
};
const minMax = readings.reduce(minMaxReducer, initMinMax);
console.log(minMax);
// ⦘ {minReading: 0.2, maxReading: 5.5}
```

这个特殊例子的问题在于我们并没有真正提升性能。我们最终仍然执行相同数量的计算。但是，有些情况下它可能会产生真正的差异。例如，如果我们要组合.map()和.filter()操作......

### 2.4 将 map 和 filter 组合在一次操作

想象一下，我们之前有同样的 peopleArr。我们希望找到最近的登录，不包括没有电子邮件地址的人。一种方法是使用三个单独的操作：

1、过滤掉没有电子邮件的条目;  
2、然后 提取 lastSeen 属性;  
3、最后 找到最大值。

将它们放在一起看起来像这样：

```js
function notEmptyEmail(x) {
  return x.email !== null && x.email !== undefined;
}

function getLastSeen(x) {
  return x.lastSeen;
}

function greater(a, b) {
  return a > b ? a : b;
}

const peopleWithEmail = peopleArr.filter(notEmptyEmail);
const lastSeenDates = peopleWithEmail.map(getLastSeen);
const mostRecent = lastSeenDates.reduce(greater, '');

console.log(mostRecent);
// ⦘ 2019-05-13T11:07:22+00:00
```

现在，这段代码有着完美的可读性，并且正确运行。对于样本数据，它很好。但是如果我们有一个庞大的数组，那么我们就有可能开始遇到内存问题。这是因为我们用一个变量来存储每个中间数组。如果我们修改我们的 reducer 回调，那么我们可以一次完成所有事情：

```js
function notEmptyEmail(x) {
  return x.email !== null && x.email !== undefined;
}

function greater(a, b) {
  return a > b ? a : b;
}
function notEmptyMostRecent(currentRecent, person) {
  return notEmptyEmail(person)
    ? greater(currentRecent, person.lastSeen)
    : currentRecent;
}

const mostRecent = peopleArr.reduce(notEmptyMostRecent, '');

console.log(mostRecent);
// ⦘ 2019-05-13T11:07:22+00:00
```

在这个版本中，我们只遍历一次数组。但如果人员名单总是很小的话，这可能不会有所改善。我的建议是默认使用.filter()和.map()。如果您确定内存使用或性能问题，请查看此类替代方案。

### 2.5 按顺序执行异步操作

另一件我们可以用.reduce()做的事情，是让 promise 按顺序执行（与平行相反）<sup>[4]</sup>。如果您对 API 请求有速率限制，或者您需要将每个 promise 的结果传递给下一个 promise，这可能很方便。举个例子，假设我们想在 peopleArr 数组中为每个人获取消息。

```js
function fetchMessages(username) {
  return fetch(`https://example.com/api/messages/${username}`).then(response =>
    response.json()
  );
}

function getUsername(person) {
  return person.username;
}

async function chainedFetchMessages(p, username) {
  // In this function, p is a promise. We wait for it to finish,
  // then run fetchMessages().
  const obj = await p;
  const data = await fetchMessages(username);
  return { ...obj, [username]: data };
}

const msgObj = peopleArr
  .map(getUsername)
  .reduce(chainedFetchMessages, Promise.resolve({}))
  .then(console.log);
// ⦘ {glestrade: [ … ], mholmes: [ … ], iadler: [ … ]}
```

请注意： 要使这个生效，我们要在使用 Promise.resolve()时候给它传一个初始值。它会立即执行（这就是 Promise.resolve()所做的）。然后我们的第一个 API 调用将立即运行。

## 3 为什么我们不经常看到 reduce 呢？

所以，我们已经看到了一些你可以用.reduce()做的有趣的事情。希望能够在您自己的项目里就如何将它提出一些想法。但是，既然.reduce()如此强大和灵活，那么为什么我们不经常看到它呢？具有讽刺是，它的灵活性和强大功问题是，你可以做很多不同的事情，减少它会给你更少的信息。 map()，.filter()和.flatMap()等方法更具体，灵活性更低。但他们告诉我们更多关于作者的意图。我们说这使他们更具表现力。所以通常使用更具表现力的方法更好，而不是使用 reduce 来解决所有问题。

## 4 对你 我的朋友

既然你已经看到了关于如何使用.reduce()的一些想法，为什么不试一试呢？如果你这样做，或者如果你发现我没有写过的新颖用法，请务必告诉我。我很想听听它。

## 5 附录

1、[Tweet by @JS_Cheerleader, 15 May 2019](https://twitter.com/JS_Cheerleader/status/1128420687712886784)

2、If you [look at the .reduce() documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce), you will see that the reducer takes up to four parameters. But only the accumulator and the arrayElement are required. I’ve left them out in the interest of keeping things simple. It can confuse people to include too much detail.

3、Some readers might point out that we could get a performance gain by mutating the accumulator. That is, we could change the object, instead of using the spread operator to create a new object every time. I code it this way because I want to keep in the habit of avoiding mutation. If it proved to be an actual bottleneck in production code, then I would change it.

4、If you'd like to know how to run Promises in parallel, check out [How to run async JavaScript functions in sequence or parallel](https://jrsinclair.com/articles/2019/how-to-run-async-js-in-parallel-or-sequential/).
