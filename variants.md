# 变体类型

> 变体类型（简称：变体）是许多函数式编程语言支持的数据类型。 它们是 ReasonML 中的一个重要组成部分，C 语言（C，C ++，Java，C＃等）不支持。 本章介绍它们如何工作。

## 变体作为一组符号（枚举）

变体允许您定义符号集。像这样使用时，它们与 C 风格语言中的枚举类似。例如，以下类型的颜色定义了六种颜色的符号。

```ocaml
type color = Red | Orange | Yellow | Green | Blue | Purple;
```

这个类型定义中有两个要素：

- 类型的名称，颜色，必须以小写字母开头。
- 构造函数的名称（ `Red` ， `Orange` ，...），必须以大写字母开头。一旦我们使用变体作为数据结构，为什么构造函数被称为构造函数将变得清晰。

构造函数的名称在当前范围内必须是唯一的。这使 ReasonML 能够轻松推导出它们的类型：

```ocaml
# Purple;

- : color = Purple
```

变体可以通过 `switch` 和模式匹配进行处理：

```ocaml
let invert = (c: color) =>
    switch c {
        | Red => Green
        | Orange => Blue
        | Yellow => Purple
        | Green => Red
        | Blue => Orange
        | Purple => Yellow
    };
```

在这里，构造函数既用作模式（ `=>` 的左侧），也用于值的右侧（ `=>` 的右侧）。这是 `invert()` 的作用：

```ocaml
# invert(Red);
- : color = Green

# invert(Yellow);
- : color = Purple
```

#### 提示：用变体替换布尔变量

在 ReasonML 中，变体通常是比布尔变量更好的选择。举个例子，这个函数定义。 （请记住，在 ReasonML 中，主要参数最终会启用 currying ( 柯里化 )。）

```ocaml
let stringOfContact(includeDetails: bool, c: contact) => ...;
```

这是怎样调用 `stringOfContact` 的：

```ocaml
let str = stringOfContact(true, myContact);
```

目前还不清楚最后的布尔值是什么。您可以通过标记参数来改进此功能。

```ocaml
let stringOfContact(~includeDetails: bool, c: contact) => ...;
let str = stringOfContact(~includeDetails = true, myContact);
```

更具自我描述性的是为 `~includeDetails` 的值引入一个变体：

```ocaml
type includeDetails = ShowEverything | HideDetails;
let stringOfContact(~levelOfDetail: includeDetails, c: contact) => ...;
let str = stringOfContact(~levelOfDetail = ShowEverything, myContact);
```

使用变体 includeDetails 有两个优点：

- 立即清楚“不显示细节”的含义。
- 稍后添加更多模式很容易。

#### 将变体值与数据关联

有时，您想使用变体值作为查找数据的关键字。这样做的一种方式是通过一个将变量值映射到数据的函数：

```ocaml
type color = Red | Orange | Yellow | Green | Blue | Purple;
let stringOfColor(c: color) => 
    switch c {
        | Red => "Red"
        | Orange => "Orange"
        | Yellow => "Yellow"
        | Green => "Green"
        | Blue => "Blue"
        | Purple => "Purple"
    };
```

## 变体作为数据结构

每个构造函数也可以保存一个或多个值。这些值由位置标识。也就是说，单独的构造函数与元组相似。以下代码演示了此功能。

```ocaml
type point = Point(float, float);

type shape = 
    | Rectangle(point, point)
    | Circle(point, float);

```

类型 `point` 是具有单个构造函数的变体。它拥有两个浮点数。`shape` 是另一种变体。它可以是：

- 由两个角点定义的 `Rectangle`
- 由中心和半径定义的 `Circle` 。

使用多个构造函数参数，它们是位置的，没有标签成为一个问题 - 我们必须在别处描述他们的角色是什么。在这种情况下，记录是另一种选择（它们在他们自己的章节中有描述）。

这是你如何使用构造函数：

```ocaml
# let bottomLeft = Point(-1.0, -2.0);
let bottomLeft: point = Point(-1., -2.);

# let topRight = Point(7.0, 6.0);
let topRight: point = Point(7., 6.);

# let circ = Circle(topRight, 5.0);
let circ: shape = Circle(Point(7., 6.), 5.);

# let rect = Rectangle(bottomLeft, topRight);
let rect: shape = Rectangle(Point(-1., -2.), Point(7., 6.)); 

```

由于每个构造函数名称都是唯一的，因此 ReasonML 可以轻松推断出这些类型。

