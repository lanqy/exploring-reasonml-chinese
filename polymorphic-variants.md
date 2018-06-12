# 多态变体类型

> 在本章中，我们看看多态变体，这是一个更灵活的正常变体版本。但这种灵活性也使得它们更加复杂。

## 什么是多态变体？

多态性变体与正常变体相似。最大的区别是构造器不再与类型绑定;他们独立存在。这使得多态变体更加通用，并为类型系统带来了有趣的后果。其中一些后果是消极的 - 它们是您为多功能性付出的代价。

让我们从以下正常变体开始，然后将其变为多态变体。

```ocaml
type rgb = Red | Green | Blue;
```

`rgb` 的多态版本定义如下：

```ocaml
type rgb = [`Red | `Green | `Blue];
```

多态变体的构造函数必须用方括号包装，并且它们的名称必须以反引号开头，后跟小写字母或大写字母：

```ocaml
# `red;
- : [> `red ] = `red

# `Red;
- : [> `Red ] = `Red
```

像往常一样，类型如 rgb 必须以小写字母开头。

#### 多态构造函数是独立存在的

如果它是非多态变体的一部分，则只能使用非多态构造函数：

```ocaml
# Int(123);

错误：未绑定的构造函数 Int

# type data = Int(int) | Str(string);
type data = Int(int) | Str(string);

# Int(123);
- : data = Int(123)
```

相反，你可以使用多态的构造函数。事先不需要定义它们：

```ocaml
# `Int(123);
- : [> `Int(int) ] = `Int(123)

```

请注意 `Int(123) 有一个有趣的类型：[> `Int(int) ]。我们将在后面讨论这意味着什么。

多态构造函数独立存在，可以多次使用相同的构造函数：

```ocaml
type color = [`Red | `Orange | `Yellow | `Green | `Blue | `Purple];

type rgb = [`Red | `Green | `Blue];

```

相反，对于正常变体，您应该避免在具有相同构造函数的同一范围内的多个变体。

您甚至可以使用具有不同 arities 和/或类型参数的相同多态构造函数：

```ocaml
# `Int("abc", true);
- : [`Int((string, bool)) ] = `Int(("abc", true))

# `Int(1.0, 2.0, 3.0);
- : [> `Int((float, float, float)) ] = `Int((1., 2., 3.))
```

#### 扩展多态变体

定义多态变体时，可以扩展现有变体。在下面的例子中，`color` 扩展了 `rgb` ：

```ocaml
type rgb = [`Red | `Green | `Blue];
type color = [rgb | `Orange | `Yellow | `Purple];
```

正如你所看到的，这里的情况很重要： `rgb` 被小写表示它的构造函数被插入。

扩展多个变体也是可能的：

```ocaml
type red = [ `Red ];
type green = [ `Green ];
type blue = [ `Blue ];
type rgb = [red | green | blue];
```

#### 多态构造函数的名称空间是全局的

非多态构造函数的名称是它们作用域的一部分（例如它们的模块），而多态构造函数的名称空间是全局的。你可以在下面的例子中看到：

```ocaml
module M = {
    type data = [`Int(int) | `Str(str) ];
    let stringOfData = (x: data) =>
        switch x {
            | `Int(i) => string_of_int(i)
            | `Str(s) => s
        };
};

M.stringOfData(`Int(123));
```

在最后一行中，`M.stringOfData()` 的参数是通过一个不属于 `M` 作用域的构造函数创建的。由于多态构造函数的全局名称空间，参数与类型数据兼容。

#### 类型兼容性

ReasonML 如何确定实际参数（函数调用）的类型是否与形式参数（函数定义）的类型兼容？我们来看看例子。我们首先创建一个类型和一个单一参数为该类型的函数。

```ocaml
# type rgb = [`Red | `Green | `Blue];
type rgb = [`Blue | `Green | `Red ];

# let id = (x: rgb) => x;
let id: (rgb) => rgb = <fun>;
```

接下来，我们创建一个新类型的 `rgb2` ，它具有与 `rgb` 相同的多态构造函数，并调用 `id()` 的值为 `rgb2` ：

```ocaml
# type rgb2 = [`Red | `Green | `Blue];
type rgb2 = [`Blue | `Green | `Red];

# let arg: rgb2 = `Blue;
let arg: rgb2 = `Blue;

# id(arg);
- : rgb = `Blue

```

通常情况下，形式参数 `x` 的类型和实际参数 `arg` 必须相同。但是对于多态变体，如果你的类型具有相同的构造函数就足够了。

但是，如果实际参数 `arg2` 的类型具有较少的构造函数，那么 ReasonML 将不会让您执行函数调用：

```ocaml
# type rg = [`Red | `Green];
type rg = [`Red | `Green ];

# let arg2: rg = `Red;
let arg2: rg = `Red;

# id(arg2);
错误：该表达式的类型为 rg，但表达式的类型为 rgb 第一种变体类型不允许使用 tag(s) `Blue

```

原因是 id 对它的需求做出了精确的要求：确切地说是构造函数 `Red，`Green 和 `Blue。因此，`rg` 类型是不够的。

#### 从具体类型到类型约束

有趣的是，下面的工作是有效的：

```ocaml
# type rgb = [`Red | `Green | `Blue];
type rgb = [`Blue | `Green | `Red ];

# let id = (x: rgb) => x;
let id: (rgb) => rgb = <fun>;

# let arg3 = `Red;
let arg3: [> `Red] = `Red;

# id(arg3);
- : rgb = `Red
```

为什么？由于 ReasonML 没有推断出 `arg3` 的固定类型，因此推断出类型约束 [> `Red ]。这个约束是一个所谓的下界，意思是：“所有至少具有构造函数 `Red 的类型”。该约束与 `x` 的类型 `rgb` 兼容。

#### 参数的类型约束

我们也可以使用约束作为参数的类型。例如，以下对 id() 的调用失败，因为参数的类型 `rgbw` 具有太多的构造函数：

```ocaml
# type rgbw = [`Red | `Green | `Blue | `White ];
type rgbw = [`Blue | `Green | `Red | `White ];

错误：此表达式的类型为 rgbw，但表达式的类型为 rgb 第二种变体类型不允许使用 tag(s) `White
```

这一次，我们调用 `id()` 并直接指定参数的类型 `rgbw` （没有中间 `let` 绑定）。我们可以通过改变 id() 来使函数调用工作而不改变函数调用：

```ocaml
# let id = (x: [> `Red | `Green | `Blue]) => x;
let id: (([> `Blue | `Green | `Red ] as 'a)) => 'a = <fun>

# id(`Red: rgbw); /* 跟前面一样调用 */

- : rgbw = `Red

```

现在 `x` 有一个下界，并接受所有至少有给定三个构造函数的类型。

您也可以通过引用变体来定义约束。例如，以下 id() 的定义基本上等同于前一个。

```ocaml
let id = (x: [> rgb]) => x;
```

稍后我们将深入研究类型约束。

## 用多态变量编写可扩展的代码

多态变体的一个主要优点是代码变得更具可扩展性。

#### 我们想要扩展的类型和代码

作为一个例子，为形状采取以下类型定义。它们是变体章节中示例的多态版本。

```ocaml
type point = [`Point(float, float) ];
type shape = [
    | `Rectangle(point, point)
    | `Circle(point, float)
];

```

基于这些类型定义，我们可以编写一个函数来计算形状的面积：

```ocaml
let pi = 4.0 *. atan(1.0);
let computeArea = (s: shape) => 
    switch s {
        | `Rectangle(`Point(x1, y1), `Point(x2, y2)) =>
            let width = abs_float(x2 -. x1);
            let height = abs_float(y2 -. y1);
            width * height;
        | `Circle(_, radius) => pi *. (radius ** 2.0)
    };

```

#### 延伸 `shape`：失败的第一次尝试

假设我们想要扩展 `shape` - 三角形。我们将如何做到这一点？我们可以简单地定义一个新的类型 `shapePlus` ，它重用现有的多态构造函数 `Rectangle` 和 `Circle` ，并添加构造函数 `Triangle` ：


```ocaml
type shapePlus = [
    | `Rectangle(point, point)
    | `Circle(point, float)
    | `Triangle(point, point, point)
];
```

现在我们还需要扩展 `computeArea()` 。下面的函数是我们写这个扩展的第一次尝试：

```ocaml
let shoelaceFormula = (`Point(x1, y1), `Point(x2, y2), `Point(x3, y3)) =>
    0.5 *. abs_float(x1 *. y2 -. x3 *. y2 +. x3 *. y1 -. x1 *. y3 +. x2 *. y3 -. x2 *. y1);

let computeAreaPlus = (sp: shapePlus) => 
    switch sp {
        | `Triangle(p1, p2, p3) => shoelaceFormula(p1, p2, p3)
        | `Rectangle(_, _) => computeArea(sp) /* A */
        | `Circle(_, _) => computeArea(sp) /* B */
    };
```

唉，这段代码不起作用：在 A 行和 B 行， `sp` 的类型 `shapePlus` 与 `computeArea` 参数的类型形状不兼容。我们收到以下错误消息：

错误：此表达式具有类型 shapePlus 但预期表达形式 第二种变体类型不允许使用 tag(s) `Triangle`

#### 通过 as 子句修复 computeAreaPlus

谢天谢地，我们可以通过在最后两种情况下使用 `as` 子句来解决这个问题：

```ocaml
let computeAreaPlus = (sp: shapePlus) =>
    switch sp {
        | `Triangle(p1, p2, p3) => shoelaceFormula(p1, p2, p3)
        | `RectTangle(_, _) as r => computeArea(r)
        | `Circle(_, _) as c => computeArea(c)
    };
```

`as` 如何帮助我们？通过多态变体，它可以选择最普通的类型。那是：

- `r` 的类型是 [> `Rectangle(point, point) ]
- `c` 的类型是[> `Circle(point, float) ]

这两种类型都与 `computeArea` 参数的类型（`shape`）兼容。

#### 多态变体缩写模式

我们还可以做出更多改进。目前，我们提到每个类型 `shape` 的构造函数：

```ocaml
| `Rectangle(_, _) as r => computeArea(r)
| `Circle(_, _) as c => computeArea(c)
```

这是不必要的多余的;更大的类型更是如此。因此，ReasonML 为我们提供了一个缩写：

```ocaml
| #shape as s => computeArea(s)
```

这涵盖了 `shape` 的所有构造函数。

一般来说，只要你有一个如下所示的变体：

```ocaml
type myvariant = [`C1(t1) | `C2(t2)];
```

然后你可以使用这种模式：

```ocaml
#myvariant
```

它是以下的缩写：

```ocaml
(`C1(_: t1) | `C2(_: t2))
```

换句话说，`#myvariant` 匹配 `[< myvariant]` 类型的所有值。

使用 `#shape` ，我们的代码如下所示：

```ocaml
let computeAreaPlus = (sp: shapePlus) =>
  switch sp {
  | `Triangle(p1, p2, p3) => shoelaceFormula(p1, p2, p3)
  | #shape as s => computeArea(s)
  };
```

让我们使用具有两种形状的 `computeAreaPlus()` ：

```ocaml
let top = `Point(3.0, 5.0);
let left = `Point(0.0, 0.0);
let right = `Point(3.0, 0.0);

let circ = `Circle(top, 3.0);
let tri = `Triangle(top, left, right);

computeAreaPlus(circ); /* 28.274333882308138 */
computeAreaPlus(tri); /* 7.5 */
```

因此，我们成功地扩展了它的类型形状和函数 `computeArea()` 。

## 最佳实践：正常变体与多态变体

一般来说，您应该更喜欢正态变体而不是多变体变体，因为它们的效率稍高一些，并且可以实现更严格的类型检查。引用 OCaml 手册：

> 多态变体虽然是类型安全的，但却导致了较弱的类型规则。也就是说，核心语言变体实际上做的不仅仅是确保类型安全，它们还检查您只使用声明的构造函数，检查数据结构中出现的所有构造函数都是兼容的，并且它们对它们的参数执行类型约束。


然而，多态变体有一些明显的优势。如果在给定情况下有任何问题，则应使用多态变体：

- 重用：构造函数（可能随代码处理它）对于多个变体很有用。例如，颜色的构造器就属于这个类别。
- 解耦：构造函数用于多个位置，但不希望这些位置依赖于定义构造函数的单个模块。相反，您可以简单地使用构造函数而不用定义它。
- 可扩展性：您期望稍后扩展一个变体。与本章前面的 `shapePlus` 是 `shape` 的扩展相似。
- 简洁性：由于多态构造函数的全局名称空间，您可以使用它们而无需限定它们或打开它们的模块（请参阅下一小节）。
- 使用没有事先定义的构造函数：您可以使用多态构造函数，而无需事先通过变体定义它们。这对于仅用于单个位置的丢弃类型偶尔很方便。

值得庆幸的是，如果需要的话，从一个正常变体转向多态变体相对容易。

## 简洁性：正常变体与多态变体

我们来比较正常变体和多态变体的简洁性。

对于常规变体 `bwNormal` ，您需要限定A行中的黑色（或打开其模块）：

```ocaml
module MyModule = {
    type bwNormal = Black | White;

    let getNameNormal(bw: bwNormal) =
        switch bw {
            | Black => "Black"
            | White => "While"
        };
};

print_string(MyModule.getNameNormal(MyModule.Black)) /* A */

/* "Black" */

```

对于多态变体 bwPoly ，对于 A 中的黑色不需要进行限定：

```ocaml
module MyModule = {
    type bwPoly = [ `Black | `White ];

    let getNamePoly(bw: bwPoly) = 
        switch bw {
            | `Black => "Black"
            | `White => "White"
        };
};

print_string(MyModule.getNamePoly(`Black)); /* A */

/* "Black" */

```

在这个例子中，不需要限定条件并没有太大的作用，但是如果你多次使用构造函数就很重要。

## 防止使用多态变体的拼写错误

多态变体的一个问题是，你会得到更少的关于拼写错误的警告，因为你可以在不定义它们的情况下使用构造函数。例如，在下面的代码中，`Green 在 A 行中拼写为 `Gren：

```ocaml
type rgb = [ `Red | `Green | `Blue ];
let f = (x) => 
    switch x {
        | `Red => "Red"
        | `Gren => "Green" /* A */
        | `Blue => "Blue"
    }

    /* lef f: ([< `Blue | `Gren | `Red ]) => string = <fun>; */

```

如果为参数 `x` 添加类型注释，则会发出警告：

```ocaml
let f = (x: rgb) => 
    switch x {
        | `Red => "Red"
        | `Gren => "Green"
        | `Blue => "Blue"
    };

    /*
    错误：此模式匹配 [? `Gren ] 
    但预期与 rgb 类型的值匹配的模式 
    第二种变体类型不允许使用 tag(s) `Gren
    */
```

如果您从函数返回多态变量值，则也可以指定该函数的返回类型。但是这也增加了你的代码量，所以确保你真的受益。如果输入所有参数，则应该捕捉许多（如果不是大多数）问题。

## （高级）

> 以下所有部分都包含高级主题。

#### 类型变量的类型约束

在我们仔细研究多态变体的约束之前，我们首先需要了解类型变量的一般约束（前者是一种特殊情况）。

在类型定义的末尾，可以有一个或多个类型约束。这些语法如下：

```ocaml
constraint «typeexpr» = «typeexpr»
```

这些约束用于改进前面类型定义中的类型变量。这是一个简单的例子：

```ocaml
type t('a) = 'a constraint 'a = int;
```

如何处理约束？在我们研究这个之前，我们先来了解 `unification` 以及它如何构建模式匹配。

模式匹配沿着一个方向进行：一个没有变量的词用于填充另一个词中的变量。在下面的例子中，`Int(123) 是没有变量的术语，`Int(x) 是包含变量的术语：

```ocaml
switch(`Int(123)) {
    | `Int(x) => print_int(x)
}
```

`unification` 是模式匹配，可以在两个方向上工作：两个术语都可以有变量，并且双方的变量都被填充。作为示例，请考虑：

```ocaml
# type t('a, 'b) = ('a, 'b) constraint ('a, int) = (bool, 'b);
type t('a, 'b) = ('a, 'b) constraint 'a = bool constraint 'b = int;
```

尽可能简化 ReasonML ：将等号两边变量的原始复杂约束转换为仅在左侧具有变量的两个简单约束。

这是一个可以简化事物的例子，不再需要约束：

```ocaml
# type t('a, 'b) = 'c constraint 'c = ('a, 'b);
type t('a, 'b) = ('a, 'b);
```

## 多态变体的类型约束

我们在本章前面看到的类型约束实际上只是多态变量特有的类型约束。

例如，以下两个表达式是等价的：

```ocaml
let x: [> `Red ] = `Red;
let x: [> `Red] as 'a = `Red;
```

另一方面，以下两个类型表达式也是等价的（但不能在 let 绑定和参数定义中使用约束）：

```ocaml
[> `Red ] as 'a
'a constraint 'a = [> `Red ]
```

也就是说，对于迄今为止我们使用的所有多态变体约束，总是存在一个隐式（隐藏）类型变量。我们可以看到，如果我们尝试使用这样的约束来定义类型 t ：

```ocaml
# type t = [> `Red ];
错误:类型变量在此类型声明中未绑定。
在类型[> `Red]中，作为变量的 'a 是未绑定的
```

我们如下解决这个问题。请注意由 ReaonML 计算​​出的最终类型。

```ocaml
# type t('a) = [> `Red ] as 'a;
type t('a) = 'a constraint 'a = [> `Red ];
```

## 多态变体的上下界

对于本章的其余部分，我们将多态变体的类型约束简称为类型约束或约束。

类型约束包含以下任一个或两个：

- 下界：指示类型必须至少包含哪些元素。例如：[> `Red | `Green] 接受所有包含构造函数 `Red 和 `Green 的类型。换句话说：作为一个集合，约束接受所有类型的超集。
- 上界：指示类型最多必须包含的元素。例如：[< `Red | `Green] 接受以下类型：[ `Red | `Green]，[ `Red ]，[ `Green ]。

您可以使用类型约束：

- 类型定义
- `let` 绑定
- 参数定义

对于后两者，您必须使用简写形式（无约束）。

## 类型约束匹配什么?

#### 下界

让我们来看看下界 [> `Red | `Green] 通过使用它作为函数参数的类型 x：

```ocaml
let lower = (x: [> `Red | `Green]) => true;
```

可以接受的类型值 [`Red | `Green | `Blue] 和 [`Red | `Green]

```ocaml
# lower(`Red: [`Red | `Green | `Blue]);
- : bool = true

# lower(`Red: [`Red | `Green]);
- : bool = true
```

然而，类型 [`Red] 的值不被接受，因为该类型不包含约束的两个构造函数。

```ocaml
# lower(`Red: [`Red]);
错误：此表达式有类型 [ `Red ]
但预期的表达类型 [> `Green | `Red]
第一种变体类型不允许 tag(s) `Green
```

#### 上界

下面的交互实验的上界 [< `Red | `Green]:

```ocaml
# let upper = (x: [< `Red | `Green]) => true;
let uppder: ([< `Green | `Red ]) => bool = <fun>;

# upper(`Red: [`Red | `Green]); /* OK */
- : bool = true

# upper(`Red: [`Red]); /* OK */
- : bool = true

# upper(`Red: [`Red | `Green | `Blue]);
错误：此表达式有类型 [ `Blue | `Green | `Red ]
但预期的表达类型 [< `Green | `Red ]
第二种变体类型不允许 tag(s) `Blue 
```

## 推断的类型约束

如果使用多态构造函数，ReasonML 将为您推断类型约束。我们来看几个例子。

## 下界

```ocaml
# `Int(3);
- : [> `Int(int) ] = `Int(3)
```

值 `Int(3) 具有推断类型 [> `Int(int) ] ，它与至少具有构造函数 `Int(int) 的所有类型兼容。

使用元组，您可以得到两个单独的推断类型约束：

```ocaml
# (`Red, `Green);
- : ([> `Red ], [> `Green ]) = (`Red, `Green)
```

另一方面，列表中的元素必须都具有相同的类型，这就是合并两个推断约束的原因：

```ocaml
# [`Red, `Green];
- : list([> `Green | `Red ]) = [`Red, `Green]
```

只要预期类型是一个元素列表，其类型至少包含构造函数 `Red 和 `Green，就可以接受此列表。

如果您尝试使用具有不同类型参数的相同构造函数，则会出现错误，因为 ReasonML 无法合并两个推断类型：

```ocaml
# [`Int(3), `Int("abc")];
错误：此表达式具有类型字符串但是 预期类型为 int 的表达式
```

#### 上界

到目前为止，我们只看到下界被推断。在以下示例中，ReasonML 推断参数 x 的上界：

```ocaml
let f = (x) => 
    switch x {
        | `Red => 1
        | `Green => 2
    };
    /* let f: ([< `Green | `Red ]) => int = <fun>; */
```

由于 switch 表达式，f 最多可以处理两个构造函数 `Red 和 `Green。

如果 f 返回其参数，则推断的类型变得更复杂：

```ocaml
let f = (x) => 
    switch x {
        | `Red => x
        | `Green => x
    };
/* let f: (([< `Green | `Red ] as 'a)) => 'a = <fun>; */
```

类型参数 'a 用于表示参数的类型和结果的类型相同。

如果我们使用 as 子句，情况就会改变，因为它们将输入类型与输出类型分离开来：

```ocaml
let f = (x) => 
    switch x {
        | `Red as r => r
        | `Green as g => g
    };

/* let f: ([< `Green | `Red ]) => [> `Green | `Red ] = <fun>; */
```

#### ReasonML 的类型系统的限制

有些事情超出了 ReasonML 的类型系统的能力：

```ocaml
let f = (x) => 
    switch x {
        | `Red => x
        | `Green => `Blue
    };
/*
错误：此表达式有类型 [> `Blue ]
但预期的表达类型 [< `Green | `Red ]
第二种变体类型不允许 tag(s) `Blue
*/
```

f 的返回类型是：“ x 的类型或类型 [> `Blue] ”。但是，无法通过约束来表达这一点。

#### 更复杂的约束

类型约束可能变得相当复杂。我们从两个简单的函数开始：

```ocaml
let even1 = (x) =>
    switch x {
        | `Data(n) => (n mod 2) == 0
    };
/*
let even1: ([< `Data(int) ]) => bool = <fun>;
*/

let even2 = (x) =>
    switch x {
        | `Data(s) => (String.length(s) mod 2) == 0
    };

/* let even2: ([< `Data(string) ]) => bool = <fun>; */

```

这两种推断类型都包含由 switch 语句引起的上界。

我们使用相同的变量 `x` 作为两个函数的参数：

```ocaml
let even = (x) => even1(x) && even2(x);
/* let even: ([< `Data(string & int)]) => bool = <fun>; */
```

对于 `x` 类型，ReasonML会合并以下两种类型：

```ocaml
[< `Data(int) ]
[< `Data(string) ]
```

结果包含以下构造函数。

```ocaml
`Data(string & int)
```

它代表“构造函数 `Data ”，其类型参数既有 `string` 类型又有 `int` 类型。这样的构造函数不存在，这意味着你不能调用 `even()`，因为没有与其参数类型兼容的值。

## 类型推断使用统一（这是双向的）

当 ReasonML 通过推理计算类型，并确定两个类型 `t1`，`t2` 必须相等（例如实际参数的类型和形式参数的类型）时，它使用统一来求解方程 `t1 = t2`。

我将通过几个函数来演示。每个函数都返回唯一的参数。它为参数指定的类型 tp 与它为结果指定的类型 tr 不同。 ReasonML 将尝试统一 tp 和 tr，让我们观察到统一是双向的。

#### 两个约束

如果参数类型和结果类型都是约束，则 ReasonML 会尝试合并约束。

```ocaml
# let f = (x: [>`Red]): [`Red | `Green] => x;
let f: ([`Green | `Red]) => [`Green | `Red] = <fun>;
# let f = (x: [`Red]): [< `Red | `Green] => x;
let f: ([ `Red ]) => [ `Red ] = <fun>;

```

## 两种单形类型

如果两种类型都是单形的，那么它们必须具有相同的构造函数。

```ocaml
# let f = (x: [`Red]): [`Red | `Green] => x;
错误：此表达式具有类型 [`Red]
但预期的表达类型 [ `Green | `Red ]
第一种变体类型不允许 tag(s) `Green

# let f = (x: [`Red | `Green]): [`Red | `Green] => x;
let f: ([`Green | `Red ])  => [`Green | `Red ] = <fun>;
```

## 单形类型与约束

单形态和约束之间的区别的另一个证明。考虑以下两个功能：

```ocaml
type rgb = [`Red | `Green | `Blue];

let id1 = (x: rgb) => x'
/* let id1: (rgb) => rgb = <fun>; */

let id2 = (x:[> rgb]) => x;
/* let id2: (([> rgb ] as 'a)) => 'a = <fun>; */
```

我们来比较这两个函数：

- id1() 有一个参数，其类型 rgb 是单形的（它没有类型变量）。
- id2() 有一个参数，其类型 [> rgb] 是一个约束。定义本身看起来不是多态的，但计算出的签名确实存在 - 现在有一个类型变量 'a。

让我们来看看如果我们用参数的类型是一个新的多态变量来调用这些函数会发生什么，该变量具有与 rgb 相同的构造函数：

```ocaml
type c3 = [ `Blue | `Green | `Red ];
```

使用id1()，x 的类型以及结果是固定的。也就是说，它保持 rgb ：

```ocaml
# id1(`Red: c3);
- : rgb = `Red
```

我们只能调用id1()，因为 rgb 和 c3 是相同的。

相反，使用id2()，x 的类型是多态的，并且更加灵活。统一时，'a 必然会 c3。函数调用的结果是类型 c3（'a 的值）。

```ocaml
# id2(`Red: c3);
- : c3 = `Red
```

## 资料

多态变体以及如何使用它们：

- OCaml手册中的“[多态变体](http://caml.inria.fr/pub/docs/manual-ocaml/lablexamples.html#sec46)” 章节。
- [Daniel Bünzli关于何时使用多态变体的技巧](https://stackoverflow.com/questions/9367181/variants-or-polymorphic-variants/9367778#9367778)（Stack Overflow）
- [通过多态变体重复使用代码](https://www.math.nagoya-u.ac.jp/~garrigue/papers/fose2000.html) Jacques Garrigue（2000）。

类型变量约束：

- [OCaml中的“constraint”关键字可以做什么？](https://stackoverflow.com/questions/24490648/what-can-be-done-with-the-constraint-keyword-in-ocaml)（Stack Overflow）
- [为什么约束子句存在于类型定义中？](https://www.reddit.com/r/ocaml/comments/1ss987/why_does_the_constraint_clause_exist_in_type/)
- 张宏波 “OCaml书”中的“[多态变体](https://github.com/bobzhang/ocaml-book/blob/master/lang/features.org)”章节。
- OCaml手册中的“[类型定义](https://caml.inria.fr/pub/docs/manual-ocaml/typedecl.html)”章节。

多态变体的语义：

- [结构多态性的简单类型推断](http://caml.inria.fr/pub/papers/garrigue-structural_poly-fool02.pdf)[PDF] Jacques Garrigue （2000）。
- [行多态性](https://www.cl.cam.ac.uk/teaching/1415/L28/rows.pdf)[PDF]  Leo White （2015）。