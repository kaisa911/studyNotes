# JavaScript 整洁代码-最佳实践

作者：Milos Protic

原文地址：`https://devinduct.com/blogpost/22/javascript-clean-code-best-practices`

## 简介

如果你关心代码本身和代码是怎么写的，而不是担心它是否有效果，你可以说你在实践并关注整洁代码。一个专业的开发者写代码是为了未来的自己和其他人，而不仅仅是让机器认识。你写的任何代码都不会只写一次，而是要坐好并等待未来的某个人，来让他感到惨不忍睹。希望未来的那个人不是你。

基于此，整洁的代码可以被定义为：**代码应该以一种能够自我解释，便于其他人理解、便于改变和扩展方式来写**

问问你自己，多少次当你第一印象是下面“WTF”的问题之一时，你会继续完成别人的工作？

"WTF is that?"  
"WTF did you do here?"  
"WTF is this for?"

这里有一张非常火的图，可以解释上面的问题。

![图图图](https://github.com/kaisa911/studyNotes/blob/master/public/image/wtfexpress.jfif?raw=true)

罗伯特·C·马丁（鲍勃大叔）的一句引言，应该能让你思考你的方式。

> 即使是糟糕的代码也能运行，但如果代码没有规范，它也能够让开发组织陷入困境。

在这篇文章中，关注的是 JavaScript，但是其中的原则，也可以用于其他的编程语言中。

## 你来到这里的实际阅读部分-整洁代码的最佳实践

### 1 强类型校验

使用 === 而不是 ==

```js
//如果处理不当，它会极大地影响程序逻辑。
//就像，你希望向左，但由于某种原因，你向右了
0 == false; // true
0 === false; // false
2 == '2'; // true
2 === '2'; // false

// 举例
const value = '500';
if (value === 500) {
  console.log(value);
  // 这不会成功的
}

if (value === '500') {
  console.log(value);
  // 这不会成功的
}
```

### 2 变量

用一种能够解释其含义的方式去命名变量，这样，后面的人去看它的时候，就更容易理解和搜索

Bad：

```js
let daysSLV = 10;
let y = new Date().getFullYear();

let ok;
if (user.age > 30) {
  ok = true;
}
```

Good:

```js
const MAX_AGE = 30;
let daysSinceLastVisit = 10;
let currentYear = new Date().getFullYear();

...

const isUserOlderThanAllowed = user.age > MAX_AGE;
```

不要在变量名中添加额外不需要的单词
Bad:

```js
let nameValue;
let theProduct;
```

Good:

```js
let name;
let product;
```

不要强制记忆变量的上下文
Bad:

```js
const users = ['John', 'Marco', 'Peter'];
users.forEach(u => {
  doSomething();
  doSomethingElse();
  // ...
  // ...
  // ...
  // ...
  // 这里就有wtf的情况，这个'u'是干什么的
  register(u);
});
```

Good:

```js
const users = ['John', 'Marco', 'Peter'];
users.forEach(user => {
  doSomething();
  doSomethingElse();
  // ...
  // ...
  // ...
  // ...
  register(user);
});
```

不要添加不必要的上下文

Bad:

```js
const user = {
  userName: "John",
  userSurname: "Doe",
  userAge: "28"
};

...

user.userName;
```

Good:

```js
const user = {
  name: "John",
  surname: "Doe",
  age: "28"
};

...

user.name;
```

### 3 函数

使用长的具有描述性的名字。考虑到它需要表示某种行为，函数名称应该是动词或短 语，完全揭示其背后的意图以及参数的意义。他们的名字应该说明他们做了什么。

Bad:

```js
function notif(user) {
  // implementation
}
```

Good:

```js
function notifyUser(emailAddress) {
  // implementation
}
```

函数应避免有大量参数。理想情况下，函数应该指定两个或更少的参数。参数越少，测试该函数就越容易。

Bad:

```js
function getUsers(fields, fromDate, toDate) {
  // implementation
}
```

Good:

```js
function getUsers({ fields, fromDate, toDate }) {
  // implementation
}

getUsers({
  fields: ['name', 'surname', 'email'],
  fromDate: '2019-01-01',
  toDate: '2019-01-18'
});
```

使用默认参数而不是条件判断

Bad:

```js
function createShape(type) {
  const shapeType = type || 'cube';
  // ...
}
```

Good:

```js
function createShape(type = 'cube') {
  // ...
}
```

一个函数应该做一件事。避免在单个函数中执行多个操作

Bad:

```js
function notifyUsers(users) {
  users.forEach(user => {
    const userRecord = database.lookup(user);
    if (userRecord.isVerified()) {
      notify(user);
    }
  });
}
```

Good:

```js
function notifyVerifiedUsers(users) {
  users.filter(isUserVerified).forEach(notify);
}

function isUserVerified(user) {
  const userRecord = database.lookup(user);
  return userRecord.isVerified();
}
```

使用`Object.assign()` 来设置默认对象.

Bad:

```js
const shapeConfig = {
  type: 'cube',
  width: 200,
  height: null
};

function createShape(config) {
  config.type = config.type || 'cube';
  config.width = config.width || 250;
  config.height = config.width || 250;
}

createShape(shapeConfig);
```

Good:

```js
const shapeConfig = {
type: "cube",
width: 200
// Exclude the 'height' key
};

function createShape(config) {
config = Object.assign(
{
type: "cube",
width: 250,
height: 250
},
config
);

...
}

createShape(shapeConfig);
```

不要使用标志来当作参数，因为这样做表明这个函数正在做很多比它应该做的事情

Bad:

```js
function createFile(name, isPublic) {
  if (isPublic) {
    fs.create(`./public/${name}`);
  } else {
    fs.create(name);
  }
}
```

Good:

```js
function createFile(name) {
  fs.create(name);
}

function createPublicFile(name) {
  createFile(`./public/${name}`);
}
```

不要污染全局变量。如果需要扩展现有对象，请使用 ES 类和继承，而不是在原生对象的原型链上创建函数

Bad:

```js
Array.prototype.myFunc = function myFunc() {
  // implementation
};
```

Good:

```js
class SuperArray extends Array {
  myFunc() {
    // implementation
  }
}
```

### 4 条件

避免使用否定条件

Bad:

```js
function isUserNotBlocked(user) {
  // implementation
}

if (!isUserNotBlocked(user)) {
  // implementation
}
```

Good:

```js
function isUserBlocked(user) {
  // implementation
}

if (isUserBlocked(user)) {
  // implementation
}
```

使用短路判断。这可能是微不足道的，但是却值得被提起。使用这个方法仅当判断的是你已经确定值将不会是`undefined`或者`null`的布尔值，

Bad:

```js
if (isValid === true) {
  // do something...
}

if (isValid === false) {
  // do something...
}
```

Good:

```js
if (isValid) {
  // do something...
}

if (!isValid) {
  // do something...
}
```

尽可能避免条件。请改用多态和继承

Bad:

```js
class Car {
  // ...
  getMaximumSpeed() {
    switch (this.type) {
      case 'Ford':
        return this.someFactor() + this.anotherFactor();
      case 'Mazda':
        return this.someFactor();
      case 'McLaren':
        return this.someFactor() - this.anotherFactor();
    }
  }
}
```

Good:

```js
class Car {
  // ...
}

class Ford extends Car {
  // ...
  getMaximumSpeed() {
    return this.someFactor() + this.anotherFactor();
  }
}

class Mazda extends Car {
  // ...
  getMaximumSpeed() {
    return this.someFactor();
  }
}

class McLaren extends Car {
  // ...
  getMaximumSpeed() {
    return this.someFactor() - this.anotherFactor();
  }
}
```

### 5 ES 类

Class 在 JavaScript 中是新的语法糖。一切都像以前的原型一样工作，只有看起来不同，你应该更喜欢它们而不是 ES5 的普通函数

Bad:

```js
const Person = function(name) {
  if (!(this instanceof Person)) {
    throw new Error('Instantiate Person with `new` keyword');
  }

  this.name = name;
};

Person.prototype.sayHello = function sayHello() {
  /\*\*/;
};

const Student = function(name, school) {
  if (!(this instanceof Student)) {
    throw new Error('Instantiate Student with `new` keyword');
  }

  Person.call(this, name);
  this.school = school;
};

Student.prototype = Object.create(Person.prototype);
Student.prototype.constructor = Student;
Student.prototype.printSchoolName = function printSchoolName() {
  /\*\*/;
};
```

Good:

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  sayHello() {
    /_ ... _/;
  }
}

