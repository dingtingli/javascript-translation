# Understanding Execution Context and Execution Stack in Javascript
# 翻译：理解JavaScript中的执行上下文和执行栈

点击访问：[原文地址](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)

作者：[Carson](https://medium.com/@Sukhjinder)

学习 JavaScript 程序内部是如何运行的。

如果你希望成为一名 JavaScript 开发人员，那么你就必须知道 JavaScript 程序内部是如何执行的。理解执行上下文和执行栈对于理解 JavaScript 中的其他概念至关重要，比如：提升，作用域和闭包。 

正确理解执行上下文和执行栈的概念将会使你成为一名更优秀的 JavaScript 开发人员。 

所以，不再赘述，让我们开始吧。

## 什么是执行上下文？ 

简单地说，执行上下文是一个抽象的环境概念，在这个环境里 JavaScript 代码被解释和执行。每当代码在 JavaScript 中运行，它都是在一个执行上下文中运行。

### 执行上下文的类型 

在 JavaScript 中有三种类型的执行上下文。

* **全局执行上下文** —— 这是一个默认的或者说基础的执行上下文。不在任何函数内的代码都属于全局执行上下文。它做了两件事：创建了一个全局对象（浏览器中这个对象是 windows ），并且将 this 的值设置为指向该全局对象。一个程序中只能有一个全局执行上下文。 

* **函数执行上下文** —— 每当一个函数被调用时，都会为该函数创建一个新的执行上下文。每个函数都有自己的执行上下文，但它是在函数被调用时创建的。一个程序可以存在很多个函数的执行上下文。每当一个新的执行上下文被创建，它都会按照规定的顺序执行一系列的步骤，我会在文章的后面介绍这些内容。 

* **eval函数执行上下文** —— 在eval函数中执行的代码也会有自己的执行上下文，但是因为并不推荐JavaScript开发人员使用evel，所有本文将不会讨论相关内容。 

## 执行栈

执行栈，在其他编程语言中也被称为“调用栈”，是一个拥有后进先出（ Last in First Out ：LIFO ）功能的栈数据结构，它将用于存储所有在代码执行期间创建的执行上下文。

当 JavaScript 引擎第一次读取代码时，它会创建一个全局执行上下文，并将其存储到（压入）当前的执行栈中。每当引擎发现函数调用，它就会为该函数创建一个新的执行上下文，并将其压入到执行栈的顶部。 

JavaScript 引擎会执行在栈顶部的执行上下文的函数。当函数执行完成后，对应的执行上下文会从栈中弹出，准备执行下一个在栈顶部的执行上下文。

让我们通过下面的代码示例来理解一下： 

```javascript
let a = 'Hello World!';

function first() {
    console.log('Inside first function');
    second();
    console.log('Again inside first function');
}

function second() {
    console.log('Inside second function');
}

first();
console.log('Inside Global Execution Context');
```

![PNG 01](./illustrations/JSUnderstand01/png01.png)

当上面的代码加载到浏览器中时，JavaScript 引擎创建一个全局执行上下文，并将其压入当前执行栈中。当遇到对 `first()` 函数的调用时，JavaScript 引擎为该函数创建一个新的执行上下文，并将其压入当前执行栈的顶部。 

当 `second()` 函数在 `first()` 函数中被调用的时候，JavaScript 引擎为 `second` 函数创建一个新的执行上下文，并将其压入当前执行栈的顶部。当 `second` 函数执行完成后，对应的执行上下文就从当前执行栈中弹出，此时执行栈的顶部就是 `first` 函数的执行上下文。 

当 `first` 函数执行完成，它的执行上下文也会被弹出。这时候就剩下全局执行上下文留在栈中了。一旦所有的代码都执行完成，JavaScript 引擎会将全局执行上下文从当前栈中弹出。

## 如何创建执行上下文？

到目前为止，我们已经看到了 JavaScript 引擎如何管理执行上下文，现在我们来了解一下执行上下文是如何被 JavaScript 引擎创建的。 

执行上下文的创建分为两个阶段：1) 创建阶段 和 2）执行阶段

### 创建阶段

执行上下文是在创建阶段创建的。在创建阶段会发生以下一些事情：

1. 词法环境（ Lexical Environment ）组件的创建。 
2. 变量环境（ Variable Environment ）组件的创建。 

因此，执行上下文在概念上可以表示如下： 

```javascript
ExecutionContext = {
    LexicalEnvironment = <ref. to LexicalEnvironment in memory>,
    VariableEnvironment = <ref. to VariableEnvironment in memory>,
}
```

### 词法环境（ Lexical Environment ）

