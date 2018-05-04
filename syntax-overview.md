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