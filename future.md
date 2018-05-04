# ReasonML 计划做什么？

本章简要介绍一些仍在研究中的 [ReasonML](https://reasonml.github.io/) 关键部分：

- 更好地支持编写与 JavaScript Promises 兼容的异步代码。
    - 一种选择是提供特殊的语法（请参阅 [GitHub上的问题](https://github.com/BuckleScript/bucklescript/issues/1326) ）。
    - 另一种选择是向 OCaml 的并发库 [Lwt](https://github.com/ocsigen/lwt) （它是其中一个分支）添加支持。

- 更好地支持多态。目前，不同类型表示 ReasonML 的不同功能或操作员名称（例如，有整数有 `+` 和 浮点数 `+.` ）。Haskell 有解决这个问题的类型类。正在为 OCaml（因此 ReasonML ）开发类似的方法：[模块化含义](http://ocamllabs.io/doc/implicits.html)。

- 一个更好的标准库。这方面正在探索许多事情（参见 github 仓库 [reasonml-community/belt](https://github.com/reasonml-community/belt) ）。OCaml 在这里存在相当多的碎片，因此标准化将受到欢迎。

- 更好地支持 Unicode。目前，OCaml 不支持 Unicode，OCaml 字符的大小为 8 位。您可以通过 BuckleScript 的自定义字符串文字（编译为 JavaScript 字符串）获得一些 Unicode 支持：

```ocaml
Js.log({js|äöü|js});
```

将来，ReasonML 可能会增加对 OCaml 的更多支持。他们可以将字符串视为 UTF-8，并使用工具函数访问字形集群和代码点。编译为 JavaScript 会带来挑战（例如访问字符/单位），因为 JavaScript 基本上是 UTF-16。

- 长期来看，ReasonML 通过 [OCaml 的代数效应](http://ocamllabs.io/doc/effects.html)还为多核代码提供了一个引人入胜的故事。

有关 ReasonML 计划的更多信息，请参阅其[常见问题](https://reasonml.github.io/docs/en/faq.html)。

我对以上列出的东西感到兴奋。更好的异步支持对我来说尤其重要，因为它会使 Node.js 的开发更加愉快。有没有你想要的东西不在这个列表中？