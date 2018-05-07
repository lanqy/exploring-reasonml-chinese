# 模式匹配：解构，switch，if 表达式

> 在本章中，我们看看与模式匹配有关的三个特征：解构，`switch` 和 `if` 表达式。

### 离题：元组

为了说明模式和模式匹配，我们将使用元组。元组基本上是由位置（而不是名称）标识部分的记录。元组的部分称为组件。

让我们在 rtop 中创建一个元组：

```ocaml
# (true, "abc");
- : (bool, string) = (true, "abc")
```

这个元组的第一个元素是布尔值 true，第二个元素是字符串 “abc”。因此，元组的类型是（bool，string）。

让我们创建更多元组：

```ocaml
# (1.8, 5, ('a', 'b'));
- : (float, int, (char, char)) = (1.8, 5, ('a', 'b'))
```

### 模式匹配

在我们研究解构之前，`switch` 和 `if` 我们需要学习他们的基础：模式匹配。

模式是一种帮助处理数据的编程机制。它们有两个目的：

* 检查有什么结构的数据。
* 提取部分数据。

这是通过将数据与模式匹配来完成的。在语法上，模式的工作原理如下：

* ReasonML 具有创建数据的语法。例如：元组通过用逗号分隔数据并将结果放在括号中来创建。
* ReasonML 具有处理数据的语法。模式的语法反映了创建数据的语法。

我们从支持元组的简单模式开始。他们有以下语法：

* 一个变量名称就是一个模式。
  * 例如：`x`,`y`,`foo`。
* 文字是一种模式。
  * 例如：`123`,`"abc"`,`true`。
