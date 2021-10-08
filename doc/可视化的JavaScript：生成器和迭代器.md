# 💡🎁JavaScript Visualized: Generators and Iterators
# 翻译：💡🎁可视化的 JavaScript：生成器和迭代器

点击访问：[原文地址](https://dev.to/lydiahallie/javascript-visualized-generators-and-iterators-e36)

作者：[Lydia Hallie](@lydiahallie)

ES6 引入了一个很酷的东西，叫做生成器函数（ gengrator function ）🎉 。每当我问别人有关生成器函数的问题时，回答基本都是“我曾经看过一次，很困惑，就没有再看了”，“我读过很多关于生成器函数的文章，但还是不明白”，“我了解这个东西，但是为什么有人会用它”🤔 也许这只是我和自己的对话，因为我曾经很长一段时间都是这么想的！但是，生成器函数真的非常酷。

那么，什么是生成器函数（ gengrator function ）呢？让我们先来看一个常规的、老派函数👵🏼。

```javascript
function normalFunction(){
    console.log("I am the coolest function!");
    console.log("There is no way of stopping me!");
    console.log("Oh boi we're still going go");
    console.log("Okay finally all done now 🚀");
}
```

是的，这没有什么特别的地方！只是一个普通的函数，会输出 4 次。我们来调用一下它：

![GIF 01](./illustrations/JSVisual06Generator/gif01.gif)

“但是，为什么要我浪费5秒种时间看这么一个普通无聊的函数？”这是一个好问题。普通函数遵循着一种称为 运行-完成 （run to completoin）模式：当我们调用一个函数时，它会一直运行直到完成（除非某处报错）。我们不能随意地在中间某个地方暂停一个函数。

现在最酷的部分来了：生成器函数（ gengrator function ）并不遵循运行-完成 （run to completoin）模式🤯 。这是否意味着我们可以在生成器函数执行的过程中随意地暂停它？嗯，算是吧。让我们来看看什么是生成器函数，以及我们如何使用它。

我们通过在 `function` 关键字后面写一个 `*` 来创建生成器函数。

```javascript
function* generatorFunction(){

}
```

但是这并不是我们使用生成器函数所要做的一切。与普通函数相比，生成器函数实际上以完全不同的方式工作：

* 调用一个生成器函数会返回一个生成器对象（ generator object ），这个对象是一个迭代器（ iterator ）。
* 我们可以在生成器函数中使用 `yield` 关键字来“暂停”执行。

但这到底是什么意思？

让我们先来看看第一条：调用生成器函数（ gengrator function ）会返回一个生成器对象（ generator object ）。

当我们调用一个普通函数的时候，函数体会被执行，最终返回一个值。然而，当我们调用一个生成器函数时，返回一个生成器对象！让我们来看看把生成器对象输出后是个什么样子。

![GIF 02](./illustrations/JSVisual06Generator/gif02.gif)

现在，我可以听到你内心的尖叫（可能你已经叫出声了🙃）。这看起来有点让人望而生畏。但是别担心，我们并不真的要要使用你在这里看到的任何属性。那么，生成器对象到底有什么用呢？

