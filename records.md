# 记录

> 本章将探讨 ReasonML 的记录是如何工作的。

记录类似于元组：它具有固定的大小，并且它的每个部分可以具有不同的类型并且可以直接访问。然而，在一个元组（它的组件）的部分按位置访问的地方，一个记录的部分（它的字段）是按名称访问的。默认情况下，记录是不可变的。

## 基本使用

#### 定义记录类型

在创建记录之前，您必须为其定义一个类型。例如：

```ocaml
type point = {
    x: int,
    y: int, /* 最后一个逗号可选 */
}
```

我们已经定义了记录类型点，它有两个字段 `x` 和 `y` 。字段的名称必须以小写字母开头。

在相同的范围内，没有两个记录应该具有相同的字段名称。此限制性规则的原因是字段名称用于确定记录的类型。为了实现这个任务，每个字段名称只与一个记录类型相关联。

可以在多条记录中使用相同的字段名称，但是可用性会受到影响：最后一个字段名称为 “wins” 的记录类型为 w.r.t.类型推断。因此，使用其他记录类型变得更加复杂。所以我更喜欢假装重复使用字段名称是不可能的。

稍后我们将研究如何解决此限制。

#### 嵌套记录类型

是否可以嵌套记录类型？例如，我们可以做以下事情吗？

```ocaml
type t = {a: int, b: {c: int}};
```

不，我们不能。我们会得到一个语法错误。这是如何正确定义 `t` ：

```ocaml
type b = {c: int};
type t = {a: int, b: b}
```

用 `b:b` ，字段名称和字段值是相同的。那么你可以简称为 `b` 。这就是所谓的双关语：

```ocaml
type t = {a: int, b};
```

#### 从头开始创建记录

这是您从头开始创建记录的方式：

```ocaml
# let pt1 = {x: 12, y: -2};
let pt1: point = {x: 12, y: -2};
```

请注意字段名称是如何用来推断 `pt1` 具有类型 `point` 。

双关语这样也是可以的：

```ocaml
let x = 7;
let y = 8;

let pt2 = {x, y}
/* 和 {x: x, y: y} 一样 */

```

#### 访问字段值

字段值通过点(`.`)运算符访问：

```ocaml
# let pt = {x: 1, y: 2};
let pt: point = {x: 1, y: 2};

# pt.x;
- : int = 1
# pt.y;
- : int = 2

```

#### 记录的非破坏性更新

记录是不可变的。要更改记录 `r` 的字段 `f` 的值，我们必须创建一个新记录 `s` 。 `s.f` 具有新的值， `s` 的所有其他字段与 `r` 中的值相同。这是通过以下语法实现的：

```ocaml
let s = {...r, f: newValue}
```

三重点（`...`）被称为扩展算子。他们必须先来，最多可以使用一次。但是，您可以更新多个字段（而不仅仅是一个字段 `f` ）。

这是使用扩展运算符的一个例子：

```ocaml
# let pt = {x: 1, y: 2};
let pt: point = {x: 1, y: 2};

# let pt' = {...pt, y: 3}
let pt': point = {x: 1, y: 3};

```

#### 模式匹配

所有常用的模式匹配机制也与记录一起工作。例如：

```ocaml
let isOrig = (pt: point) =>
    switch pt {
        | {x: 0, y: 0} => true
        | _ => false
    };
```

这就是通过 let 解构的样子：

```ocaml
# let pt = {x: 1, y: 2};
let pt: point = {x: 1, y: 2};

# let {x: xCoord} = pt;
```

您可以使用双语：

```ocaml
# let getX = ({x}) = x;
let getX: (point) => int = <fun>;

# getX(pt);
- : int = 1
```

#### 关于缺少字段的警告

在模式匹配过程中，默认情况下，您可以省略所有您不感兴趣的字段。例如：

```ocaml
type point = {
    x: int,
    y: int,
}

let getX = ({x}) => x; /* 不要这样做 */
``

对于 `getX()` ，我们对字段 `y` 不感兴趣，只提及字段 `x` 。但是，最好明确地省略字段：

```ocaml
let getX = ({x, _}) => x;
```

`x` 后面的下划线告诉 ReasonML：我们忽略了所有剩余的字段。

为什么更明确一点？因为现在您可以让 ReasonML 通过向 bsconfig.json 添加以下条目来警告您缺少字段名称：

```ocaml
"warnings": {
  "number": "+R"
}
```

初始版本现在触发以下警告：

```ocaml
Warning number 9

4 │ };
5 │
6 │ let getX = ({x}) => x;

以下标签未在此记录模式中绑定： y 要么明确地绑定这些标签，要么添加'; _'的模式。
```

我建议更进一步，并使缺少的字段出现错误（编译无法完成）：

```ocaml
"warnings": {
  "number": "+R",
  "error": "+R"
}
```

[有关配置警告的更多信息](https://bucklescript.github.io/docs/en/build-configuration.html#warnings)，请参阅BuckleScript手册。

检查缺少的字段对于使用所有当前字段的代码尤其重要：

```ocaml
let string_of_point = ({x, y}: point) =>
  "(" ++ string_of_int(x) ++ ", "
  ++ string_of_int(y) ++ ")";

string_of_point({x:1, y:2});
  /* "(1, 2)" */