* 一组元组模式是一种模式。
  * 例如：`(8, x)`,`(3.2, "abc", true)`,`(1, (9, foo))

相同的变量名称不能在两个不同的位置使用。也就是说，以下模式是非法的：`(x, x)`。

#### 相等检查

最简单的模式没有任何变量。匹配这种模式基本上与平等检查相同。我们来看几个例子：

| 模式             |      数据       | 是否匹配 |
| ---------------- | :-------------: | -------: |
| 3                |        3        |       是 |
| (true, 12, 'p')  | (true, 12, 'p') |       是 |
| (false, 12, 'p') | (true, 12, 'p') |       否 |

到目前为止，我们已经使用该模式来确保数据具有预期的结构。作为下一步，我们引入变量名称。这些使结构检查更加灵活，让我们提取数据。

#### 模式中的变量名称

变量名称与其位置上的任何数据匹配，并导致创建绑定到该数据的变量。

| 模式   |  数据  | 是否匹配 |     变量绑定 |
| ------ | :----: | -------: | -----------: |
| x      |   3    |       是 |        x = 3 |
| (x, y) | (1, 4) |       是 | x = 1, y = 4 |
| (1, y) | (1, 4) |       是 |        y = 4 |
| (2, y) | (1, 4) |       否 |              |

特殊变量名称 `_` 不会创建变量绑定，可以多次使用：

| 模式    |  数据  | 是否匹配 | 变量绑定 |
| ------- | :----: | -------: | -------: |
| (x, \_) | (1, 4) |       是 |    x = 1 |
| (1, \_) | (1, 4) |       是 |          |
| (_, _)  | (1, 4) |       是 |          |

#### 选择的模式

让我们来看看另一个模式特征：两个或多个由竖线分隔的子模式构成了一个替代方案。如果其中一个子模式匹配，则这种模式匹配。如果一个变量名存在于一个子模式中，它必须在所有子模式中退出。

例如：

| 模式            |  数据  | 是否匹配 | 变量绑定 |
| --------------- | :----: | -------: | -------: |
| 1❘2❘3           |   1    |       是 |          |
| 1❘2❘3           |   2    |       是 |          |
| 1❘2❘3           |   3    |       是 |          |
| 1❘2❘3           |   4    |       否 |          |
| (1❘2❘3, 4)      | (1, 4) |       是 |          |
| (1❘2❘3, 4)      | (2, 4) |       是 |          |
| (1❘2❘3, 4)      | (3, 4) |       是 |          |
| (x, 0) ❘ (0, x) | (1, 0) |       是 |    x = 1 |

#### as 运算符：同时绑定和匹配

到现在为止，您必须决定是否要将一段数据绑定到变量或通过子模式匹配它。`as` 运算符可以让你做到这一点：它的左边是一个子模式来匹配，它的右边是当前数据绑定到的变量的名称。

| 模式            |    数据     | 是否匹配 |          变量绑定 |
| --------------- | :---------: | -------: | ----------------: |
| 7 as x          |      7      |      是 |             x = 7 |
| (8, x) as y     |   (8, 5)    |      是 | x = 5, y = (8, 5) |
| ((1,x) as y, 3) | ((1,2), 3)) |      是 | x = 2, y = (1, 2) |

#### 还有更多创建模式的方法

ReasonML 支持比元组更复杂的数据类型。例如：列表和记录。许多这些数据类型也通过模式匹配来支持。更多内容将在后面的章节中介绍。

### 通过let进行模式匹配（解构）

你可以通过 `let` 做模式匹配。作为一个例子，我们首先创建一个元组：

```ocaml
# let tuple = (7, 4);
let tuple: (int, int) = (7, 4);
```

我们可以使用模式匹配来创建变量 `x` 和 `y` ，并分别将它们绑定到 7 和 4：

```ocaml
# let (x, y) =  tuple;
let x: int = 7;
let y: int = 4;
```
变量名称 `_` 也适用，不会创建变量：

```ocaml
# let (_, y) = tuple;
let y: int = 4;
# let (_, _) = tuple;
```

如果一个模式不匹配，你会得到一个异常：

```ocaml
# let (1, x) = (5, 5);
Warning: this pattern-matching is not exhaustive.
(警告：这种模式匹配并非详尽无遗。)
Exception：Match_failure
(异常：Match_failure.)
```

我们从ReasonML获得两种反馈：

* 在编译时:警告模式不包含 `(int, int)` 元组。当我们学习 `switch` 表达式时，我们会看看这意味着什么。
* 运行时：匹配失败的异常。

### switch

让匹配数据的单一模式。有了 `switch` 表达式，我们可以尝试多种模式。第一个匹配决定表达式的结果。看起来如下:

```ocaml
switch «value» {
| «pattern1» => «result1»
| «pattern2» => «result2»
···
}
```

`switch` 按顺序遍历分支:匹配值的第一个模式导致相关的表达式成为 `switch` 表达式的结果。让我们看一个模式匹配很简单的例子:

```ocaml
let value = 1;
let result = switch value {
    | 1 => "one"
    | 2 => "two"
}; /* result == "one" */
```
如果 `switch` 值不止一个实体（变量名称，限定变量名称，文字等），它需要放在括号内：

```ocaml
let result =  switch (1 + 1) {
    | 1 => "one"
    | 2 => "two"
}; /* result == "two" */
```

#### 关于详尽性的警告

编译上一个示例或在 `rtop` 中输入时，会出现以下编译时警告：

`Warning: this pattern-matching is not exhaustive. ( 警告:这种模式匹配不是详尽的。)`

这意味着：操作数 1 的类型为 int，分支不包含该类型的所有元素。这个警告非常有用，因为它告诉我们有些情况我们可能错过了。也就是说，我们对未来潜在的麻烦提出警告。如果没有警告，`switch` 总是会成功。

如果你没有解决这个问题，当操作数没有匹配的分支时，ReasonML 会抛出一个运行时异常：

```ocaml
let result = switch 3 {
    | 1 => "one"
    | 2 => "two"
}; /* Exception (异常): Match_failure */
```

让这个警告消失的一种方法是处理一个类型的所有元素。我将简要介绍如何为递归定义的类型做这件事。这些通过以下定义：

* 一个或多个（非递归）基本情况。
* 一个或多个递归案例。

例如，对于自然数，基本情况是零，递归情况是一个加上一个自然数。您可以通过两个分支彻底覆盖自然数，每个分支一个分支。

就目前而言，只要可以，就足以知道，你应该做详尽的报道。然后，如果您错过了一个案例，编译器会警告您，防止出现整个类别的错误。

如果详尽的报道不是一个选项，你可以引入一个全面的分支。下一节将介绍如何做到这一点。

#### 变量作为模式

如果添加模式为变量的分支，关于穷举性的警告将消失：

```ocaml
let result = switch 3 {
    | 1 => "one"
    | 2 => "two"
    | x => "unkown: " ++ string_of_int(x)
}; /* result == "unknown: 3" */
```

我们通过将它 `switch` 值相匹配来创建新的变量 `x`。该新变量可用于分支的表达式中。

这种分支被称为“全部捕获”：它最后到来，并且如果所有其他分支失败，则被执行。它总是成功并匹配一切。在C风格的语言中，全部分支都称为默认分支。

如果你只想匹配所有的东西，不管匹配的是什么，你可以使用下划线:

```ocaml
let result = switch 3 {
    | 1 => "one"
    | 2 => "two"
    | _ => "unknown"
}; /* result == "unknown" */
```

#### 元组的模式

让我们通过 `switch` 表达式来实现逻辑和（&&）：

```ocaml
let tuple = (true, true);

