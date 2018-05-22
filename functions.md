# 函数

>本章探讨函数如何在 ReasonML 中工作。

### 定义函数

匿名（无名）函数如下所示：

```ocaml
(x) => x + 1;
```
该函数具有单个参数 `x` 和主体 `x + 1`。

您可以通过将该函数绑定到一个变量来为该函数命名：

```ocaml
let plus1 = (x) => x + 1;
```

这就是你调用 plus1 函数的方式：

```ocaml
# plus1(5);
- : int = 6
```

#### 作为其他函数的参数（高阶函数）

函数也可以是其他函数的参数。为了演示这个功能，我们简要地使用列表，这些[列表](lists.md)在他们自己的章节中有解释。列表大致是单链表，类似于不可变数组。

列表函数 `List.map(func，list)` 需要列表，将 `func` 应用到它的每个元素并将结果返回到一个新列表中。例如：

```ocaml
# List.map((x) => x + 1, [12, 5, 8, 4]);
- : list(int) = [13, 6, 9, 5]
```

具有函数作为参数或结果的函数称为高阶函数。不被称为一阶函数的函数。 `List.map()` 是一个高阶函数。 `plus1()` 是一个一阶函数。

#### 块作为函数体

函数的主体是一个表达式。鉴于作用域块是表达式，加号 1 的以下两个定义是等效的。

 ```ocaml
 let plus1 = (x) => x + 1;

 let plus1 = (x) => {
     x + 1
 };
 ```

 ### 没有括号的单参数

 如果函数具有单个参数，并且该参数是通过标识符定义的，则可以省略括号：

 ```ocaml
 let plus1 = x => x + 1;
 ```

 ### 通过递归绑定 `let rec`

 通常情况下，您只能引用已存在的限制值。这意味着你不能定义相互递归和自递归函数。

 #### 定义相互递归的函数

 我们先来看看相互递归的函数。以下两个函数 `even` 和 `odd` 是相互递归的（这是一个例子，而不是你实际实现这些函数的方式）。您必须使用特殊的 `let rec` 来定义它们：

```ocaml
let rec even = (x) =>
    if (x <= 0) {
        true;
    } else {
        odd(x - 1);
    }
and odd = (x) =>
    if (x <= 0) {
        false;
    } else {
        even(x - 1);
    }
```

请注意如何连接多个互相认识的 `let rec` 条目。在和之前没有分号。最后的分号表示让rec结束。

我们来使用这些函数：

 ```ocaml
 # even(11);
 - : bool = false
 # even(2);
 - : bool = true
 # odd(11);
 - : bool = true
 # odd(2);
 - : bool = false
 ```

 #### 定义自递归函数

您还需要让递归调用自身的函数 `rec`，因为在进行递归调用时，绑定不存在。例如：

```ocaml
let rec factorial = (x) =>
    if (x <= 2>) {
        x
    } else {
        x * factorial(x - 1)
    };

factorial(3); /* 6 */
factorial(4); /* 24 */

```

### 术语：元数


### 函数的类型

函数是我们第一次接触到复杂类型：通过组合其他类型构建的类型。我们使用 `rtop` 来确定两个函数的类型。

#### 一阶函数的类型

首先，一个函数 `add()`：

```ocaml
# let add = (x, y) => x + y;
let add:(int, int) => int = <fun>;
```

因此，`add` 的类型是：

```ocaml
(int, int) => int
```

箭头表示 `add` 是一个函数。它的参数是两个整数。其结果是一个 `int`。

符号 `(int, int)=> int` 也被称为 `add` 的（类型）签名。它描述了其输入和输出的类型。

#### 高阶函数的类型

其次，一个高阶函数 `callFunc()`:

```ocaml
# let callFunc = (f) => f(1) + f(2);
let callFunc:((int) => int) => int = <fun>;
```

你可以看到 `callFunc` 的参数本身就是一个函数，并且具有类型 `(int) => int`。

这是如何使用 `callFunc()`：

```ocaml
# callFunc(x => x);
- : int = 3
# callFunc(x => 2 * x);
- : int = 6
```

#### 类型注解和类型推断

ReasonML 中的类型注释是可选的，但它们提高了类型检查的准确性。最极端的是要注解一切：

```ocaml
let add = (x: int, y: int): int => x + y;
```
我们为参数 `x` 和 `y` 以及函数的结果（最后一个：箭头之前的 `int` ）提供了类型注释。