class Student extends Person {
  constructor(name, school) {
    super(name);
    this.school = school;
  }

  printSchoolName() {
    /_ ... _/;
  }
}
```

使用方法链。许多库如 jQuery 和 Lodash 都使用这种模式。这样，你的代码将不那么冗长。在你的类中，只需在每个函数的末尾返回它，你就可以将更多的类方法链接到它上面。

Bad:

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  setSurname(surname) {
    this.surname = surname;
  }

  setAge(age) {
    this.age = age;
  }

  save() {
    console.log(this.name, this.surname, this.age);
  }
}

const person = new Person('John');
person.setSurname('Doe');
person.setAge(29);
person.save();
```

Good:

```js
class Person {
  constructor(name) {
    this.name = name;
  }

  setSurname(surname) {
    this.surname = surname;
    // 返回 this，形成链
    return this;
  }

  setAge(age) {
    this.age = age;
    // 返回 this，形成链
    return this;
  }

  save() {
    console.log(this.name, this.surname, this.age);
    // 返回 this，形成链
    return this;
  }
}

const person = new Person('John')
  .setSurname('Doe')
  .setAge(29)
  .save();
```

### 6 避免一致

一般来说，你要尽量不要重复自己，这意味着不能编写重复的代码，也不要在你留下像未使用的函数或者死代码这样的小尾巴~

出于各种原因，你最终可能会遇到重复的代码。例如，你可以有两个略有不同的事情，它们有很多共同之处，它们之间的差异性或紧迫的 DDL 迫使你创建包含几乎相同代码的单独函数。在这种情况下删除重复代码意味着抽象差异并在该级别上处理它们。

关于死代码，正如它名字所说的。它代码中的代码没有做任何事情，因为在某些开发阶段，你已经决定不再使用它了。你应该在代码库中搜索并删除所有不需要的函数和代码块。我可以给你的建议是，一旦你决定不再需要它，那就删除它。以后你可能会忘记它的用途。

这是一张图片，展示了你在那时的感受。

![tututu](https://github.com/kaisa911/studyNotes/blob/master/public/image/jscleancode2.png?raw=true)

## 总结

这只是改进代码所能做的一小部分。在我看来，这里所说的原则是人们经常不遵循的原则。他们试图但不总是因各种原因而成功。也许在项目开始时，代码整洁干净，但在满足最后期限时，原则经常被忽略并转移到“TODO”或“REFACTOR”部分。那时，你的客户宁愿让你满足截止日期，而不是编写整洁的代码。

以上

感谢你的阅读并期待在下一篇文章中见到你