如果构造函数保存数据，通过 `switch` 进行模式匹配更加方便，因为它还允许您访问该数据：

```ocaml
let pi = 4.0 *. atan(1.0);

let computeArea = (s: shape) =>
    switch s {
        | Rectangle(Point(x1, y1), Point(x2, y2)) =>
            let width = abs_float(x2 -. x1);
            let height = abs_float(y2 -. y1);
            width *. height;
        | Circle(_, radius) => pi *. (radius ** 2.0)
    }
```

让我们使用 `computeArea` ，继续我们之前的交互式 `rtop` 会话：

```ocaml
# computeArea(circ);
- : float = 78.5398163397448315

# computeArea(rect);
- : float = 64.

```

## 通过变体的自递归数据结构

您也可以通过变体定义递归数据结构。例如，节点包含整数的二叉树：

```ocaml
type intTree = 
    | Empty
    | Node(int, intTree, intTree);
```

`intTree` 的值是这样构造的：

```ocaml
let myIntTree = Node(1,
  Node(2, Empty, Empty),
  Node(3,
    Node(4, Empty, Empty),
    Empty
  )
);
```

`myIntTree` 的外观如下：1有两个子节点 2 和 3 . 2 有两个空子节点。等等。

```ocaml
1
  2
    X
    X
  3
    4
      X
      X
    X
```

#### 通过递归处理自递归数据结构

为了演示处理自递归数据结构，让我们实现一个函数 `computeSum` ，它计算存储在节点中的整数的总和。

```ocaml
let rec computeSum = (t: intTree) =>
    switch t {
        | Empty => 0
        | Node(i, leftTree, rightTree) =>
            i + computeSum(leftTree) + computeSum(rightTree)
    };

computeSum(myIntTree); /* 10 */
```

使用变体类型时，这种递归是典型的：

- 一组有限的构造函数用于创建数据。在这种情况下：`Empty` 和 `Node()`。
- 使用相同的构造函数作为模式来处理数据。

这确保我们能够正确处理传递给我们的任何数据，只要它是 `intTree` 类型即可。 ReasonML 会警告我们，如果 `switch` 没有详尽地涵盖 `intTree` 。这可以保护我们避免遗忘我们应该考虑的情况。为了说明，我们假设我们忘记了 `Empty` ，并写下了这样的 `computeSum` ：

```ocaml
let rec computeSum = (t: intTree) => 
    switch t {
        /* Missing: Empty */
        | Node(i, leftTree, rightTree) =>
            i + computeSum(leftTree) + computeSum(rightTree)
    };
```

然后我们得到以下警告。

`警告：这种模式匹配并非详尽无遗。 以下是一个不匹配的值的示例： 空`

正如在函数一章中提到的，引入所有的情况意味着您失去了这种保护。这就是为什么你应该避开他们。

## 通过变体相互递归的数据结构

回想一下，当我们使用递归时，我们必须使用 `let rec`:

- 一个自我递归的定义是通过 `let rec` 完成的。
- 多个相互递归的定义通过 `let rec`和通过 `and` 连接完成。

`type` 是隐式 `rec` 。这使我们可以做自我递归的定义，如 `intTree` 。对于相互递归的定义，我们还需要通过 `and` 连接这些定义。下面的例子再次定义了 `int` 树，但是这次使用了单独的节点类型。

```ocaml
type intTree = 
    | Empty
    | IntTreeNode(intNode)
and intNode = 
    | IntNode(int, intTree, intTree);
```

`intTree` 和 `intNode` 是相互递归的，这就是为什么它们需要在相同的 `type` 声明中定义，通过 `and` 分隔。

## 参数化变体

让我们回顾我们对 `int` 树的原始定义：

```ocaml
type intTree = 
    | Empty
    | Node(int, intTree, intTree);
```

我们如何将这个定义转化为树的一个通用定义，树的节点可以包含任何类型的值？为此，我们必须为 `Node` 的内容类型引入一个变量。类型变量在 ReasonML 中以撇号作为前缀。例如：`'a` 。因此，通用树看起来如下所示：

```ocaml
type tree('a) = 
    | Empty
    | Node('a, tree('a), tree('a));
```

有两件事值得注意。首先，之前具有 `int` 类型的节点的内容现在具有类型 `'a`。其次，类型变量 `'a` 已成为类型树的参数。节点将该参数传递给其子树。也就是说，我们可以为每棵树选择不同的节点值类型，但在树中，所有节点值必须具有相同的类型。

