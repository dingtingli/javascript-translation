# Ignition: V8 Interpreter
# Ignition: V8 解释器

点击访问：[原文地址](https://docs.google.com/document/d/11T2CRex9hXxoJwbYqVQ32yIPMh0uouUZLdyrtmMoL44/mobilebasic)

作者：[rmcilroy@, oth@]

## 背景

V8 的 Full-Codegen 编译器生成的机器码是冗长的，因此对于普通的网页来说，这些机器码明显增加了 V8 Heap 所使用的内存空间（之前的研究表明，代码空间占 JS Heap 内存空间的 15%-20% 左右）。

除了造成内存压力之外，这也意味着 V8 非常努力地避免生成那些它认为可能不会执行的代码。为此 V8 实现了惰性解析和编译，通常函数只在第一次运行时被编译。这种做法在网页启动时有很大的代价，因为在惰性编译时需要重新解析函数的源代码（比如：crbug.com/593477）。

Ignition 的目的是为 V8 建立一个解释器，该解释器可以执行低级别字节码，以便让那些运行一次的代码或者非热点代码能够以字节码的形式压缩存储。因为字节码更小，所以编译时间大大缩减，我们也能在初始编译时更激进，从而明显地提高启动速度。还有一个额外的好处是，字节码可以直接传给 Turbofan 图生成器，从而避免了在 Turbofan 中优化一个函数时重新解析 JavaScript 源代码。

Ignition 项目的目标：

* 将代码空间减少至当前的 50% 左右。

* 与 Full-Codegen 相比有合理的性能（在真实的网站上，Ignition 会变慢一些；在一些峰值基线测试，比如 Octane 中，大约慢 2 倍）。

    * 注意：由于热代码会被 Crankshaft 或者 TurboFan 优化，所以整体的速度不会明显下降，希望可以忽略不计。

    <br/>
* 完全支持 DevTools 调试和 CPU 性能分析。

* 替代 Full-Codegen 成为第一级编译。

    * 在 CrankShaft 被完全弃用之前，我们无法完全取代 Full-Codegen，因为 CrankShaft 不能反优化 Ignition，它只能反优化 Full-Codegen 代码。

    <br/>
* 成为 Turbofan 编译器新的前端，可以在不重新解析 JS 源码的情况下进行优化在编译。

* 支持从 TurboFan 到解释器的反优化。

<br/>
明确不是 Ignition 项目目标的是（至少现阶段不是）：

* 支持其他允许 JIT 代码的平台（比如，iOS）。

    * JIT 代码的生成仍然需要 IC 和 Code Stub。

    <br/>
* 支持 V8 执行非 JavaScript 代码（比如，WASM）。

* 与 Full-Codegen 具有相同的性能

* 完全替代 Full-Codegen 编译器。

    * 如上所说，我们需要 Full-Codegen 作为 CrankShaft 的反优化目标，并且为 CrankShaft 生成 Ignition 所无法提供的类型反馈。因此，Full-Codegen 将成为 CrankShaft 优化的热代码的中间层（任何由解析器决定需要被 TurboFan 优化的函数，将不会通过 Full-Codegen 编译，而是由 TurboFan 直接优化）。

    
## 整体设计

这小节将概述 Ignition 字节码解释器的整体设计，下面几节将详细介绍具体细节。

解释器本身由一组字节码处理程序所组成，每个处理程序处理一个特定的字节码，并将其分配到下一个字节码的处理程序。这些处理程序使用高层的，与机器架构无关的汇编代码写成，由 `CodeStubAssembler` 类实现，并由 Turbofan 编译。

因此，解释器可以一次编写，然后使用 TurboFan 为 V8 支持的每个架构生成相应的机器指令。当解释器启动时，每个 V8 isolate（隔离区） 都包含一个全局解释器调度表，由字节码值作为索引，包含每一个字节码处理程序的代码对象指针。这些字节码处理程序可以包含在启动快照中，并且在一个新的 isolate 创建时被反序列化。

函数为了被解释器运行，在首次未优化编译阶段，被 `BytecodeGenerator` 转换成字节码。`BytecodeGenerator` 是一个 `AstVisitor`，它遍历函数的 AST，为每个 AST 节点生成合适的字节码。这个字节码作为 `SharedFunctionInfo` 对象的一个字段与该函数相关联，并且该函数的代码入口地址被设置为 `InterpreterEntryTrampoline` Stub 中。

当函数在运行时被调用时，也就进入了 `InterpreterEntryTrampoline` Stub 中。这个 Stub 会初始化合适的栈桢，然后将函数的第一个字节码分派给解释器的字节码处理程序，以便在解释器中开始执行该函数。每个字节码处理程序的结尾都会根据下一个字节码，转到下一个字节码处理程序（通过全局解释器调度表中字节码索引进行转换）。

Ignition 是一个基于寄存器的解释器。这些寄存器不是传统意义上的机器寄存器，而是寄存器文件里特定 slot，这个寄存器文件是分配在函数栈帧中的。字节码可以通过字节码参数指定它们所操作的输入和输出寄存器，这些字节码参数在 `BygecodeArray` 流中紧紧跟着字节码本身。

为了减少字节码流的大小，Ignition 有一个累加寄存器，它被许多字节码作为隐式的输入和输出寄存器。这个寄存器不是栈中寄存文件的一部分，而是由 Ignition 维护在机器寄存器中。这将会最大限度地减少重复加载和读取内存中的寄存器的操作。并且这也减少了字节码的大小，因为许多指令可以不需要显式的指定输入和输出的寄存器。比如二元操作字节码，只需要用一个操作数来指定其中的输入，其他的输入和输出寄存器被隐式地包含在累加寄存器中，而不是显式地指定所需的三个寄存器。

## 字节码处理程序的生成

字节码处理程序由 TurboFan 编译器生成。每个处理程序都有它自己的代码对象，是独立生成的。 处理程序通过 `InterpreterAssembler` 被写成 TurboFan 可操作的 Graph，其中具有解释器需要的一些额外的高层 primitive（比如, Dispatch，GetBytecodeOperand 等）。`InterpreterAssembler` 是 `CodeStubAssembler` 的一个子类。

比如， Ldar 字节码处理程序的生成器函数如下：

Ldar(Load Accumlator from Register, 将寄存器加载到累加器)

```c++
void Interpreter::DoLdar(InterpreterAssembler* assembler) {
   Node* reg_index = __ BytecodeOperandReg(0);
   Node* value = __ LoadRegister(reg_index);
   __ SetAccumulator(value);
   __ Dispatch();
 }
```

字节码处理程序不是被直接调用的，而是被每个字节码处理程序分配到下一个字节码。在 TurboFan 中，字节码分配是尾部调用操作。解释器加载下一个字节码，索引到调度表以获得目标字节码处理程序的代码对象，然后尾部调用该代码对象以分配下一个字节码处理程序。

解释器还需要在固定的机器寄存器中维护跨字节码处理程序的状态。比如，指向 `BytecodeArray` 的指针，当前字节码的偏移量和累加器的值。这些值被 TurboFan 视为参数，作为前一个字节码的输入被接收，并被当作字节码尾部调用的参数传递给下一个字节码处理程序。字节码处理程序分配的调用惯例是为这些参数指定固定的机器寄存器，这使得它们可以通过解释器调度进行线程化，而不需要将它们压栈和弹栈。

字节码处理程序的 Graph 生成之后，它就会通过 TurboFan 管道的简化版本，并分配给解释器表中的相应条目。

