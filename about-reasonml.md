# 什么是 ReasonML ?

> 本章简要介绍了 Facebook 的新编程语言 [ReasonML](https://reasonml.github.io/)

## 什么是 ReasonML ?

ReasonML 是在 Facebook 创建的一种新的对象函数编程语言。实质上，它是编程语言 OCaml 的新 C 语言语法。新的语法旨在与JavaScript进行互操作，并且更容易被
JavaScript 程序员采用。此外，它消除了 OCaml 语法的特性。ReasonML 还支持 JSX（ Facebook 的 React 框架使用的 JavaScript 内部 HTML 模板的语法）。由于
ReasonML 基于 OCaml，许多人可以互换使用这两个名称。下图显示了 ReasonML 如何适合OCaml生态系统。

![这就是 ReasonML 如何适应OCaml生态系统。](4a1823ac61589e61eae453cfe9421d70809f2fba.svg)

目前，ReasonML 的默认编译目标是 JavaScript（浏览器和 Node.js ）。

这就是 ReasonML 代码的样子：

```ocaml

type color = Red | Green | Blue;

let stringOfColor(c) =>
    switch (c) {
        | Red => "Red"
        | Green => "Green"
        | Blue => "Blue"
    };

```

有几件事值得注意：

- 该语法的许多元素都是从JavaScript中借用的。例如：

    - 名称 switch（对应 OCaml 的 match）
    - 语法 (x) => ...用于函数
    - 分号

- 其他元素对于函数式编程语言来说是典型的。例如：

    - color 是 [变体类型](variants.md) 。
    - switch 执行模式匹配。

- 不需要类型注释（例如，stringOfColor 的参数 c 没有注释）。

## OCaml 的好处

作为 ReasonML 的基础 OCaml 带来以下好处：

- 这是一种成熟的语言（创建于1996年），已在许多项目中得到证明。
- Facebook本身在几个项目中使用它（例如Flow）。
- 其核心是具有全功能类型系统的函数式编程语言。但它也支持面向对象和可变状态。
- 它可以编译为字节码，快速的 native 代码或 JavaScript。
- 编译成JavaScript很快。引用博客文章 “ [Messenger.com 现在已经 50％ 转化为 Reason](https://reasonml.github.io/blog/2017/09/08/messenger-50-reason.html) ”：

  > 代码库的 Reason 部分的完全重建是〜2s（几百个文件），增量构建（标准）平均<100ms。BuckleScript 作者估计，构建系统在当前情况下应该扩展到几十万个文件。