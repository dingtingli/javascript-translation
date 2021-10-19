# JavaScript basics: The Execution Context and the Lexical Environment
# 翻译：JavaScript基础：执行上下文和词法环境 

点击访问：[原文地址](https://medium.com/dailyjs/javascript-basics-the-execution-context-and-the-lexical-environment-3505d4fe1be2)

作者：[Dan Park](https://medium.com/@danparkk)

当代码被编译（ compile ）或者解释（ interpret ）的时候，会有很多神奇的事情在幕后发生。

JavaScript 被称为解释型语言，而不是像 `C` 或者 `Java` 那种编译型语言。关于解释型语言和编译型语言的区别，可能需要一整篇文章来讨论，这里先不详述了。然而，对于很多 JavaScript 开发新手来说，许多困惑都是由编译阶段产生的。

从语义上讲，你有两种理解编译的方式： 

1. 编译器将整个程序的代码“编译”成机器代码，然后再执行代码。
2. 编译仅仅意味着让程序代码易于机器运行。

第一种方式是作为编译型语言的传统定义。毕竟，如果你仔细想想，如果编译只意味着将程序代码变成机器可读的代码，那么所有的语言都是“编译型”。在传统的定义下，JavaScript 被理解为解释型语言。谷歌的 V8 引擎在程序运行的时候逐行编译程序代码——称为即时编译器（ Just-In-Time Compiler：JIT ）。但关于这一点不是本文讨论的重点。

## 执行上下文

执行上下文是当前代码在其计算环境中的抽象概念。

在 ES5 的文档中，执行上下文“包含跟踪相关代码执行进程所需的任何状态”。

通常，这里的“环境”是指：1.默认的全局环境，或者 2.函数环境，即函数内的代码。注意：eval 代码是不同的环境。这篇文章接下来只会讨论全局和函数环境。 

当 JavaScript 解释器在执行前逐行“编译”代码的时候，它会创建调用栈（ call stack ）。因为 JavaScript 是一个单线程语言，在浏览器中一次只发生一件事情，其余的事情都在这个调用栈中等待运行。 

想象一下一个典型的栈结构，底部是默认的全局执行上下文，上面是无数的函数执行上下文。同样的情况也适用于在函数内部调用另一个函数。简单地说，函数的执行上下文可以一个接一个地堆在栈的上面。可以想象成，盒子上面堆盒子。 

回到代码执行前 JavaScript 逐行编译的阶段，其实在幕后也发生了一些事情。这就是所谓的创建阶段。

## 第一阶段：创建阶段

在 JavaScript 被执行前的几微秒里，创捷阶段发生了三件事情： 

1. ThisBinding状态组件被确定 

2. 词法环境的状态组件被定义 

3. 变量环境的状态组件被定义 

首先，ThisBinding状态组件……

这只是 this 关键字在当前执行上下文中的值。 

在全局执行上下文中，this 关键字被绑定到全局对象上（在浏览器中，是 windows 对象）。 

然而，在函数代码中，this 关键字取决于函数如何被调用。 

比如： 

```javascript
let foo = {
    bar: function () {
        console.log(this);
    }
};

foo.bar(); // 'this' is bound to the 'foo' object because 'bar' was called with reference to 'foo'

let baz = foo.bar;
baz(); // 'this' is bound to the Global Window Object because no reference was given when called
```

总的来说，当函数调用时给出了一个对象引用，this 就指向这个对象。然而，如果没有给出对象引用（或者该引用为 `null` 或者 `undefined` ），this 将指向全局的 windows 对象。

第二，词法环境状态组件……

我认为思考一下为什么 this 很重要是有帮助的。简单的说，我们必须要知道，机器是如何知道在哪里查询代码中声明的变量的？词法环境就是这种“标识符解析”发生的地方。  

在官方的 ES5 文档中，词法环境（其实是一个抽象的概念）定义为“根据 ECMAScript 代码的词法嵌套结构，将特定变量的标识符和函数关联起来”的地方。这个定义有两个主要的结论：

1. 标识符关联只是变量和函数声明及其值之间的绑定。（例如，`let x=10` ，`x` 被绑定为 `10` ）。

2. 词法结构只是描述代码编写的实际位置。

```javascript
let foo = function (a) { 
    let b = 10;  // 'b' and 'bar' is in the lexical structure of 'foo' 
    let bar = function(a) {
        let c = 20;        // however, 'c' would only be in the lexical structure of 'bar' because 'c' is written inside the 'bar' function.
        return a + b + c; 
};
```
这是代码编写的位置，这也是你如何知道变量和函数声明与词法环境相关联的地方。

现在，词法环境中有两个组件：1. 环境记录（ environment record ） 和 2.外部环境的引用（ outer ）。

1. 环境记录是存放变量和函数声明的地方。

2. 外部环境的引用（ outer ）是机器标识符解析（作用域）的路径。

* 在全局代码环境中，你可以期待这个环境记录中内置 Object / Array /等等原型函数，以及任何用户自定义的全局变量。而外部环境的引用将被置为 null（因为他是最外层的环境）。 

* 在函数代码中，函数内部用户自定义的变量和词法结构都存储在环境记录中。外部环境的引用可以是全局环境，也可以是包裹内部函数的外部函数。

让事情更加混乱的是，有两种类型的环境记录…… 

1. 声明式环境记录（ declarative environment record ）处理变量、函数和参数（类似与ES3中的“activation object”概念）。

2. 对象环境记录（ object environment record ）描述了在全局上下文中定义标识符管理的方式（以及 `with` 语句）。

简单地说，

* 在全局代码中，环境记录是对象环境记录。
* 在函数代码中，环境记录是声明式环境记录。

一个快速但是重用的说明：对于函数代码，声明式环境记录也包含了参数对象（ argument object ），该对象有映射到参数的索引，就像 key:value 键值对，并且还包含了传递到函数的参数长度。

* 正如上面所暗示的那样，对于全局代码，词法环境被设置为全局环境。 

* 对于函数代码，词法环境将包含所有的变量和函数声明。这实际上是我们看到“提升（ Hoisting ）”现象的地方——在任何赋值之前，声明被带到其词法范围的顶部。

最后，变量环境状态组件……

这实际上是词法环境的一个副本。两者实际上没有区别，除了在 `with` 语句的使用上。`with` 语句实际上修改的是词法环境，而不是变量环境。`with` 语句的使用不在本文讨论的范围，实际上一些开发者认为，出于性能的考虑，应该完全避免 `with` 语句。 

因此，在大多数情况下，我们假设两者是一样的。

## 执行阶段

这是整篇文章最直接的部分。在 V8 引擎遍历代码后，绑定 this 关键字并定义了词法环境和变量环境，代码最终被执行了。 

在这里对变量/函数声明的赋值终于完成了（那些被“提升”的变量现在也被恰当地定义了）。代码运行，我们看到了代码的最终结果。

## 为什么这些内容很重要？

可能对于大多数开发者来说，你不需要知道这些概念来证明对“提升”、作用域和标识符解析的理解。然而，如果你和我一样，渴望知道更深层次到底发生了什么（依然比机器代码更抽象），那么现在你可以根据 ECMAScript 规范准确地解释正在发生什么。 

就个人学习而言，了解标识符解析实际发生的位置（词法环境）和“提升”发生的原因（创建阶段和执行阶段）是很有启发的。 

很多文章都不会深挖到这一层，因为在某种程度上，你可以通过阅读实际的文档来了解。但在这里，我试图以某种方式消化它，以便我能理解它。我希望这对像我一样喜欢深挖的读者有所帮助。

资源和参考

[1] [Dmitry Soshnikov’s post on the Theory of Lexical Environments](http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-1-lexical-environments-common-theory/)

[2] [Dmitry Soshnikov’s post on ECMAScript Implementation of Lexical Environments](http://dmitrysoshnikov.com/ecmascript/es5-chapter-3-2-lexical-environments-ecmascript-implementation/)

[3] [ES5 Documentation on Code Evaluation and Execution Context](http://es5.github.io/#x10.4)

[4] [ES5 Documentation on Lexical Environments](http://es5.github.io/#x10.2)