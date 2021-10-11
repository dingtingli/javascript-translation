# ⭐️🎀JavaScript Visualized: Promises & Async/Await
# 翻译：⭐️🎀可视化的 JavaScript：Promises & Async/Await

点击访问：[原文地址](https://dev.to/lydiahallie/javascript-visualized-promises-async-await-5gke)

作者：[Lydia Hallie](@lydiahallie)

![GIF 00](./illustrations/JSVisual07Promises/gif00.png)

你是否遇到过 JavaScript 代码没有安装你预期的方式来运行？看起来就好像是函数在随机的，不可预测的时间被执行，或者被延迟执行。这时候，你可能是在处理 ES6 中的新特性： Promises 。

多年之前的好奇心终于有了回报，在又一次的不眠之夜中我终于有时间做了一些动画，来聊聊 Promises ：为什么我们要使用它，它在底层是如何工作的，以及我们如何以现代的方式来编写它。

> 如果你还没有看过我之前写的一篇关于 JavaScript 事件循环的文章，那么我建议你先读一下。因为如果你对事件循环中的 Call Stack（调用栈）、Web API 和 Queue（队列）有了解，会对这篇文章的理解帮助。当然，这次我们还会介绍一些令人兴奋的其他功能。
<br/>
[✨♻️可视化的 JavaScript：事件循环](doc/可视化的JavaScript：事件循环.md)

如果你对 Promises 有些熟悉，这里有导航，可以跳过你熟悉的部分，以节省时间。

🥳 介绍

⚡️ Promise 语法

♻️ 事件循环: 微任务（ Microtasks ）和宏任务（ Macrotasks ）

🚀 Async / Await

---

🥳 介绍

当我们在编写 JavaScript 代码时，经常会需要处理那些依赖于其他任务的任务。比如，我们需要先获取一个图片，然后再调整大小，应用滤镜，最后保存📸。

首先，我们需要做的是获取想要编辑的图片。`getImage` 函数可以帮助我们获取图片。一旦我们成功加载图片，就需要把图片传入 `resizeImage` 函数。当图片被重新调整大小后，再使用 `applyFiler` 函数为图片添加一个滤镜。最后，我们希望保持图片，并通知用户工作一切正常🥳。

我们最后得到下面这样的代码：

```javascript
getImage('./image.png', (image, err) => {
    if (err) throw new Error(err)
    resizeImage(image, (resizeImage, err) => {
        if (err) throw new Error(err)
        applyFilter(resizeImage, (filteredImage, err) => {
            if (err) throw new Error(err)
            saveImage(filteredImage, (res, err) => {
                if (err) throw new Error(err)
                console.log("Successfully saved image!")
            })
        })
    })
})
```

注意到了吗？虽然结果正常，但代码并不友好。我们使用了许多嵌套的回调函数，它们都依赖于前一个回调的函数。这种方式通常被称为回调地狱（ [callback hell](http://callbackhell.com/) ）。大量地嵌套回调函数，使得代码很难阅读。

幸运的是，我们可以使用 Promise 来解决这个问题！让我们来看看 Promise 是什么，以及它是如何帮助我们解决上面问题的😃。

---

⚡️ Promise 语法

ES6 引入了 Pormise 。在许多教程中，我们可能会看到这样的描述。

> A promise is a placeholder for a vaule that can either resolve or reject at some time in the future.

> Promise 是一个值的占位符，可以在未来某个时刻  resolve 或者 reject 。

额……这样的解释并没有让我们对 Promise 有更加清晰的理解。相反，它只会让我们觉得 Promise 很神秘，不可预测，像是一种魔法。因此，接下来让我们看看 Promise 到底是什么？

让我们先来创建一个 Promise 。Promise 构造函数接受一个回调函数作为参数，让我们试一下：

```javascript
new Promise(() => {})
```

![GIF 01](./illustrations/JSVisual07Promises/gif01.gif)

这……返回的是什么？

Promise是一个对象，它包括一个 status（`[[PromiseStatus]]`）和一个 value （`[[PromiseValue]]`）。在上面的例子中，我们可以看到 `[[PromiseStatus]]` 的值是 `pending` ，`[[PromiseValue]]` 的值是 `undefined` 。

别担心，你永远不会和这个对象打交道，你甚至都不能访问 `[[PromiseStatus]]` 和 `[[PromiseValue]]` 这两个属性。然而，在使用 Promise 时，这些属性的值非常重要。

`PromiseStatus` 的值是 Pormise 的状态（status），它可以是下面三个值之一：

* ✅fulfilled：表示 Promise 已经被 resolved 。一切正常，在这个 Promise 内没有错误发生🥳。

* ❌rejected：表示 Promise 已经被 rejected 。说明出错了。

* ⏳pending：表示 Promise 既没有被 resolved ，也没有被rejected ，仍然处于 pending 状态。

这一切听起来很棒。但什么时候一个 Promise 的状态是 pending ， rejected 或者 fulfilled ？为什么这种状态很重要？