您可以省略返回类型的注释，并且 ReasonML 会推断它（从参数类型中推断出来）：

```ocaml
# let add = (x: int, y: int) => x + y;
let add: (int, int) => int = <fun>;
```

但是，类型推断比这更复杂。它不仅可以自顶向下工作，还可以从 `int` 加号运算符 `(+)` 推断出参数的类型：

```ocaml
# let add = (x, y) => x + y;
let add: (int, int) => int = <fun>;
```

如果你愿意，你也可以只注解一些参数：

```ocaml
let add = (x, y: int) => x + y;
```

#### 类型注解：最佳实践

我更喜欢函数的编码风格是注释所有参数，但让 ReasonML 推断返回类型。除了改进类型检查之外，参数的注释也是很好的文档。

#### 没有无参数的函数

ReasonML 没有空值函数，但您可以使用它，而不会注意到这一点。

回想一下，在许多C风格的语言中，`( )` 大致类似于 `null`。它是 `unit` 类型的唯一元素。调用函数时，省略参数与将 `unit` 值作为单个参数传递相同。也就是说，以下两个表达式是等价的。

```ocaml
func()
func(())
```

以下示例演示了这种现象：如果您调用不带参数的一元函数，`rtop` 会强调 `()` 并报该类型的表达式错误。它不会报没有提供足够的参数（它不会部分应用 - 稍后会有详细说明）。

```ocaml
# let id = (x: int) => x;
let id: (int) => int = <fun>;
# id();
Error: This expression has type unit but
an expression was expected of type int
( 错误：该表达式具有 `unit` 类型但是预期类型为 `int` 的表达式 )
```

如果你定义了一个没有参数的函数，那么 ReasonML 会为你添加一个参数，其类型为 `unit`：

```ocaml
# let f = () => 123'
let f: (unit) => int = <fun>;
```

#### 为什么没有空函数 ?

为什么 ReasonML 没有空函数？ 这是由于 ReasonML 始终执行部分应用程序（稍后详细解释）：如果不提供所有函数的参数，则会从剩余参数获得一个新函数。 因此，如果实际上根本没有提供任何参数，那么 `func()` 将与 `func` 相同，而且实际上也不会调用 `func`。

### 解构函数参数

在变量绑定到值的地方，可以使用解构。也就是说，它也适用于参数定义。让我们看看一个添加元组的元素的函数：

```ocaml
let addComponents = ((x, y)) => x + y;
let tuple = (3, 4);
addComponents(tuple); /* 7 */
```

围绕 `x，y` 的双括号表明 `addComponents` 是一个具有单个参数的函数，它是一个元素是 `x` 和 `y` 的元组。它不是具有两个参数 `x` 和 `y` 的函数。它的类型是：

```ocaml
addComponents: ((int, int)) => int
```

当涉及到类型注解时，您可以注解组件：

```ocaml
# let addComponents = ((x: int, y: int)) => x + y;
let addComponents: ((int, int)) => int = <fun>;
```
或者你可以注解整个参数：

```ocaml
# let addComponents = ((x, y): (int, int)) => x + y;
let addComponents: ((int, int)) => int = <fun>;
```

### 标签参数

到目前为止，我们只使用位置参数：实际参数在调用的位置决定了它绑定的形式参数。

但是 ReasonML 也支持标签参数。这里，标签用于将实际参数与形式参数相关联。

作为一个例子，我们来看看使用标签参数的 `add` 的版本：

```ocaml
let add = (~x, ~y) => x + y;

add(~x:7, ~y=9); /* 16 */

```

在这个函数定义中，我们对标签 `~x` 和参数 `x` 使用了相同的名称。您也可以使用不同的名称，例如 `~x` 为标签，`op1` 为参数：

```ocaml
/* 类型推导 */
let add = (~x as op1, ~y as op2) =>
    op1 + op2;

/* 指定类型 */
let add = (~x as op1: int, ~y as op2: int) =>
    op1 + op2;

```

在调用时，您可以缩写 `~x = x`，只需 `~x`：

```ocaml
let x = 7;
let y = 9;
add(~x, ~y);
```

标签的一个很好的功能是可以按任意顺序提及标签参数：

```ocaml
# add(~x=3, ~y=4);
- : int = 7
# add(~y=4,~x=3);
- : int = 7
```

#### 函数类型与标签参数的兼容性