在ES6官方文档中词法环境的定义如下： 

> A Lexical Environment is a specification type used to define the association of Identifiers to specific variables and functions based upon the lexical nesting structure of ECMAScript code. A Lexical Environment consists of an Environment Record and a possibly null reference to an outer Lexical Environment. 

> 词法环境是一种规范类型，根据 ECMAScript 代码的词法嵌套结构来定义标识符与特定变量和函数的关联。一个词法环境由一个环境记录和一个可能为 null 值的外部词法环境引用所组成。

简单地说，词法环境是一种结构，它保存标识符-变量的映射。（这里标识符指的是变量/函数的名称，变量指的是实际对象（包括函数对象和数组对象）或者原始值的引用）。

比如，考虑下面的代码片段： 

```javascript
var a = 20; 
var b = 40; 

function foo() { 
    console.log('bar'); 
} 
```

上面代码片段的词法环境看起来是这样的：

```javascript
lexicalEnvironment = { 
    a: 20, 
    b: 40, 
    foo: <ref. to foo function> 
}
```

每个词法环境有三个组件： 

* 环境记录（ Environment Record ）
* 外部词法环境的引用（ outer ） 
* This 绑定

### 环境记录（ Environment Record ）

环境记录是变量和函数声明在词法环境中存储的地方。

有两种类型的环境记录： 

* **声明式环境记录（ Declarative environment record ）** —— 就像它名字所表明的那样，就是存储变量和函数声明的。函数代码的词法环境包含一个声明式的环境记录。 

* **对象环境记录（ Object environment record ）** —— 全局代码的词法环境包含一个对象环境记录。除了变量和函数声明之外，对象环境记录中还存储了全局绑定对象（浏览器中是 windows 对象）。所以，对于绑定对象的每个属性（ property ），都会在对象环境记录中创建一个新的条目（ entry ）（浏览器中，绑定对象会包含浏览器提供给 windows 对象的属性和方法）。 

注意：对于**函数**代码而言，环境记录还包含一个参数对象，其中包括了传递给函数的参数和索引的映射，以及参数的长度。比如，下面这个函数的参数对象看起来像这样： 

```javascript
function foo(a, b) { 
    var c = a + b; 
} 
foo(2, 3); 

// argument object 
Arguments: {0: 2, 1: 3, length: 2}, 
```

### 外部词法环境的引用（ outer ）

外部词法环境的引用意味着可以通过它来访问外部的词法环境。这意味着如果有变量在当前的词法环境找不到的话，JavaScript 引擎可以在外部词法环境中继续寻找变量。 

### This 绑定

在这个组件中，`this` 的值被确定或者设置。 

在全局执行上下文中，`this` 的值指向全局对象（浏览器中，`this` 指向 windows 对象）。 

在函数执行上下文中，`this` 的值取决于函数是如何调用的。如果函数是被一个对象引用调用的，那么 `this` 会被设置成该对象，否则，`this` 的值会被设置成全局对象或者 `undefined`（在 strict mode 下）。比如： 

```javascript
const person = { 
    name: 'peter', 
    birthYear: 1994, 
    calcAge: function() { 
        console.log(2018 - this.birthYear); 
  }
} 

person.calcAge();  
// 'this' refers to 'person', because 'calcAge' was called with 'person' object reference
// 'this' 指的是 'person' 对象，因为 'calcAge' 是被 'person' 对象调用。

const calculateAge = person.calcAge; 
calculateAge(); 
// 'this' refers to the global window object, because no object reference was given
// 'this' 指的是 global windows 对象，因为没有给出调用函数的对象。
```

抽象地讲，词法环境伪代码看起来是这样的：

```js
GlobalExectionContext = { 
    LexicalEnvironment: { 
        EnvironmentRecord: { 
            Type: "Object", 
            // Identifier bindings go here
            // 标识符在此绑定
        } 
        outer: <null>, 
        this: <global object> 
    } 
}

FunctionExectionContext = { 
    LexicalEnvironment: { 
        EnvironmentRecord: { 
            Type: "Declarative", 
            // Identifier bindings go here
            // 标识符在此绑定
        } 
        outer: <Global or outer function environment reference>, 
        this: <depends on how function is called> 
    } 
} 
```

### 变量环境（ Variable Environment ）

变量环境也是一个词法环境，它的环境记录持有变量声明语句在执行上下文中创建的绑定关系。 

就像上面说的，变量环境也是一个词法环境，所以它有着词法环境所有的属性和组件。 

在 ES6 中，词法环境和变量环境之间的一个区间是，词法环境用于存储函数声明和变量（ `let` 和 `const` ）绑定，变量环境只用于存储变量（ `var` ）的绑定。

