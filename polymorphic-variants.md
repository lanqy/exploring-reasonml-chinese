# 多态变体类型

译自：http://reasonmlhub.com/exploring-reasonml/ch_polymorphic-variants.html

在本章中，我们看看多态变体，这是一个更灵活的正常变体版本。但这种灵活性也使得它们更加复杂。

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