# ✨♻️JavaScript Visualized: Event Loop
# 翻译：✨♻️可视化的 JavaScript：事件循环

点击访问：[原文地址](https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif)

作者：[Lydia Hallie](@lydiahallie)

![GIF 00](./illustrations/JSVisual01EP/gif0.png)

事件循环( Event Loop )是每个 JavaScript 开发人员都必须学习的概念之一，但是这个概念一开始理解起来会有点让人困惑。我是一个可视化的学习者，所以我尝试着用 gif 动态图片来解释这个概念。

首先，什么是事件循环( Event Loop )？我们为什么要关注它？

JavaScript 是一门单线程语言：同一时间只能运行一个任务。通常情况下这没什么问题。但是现在想像一下，我们正在运行的一个任务需要 30 秒。 那么我们就需要等待 30 秒，然后再继续运行其他任务（ JavaScript 默认运行在浏览器的主线程上，所以整个页面的 UI 都会卡住）。我想没有人希望访问一个很慢，响应不及时的网站。

幸运的是浏览器提供一些 JavaScript 引擎自身不具备的特性，比如：Web API 。它包括 DOM API , `setTimeout` , HTTP 请求 等等。这帮助我们创建一些处理异步（ async ），非阻塞（ non-blocking ）行为的操作🚀。


当我们调用一个函数的时候，该函数就会被添加到一个叫做调用栈（ Call Stack ）的地方。调用栈是 JavaScript 引擎的一部分，而不是浏览器特有的。它本质上是一个栈（ Stack ）的数据结构，这意味着先进后出（ First In, Last Out FILO ）。当函数返回值后，就会从栈中弹出👋。

![GIF 01](./illustrations/JSVisual01EP/gif1.gif)

图中函数 `respond` 返回了一个 `setTimeout` 函数，这个 `setTimeout` 是由 Web API 提供的：它可以让我们在不阻塞主线程的情况下延迟执行一个任务。`setTimeout` 中的回调函数，就是那个箭头函数（Arrow Function） `()=>{ return 'Hey'}` 会被添加到 Web API 中。同时，`setTimeout` 函数和 `respond` 函数将从栈中弹出，因为它们都已经返回了相应的值。

![GIF 02](./illustrations/JSVisual01EP/gif2.gif)

在 Web API 中，一个计时器会运行 1000 毫秒——这是我们传过来的第二个参数的值。时间到了之后，回调函数（即箭头函数）不会立即被添加到调用栈，相反它会被传递到一个叫队列（ Queue ）的地方。

![GIF 03](./illustrations/JSVisual01EP/gif3.gif)

这可能会让人困惑：回调函数并不是在 1000 毫秒后被添加到调用栈，它只是在 1000 毫秒后被添加进了队列。但这是队列，函数进入后需要排队等待运行。

现在，我们一直期盼的部分要登场了——事件循环（ Event Loop ）。它唯一的任务就是：**连接队列和调用栈**。如果调用栈是空的，也就是说之前调用的函数都已经返回了相应的值，并且从栈中弹出，那么队列中的第一个函数就会被添加到调用栈。在上面的例子中，没有其他函数被调用，也就是说调用栈是空的，同时回调函数是队列中的第一个函数。

![GIF 04](./illustrations/JSVisual01EP/gif4.gif)

回调函数被添加到调用栈，被调用并返回值，然后从栈中弹出。

![GIF 05](./illustrations/JSVisual01EP/gif5.gif)

看上面的讲解你会觉得很有意思，感觉看懂了。但是想真正理解这些内容，我们需要一遍一遍的实践。试着想一想，下面这段代码会输出什么内容到控制台：

```javascript
const foo = () => console.log("First");
const bar = () => setTimeout(() => console.log("Second"), 500);
const baz = () => console.log("Third");

bar();
foo();
baz();
```

想到了吗？让我们快速看一下当我们在浏览器中运行这段代码时都发生了什么：

![GIF 06](./illustrations/JSVisual01EP/gif6.gif)

1. 我们调用了函数 `bar`。`bar` 返回了一个 `setTimeout` 函数。
2. 传给 `setTimeout` 的回调函数被添加到了 Web API 中， `setTimeout` 和 `bar` 函数从栈中弹出。
3. 计时器（Timer）开始运行，同时函数 `foo` 被调用，并输出字符串 `Frist` 。 `foo` 返回（undefined）， `baz` 继续被调用，然后回调函数被添加到队列。
4. `baz` 输出字符串 `Third` 。`baz` 返回之后，事件循环看到调用栈是空的，就将回调函数添加到调用栈中。
5. 回调函数输出字符串 `Second`。

希望这篇文章能让你对事件循环这个概念有些清晰的理解。如果还是有些困惑，也不要担心。重要的是能将问题分解，知道那些错误/行为来自哪个过程，以便能通过正确的关键字搜索出结果，最终进入正确的 Stack Overflow 页面💪。