```

如果您要向 `point` 添加另一个字段（例如 `z` ），那么您希望 ReasonML 向您发出有关 `string_of_point` 的警告，以便您可以更新它。

#### 递归记录类型

变体是我们已经看到的递归定义类型的第一个例子。您也可以在递归定义中使用记录。例如：

```ocaml
type intTree = 
    | Empty
    | Node(intTreeNode)
and intTreeNode = {
    value: int,
    left: intTree,
    right: intTree,
};
```

变体 `intTree` 递归地依赖于记录类型 `intTreeNode` 的定义。这就是你如何创建 `intTree` 类型的元素：

```ocaml
let t = Node({
    value: 1,
    left: Node({
        value: 2,
        left: Empty,
        right: Node({
            value: 3,
            left: Empty,
            right: Empty,
        }),
    }),
    right: Empty,
});
```

#### 参数化记录类型

在 ReasonML 中，类型可以通过类型变量进行参数化。定义记录类型时可以使用这些类型变量。例如，如果我们希望树包含任意值，而不是整数，我们使字段值的类型为多态（行A）：

```ocaml
type tree('a) = 
    | Empty
    | Node(treeNode('a))
and treeNode('a) = {
    value: 'a, /* A */
    left: tree('a),
    right: tree('a),
};
```

## 记录在其他模块中

每个记录都在一个作用域内定义（例如，一个模块）。其字段名称位于该作用域的顶层。虽然这有助于类型推断，但它使得使用字段名称比其他许多语言更复杂。让我们看看如果我们将点放入另一个模块 M 中，各种与记录有关的机制如何受到影响：


```ocaml
module M = {
    type point = {
        x: int,
        y: int,
    };
};
```

#### 从其他模块创建记录

如果我们试图创建类型 `point` 记录，就好像该类型在同一个范围内一样，我们就会失败：

```ocaml
let pt = {x: 3, y: 2}
/* 错误：未绑定的记录字段 x */
```

原因是 `x` 和 `y` 在当前作用域内不作为名称存在，它们只存在于模块 `M`.

解决这个问题的方法之一是通过限定至少一个字段名称：

```ocaml
let pt1 = {M.x: 3, M.y: 2}; /* OK */
let pt2 = {M.x: 3, y: 2}; /* OK */
let pt3 = {x: 3, M.y: 2}; /* OK */
```

解决这个问题的另一种方法是对整个记录进行排位。 ReasonML 如何报告推断的类型是很有趣的 - 类型和第一个字段名称都是合格的

```ocaml
# let pt4 = M.{x: 3, y: 2};
let pt4: M.point = {M.x: 3, y: 2};

```

最后，您还可以打开 `M` ，因此可以将 `x` 和 `y` 导入当前作用域。

```ocaml
open M;
let pt = {x: 3, y: 2};
```

#### 从其他模块访问字段

如果您未打开 `M` ，则无法使用非限定名称访问字段：

```ocaml
let pt = M.{x: 3, y: 2};

print_int(pt.x);

/*
警告40：从 M.point 类型中选择了 x 。 它在当前作用域内不可见，并且不会 如果类型未知，可以选择。
*/
```

如果您限定字段名称 `x` ，警告消失：

```ocaml
print_int(pt.M.x); /* OK */
```

本地打开 `M` 也可以：

```ocaml
M.(print_int(pt.x));

print_int(M.(pt.x));
```

## 模式匹配和来自其他模块的记录

通过模式匹配，您将面临与正常访问字段相同的问题 - 您无法使用 `point` 字段名称而无需对其进行限定：

```ocaml
# let {x, _} = pt;
错误：未绑定的记录字段 x
```

如果我们有资格获得 x ，一切都会好的：

```ocaml
# let {M.x, _} = pt;
let x: int = 3;
```

唉，排位模式不起作用：

```ocaml
# let M.{x, _} = pt;
错误：语法错误
```

我们可以在本地打开 `M` ，让整个 `let` 绑定。然而，这不是一个表达式，它阻止我们将它包含在括号中。我们必须另外将它包装在一个代码块（花括号）中：

```ocaml
M.({
    let {x, _} = pt;
    ...
});
```

#### 在多个记录中使用相同的字段名称

我最初承诺我们可以在多个记录中使用相同的字段名称。这样做的诀窍是把每个记录放在一个单独的模块中。例如，这里有两种记录类型，`Person.t` 和 `Plant.t` ，都有字段 `name` 。但它们驻留在单独的模块中，名称冲突不是问题：

```ocaml
module Person = {
    type t = {name: string, age: int};
};

module Plant = {
    type t = {name: string, edible: bool};
};

```

## FAQ：记录

#### 有没有办法动态指定一个字段的名称？

在 JavaScript 中，有两种方法可以访问一个字段（在 JavaScript 中称为属性）：

```ocaml
// 静态字段名称（在编译时已知）
console.log(obj.prop);

function f(obj, fieldName) {
  // 动态字段名称（在运行时已知）
  console.log(obj[fieldName]);
}
```

在 ReasonML中，字段名称始终是静态的。 JavaScript 对象扮演着两个角色：他们既是记录又是字典。在 ReasonML中，如果您需要记录，请使用记录，如果您需要字典，请使用 map 。