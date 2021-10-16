# Understanding Scope and Scope Chain in JavaScript
# 翻译：理解JavaScript中的作用域和作用域链

点击访问：[原文地址](https://blog.bitsrc.io/understanding-scope-and-scope-chain-in-javascript-f6637978cf53)

作者：[Carson](https://medium.com/@Sukhjinder)

作用域和作用域链是 JavaScript 语言和其他编程语言中的基本概念。然而这些概念让许多 JavaScript 开发新手们觉得困惑。这些概念是掌握 JavaScript 的关键。 

对这些概念的正确理解，将帮助你写出更好，更高效和简洁的代码。这将反过来帮助你成为一个更好的 JavaScript 开发人员。 

所以，这篇文章我将解释什么是作用域和作用域链，JavaScript 引擎如何执行变量查询，以及这些概念的内部情况。 

话不多说，让我们开始吧。 

## 什么是作用域？

JavaScript 中的作用域指的是变量的可访问性或者可见性。也就是说程序的哪些部分可以访问到变量或者说变量在哪是可见的。

### 为什么作用域很重要？

1. 作用域主要的好处是安全。变量只能在程序中的某个区域中被访问。使用作用域，我们可以避免从程序中的其他位置对变量进行意外的修改。 

2. 作用域也可以减少命名空间冲突。我们可以在不同的作用域中使用相同的变量名称。

## 作用域的类型

在 JavaScript中 有三种类型的作用域 —— 1）全局作用域，2）函数作用域，3）块作用域

### 全局作用域

任何不在函数或者块（一对大括号）中的变量，都在全局作用域内。全局作用域内的变量可以被程序中的任何地方访问。比如:

```javascript
var greeting = 'Hello World!'; 

function greet() { 
    console.log(greeting); 
}

// Prints 'Hello World!' 
greet(); 
```
（译注：只存在于块（一对大括号）中的 var 变量也属于全局作用域）

### 函数作用域

在函数内声明的变量是在函数作用域中。他们只能被该函数内部访问，这意味着它们不能被函数外部的代码访问。比如：

```javascript
function greet() { 
    var greeting = 'Hello World!'; 
    console.log(greeting); 
}

// Prints 'Hello World!' 
greet(); 

// Uncaught ReferenceError: greeting is not defined 
console.log(greeting); 
```

### 块作用域

ES6 中引入了 `let` 和 `const` 变量，不同于 `var` 变量，它们的作用域是最近的一对大括号。这意味着他们不能被这对大括号外面的代码访问。比如：

```javascript
{ 
    let greeting = 'Hello World!'; 
    var lang = 'English'; 
    console.log(greeting); // Prints 'Hello World!' 
} 

// Prints 'English' 
console.log(lang); 

// Uncaught ReferenceError: greeting is not defined 
console.log(greeting); 
```

我们看到 `var` 变量可以被块外面的代码访问，因为 `var` 变量不属于块作用域。

## 作用域的嵌套

就像 JavaScript 中的函数一样，一个作用域也可以被嵌套到另外一个作用域中。比如：

```javascript
var name = 'Peter'; 

function greet() { 
    var greeting = 'Hello'; 
    { 
        let lang = 'English'; 
        console.log(`${lang}: ${greeting} ${name}`); 
    } 
} 

greet(); 
```

这里有三个作用域彼此嵌套。首先，最里面块作用域（因为有 `let` 变量）嵌套在函数作用域中，其次，函数作用域嵌套在全局作用域中。

## 词法作用域（ Lexical Scope ）

词法作用域（也被称为静态作用域）字面上看是指在词法化（编译）时而不是在运行时确定的作用域。比如：

```javascript
let number = 42; 

function printNumber() { 
    console.log(number); 
} 

function log() { 
    let number = 54; 
    printNumber(); 
} 

// Prints 42 
log(); 
```
不管从哪里调用 `printNumber()` 函数，这里的 `console.log(number)` 将会输出 `42` 。这跟动态作用域的语言不一样，在动态作用域中，`console.log(number)` 将会根据函数 `printNumber()` 的调用位置来确定 `number` 的值。 

如果上面的代码是用一种支持动态作用域的语言来编写，那么 `console.log(number)` 将会输出 `54` 。 

使用词法作用域，我们可以通过查看源代码来确定变量的作用域。然而，在动态作用域的示例中，在代码执行之前，作用域是无法确定的。 

大多数语言支持词法作用域（或者叫做静态作用域），比如 C、C++、Java 、 JavaScript 等，Perl 同时支持静态和动态作用域。

### 作用域链

当在 JavaScript 中使用一个变量时，JavaScript 引擎将会尝试在当前的作用域中查询该变量的值。如果查找不到，它就会在外部作用域中继续查找，直到查找到该变量或者到达全局作用域。 

如果引擎在全局作用域中仍然查找不到该变量，它将会在全局作用域中隐性地声明这个变量（非 strict mode 下），或者返回一个错误。 

比如：

```javascript
let foo = 'foo'; 

function bar() { 
    let baz = 'baz';   

    // Prints 'baz' 
    console.log(baz);   

    // Prints 'foo' 
    console.log(foo);   

    number = 42; 
    console.log(number);  // Prints 42 
} 

bar(); 
```

当函数 `bar()` 被执行时，JavaScript 引擎在当前作用域中查找变量 `baz` 并且找到了。接下来，它在当前变量中查找变量 `foo` ，但是并没有找到，所以他在外面一层的作用域（示例中是在全局作用域）中继续查找，并且找到了。 

之后，我们将 `42` 赋给了 `number` ，所以 JavaScript 引擎在当前作用域中查找 `number` 变量，然后在外层作用域中继续查找。 

如果代码不是在 strict mode 下，引擎将会创建一个名为 `number` 的新变量，并且将 `42` 赋给它；如果是在 strict mode 下，会返回一个错误。 

所以，当变量被使用时，引擎将会遍历整个作用域链，直到找到该变量。

## 作用域和作用域链如何工作？ 

到目前为止，我们讨论了什么是作用域和作用域的类型。现在我们来了解一下 JavaScript 引擎是如何确定变量的作用域，并且在引擎盖下是如何进行变量查找的。 

为了理解 JavaScript 引擎是如何进行变量查找的，我们必须先了解 JavaScript 中的词法环境概念。

### 什么是词法环境？

词法环境是一种结构，它保存标识符-变量的映射。（这里标识符指的是变量/函数的名称，变量指的是实际对象[包括函数对象和数组对象]或者原始值的引用）。 

简单地说，词法环境是一个存储变量和对象引用的地方。 

注意：不要混淆词法作用域和词法环境，词法作用域是在编译时确定的作用域，词法环境是在程序执行过程中变量存储的地方。 

从概念上讲，词法环境看起来像这样：

```javascript
lexicalEnvironment = {
    a: 25,
    obj: <ref. to the object>
}
```

一个新的词法环境是为每个词法作用域创建的，但是只有当该作用域中的代码被执行时才会被创建。词法环境也有一个对外部词法环境（即外部作用域）的引用。比如：

```javascript
lexicalEnvironment = { 
    a: 25, 
    obj: <ref. to the object>   

    outer: <outer lexical environemt> 
} 
```

### JavaScript引擎如何执行变量查找？

现在我们知道了什么是作用域，作用域链和词法环境，现在让我们来理解一下 JavaScript 引擎是如何使用词法环境来确定作用域和作用域链的。 

让我们通过下面这段代码示例来理解一下上述概念。

```javascript
let greeting = 'Hello'; 

function greet() { 
    let name = 'Peter'; 
    console.log(`${greeting} ${name}`); 
} 

greet(); 

{ 
    let greeting = 'Hello World!' 
    console.log(greeting); 
}
```

当上述代码被加载时，全局词法环境被创建，其中包含全局作用域内定义的变量和函数。比如：

```javascript
globalLexicalEnvironment = { 
    greeting: 'Hello' 
    greet: <ref. to greet function>   

    outer: <null> 
}
```

之后，遇到了对函数 `greet()` 的调用。因此为 `greet()` 函数创建一个新的的词法环境。比如：

```javascript
functionLexicalEnvironment = {
    name: 'Peter'
    outer: <globalLexicalEnvironment>
}
```

这里，外部词法环境 `outer` 被设置为 `globalLexicalEnvironment`，因为其外部作用域为全局作用域。 

之后，JavaScript 引擎执行语句 `console.log(``${greeting} ${name}``)` 。 

JavaScript 引擎尝试在函数词法环境中找到变量 `greeting` 和 `name` 。它在当前此法环境中找到了变量 `name` ，但是无法在当前的词法环境中找到变量 `greeting` 。 

所以，引擎在外部词法环境（由 `outer` 属性定义，示例中为全局词法环境）中找到变量 `greeting` 。 

接下来，JavaScript 引擎执行块中的代码。所以它为块创建一个新的词法环境。比如：

```javascript
blockLexicalEnvironment = { 
    greeting: 'Hello World',   

    outer: <globalLexicalEnvironment> 
}
```

然后，`console.log(greeting)` 语句被执行。JavaScript 引擎在当前的词法环境中找到该变量，并且使用该变量。所以引擎不会在外部词法环境（全局词法环境）中查找该变量。 

注意：只会为 `let` 和 `const` 声明创建一个新的词法环境，而不会为 `var` 声明创建。`var` 声明被添加到当前的词法环境中（全局或者函数词法环境）。 

所以，当一个程序中使用变量的时候，JavaScript 引擎将会尝试在当前的词法环境中查找该变量，如果找不到，它会继续在外部的词法环境中查找。所以，这就是 JavaScript 引起执行变量查找的方式。

## 总结

简单地说，作用域是一个变量可见和可访问的区域。就像函数一样，JavaScript 中的作用域也可以嵌套了，而且 JavaScript 引擎会遍历作用域链来查找程序中使用的变量。 

JavaScript 使用词法环境，意味着变量的作用域是在编译时确定的。在程序执行过程中，JavaScript 引擎使用词法环境来存储变量。 

作用域和作用域链是 JavaScript 的基本概念，每个 JavaScript 开发人员都应该理解。对这些概念有很好的理解会帮助你成为一个更有效率和更好的 JavaScript 开发人员。 

如果你觉得这篇文章对你有帮助，请点赞👏并在下面随意留言，我非常乐意与你交流。 