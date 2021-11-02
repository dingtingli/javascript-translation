# Hoisting in Modern JavaScript — let, const, and var
# 翻译：现代 JavaScript 中的提升 —— let ，const 和 var


点击访问：[原文地址](https://blog.bitsrc.io/a-beginners-guide-to-closures-in-javascript-97d372284dda)

作者：[Carson](https://medium.com/@Sukhjinder)

许多 JavaScript 程序员将提升（hoisting）解释为 JavaScript 将声明（变量和函数）移动到当前作用域（函数或者全局）顶端的行为。就好像它们被物理地移动到你的代码顶端一样，但这不是事实。 比如：

```javascript
console.log(a);
var a= 'Hello World!';
```

他们会说，上面的代码在提升之后会转化为这样的代码： 

```javascript
var a; 
console.log(a); 
a = 'Hello World!'; 
```

虽然看起来第一段代码好像发生了什么（因为代码运行结果跟第二段一样），但实际上并没有发生，你的代码没有去到任何地方。JavaScript 引擎并没有物理移动你的代码，它们还待在你输入的地方。

## 那么，什么是提升？

在编译阶段，就是代码被执行前的几微秒，它被扫描为函数和变量的声明。所有这些函数和变量的声明都会被添加到内存中一个叫做词法环境的 JavaScript 数据结构中。所以这些函数和变量可以在实际声明之前就被使用。

### 什么是词法环境？

词法环境是一个数据结构，它保持标识符和变量的映射（这里，标识符指的是变量/函数的名称，变量指的是对实际对象[包括函数对象]或者原始值的引用）。 

这就是词法环境在概念上的样子：

```javascript
LexicalEnvironment = { 
    Identifier: <value>, 
    Identifier: <function object> 
} 
```

所以简而言之，词法环境是程序执行是变量和函数所在的地方。如果你希望了解更多关于词法环境的内容，可以阅读我[之前的文章](./理解JavaScript01：执行上下文和调用栈.md)。 

现在我们知道什么是提升，让我们来看看函数和变量（var 、 let 和 const）的声明是如何提升的。 

### 提升函数声明

```javascript
helloWorld();  // prints 'Hello World!' to the console

function helloWorld() {
    console.log('Hello World!');
}
```

正如我们已经知道的那样，函数声明在编译阶段已经添加到内存中了，所以我们可以在实际函数声明的位置之前访问它。 

因此，上面代码的词法环境看起来像这样：

```javascript
lexicalEnvironment = {
    helloWorld: < func >
}
```

所以当 JavaScript 引擎遇到 `helloWorld` 函数调用的时候，它将在词法环境中查找，找到该函数后执行它。

### 提升函数表达式 

在 JavaScript 中，只有函数声明被提升，**函数表达式不会被提升**。比如下面这段代码示例就不会正常运行：

```javascript
helloWorld();  // TypeError: helloWorld is not a function

var helloWorld = function() {
    console.log('Hello World!');
}
```

因为 JavaScript 只提升声明，不提升初始化（赋值），所以 `helloWorld` 将会被视为一个变量，而不是函数。因为 `helloWorld` 是一个 `var` 变量，所以引擎在提升过程中将会分配给它一个 `undefined`。 

所以下面这段代码将会正常运行：

```javascript
var helloWorld = function() { 
    console.log('Hello World!');  // prints 'Hello World!' 
} 

helloWorld(); 
```

### 提升var变量

让我们来看几个示例理解一下 `var` 变量的提升。

```javascript
console.log(a); // outputs 'undefined'
var a = 3;
```

我们期望输出的是 `3` ，但得到的是 `undefined`。为什么？ 

记住，JavaScript 只提升声明（declaration），而不提升初始化（initialization）。也就是说，在编译的时候，JavaScript 引擎只在内存中存储函数和变量的声明，而不是他们的赋值。 

但是，为什么是 `undefined`？ 

在编译阶段，当 JavaScript 引擎发现 `var` 变量声明的时候，它将该变量添加到词法环境中，并将其初始化为 `undefined`。在执行阶段，当到达代码中实际赋值的那一行的时候，它会将值赋给该变量。 

因此，上述代码的初始词法环境看起来像这样：

```javascript
lexicalEnvironment = { 
    a: undefined 
} 
```

这就是为什么我们得到的是 `undefined` 而不是 `3`。当引擎到达实际赋值那一行的时候（执行阶段），它将会更新词法环境中变量的值。因此，赋值后词法环境将会是这样：

```javascript
lexicalEnvironment = { 
    a: 3 
} 
```

### 提升 let 和 const 变量

让我们先来看一段代码示例： 

```javascript
console.log(a); 
let a = 3; 
```

输出：

`ReferenceError: a is not defined`

所以 `let` 和 `const` 变量没有提升吗？

这个问题的答案比之前的复杂。**JavaScript 中所有的声明（函数， var ， let ， const 和 class ）都被提升，而 `var` 声明被初始化为 `undefined`，但是 `let` 和 `const` 声明没有被初始化。** 

`let` 和 `const` 声明只有在运行期间被 JavaScript 引擎评估词法绑定（赋值）时才会被初始化。这意味着在此之前，你不能访问该变量。这就是我们所说的 “暂时性死区” （Temporal Dead Zone：TDZ），在变量创建和初始化之间的时间段，不能被访问。 

如果 JavaScript 引擎在声明 `let` 和 `const` 变量的那一行还是找不到对应的值，引擎就会给变量赋值为 `undefined` 或者返回一个错误（`const` 变量）。 

让我们再看一个代码示例：

```javascript
let a;
console.log(a); // outputs undefined
a = 5;
```

在编译阶段，JavaScript 引擎遇到了变量 `a`，并将其存储到词法环境中，但是由于它是 `let` 变量，引擎并没有用任何值来初始化该变量。所以在编辑阶段，词法环境看起来像这样：

```javascript
lexicalEnvironment = {
    a: <uninitialized>
}
```

如果你试图在变量声明之前访问它，JavaScript 引擎会试图从词法环境中获取该变量的值，而该变量未被初始化，所以会抛出引用错误（reference error）。 

在执行阶段，当引擎到达变量声明的那一行时，它将试图评估绑定（值），因为该变量没有赋值，所以它将会被赋值为 `undefined`。 

执行完第一行代码后，词法环境看起来像是这样：

```javascript
lexicalEnvironment = {
    a: <undefined>
}
```

随后，`undefined` 会被输出，之后 `5` 会被赋值给变量 `a`，词法环境中 `a` 的值从 `undeined` 变成了 `5`。

```javascript
lexicalEnvironment = {
    a: 5
}
```

注意：只要代码在变量声明之前还没有被执行，我们是可以在 `let` 和 `const` 变量声明之前的代码中引用相应变量。 

比如，下面这段代码示例是完全有效的：

```javascript
function foo () {
    console.log(a);
}

let a = 20;
foo();  // This is perfectly valid
```

但是，下面这段代码将会产生引用错误：

```javascript
function foo() {
    console.log(a); // ReferenceError: a is not defined
}

foo(); // This is not valid
let a = 20;
```

### 提升class声明

就像 `let` 和 `const` 声明一样，JavaScript 中的 `class` 也会被提升，并且 `class` 也会在解释之前保持未初始化状态。所以 `class` 也会受到“暂时性死区”的影响。比如下面这段代码： 

```javascript
let peter = new Person('Peter', 25); // ReferenceError: Person is  not defined
console.log(peter);

class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
}
```

所以如果要访问 `class`，你必须先声明。比如：

```javascript
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
}

let peter = new Person('Peter', 25); 
console.log(peter);
// Person { name: 'Peter', age: 25 }
```

在编译阶段，上面代码的词法环境看起来像这样： 

```javascript
lexicalEnvironment = {
    Person: <uninitialized>
}
```

当引擎评估（译注？执行）到该 `class` 语句后，它将用值初始化该 `class`。 

```javascript
lexicalEnvironment = {
    Person: <Person object>
}
```

### 提升 Class 表达式

就像函数表达式一样，`class` 表达式也不能被提升。比如，下面这段代码不能正常工作:

```javascript
let peter = new Person('Peter', 25); // ReferenceError: Person is not defined
console.log(peter);

let Person = class {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
}
```

正确的方式应该像下面这段代码： 

```javascript
let Person = class {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
}

let peter = new Person('Peter', 25); 
console.log(peter);
// Person { name: 'Peter', age: 25 }
```

## 总结

所以我们现在知道，我们的代码在提升过程中并没有被 JavaScript 引擎物理移动过。对提升机制的正确理解，将会帮助我们避免由于提升导致的 bug 和困惑。为了避免提升带来的副作用，比如 `undefined` 变量或者引用错误，总是将变量的声明放在各自作用域的顶端，并且总是在变量声明是对其初始化。

就这样吧，如果你觉得这篇文章对你有帮助，请点赞👏并在下面随意留言，我非常乐意与你交流。