有一个不幸的警告是能够以任何顺序提及带标签的参数：只有当标签按照相同的顺序被提及时，功能类型才是兼容的。

考虑以下三个函数：

```ocaml
let add = (~x, ~y) => x + y;
let addxy = (add: ((~x: int, ~y: int) => int)) => add(5, 2);
let addyx = (add: ((~y: int, ~x: int) => int)) => add(5, 2);
```

`addxy` 与 `add` 按预期工作：

```ocaml
# addxy(add);
- : int = 7
```

但是，使用 `addyx`，我们会遇到错误，因为标签的顺序是错误的：

```ocaml
# addyx(add);
错误：此表达式有类型 (~x: int，~y: int)=> int
但预期的表达类型 (~y: int，~x: int)=> int
```

### 可选参数

在 ReasonML中，只有标签参数可以是可选的。在下面的代码中，`x` 和 `y` 都是可选的。

```ocaml
let add = (~x=?, ~y=?, ()) =>
    switch (x, y) {
        | (Some(x'), Some(y')) => x' + y'
        | (Some(x'), None) => x'
        | (None, Some(y')) => y'
        | (None, None) => 0
    };
```
让我们看看相对复杂的代码做了什么。

为什么 `()` 作为最后一个参数？这在下一节中解释。

`switch` 表达式做了什么？如果您将参数声明为可选参数，则始终具有 `option(t)` 类型，其中t是实际值所具有的任何类型。`option` 是一个变体（在[单独的章节](variants.md)中进行解释）。现在，我将简要预览一下。`option` 的定义是：

```ocaml
type option('a) = None | Some('a);
```

它的用法如下：

- 省略 `~x` 和 `x` 将被绑定到 `None`。
- 为 `~x` 提供值 `123` ，`x` 将绑定到 `Some(123)`。

换句话说，`option` 会包装值，而示例中的 `switch` 表达式会打开它们。

#### 使用可选参数，您至少需要一个位置参数

为什么 `add` 在最后有一个类型为 `unit` 的参数（如果你愿意的话，它是一个空参数）？

```ocaml
let add = (~x=?, ~y=?, ()) =>
    ...
```

原因与部分应用程序有关，稍后将详细解释。简而言之，这里有两件事情是冲突的：

> 在部分应用程序中，如果省略了参数，就会创建一个函数，让您可以填充这些剩余的参数。
> 有了可选参数，如果省略了参数，就应该绑定到它们的默认值。

为了解决这个冲突，当遇到第一个位置参数时，ReasonML 为缺失的可选参数填充所有缺省值。在遇到位置参数之前，它仍然等待丢失的可选参数。也就是说，你需要一个位置参数来触发该调用，并且由于 `add()` 没有，因此我们添加了一个空的参数。

这种稍微怪异的方法的优点是你可以得到两全其美的效果：你可以获得部分应用程序和可选参数。

#### 可选参数的类型注解。

当您注释可选参数时，它们必须全都具有 `option(...)` 类型：

```ocaml
let add = (~x: option(int)=?, ~y: option(int)=?, ()) =>
  ...
```
`add` 的类型签名是：

```ocaml
(~x: int=?, ~y: int=?, unit) => int
```

不幸的是，在这种情况下，定义与类型签名不同。但它与具有默认值的参数（下面会解释它）的意义相同。这个想法是隐藏如何处理可选参数的实现细节。

#### 参数默认值

处理缺少的参数可能非常麻烦：

```ocaml
let add = (~x=?, ~y=?, ()) =>
  switch (x, y) {
    | (Some(x'), Some(y')) => x' + y'
    | (Some(x'), None) => x'
    | (None, Some(y')) => y'
    | (None, None) => 0
  };
```

在这种情况下，我们只需要 `x` 和 `y` 在省略时为零。 ReasonML 具有特殊的语法：

```ocaml
let add = (~x=0, ~y=0, ()) => x + y;
```

#### 用参数默认值输入注解。

如果有默认值，类型注解更直观（无 `option()`）：

```ocaml
let add = (~x: int=0, ~y: int=0, ()) =>
  x + y;
```

`add` 的类型签名为：

```ocaml
(~x: int=?, ~y:int=?, unit) => int
```

#### 将 `option` 值传递给可选参数（高级）

在内部，可选参数作为选项类型（`None` 或 `Some(x)`）的元素接收。到目前为止，您只能通过提供或省略参数来传递这些值。但也有一种方法可以直接传递这些值。在我们开始使用这个特性之前，我们先通过下面的函数试试看。

