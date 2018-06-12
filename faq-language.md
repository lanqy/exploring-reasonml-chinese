# ReasonML 常见问题

## BuckleScript中的 Str 模块在哪里？

当你编译成 JavaScript 时，你不能使用 Str 模块，它的功能太依赖于本地字符串。相反，您必须使用 JavaScript 特定的模块：

- 字符串模块：[Js.String](https://bucklescript.github.io/bucklescript/api/Js.String.html)

- 正则表达式模块：[Js.Re](https://bucklescript.github.io/bucklescript/api/Js.Re.html)