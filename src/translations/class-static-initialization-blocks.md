# ES2022 新特性：类的静态初始化块

- 原文地址: [https://2ality.com/2021/09/class-static-block.html](https://2ality.com/2021/09/class-static-block.html)

标签: `dev`, `javascript`, `es proposal`, `es2022`

Ron Buckton 的 ECMAScript 提案“类静态初始化块”已经处于第 4 阶段，并计划包含在 ECMAScript 2022 中。

为了设置一个类的实例，我们在 JavaScript 中有两个结构：

- 字段：创建（并可选择初始化）实例属性。
- 构造函数：在设置完成之前执行的代码块。

对于设置类的静态部分，我们只有静态字段。 ECMAScript 提案为类引入了静态初始化块，大致上，对于静态类来说，构造函数对于实例来说是一样的。

目录：

1. [为什么我们在类里需要静态块](#1为什么我们在类里需要静态块)
2. [一个更复杂的例子](#2一个更复杂的例子)
3. [详细内容](#3详细内容)
4. [支持类静态块的引擎](#4详细内容)
5. [JavaScript 是否变得很像 Java 或一团糟](#5JavaScript是否变得很像Java或一团糟)
6. [结论](#6结论)

## 1.为什么我们在类里需要静态块

设置静态字段时，使用外部函数通常效果很好:

```js
class Translator {
  static translations = {
    yes: 'ja',
    no: 'nein',
    maybe: 'vielleicht',
  };
  static englishWords = extractEnglish(this.translations);
  static germanWords = extractGerman(this.translations);
}

function extractEnglish(translations) {
  return Object.keys(translations);
}

function extractGerman(translations) {
  return Object.values(translations);
}
```

在这种情况下，使用外部函数 extractEnglish() 和 extractGerman() 效果很好，因为我们可以看到它们是从类内部调用的，并且它们完全独立于类。

如果我们想同时设置两个静态字段的时候，事情就会变得不那么优雅：

```js
class Translator {
  static translations = {
    yes: 'ja',
    no: 'nein',
    maybe: 'vielleicht',
  };
  static englishWords = [];
  static germanWords = [];
  static _ = initializeTranslator(
    // (A)
    this.translations,
    this.englishWords,
    this.germanWords
  );
}

function initializeTranslator(translations, englishWords, germanWords) {
  for (const [english, german] of Object.entries(translations)) {
    englishWords.push(english);
    germanWords.push(german);
  }
}
```

这个时候，有会几个问题：

- 调用 initializeTranslator() 是一个额外的步骤，必须在创建类之后在类之外执行。或者通过变通方法执行（A 行）。
- initializeTranslator() 无权访问 Translator 的私有数据。

使用建议的静态块（A 行），我们有一个更优雅的解决方案。

```js
class Translator {
  static translations = {
    yes: 'ja',
    no: 'nein',
    maybe: 'vielleicht',
  };
  static englishWords = [];
  static germanWords = [];
  static {
    // (A)
    for (const [english, german] of Object.entries(this.translations)) {
      this.englishWords.push(english);
      this.germanWords.push(german);
    }
  }
}
```

## 2.一个更复杂的例子

在 JavaScript 中实现枚举的一种方法, 是通过具有辅助功能的超类 Enum（有关此想法的更强大实现，请参阅库 enumify）：

```js
class Enum {
  static collectStaticFields() {
    // Static methods are not enumerable and thus ignored
    this.enumKeys = Object.keys(this);
  }
}
class ColorEnum extends Enum {
  static red = Symbol('red');
  static green = Symbol('green');
  static blue = Symbol('blue');
  static _ = this.collectStaticFields(); // (A)

  static logColors() {
    for (const enumKey of this.enumKeys) { // (B)
      console.log(enumKey);
    }
  }
}
ColorEnum.logColors();

// Output:
// 'red'
// 'green'
// 'blue'
```

我们需要收集静态字段，以便我们可以迭代枚举条目的键（B 行），这是创建所有静态字段后的最后一步。我们再次使用一种解决方法（A 行）。静态块会更优雅。

## 3.详细内容

静态块的细节是相对合乎逻辑的（相对于更复杂的实例成员规则）：

- 每个类可以有多个静态块
- 静态块的执行与静态字段初始值设定项的执行交错​​执行。
- 超类的静态成员在子类的静态成员之前执行。

以下代码演示了这些规则：

```js
class SuperClass {
  static superField1 = console.log('superField1');
  static {
    assert.equal(this, SuperClass);
    console.log('static block 1 SuperClass');
  }
  static superField2 = console.log('superField2');
  static {
    console.log('static block 2 SuperClass');
  }
}

class SubClass extends SuperClass {
  static subField1 = console.log('subField1');
  static {
    assert.equal(this, SubClass);
    console.log('static block 1 SubClass');
  }
  static subField2 = console.log('subField2');
  static {
    console.log('static block 2 SubClass');
  }
}

// Output:
// 'superField1'
// 'static block 1 SuperClass'
// 'superField2'
// 'static block 2 SuperClass'
// 'subField1'
// 'static block 1 SubClass'
// 'subField2'
// 'static block 2 SubClass'
```

## 4.支持类静态块的引擎

- V8：在 v9.4.146 中未标记
- SpiderMonkey：在 v92 中的标志后面，意图在 v93 中添加 
- TypeScrpt：v4.4

## 5.JavaScript是否变得很像Java或一团糟

这是一个小功能，不会与其他功能竞争。我们已经可以通过带有 static _ = ... 解决方法的字段运行静态代码。静态块意味着不再需要这种解决方法。

除此之外，类只是 JavaScript 程序员的众多工具之一。我们中的一些人使用它，其他人不使用它，并且有很多替代方案。即使是使用类的 JavaScript 代码也经常使用函数，并且往往是轻量级的。

## 6.结论

类静态块是一个相对简单的特性，它完善了类的静态特性。粗略地说，它是实例构造函数的静态版本。当我们必须设置多个静态字段时，它主要有用。