```ocaml
let multiply = (~x=1, ~y=1, ()) => x * y;
```

`multiply` 有两个可选参数，我们首先通过 `option` 元素提供 `~x` 和省略 `~y`：

```ocaml
# multiply(~x=?Some(14), ~y=?None, ());
- : int = 14
```

传递 `option` 值的语法是：

```ocaml
~label = ?expression
```

如果 `expression` 是名称为 `label` 的变量，则可以缩写：以下两种语法是等效的。

```ocaml
~foo =?foo
~foo?
```

那么用例是什么？这是一个将可选参数转发给另一个函数的可选参数的函数。这样，它可以依赖该函数的参数默认值，而不必自己定义一个参数。

我们来看一个例子：下面的函数 `square` 有一个可选参数，它传递给 `multiply` 的两个可选参数：

```ocaml
let square = (~x=?, ()) => multiply(~x?, ~y=?x, ());
```

`square` 不需要指定参数默认值，它可以使用 `multiply` 的默认值。


### 部分应用

部分应用程序是一种使函数更具通用性的机制：如果在函数调用 `f(...)` 的末尾省略一个或多个参数，则 `f` 返回一个将缺少的参数映射到 `f` 的最终结果的函数。

让我们看看它是如何工作的。我们首先创建一个带有两个参数的函数 `add`：

```ocaml
# let add = (x, y) => x + y;
let add: (int, int) => int = <fun>;
```

然后我们部分调用二元函数 `add` 来创建一元函数 `plus5`：

```ocaml
# let plus5 = add(5);
let plus5: (int) => int = <fun>;
```

我们只提供了 `add` 的第一个参数 `x`。每当我们调用 `plus5` 时，我们都会提供 `add` 的第二个参数 `y`：

```ocaml
# plus5(2);
- : int = 7
```

#### 为什么部分应用程序有用？

部分应用程序可让您编写更紧凑的代码。为了演示如何，我们将使用数字列表：

```ocaml
# let numbers = [11, 2, 8];
let numbers: list(int) = [11, 2, 8];
```

接下来，我们将使用标准库函数 `List.map`。 `List.map(func, myList)` 接受 `myList`，将 `func` 应用于每个元素并将它们作为新列表返回。

我们使用 `List.map` 为每个数字元素添加 `2`：

```ocaml
# let plus2 = x => add(2, x);
let plus2: (int) => int = <fun>;
# List.map(plus2, numbers);
- : list(int) = [13, 4, 10]
```

通过部分应用程序，我们可以使代码更紧凑：

```ocaml
# List.map(add(2), numbers);
- : list(int) = [13, 4, 10]
```

我们直接比较两个版本：

```ocaml
List.map(x => add(2, x), numbers);

List.map(add(2), numbers)
```

哪个版本更好？这取决于你的口味。第一个版本 - 可以说 - 更具自描述性，第二个版本更简洁。

部分应用程序真正使用管道运算符（ `|>` ）进行函数组合（后面会介绍）。

#### 部分应用和标签参数

到目前为止，我们只能看到具有位置参数的部分应用程序，但它也适用于带标签的参数。再次考虑标记版本的 `add`：

```ocaml
# let add = (~x, ~y) => x + y;
let add: (~x: int, ~y: int) => int = <fun>;
```

如果我们只用第一个标签参数调用 `add`，我们得到一个将第二个参数映射到结果的函数：

```ocaml
# add(~x=4);
- : (~y: int) => int = <fun>
```

仅提供第二个标记的参数类似地起作用。

```ocaml
# add(~y=4);
- : (~x: int) => int = <fun>
```

也就是说，标签不会在这里强制下单。这意味着部分应用程序对于标签​​更具多功能性，因为您可以部分应用任何标记的参数，而不仅仅是最后一个。

#### 部分应用和可选参数

可选参数如何?以下版本的 `add` 只有可选参数:

```ocaml
# let add = (~x=0, ~y=0, ()) => x + y;
let add: (~x: int=?, ~y: int=?, unit) => int = <fun>;
```

如果仅提到标签 `〜x` 或仅标签 `〜y`，则部分应用程序与以前一样工作，但有一点不同：`unit` 类型附加位置参数也必须填写。

