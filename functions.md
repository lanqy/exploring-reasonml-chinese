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