# 基本值和类型

在本章中，我们将看看 ReasonML 对布尔值，整数，浮点数，字符串，字符和单位类型的支持。我们还会看到一些操作符的例子。

为了探索，我们将使用交互式RationalML命令行rtop，它是包 reason-cli 的一部分（[文档解释如何安装它](https://reasonml.github.io/docs/en/global-installation.html)）。

## 在 rtop 中互动

rtop中的交互如下所示。 

```ocaml
# !true;
- : bool = false
```

两点意见：

- 你必须用分号来结束`!true;`这个表达。

- rtop 总是打印出计算结果的类型。这在稍后特别有用，并且具有更复杂的类型，例如函数的类型。

## ReasonML是静态类型 - 这是什么意思？

ReasonML中的值是静态类型的。静态类型意味着什么？

一方面，我们有术语类型。在这种情况下，类型意味着“一组值”。例如，bool是所有布尔值类型的名称：（数学）集合{false，true}。

另一方面，我们在代码的生命周期的上下文中做出以下区分：

- 静态：在编译时，不运行代码。
- 动态：在运行时，代码正在运行。

因此静态类型意味着：ReasonML 在编译时知道值的类型。编辑代码时也会知道类型，它支持智能编辑功能。

### 习惯使用类型

我们已经遇到了静态类型的一个好处：编辑支持。它还有助于检测某些类型的错误。它通常有助于记录代码的工作方式（以自动检查一致性的方式）。

为了获得这些好处，你应该习惯使用类型。你有两种方式获得帮助：

- ReasonML 经常推断类型（为你写）。也就是说，对类型的被动知识会让你感到惊讶。

- 当出现错误时，ReasonML会提供描述性错误消息，甚至可能包含修复问题的提示。也就是说，您可以使用试用和错误来学习类型。

### 没有特别的多态性（尚未）

特别的多态性听起来很聪明，但它对 ReasonML 有一个简单的定义和可见的实际后果。所以请忍受我。

ReasonML 目前不支持特别多态性，其中根据参数的类型不同地实现相同的操作。 Haskell 是另一种函数式编程语言，它通过类型类来支持特别多态性。ReasonML 最终可能会通过类似的模块化含义来支持它。

ReasonML不支持特别多态意味着大多数运算符如+（整数相加），+. （浮点数相加）和++（字符串相连）只支持单一类型。因此，您有责任将操作数转换为适当的类型。另外，ReasonML 会在编译时警告你，如果你忘记了。这是静态类型的好处。

## 注释

在我们进入值和类型之前，让我们学习注释。

ReasonML 仅有一种撰写注释的方式：

```ocaml
/* 这是一个注释 */
```

方便的是，可以嵌套这种注释（具有 C 风格语法的语言往往无法做到这一点）：

```ocaml
/* 外部 /* 内部注释 */ 注释 */
```

嵌套对于注释代码段很有用： 

```ocaml
/*
foo(); /* foo */
bar(); /* bar */
*/
```

## 布尔 

让我们输入一些布尔表达式： 

```ocaml
# true;
- : bool = true
# false;
- : bool = false
# !true;
- : bool = false
# true || false;
- : bool = true
# true && false;
- : bool = false
```

## 数字

这些是整数表达式：

```ocaml
# 2 + 1;
- : int = 3
# 7 - 3;
- : int = 4
# 2 * 3;
- : int = 6
# 5 / 3;
- : int = 1
```

浮点表达式如下所示：

```ocaml
# 2.0 +. 1.0;
- : float = 3.
# 2. +. 1.;
- : float = 3.
# 2.25 +. 1.25;
- : float = 3.5

# 7. -. 3.;
- : float = 4.
# 2. *. 3.;
- : float = 6.
# 5. /. 3.;
- : float = 1.66666666666666674
```

## 字符串

普通字符串文字用双引号分隔：

```ocaml
# "abc";
- : string = "abc"
# String.length("ü");
- : int = 2

# "abc" ++ "def";
- : string = "abcdef"
# "There are " ++ string_of_int(11 + 5) ++ " books";
- : string = "There are 16 books"

# {| Multi-line
string literal
\ does not escape
|};
- : string = " Multi-line\nstring literal\n\\ does not escape\n"
```

ReasonML 字符串编码为 UTF-8，与 JavaScript 的 UTF-16 字符串不兼容。ReasonML 对 Unicode 的支持也比 JavaScript 更糟糕 - 它已经很有限了。作为一个短期的解决方法，您可以在 ReasonML 中使用 BuckleScript 的 JavaScript 字符串：

```ocaml
Js.log("äöü"); /* garbage */
Js.log({js|äöü|js}); /* äöü */
```

这些字符串是通过用js注释的多行字符串文字生成的，这些文字只能由BuckleScript专门处理。在本地ReasonML中，您将获得普通字符串。

## 字符

字符由单引号分隔。只支持Unicode的前7位（不含变音器等）：

```ocaml
# 'x';
- : char = 'x'
# String.get("abc", 1);
- : char = 'b'
# "abc".[1];
- : char = 'b'
```

`"x".[0]` 是` String.get("x", 0)` 的语法糖.

## 单位类型

有时候，你需要一个表示“无”的值。为此，ReasonML 具有特殊的值（）。 （）有它自己的类型，单位，并且是这种类型的唯一元素：

```ocaml
# ();
- : unit = ()
```

与 C 风格语言中的 null 不同，（）不是任何其他类型的元素。

除此之外，类型单位用于具有不返回任何内容的副作用的函数。例如：

```ocaml
# print_string;
- : (string) => unit = <fun>
```

函数 print_string 将一个字符串作为参数并打印该字符串。它没有真正的结果。

## 在基本类型之间进行转换

ReasonML的标准库具有在基本类型之间转换的功能：

```ocaml
# string_of_int(123);
- : string = "123"
# string_of_bool(true);
- : string = "true"
```

所有的转换函数的命名如下。

«输出类型»_of_«输入类型» （ 译者注： string_of_int 就是将整型转换为字符串 ）

## 更多的操作符

### 比较运算符 

以下是比较运算符。它们是少数几种类型操作符的一部分（它们是多态的）。

```ocaml
# 3.0 < 4.0;
- : bool = true
# 3 < 4;
- : bool = true
# 3 <= 4;
- : bool = true
```

但是，您不能混合操作数类型：

```ocaml
# 3.0 < 4;
错误：表达式的类型为int，但预期的类型为float
```

### 相等操作符

ReasonML有两个相等运算符。

双等号（按值相等）比较值，即使对于列表等参考类型也是如此。

```ocaml
# [1,2,3] == [1,2,3];
- : bool = true
```

相比之下，三等号（参照平等）比较引用：

```ocaml
# [1,2,3] === [1,2,3];
- : bool = false
```

== 是首选的相等运算符（除非你真的想比较引用）。