```ocaml
# add(~x=3);
- : (~y: int=?, unit) => int = <fun>
# add(~y=3);
- : (~x: int=?, unit) => int = <fun>
```

但是，一旦您提到了位置参数，就不再有部分应用程序;默认值现在填充:

```ocaml
# add(~x=3, ());
- : int = 3
# add(~y=3, ());
- : int = 3
```

即使你采取一个或两个中间步骤，`()` 始终是评估的最终信号。一个中间步骤如下所示。

```ocaml
# let plus5 = add(~x=5);
let plus5: (~y: int=? unit) => int = <fun>;
# plus5(());
- : int = 5
```

两个中间步骤：

```ocaml
# let plus5 = add(~x=5);
let plus5: (~y: int=?, unit) => int = <fun>;
# let result8 = plus5(~y=5);
let result8: (unit) => int = <fun>;
# result8(());
- : int = 8
```

### 柯里（高级）

柯里化是一种实现位置参数部分应用的技术。柯里化函数意味着将其从具有1或更多元数的函数转换为一系列一元函数调用。

例如，采取二进制函数 `add` ：

```ocaml
let add = (x, y) => x + y;
```

柯里化 `add` 意味着将其转换为以下函数：

```ocaml
let add = x => y => x + y;
```

现在我们必须像如下这样调用 `add`：

```ocaml
# add(3)(1);
- : int = 4
```

我们获得了什么？部分应用现在很容易：

```ocaml
# let plus4 = add(4);
let plus4: (int) => int = <fun>;
# plus4(7);
- : int = 11
```

值得惊喜的是：在 ReasonML 中所有函数都会自动柯里化。这就是它支持部分应用程序的方式。 你可以看到，如果你看看柯里化 `add` 的类型：

```ocaml
# let add = x => y => x + y;
let add: (int, int) => int = <fun>;
```

换句话说：`add(x, y)` 与 `add(x)(y)` 相同，以下两种类型是等价的：

```ocaml
(int, int) =>int
int => int =>int
```

让我们以一个函数来结束二进制函数。 考虑到已经 `curried` 的 `currying` 函数是毫无意义的，我们将讨论一个单参数为一对的函数。

```ocaml
let curry2 = (f:(('a, 'b)) => 'c) => x => y => f((x, y));
```

让我们将 `curry2` 与一个一元版本的 `add` 一起使用：

```ocaml
# let add = ((x, y)) => x + y;
let add: ((int, int)) => int = <fun>;
# curry2(add);
- : (int, int) => int = <fun>
```

最后的类型告诉我们，我们已经创建了一个二元函数。

### 反向应用程序运算符（|>）

操作符 `|>` 被称为反向应用程序操作符或管道操作符。它让你链函数调用: `x |> f `与 `f(x)` 相同。这看起来不太像，但是在组合函数调用时非常有用。

#### 管道整型和字符串

让我们从一个简单的例子开始。给定以下两个函数：

```ocaml
let times2 = (x: int) => x * 2;
let twice = (s: string) => s ++ s;
```

如果我们将它们用于传统的函数调用，我们会得到：

```ocaml
# twice(string_of_int(times2(4)));
- : string = "88"
```

首先，我们应用 `times2` 到 `4` ，然后将 `string_of_int`（标准库中的函数）应用到结果等。管道运算符让我们编写更接近我刚刚给出的描述的代码：

```ocaml
let result = 4 |> times2 |> string_of_int |> twice;
```

#### 例如：管道列表

用更复杂的数据和 `currying`，我们得到一种让人联想到面向对象编程中的链式方法调用的风格。

例如，以下代码使用 `ints` 列表：

```ocmal
[4, 2, 1, 3, 5]
|> List.map(x => x + 1)
|> List.filter(x => x < 5)
|> List.sort(compare);
```

这些函数在列表的章节中解释。现在，对它们的工作方式有一个大概的了解就足够了。

三个计算步骤是：

```ocaml
# let l0 = [4, 2, 1, 3, 5];
let l0: list(int) = [4, 2, 1, 3, 5];
# let l1 = List.map(x => x + 1, 10);
let l1: list(int) = [5, 3, 2, 4, 6];
# let l2 = List.filter(x => x < 5, l1);
let l2: list(int) = [3, 2, 4];
# let l3 = List.sort(compare, l2);
let l3: list(int) = [2, 3, 4];
```

我们看到，在所有这些功能中，主要参数都是最后一个。当我们管道时，我们首先通过部分应用填充次要参数，创建一个函数。

