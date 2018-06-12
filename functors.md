# 函子（高级）

> 直观上，函子是一个函数，它将一个或多个模块作为输入并返回一个新模块。本章介绍仿函数的工作原理以及为什么它们有用。

这些例子的代码可以在 GitHub 上的 repository [reasonml-demo-functors](https://github.com/rauschma/reasonml-demo-functors) 中找到。

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

这就是函子的样子：

```ocaml
/* Repetition.re */

module Make = (RC: Count) => {
  let rec repeat = (~times=RC.count, str: string) =>
    if (times <= 0) {
      "";
    } else {
      str ++ repeat(~times=times - 1, str);
    }
}

```

Make 是一个仿函数：它的参数 RC 是一个模块，其结果（在花括号中）也是一个模块。

`repeat()` 是一个相对枯燥的 ReasonML 函数。唯一的新方面是 `~times` 的默认值，它来自参数模块 `RC` 。

#### Repetition.Make 的结果接口

`Repetition.Make` 目前不定义其结果的类型。这意味着 ReasonML 推断出这种类型。如果我们想要更多的控制，我们可以为结果定义一个接口 `S`：

```ocaml
/* Repetition.re */

module type S = {
  let repeat: (~times: int=?, string) => string;
};
```

之后，我们只需要在 `Make` 的定义中添加 `S` ：

```ocaml
module Make = (RC: Count): S => ...
```

#### 使用 Repetition.Make

接下来，我们使用 `Repetition.Make` 创建一个单独的文件 `RepetitionMain.re` 。首先，我们为 `functor` 的参数定义一个模块：

```ocaml
/* RepetitionMain.re */

module CountThree: Repetition.Count = {
  let count = 3;
};
```

只要 `CountThree` 具有与该接口完全相同的结构，就不需要类型 `Repetition.Count`。但它立即清楚， `CountThree` 的目的是什么。

现在我们准备使用函子 `Repetition.Make` 为我们创建一个模块：

```ocaml
/* RepetitionMain.re */

module RepetitionThree = Repetition.Make(CountThree);
```

下面的代码调用 `RepetitionThree.repeat`，它按预期工作：

```ocaml
/* RepetitionMain.re */

let () = print_string(RepetitionThree.repeat("abc\n"));

/* 输出:
abc
abc
abc
*/
```

分开定义模块 `CountThree` ，我们也可以将其内联：

```ocaml
module RepetitionThree = Repetition.Make({
  let count = 3;
})
```

#### 模块 Repetition 的结构

我们拥有结构化模块 Repetition 的方式是使用函子的常用模式。它有以下几部分：

- Make：函子。
- Make 的参数的一个或多个接口（在我们的例子中为 `Count` ）。
- S：`Make` 的结果接口。

也就是说， `Repetition` 将函子 `Make` 和它需要的所有东西封装起来。

## 数据结构的函子

函子的一个常见用例是实现数据结构：

- 函子的参数指定由数据结构管理的元素。由于参数是模块，您不仅可以指定元素的类型，还可以指定数据结构可能需要的辅助函数来管理它们。
- 仿函数的结果是为特定元素量身定做的模块。

例如，[数据结构Set](https://reasonml.github.io/api/Set.html)（我们将在后面详细介绍）必须能够比较它的元素。因此，集合的仿函数具有带以下接口的参数：

```ocaml
module type OrderedType = {
  type t;
  let compare: (t, t) => int;
};
```

`OrderedType.t` 是集合中元素的类型，`OrderedType.compare` 用于比较这些元素。  `OrderedType.t` 类似于多态类型 `list('a)` 的类型变量 `'a`。

#### PrintablePair1：可打印对的函子的第一个版本

让我们实现一个非常简单的数据结构：与可以打印（转换为字符串）的任意组件配对。为了打印对，我们必须能够打印他们的组件。这就是为什么组件必须通过以下接口指定的原因。

```ocaml
/* PrintablePair1.re */

module type PrintableType = {
  type t;
  let print: t => string;
};
```

我们再次使用名称 `Make` 作为仿函数，它使用实际的数据结构来生成模块：

```ocaml
/* PrintablePair1.re */

module Make = (Fst: PrintableType, Snd: PrintableType) => {
  type t = (Fst.t, Snd.t);
  let make = (f: Fst.t, s: Snd.t) => (f, s);
  let print = ((f, s): t) =>
    "(" ++ Fst.print(f) ++ ", " ++ Snd.print(s) ++ ")";
}

```

该仿函数有两个参数： `Fst` 指定可打印对的第一个组件， `Snd` 指定第二个组件。

`PrintablePair1.Make` 返回的模块包含以下部分：

-  `t` 是这个函子支持的数据结构的类型。注意它是如何引用函子参数 `Fst` 和 `Snd` 的。
-  `make` 是创建类型 `t` 的值的函数。
- `print` 函数是用于处理可打印对的功能。它将可打印的对转换为字符串。

#### 使用函子

让我们使用函子 `PrintablePair1.Make` 创建一个可打印对，其第一个组件是一个字符串，其第二个组件是一个`int`。

首先，我们需要为仿函数定义参数：

```ocaml
/* PrintablePair1Main.re */

module PrintableString = {
  type t = string;
  let print = (s: t) => s;
};

module = PrintableInt = {
  type t = int;
  let print = (i: t) => string_of_int(i);
}

```

接下来，我们使用仿函数来创建 `PrintableSI` 模块：

```ocaml
/* PrintablePair1Main.re */

module PrintableSI = PrintablePair1.Make(PrintableString, PrintableInt);
```

最后，我们创建并打印一对：

```ocaml
/* PrintablePair1Main.re */

let () = PrintableSI.({
  let pair = make("Jane", 53);
  let str = print(pair);
  print_string(str);
});

```

#### 改进封装

目前的实现有一个缺陷，我们可以在不使用 `PrintableSI.make()` 的情况下创建类型 `t` 的元素：

```ocaml
let pair = ("Jane", 53);
let str = print(pair);
```

为了防止这种情况，我们需要通过一个接口来创建类型 `Make.t` 摘要：

```ocaml
/* PrintablePair1Main.re */

module type S = (Fst: PrintableType, Snd: PrintableType) => {
  type t;
  let make: (Fst.t, Snd.t) => t;
  let print: (t) => string;
}

```

这就是我们如何定义 `Make` 以获得类型 `S` ：

```ocaml
module Make: S = ...
```

请注意，`S` 是整个仿函数的类型。

#### PrintablePair2：仅用于结果的接口

定义一个仅用于函数的结果的接口更为常见（而不是像之前那样完整的函数）。这样，您可以将该接口用于其他目的。对于正在进行的示例，这样的界面看起来如下。

```ocaml
/* PrintablePair2.re */

module type S = {
  type fst;
  type snd;
  type t;
  let make: (fst, snd) => t;
  let print: (t) => string;
}

```

现在我们不能再引用函子的参数 `Fst` 和 `Snd` 了。因此，我们需要引入两个新的类型 `fst` 和 `snd` ，我们需要定义 `make()` 的类型。此函数以前有以下类型：

```ocaml
let make: (Fst.t, Snd.t) => t;
```

那么我们如何将 `fst` 和 `snd` 与 `Fst.t` 和 `Snd.t` 连接呢？我们通过所谓的共享约束来修改接口。他们是这样使用的：

```ocaml
/* PrintablePair2.re */

module Make = (Fst: PrintableType, Snd: PrintableType): (S with type fst = Fst.t and type snd = Snd.t) => {
  type fst = Fst.t;
  type snd = Snd.t;
  type t = (fst, snd);
  let make = (f: fst, s: snd) => (f, s);
  let print = ((f, s): t) => 
    "(" ++ Fst.print(f) ++ ", " ++ Snd.print(s) ++ ")";
};

```

以下两个等式是共享约束条件：

```ocaml
S with type fst = Fst.t and type snd = Snd.t
```

`S with` 提示改变接口 `S` 的约束。他们的确如此。  `Make` 的结果界面如下：

```ocaml
{
  type fst = Fst.t;
  type snd = Snd.t;
  type t;
  let make: (fst, snd) => t;
  let print: (t) => string;
}
```

还有一件事我们可以改进： `fst` 和 `snd` 是多余的。如果结果接口直接引用 `Fst.t` 和 `Snd.t` 会更好（就像我们为完整仿函数创建接口时那样）。这是通过破坏性替代完成的。

#### PrintablePair3：破坏性替换

破坏性替代与共享约束非常相似。然而：

- 共享约束 ` S with type t = u ` 提供更多信息给  S.t。
- 用 `t:= u` 类型的破坏性替换用 `s` 代替 `S` 内所有 `t` 的出现。

这就是 `Make` 看起来像是如果我们使用破坏性替换：

```ocaml
module Make = (Fst: PrintableType, Snd: PrintableType): (S with type fst := Fst.t and type snd := Snd.t) => {
  type t = (Fst.t, Snd.t);
  let make = (fst: Fst.t, snd: Snd.t) => (fst, snd);
  let print = ((fst, snd): t) => 
    "(" ++ Fst.print(fst) ++ ", " ++ Snd.print(snd) ++ ")";
};
```

破坏性替换从 `S` 中移除 `fst` 和 `snd` 。因此，我们不需要在 `Make` 的 `body` 中定义它们，并且总是可以直接引用 `Fst.t` 和 `Snd.t`.由于破坏性替换， `Make` 内部的定义与接口需要的内容相匹配。 `Make` 的结果签名现在是：

```ocaml
{
  type t;
  let make: (Fst.t, Snd.t) => t;
  let print: (t) => string;
}
```

#### 例子：使用函数 Set.Make

标准模块 Set 的值集合遵循我已经解释的约定：

- `Set.Make` 是生成处理集合的实际模块的仿函数。
- `Set.OrderedType` 是 `Make` 的参数的接口：
```ocaml
module type OrderedType = {
  type t;
  let compare: (t, t) => int;
};
```
- `Set.S` 是 `Make` 结果的接口。

这是您为字符串集创建和使用模块的方式：

```ocaml
module StringSet = Set.Make({
  type t = string;
  let compare = Pervasives.compare;
});

let set1 = StringSet.(empty |> add("a") |> add("b") |> add("c"));
let set2 = StringSet.of_list(["b", "c", "d"]);
/* StringSet.elements(s) 将 set s 转换为列表 */
StringSet.(elements(diff(set1, set2)));
  /* list(string) = ["a"] */
```

方便起见， ReasonML 的标准库带有一个模块 `String` ，它作为 `Set.Make` 的一个参数，因为它同时具有 `String.t` 和 `String.compare` 。因此，我们也可以写下：

```ocaml
module StringSet = Set.Make(String);
```

## 用于扩展模块的函子

函子也可以用来扩展现有模块的功能。以这种方式使用它们类似于多重继承和混合（抽象子类）。

作为示例，让我们扩展一个现有模块，该模块只能向其数据结构中添加单个元素，并使用一个函数添加给定数据结构的所有元素。AddAll是这样做的一个函数:

```ocaml
module AddAll = (A: AddSingle) => {
  let addAll = (~from: A.t, into: A.t) => {
    A.fold((x, result) => A.add(x, result), from, into);
  };
};
```

函数 `addAll()` 使用 `fold()` 遍历 `~from` 的元素并将它们添加到一个元素中。结果总是与已经计算的结果绑定（首先进入，然后将第一个 `x` 加入到结果中，等等）。

在这种情况下，我们让 ReasonML 推断出 `AddAll` 的结果类型，并且不提供它的接口。如果我们这样做，它将具有名称 `S` 并具有抽象类型 `t` （用于参数和 `addAll` 的结果）。它会像这样使用：

```ocaml
module AddAll = (A: AddSingle): (S with type t := A.t) => {
  ...
};
```

从源代码中，我们可以推断 `addAll` 需要什么，并在接口 `AddSingle` 中收集它：

```ocaml
module type AddSingle = {
  type elt;
  type t; /* 数据结构的类型 */

  let empty: t;
  let fold: ((elt, 'r) => 'r, t, 'r) => 'r;
  let add: (elt, t) => t;
};
```

#### AddAll 为字符串集合

我们先前已经定义了 `StringSet` 并使用它来创建 `StringSetPlus` ：

```ocaml
module StringSetPlus = {
  include StringSet;
  include AddAll(StringSet);s
}
```

新模块 `StringSetPlus` 包含模块 `StringSet` 和将函数 `AddAll` 应用于模块 `StringSet` 的结果。如果您愿意，我们正在模块之间进行多重继承。

这是 `StringSetPlus` 的使用：

```ocaml
let s1 = StringSetPlus.of_list(["a", "b", "c"]);
let s2 = StringSetPlus.of_list(["b", "c", "d"]);
StringSetPlus.(elements(addAll(~from=s2, s1)));
/* list(string) = ["a", "b", "c", "d"] */
```

#### 我们可以简化 AddAll 吗？

目前，我们需要将基本模块 `StringSet` 与扩展 `AddAll(StringSet)` 结合起来创建 `StringSetPlus` ：

```ocaml
module StringSetPlus = {
  include StringSet; /* 基础 */
  include AddAll(StringSet); /* 扩展 */
}
```

如果我们可以按如下方式创建它会怎样？

```ocaml
module StringSetPlus = AddAll(StringSet);

```

有两个原因我们不这样做。

- 首先，我们希望将 `AddAll` 的参数和 `StringSetPlus` 的基础分开。当我们使用 `AddAll` 作为列表时，我们需要这种分离。
- 其次，无法实现 `AddAll` ，以便扩展其参数。理论上，这看起来像这样：

```ocaml
module AddAll = (A: AddSingle) => {
  include A;
  ···
};
```
在实践中，包括 `A` 仅包括接口 `AddSingle` 中的内容。这通常是不够的。

#### 数据结构：多态数据类型与函子

ReasonML标准库提供了两种数据结构：

- `list` 和其他实现为多态数据类型，其元素类型通过类型变量指定。
- `Set` 和其他通过仿函数来实现。元素类型通过模块指定。

#### AddAll 列表的字符串

唉， `AddAll` 最适合通过函子实现的数据结构。如果我们想将它用于列表，我们必须将 `list('a)` 的类型变量绑定到一个具体类型（在本例中为 `string` ）。这导致 `AddAll` 的以下参数：

```ocaml
module AddSingleStringList: AddSingle with type t = list(string) = {
  type elt = string;
  type t = list(elt);
  let empty = [];
  let fold = List.fold_right;
  let add = (x: elt, col1: t) => [x, ...col1];
};
```

[这是我能想到的最简单的解决方案 - 欢迎提供改进建议。]

之后，这就是我们如何创建和使用支持所有列表操作和 `addAll()` 的模块：

```ocaml
module StringListPlus = {
  include List;
  include AddAll(AddSingleStringList);
};

StringListPlus.addAll(~from=["a", "b"], ["c", "d"]);
/* list(string) = ["a", "b", "c", "d"] */
```

## 资料

-  OCaml手册中的“[模块系统](http://caml.inria.fr/pub/docs/manual-ocaml/moduleexamples.html)” 章节
- “真实世界OCaml” 中的 “[Functors](https://realworldocaml.org/v1/en/html/functors.html)” 章节
- “[模块](https://ocaml.org/learn/tutorials/modules.html)”（OCaml.org教程）