let result = switch tuple {
    | (false, false) => false
    | (false, true) => false
    | (true, false) => false
    | (true, true) => true
}; /* result == true */
```

这段代码可以通过使用下划线和变量来简化：

```ocaml
let result = switch tuple {
    | (false, _) => false
    | (true, x) => x
}; /* result == true */
```

#### `as` 运算符

`as` 运算符也可用于 `switch` 模式：

```ocaml
let tuple  = (8, (5,9));
let result = switch tuple {
    | (0, _) => (0, (0, 0))
    | (_, (x, _) as t) => (x, t)
}; /* result == (5 (5, 9)) */
```

#### 选择模式

在子模式中使用替代方案如下所示。

```ocaml
switch someTuple {
    | (0,1 | 2 | 3) => "first branch"
    | _ => "second branch"
};
```

其他选择也可以在顶层使用:

```ocaml
switch "Monday" {
| "Monday"
| "Tuesday"
| "Wednesday"
| "Thursday"
| "Friday" => "weekday"
| "Saturday"
| "Sunday" => "weekend"
| day => "Illegal value: " ++ day
};
/* Result: "weekday" */
```

#### 条件分支

分支机构的警卫（条件）是一个 `switch` 特定的功能：它们在模式之后，并且在关键字之前。我们来看一个例子：

```ocaml
let tuple = (3, 4);
let max = switch tuple {
    | (x, y) when x > y => x
    | (_, y) => y
}; /* max == 4 */

```

第一个分支仅在条件 `x > y` 为真时评估。

### `if` 表达式

ReasonML 的 `if` 表达式如下所示（ `else` 可以省略）：

```ocaml
if («bool») «thenExpr» else «elseExpr»;
```

例如：

```ocaml
# let bool =  true;
let bool: bool = true;
# let boolStr = if (bool) "true" else "false";
let boolStr: string = "true";
```

鉴于作用域块也是表达式，以下两个if表达式是等价的：

```ocaml
if (bool) "true" else "false"
if (bool) {"true"} else {"false"}
```

事实上，`refmt` 把前者的表达式作为后者的表达式进行格式化打印。

`then` 表达式和 `else` 表达式必须具有相同的类型。

```ocaml
Reason # if (true) 123 else "abc";
Error: This expression has type string
but an expression was expected of type int
```

#### 省略 `else` 分支

你可以省略 `else` 分支 - 以下两个表达式是等价的。

```ocaml
if (b) expr else ()
if (b) expr
```

鉴于两个分支必须具有相同的类型，expr 必须具有类型单元（其唯一元素是 ()）。

例如，print_string() 的计算结果为 ()，以下代码工作：

```ocaml
# if (true) print_string("hello\n");
hello
- : unit = ()
```

相反，这不起作用：

```ocaml
# if (true) "abc";
Error: This expression has type string
but an expression was expected of type unit
( 错误：此表达式具有字符串类型 但类型单位预计表达式 )
```

### 三元运算符 (_?_:_)

ReasonML 还为您提供三元运算符作为 `if` 表达式的替代方法。以下两个表达式是等价的。

```ocaml
if (b) expr1 else expr2

b ? expr1 : expr2
```

以下两个表达式也是相同的。甚至很漂亮 - 将前者作为后者进行打印。

```ocaml
switch (b) {
    | true => expr1
    | false => expr2
};

b ? expr1 : expr2;
```

我没有发现三元运算符运算符在 ReasonML 中非常有用：它在 C 语言类似的语言中的用途是具有 `if` 语句的表达式版本。但是，如果已经是 ReasonML 中的表达式。