然后管道操作员通过调用该函数来填充主参数。

主要参数与面向对象编程语言中的 `this` 或者 `self` 类似。

### 设计函数签名的技巧

以下是设计函数类型签名的一些技巧:

- 如果函数具有单个主参数，则将其作为位置参数并放在最后。这支持管道操作员的函数组成。
- 一些函数具有多个类似的主要参数。最后将它们转换为多个位置参数。一个例子是将两个列表连接成一个列表的函数。在这种情况下，两个位置参数都是列表。
- 所有其他参数应该为标签参数。
- 如果有两个或多个不同的主参数，那么所有这些参数都应该为标签参数。
- 如果一个函数只有一个参数，那么它就趋向于没有标记，即使它不是严格的主函数。

这些规则背后的想法是使代码尽可能自描述：主要（或唯一）参数由函数名称描述，其余参数由其标签描述。

只要函数具有多个位置参数，通常很难分辨每个参数的作用。例如，比较以下两个函数调用。第二个更容易理解。

```ocaml
blit(bytes, 0, bytes, 10, 10);
blit(~src=bytes, ~src_pos=0, ~dst=bytes, ~dst_pos=10, ~len=10);
```

我也喜欢可选参数，因为它们使您能够在不破坏现有调用者的情况下向函数添加更多参数。这有助于不断发展的API。

本节的来源：Sect。 OCaml手册中的“[标签建议](https://caml.inria.fr/pub/docs/manual-ocaml/lablexamples.html#sec45)”。

### 单参数匹配函数

ReasonML为一元函数提供了一个缩写，立即切换其参数。以下面的函数为例。

```ocaml
let divTuple = (tuple) =>
  switch tuple {
    | (_, 0) => (-1)
    | (x, y) => x / y
  };
```

该函数使用如下：

```ocaml
# divTuple((9, 3));
- : int = 3
# divTuple((9, 0));
- : int = -1
```

如果您使用 `fun` 操作符来定义 `divTuple`，则代码将变得更短：

```ocaml
let divTuple =
  fun
  | (_, 0) => (-1)
  | (x, y) => x / y;
```

### （高级）

所有其余部分都是高级的。

### 操作符

ReasonML的一个很好的特点是操作符也是函数。如果将它们放在圆括号中，可以将它们用作函数：

```ocaml
# (+)(7, 1);
- : int = 8
```

你可以定义你自己的操作符：

```ocaml
# let (+++) = (s, t) => s ++ " " ++ t;
let (+++): (string, string) => string => <fun>;
# "hello" +++ "world";
- : string = "hello world"
```

通过将操作符放在括号中，您还可以轻松查找其类型：

```ocaml
# (++);
- : (string, string) => string = <fun>
```

#### 操作符的规则

有两种操作符：中缀操作符（在两个操作数之间）和前缀操作符（在单个操作数之前）。

以下操作符可以用于这两种操作符：

```ocaml
! $ % & * + - . / : < = > ? @ ^ | ~
```

中缀操作符：

第一个字符 | 随后是运算符字符
------------ | -------------
= < > @ ^ ❘ & + - * / $ % | 0+
 # | 1+

此外，以下关键字是中缀操作符：

```ocaml
* + - -. == != < > || && mod land lor lxor lsl lsr asr
```

前缀运算符：

第一个字符 | 随后是运算符字符
------------ | -------------
! | 0+
? ~ | 1+

此外，以下关键字是前缀运算符：

```ocaml
- -.
```

本节的来源：Sect。 “OCaml手册”中的“[前缀和中缀符号](https://caml.inria.fr/pub/docs/manual-ocaml/lex.html#infix-symbol)”。

#### 操作符的优先级和相关性

下表列出了运算符及其相关性。运算符越高，其优先级越高（绑定越强）。例如，`*` 的优先级高于 `+`。

构造或操作符 |	关联性
------------ | -------------
prefix operator|	–
. .( .[ .{	| –
[ ] (array index)	| –
 #···	| –
applications, assert, lazy	| left
- -. (prefix)	| –
 **··· lsl lsr asr	 | right
 *··· /··· %··· mod land lor lxor	| left
+··· -···	| left
@··· ^···	| right
=··· <··· >··· ❘··· &··· $··· !=	| left
&&	| right
❘❘	| right
if	| –
let switch fun try	| –
