# 🚀⚙️JavaScript Visualized: the JavaScript Engine
# 翻译：🚀⚙️可视化的 JavaScript：JavaScript 引擎

点击访问：[原文地址](https://dev.to/lydiahallie/javascript-visualized-the-javascript-engine-4cdf)

作者：[Lydia Hallie](@lydiahallie)

![GIF 00](./illustrations/JSVisual04Engine/gif00.png)

JavaScript 是一门非常酷的语言，但是我们有没有想过机器是如何真正理解我们编写的代码的？作为 JavaScript 开发人员，我们通常不需要自己处理编译器（ compiler）。然而如果可以了解 JavaScript 引擎的基础知识，这将绝对是一件好事🥳。我们可以看看引擎是如何处理 JS 代码，并将代码变成机器可以理解的内容的。

>提示：这篇文章介绍的内容是基于 V8 引擎，该引擎被用于 Node.js 和以 Chromiume 为内核的浏览器。

HTML 解析器在遇到 `script` 标签时，这个标签中的源代码会从网络（ network ）、缓存（ cache ）或者已经安装的 Service Worker 中加载。脚本代码以字节流的形式被响应，会由字节流解码器（ byte stream decoder ）来处理。字节流解码器对正在下载的字节流进行解码。

![GIF 01](./illustrations/JSVisual04Engine/gif01.gif)

字节流解码器从解码的字节中创建出 Token 。比如：
* `0066` 解码为 `f` 
* `0075` 解码为 `u` 
* `006e` 解码为 `n` 
* `0063` 解码为 `c` 
* `0074` 解码为 `t` 
* `0069` 解码为 `i` 
* `006f` 解码为 `o` 
* `006e` 解码为 `n` 

这就是 `function` ，JavaScript 中它是一个保留关键字（ keyword ），这时候一个 Token 就创建好了，然后它会被发送到解析器（ parser ）。其他字节流也像这样处理（这里没有提及预编译器（ pre-parser ），我们会在后面的内容中介绍）。

![GIF 02](./illustrations/JSVisual04Engine/gif02.gif)

JavaScript 引擎使用两个解析器：预解析器（ pre-parser ）和解析器（ parser ）。为了减少网站加载的时间，引擎试图避免解析那些不需要马上运行的代码。预解析器（ pre-parser ）处理以后可能会用到的代码；解析器（ parser ）处理需要立即使用的代码。

*译注：预解析器（ pre-parser ）会先检查代码是否符合语法规则，如果不符合会直接抛出错误。这个提前检查的机制可以提高解析器（ parser ）的效率。

如果某个函数只在用户点击按钮后才会被调用，那么就没有必要在网站加载时立即编译这个函数。如果用户最终点击了按钮，这段函数代码就会被发送到解析器中。

解析器会根据传过来的 Token 数组创建节点（ node ）。通过这些节点，会创建出**抽象语法树（ Abstract Syntax Tree AST ）🌳**。

*译注：创建 AST 的同时，也生成执行上下文。

![GIF 03](./illustrations/JSVisual04Engine/gif03.gif)

接下来，就该解释器（ interpreter ）上场了。解释器遍历 AST ，并根据 AST 中包含的信息生成字节码（ byte code ）。字节码生成后，AST 会被删除以节省内存空间。最终我们得到了机器可以运行的东西🎉。

![GIF 04](./illustrations/JSVisual04Engine/gif04.gif)

虽然字节码的速度很快，但还可以更快。当字节码运行的时候，信息就会被生成。引擎可以检测到某些行为是否经常发生，以及使用的数据类型。也许一个函数被调用了好几十次，这时候可以优化一下，让函数运行的速度更快🏃‍♀️。

字节码，连同生成的类型反馈信息，被发送到优化编译器中（ optimizing compiler ）。优化编译器接收字节码和类型反馈信息后，将生成高度优化的机器码（ machine code ）🚀。

![GIF 05](./illustrations/JSVisual04Engine/gif05.gif)

JavaScript 是一门动态类型语言，这意味着数据类型可以不断地变化。如果 JavaScript 引擎每次都要检查某个值的数据类型，那就太慢了。

为了减少解释代码的时间，优化的机器代码只处理引擎在运行字节码时曾经遇到的情况。

如果我们反复运行某段代码，反复返回相同的数据类型，那么优化后的机器代码可以简单地重复使用该数据类型，以加快速度。

然而，由于 JavaScript 的数据类型是动态的，可能会发生同一段代码突然返回不同的数据类型的情况。如果发生这种情况，机器代码就会执行反优化操作，引擎会退回到之前的操作：解释原先生成的字节码。