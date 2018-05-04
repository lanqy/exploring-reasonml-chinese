# 首先看一下 ReasonML 的语法

在本章中，我想给你一个关于 ReasonML 代码的第一印象。因此：不要试图理解（:)） - 在下面的章节中将逐步提供适当的解释。

这是ReasonML代码：

```ocaml
/* 注释 ( 没有单行注释 ) */
/* 你可以 /* 嵌套 */ 注释 */

/* 变量绑定 */
let myInt = 123;

/* 函数 */
let id = x => x;
let add = (x, y) => x + y;

/* 定义变体类型 */
type color = Red | Green | Blue;

/* 一种切换变体类型的函数 */
let stringOfColor = (c: color) =>
    switch (c) {
        | Red => "Red"
        | Green => "Green"
        | Blue => "Blue"
    };

/* 调用 stringOfColor() */
stringOfColor(Red); /* "Red" */


```

再次说明：没有必要了解我刚才向你展示的内容。但是如果你想现在深入挖掘，你可以：

- [模式匹配 and switch](http://reasonmlhub.com/exploring-reasonml/ch_pattern-matching.html)
- [函数](http://reasonmlhub.com/exploring-reasonml/ch_functions.html)
- [变体类型](http://reasonmlhub.com/exploring-reasonml/ch_variants.html)

## 大多数东西都是表达式

例如，你几乎可以在任何地方使用`if-then-else`：

```ocaml

let myBool =  true;
id(if (myBool) "yes" else "no");

```

而且你几乎可以在任何地方使用块：

```ocaml
let abcabc = {
    let abc = "abc";
    abc ++ abc; /* "abcabc" */
};
```

实际上，以下两个表达式是等价的：

```ocaml
if (myBool) "yes" else "no";

if (myBool) {
    "yes";
} else {
    "no";
}
```

## 分号很重要

您可能已经注意到本章中的代码中有很多分号。其中大多数是强制性的。额外的分号可以被忽略。特别是交互式命令行rtop只会评估以分号结尾的表达式。

起初，即使在代码块后面看到分号也是有点奇怪，但它是有道理的，因为代码块也是表达式。有了这些知识，再看看两个if表达式：

```ocaml
if (myBool) "yes" else "no";

if (myBool) {
    "yes";
} else {
    "no";
};

```

第一行末尾的分号看起来合乎逻辑。但是，最后的分号也是合乎逻辑的，因为我们只是用块代替了“no” 表达式。

## ReasonML 中的所有内容都是骆驼式的

ReasonML 基于 OCaml，它使用小写字母的蛇壳（create_resource）和大写字母（StringUtilities）的骆驼壳。这就是为什么你偶尔会看到以蛇为名的名字。

但是所有新的 ReasonML 代码都是驼峰式的（StringUtilities，createResource）。

## 变量名称的特殊前缀和后缀

前缀下划线的意思是：不要警告我这个变量没有被使用。

```ocaml
let f = (x, _y) => x;
  /* 没有警告 _y */
```

后缀撇号是合法的（在数学中，x' 表示 x 的修改版本）：

```ocaml
let x = 23;
let x' = x + 1;
```

带前缀的撇号是为类型变量保留的（思考C风格语言中的泛型）：

```ocaml
let len(arr: array('a)) = Array.length(arr);
```

类型变量在[变体类型的章节](variants.md)中进行了解释。它们与C风格语言中的泛型类似。