### 执行阶段

在这个阶段，所有变量的赋值都会完成，代码最终被执行。

让我们看一些代码示例来理解上面的概念： 

```javascript
let a = 20;
const b = 30;
var c;

function multiply(e, f) {
    var g = 20;
    return e * f * g;
}

c = multiply(20, 30);
```

当上面这段代码执行的时候，JavaScript 引擎创建一个全局执行上下文来执行全局代码。所以全局执行上下文在创建阶段看起来是这样的：

```javascript
GlobalExectionContext = {
    LexicalEnvironment: {
        EnvironmentRecord: {
            Type: "Object",
            // Identifier bindings go here
            a: < uninitialized >,
            b: < uninitialized >,
            multiply: < func >
        }
        outer: <null>,
        ThisBinding: <Global Object>
    },

    VariableEnvironment: {
        EnvironmentRecord: {
            Type: "Object",
            // Identifier bindings go here
            c: undefined,
        }
        outer: <null>,
        ThisBinding: <Global Object>
    }
}
```

在执行阶段，变量赋值已经完成。因此全局执行上下文在执行阶段看起来是这样的：

```javascript
GlobalExectionContext = {
    LexicalEnvironment: {
        EnvironmentRecord: {
            Type: "Object",
            // Identifier bindings go here
            a: 20,
            b: 30,
            multiply: < func >
        }
        outer: <null>,
        ThisBinding: <Global Object>
    },

    VariableEnvironment: {
        EnvironmentRecord: {
            Type: "Object",
            // Identifier bindings go here
            c: undefined,
        }
        outer: <null>,
        ThisBinding: <Global Object>
    }
}
```

当调用函数 `multiply(20,30)` 时，会创建一个新的函数执行上下文来执行该函数代码。所以该函数执行上下文在创建阶段看起来是这样的：

```javascript
FunctionExectionContext = {
    LexicalEnvironment: {
        EnvironmentRecord: {
            Type: "Declarative",
            // Identifier bindings go here
            Arguments: {0: 20, 1: 30, length: 2},
        },
        outer: <GlobalLexicalEnvironment>,
        ThisBinding: <Global Object or undefined>,
    },

    VariableEnvironment: {
        EnvironmentRecord: {
            Type: "Declarative",
            // Identifier bindings go here
            g: undefined
        },
        outer: <GlobalLexicalEnvironment>,
        ThisBinding: <Global Object or undefined>
    }
}
```

之后，函数执行上下文经历执行阶段，这意味着函数内部变量的赋值已经完成。所以该函数执行上下文在执行阶段看起来是这样的：

```javascript
FunctionExectionContext = {
    LexicalEnvironment: {
        EnvironmentRecord: {
            Type: "Declarative",
            // Identifier bindings go here
            Arguments: {0: 20, 1: 30, length: 2},
        },
        outer: <GlobalLexicalEnvironment>,
        ThisBinding: <Global Object or undefined>,
    },

    VariableEnvironment: {
        EnvironmentRecord: {
            Type: "Declarative",
            // Identifier bindings go here
            g: 20
        },
        outer: <GlobalLexicalEnvironment>,
        ThisBinding: <Global Object or undefined>
    }
}
```

函数执行完成后，返回值存储在变量 `c` 中。所以全局执行上下文被更新。之后，全部代码执行完成，程序结束。

注意：你可能已经注意到了，在创建阶段 `let` 和 `const` 变量为 `uninitialized`，而 `var`变量为 `undefined`。

这是因为在创建阶段，代码被扫描为变量和函数声明，而函数声明被完整的存储在环境中，变量最初被设置为 `undefined`（ `var` ）或者保持 `uninitiated`（ `let` 和 `const` ）。

这就是为什么你可以在声明前访问 `var` 定义的变量（尽管是 `undefined` ），但是在声明前访问 `let` 和 `const` 变量时会出现引用错误的原因。 

这就是我们所说的变量“提升”。 

注意：在执行阶段，如果 JavaScript 引擎无法在源代码中声明的实际位置找到 `let` 变量的值，那么该变量将会赋值为`undefined` 。

## 总结 

所以，我们已经讨论了 JavaScript 程序是如何在内部执行的。虽然没有必要为了成为一名优秀的 JavaScript 开发人员而学习所有这些概念，但是对上述概念有适当的了解将会帮助你更容易并深入的理解其他概念，比如：提升、作用域和闭包。 

就这样吧，如果你觉得这篇文章对你有帮助，请点赞 👏并在下面随意留言，我非常乐意与你交流😃。