# 函子（高级）

> 直观上，函子是一个函数，它将一个或多个模块作为输入并返回一个新模块。本章介绍仿函数的工作原理以及为什么它们有用。

这些例子的代码可以在 GitHub上的 repository [reasonml-demo-functors](https://github.com/rauschma/reasonml-demo-functors) 中找到。

注：函子是一个高级主题。你最初可能不知道如何编写它们。我解释了在别处需要时使用它们的基本知识。

## 什么是函子？

函子是一个来自类别理论的术语，它指的是类别之间的映射。在 ReasonML 中，有两种查看函子的方法：

- 一个函数，其参数是模块，其结果是一个模块。
- 通过模块（参数）参数化（配置）的模块（结果）。

您只能将函子定义为子模块（它不能是文件中的顶层元素）。但这不是问题，因为您通常希望在模块内部提供仿函数，它可以位于接口的参数和（可选）结果旁边。

函子的语法如下所示：

```ocaml
module «F» = («ParamM1»: «ParamI1», ···): «ResultI» => {
  «body of functor»
};
```

函子 `F` 具有一个或多个模块参数 `ParamM1` 等。这些模块必须通过接口 `ParamI1` 等进行类型化。结果类型  `ResultI` (另一个接口)是可选的。

函子的主体具有与普通模块相同的语法，但它可以引用参数及其内容。

## 第一个函子

定义函子 `Repetition.Make`

作为第一个例子，我们定义了一个函数 `Repetition.Make`，它导出一个函数 `repeat()` ，它重复它的参数，一个字符串。函子的参数配置字符串重复的频率。在我们定义函数之前，我们需要为它的参数定义一个接口 `Repetition.Count`：

```ocaml
/* Repetition.re */

module type Count = {
  let count: int;
};
```