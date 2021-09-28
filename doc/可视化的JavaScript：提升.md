# 🔥🕺JavaScript Visualized: Hoisting
# 翻译：🔥🕺可视化的 JavaScript：提升

点击访问：[原文地址](https://dev.to/lydiahallie/javascript-visualized-hoisting-478h)

作者：[Lydia Hallie](@lydiahallie)

![GIF 00](./illustrations/JSVisual02Hoist/gif01.png)

提升（ Hoisting ）是每个 JavaScript 开发人员都听说过的一个术语，因为你在 google 一个麻烦的错误时，结果会在 StackOverflow 上有人告诉你这个错误是由于提升（ Hoisting ）造成的🙃。那么，什么是提升（ Hoisting ）？（ FYI ——作用域会在另外一篇文章中介绍）

如果你刚学习 JavaScript ，你会有一些“奇怪”的经历，一些变量会随机地抛出 `undefined` 、ReferenceErrors 等等错误。提升（ Hoisting ）通常被解释为将变量和函数放到文件的顶部，虽然看起来很像是这样，但其实并不是😀。

当 JavaScript 引擎获取到我们的脚本代码，首先是为我们的代码设置内存（ Setting Memory）。