现在我们可以通过提供树的类型参数，通过类型别名为 `int` 树定义一个类型：

```ocaml
type intTree = tree(int);
```

让我们用 `tree` 类型来创建一个字符串树：

```ocaml
let myStrTree = Node("a",
    Node("b", Empty, Empty),
    Node("c",
        Node("d", Empty, Empty),
        Empty
    )
);
```

由于类型推理，您不需要提供类型参数。ReasonML 自动推断 `myStrTree` 具有类型树（字符串）。以下通用函数可以打印任何类型的树：

```ocaml
/**
 * @param ~indent 当前树或子树有多少个缩进.
 * @param ~stringOfValue 将节点值转换为字符串。
 * @param t 要转换为字符串的树。
 */

let rec stringOfTree = (~indent = 0, ~stringOfValue: 'a => string, t: tree('a)) => {
    let indentStr = String.make(indent * 2, ' ');
    switch t {
        | Empty => indentStr ++ "X" ++ "\n"
        | Node(x, leftTree, rightTree) => 
            indentStr ++ stringOfValue(x) ++ "\n" ++
            stringOfTree(~indent = indent + 1, ~stringOfValue, leftTree) ++ 
            stringOfTree(~indent = indent + 1, ~stringOfValue, rightTree)
    };
};

```

该函数使用递归遍历其参数 `t` 的节点。鉴于 `stringOfTree` 适用于任意类型 `'a`，我们需要一个类型特定函数来将 `'a` 类型的值转换为字符串。这是 `~stringOfValue` 参数要做的。

这就是我们如何打印先前定义的 `myStrTree`：

```ocaml
# print_string(stringOfTree(~stringOfValue = x => x, myStrTree));
a
  b
    X
    X
  c
    d
      X
      X
    X
```

## 有用的标准变体

我将简要介绍两种常用的标准变体。

#### 可选值类型 option('a)

在许多面向对象的语言中，具有类型字符串的变量意味着该变量可以是 `null` 或字符串值。包含 `null` 的类型称为 `nullable` (可空)。可空类型有问题，因为它很容易处理它们的值，而忘记处理 `null` 。如果异常 - `null` (空)出现，你会得到臭名昭着的空指针异常。

在 ReasonML 中，类型不可空。相反，可能缺少的值通过以下参数化变体处理：

```ocaml
type option('a) = 
    | None
    | Some('a);
```

`option` 强制你总是考虑 `None` 情况。

ReasonML 对选项的支持很小。这个变体的定义是语言的一部分，但是核心标准库还没有用于处理可选值的实用函数。在这之前，您可以使用 BuckleScript 的 [Js.Option](https://bucklescript.github.io/bucklescript/api/Js.Option.html)。

#### 使用 result('a) 类型进行错误处理

`result` 是 OCaml 中的错误处理的另一个标准变体：

```ocaml
type result('good, 'bad) = 
    | Ok('good)
    | Error('bad);
```

在 ReasonML 的核心库支持它之前，您可以使用 BuckleScript 的 [Js.Result](https://bucklescript.github.io/bucklescript/api/Js.Result.html)。


#### 示例：计算整数表达式

使用 `trees` 是 ML 风格语言的优势之一。这就是为什么它们经常用于涉及语法树（解释器，编译器等）的程序。例如，Facebook 的语法检查器 Flow 是用 OCaml 编写的。

因此，作为最后一个例子，我们实现一个简单整数表达式的求值器。

以下是整数表达式的数据结构。

```ocaml
type expression = 
    | Plus(expression, expression)
    | Minus(expression, expression)
    | Times(exprssion, expression)
    | DividedBy(expression, expression)
    | Literal(int);
```

这是用这个变体编码的表达式：

```ocaml
/* (3 - (16 / (6 + 2)) */
let expr = 
    Minus(
        Literal(3),
        DividedBy(
            Literal(16),
            Plus(
                Literal(6),
                Literal(2)
            )
        )
    );
```

最后，这是计算整型表达式的函数。

```ocaml
let rec eval(e: expression) =
    switch e {
        | Plus(e1, e2) => eval(e1) + eval(e2)
        | Minus(e1, e2) => eval(e1) - eval(e2)
        | Times(e1, e2) => eval(e1) * eval(e2)
        | DividedBy(e1, e2) => eval(e1) / eval(e2)
        | Literal(i) => i
    };

eval(expr); /* 1 */
```
