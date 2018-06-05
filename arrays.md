# 数组

译自：http://reasonmlhub.com/exploring-reasonml/ch_arrays.html

> 在本章中，我们看看 ReasonML 数据结构数组。

数组是具有随机访问的可变数据结构，其元素都具有相同的类型。它特别适合大量数据，并且随时随地访问。

当我们从标准库中访问函数时，我们会再次选择带标签的模块 `ListArray` 放在未标记的 `Array` 上。详情请参阅清单一章。

## 列表与数组

下表比较列表和数组。

|| 列表 | 数组
------------ | ------------ | ------------ 
大小         | 小-中 | 小-大
可调整大小？ | 灵活 | 固定
可变性 | 不可变 | 可变
元素类型 | 一样 | 一样
访问途径 | 解构 | 索引
最快的 | 前置/删除 第一 | 读 / 写元素

数组很像列表：它们的所有元素都具有相同的类型，并且可以按位置访问它们。但他们也不同：

- 数组是可变的。列表是不可变的。
- 数组不能调整大小。通过列表，您可以有效地预先添加元素并检索尾部。 （ BuckleScript 可以让你调整数组的大小，但如果你这样做的话，你将失去跨平台兼容性。）
- 数组提供快速索引访问。列表通过递归和模式匹配进行处理。

## 创建数组

以下小节介绍创建数组的三种常用方法。


#### 数组字面量

```ocaml
# [| "a", "b", "c" |];
- : array(string) = [|"a", "b", "c"|]
```
#### ArrayLabels.make()

签名：

```ocaml
let make: (int, 'a) => array('a);
```
第一个参数指定结果的长度。第二个参数指定了要填充的值。为什么第二个参数是强制性的？ `make()` 的结果只能包含 `'a` 类型的值。 ReasonML 没有 `null` ，所以你必须手动选择一个 `'a` 类型的成员。

这是 `make()` 的工作原理：

```ocaml
# ArrayLabels.make(3, "x");
- : array(string) = [|"x", "x", "x"|]

# ArrayLabels.make(3, true);
- : array(bool) = [|true, true, true|]

```

#### ArrayLabels.init()

签名：

```ocaml
let init: (int, ~f: int => 'a) => array('a);
```

第一个参数指定结果的长度。函数 `~f` 将索引映射到该索引处的初始值。例如：

```ocaml
# ArrayLabels.init(~f = i => i, 3);
- : array(int) = [|0 , 1, 2|]

# ArrayLabels.init(~f = i => "abc".[i], 3);
- : array(char) = [|'a', 'b', 'c'|]
```

## 获取数组的长度

`ListLabels.length()` 返回数组的长度：

```ocaml
ArrayLabels.length([| "a", "b", "c" |]);
- : int = 3
```

## 读取和写入数组元素

这就是读取和写入数组元素的方式：

```ocaml
# let arr = [| "a", "b", "c" |];
let arr: array(string) = [|"a", "b", "c"|];

# arr[1] /* 读 */ 
- : string = "b"
# arr[1] = "x"; /* 写 */
- : unit = ()
# arr;
- : array(string) = [| "a", "x", "c" |]

```

## 模式匹配和数组

模式匹配数组类似于匹配元组，而不是匹配列表。让我们从元组和列表开始（我们可以忽略穷尽性警告，因为我们正在处理固定数据）：

```ocaml
# let (a, b) = (1, 2);
let a: int = 1;
let b: int = 2;

# let [a, ...b] = [1, 2, 3];
警告：这种模式匹配并非详尽无遗。

let a: int = 1;
let b: list(int) = [2, 3];

```

接下来我们将解构一个数组：

```ocaml
# let [|a, b|] = [| 1, 2 |];
警告：这种模式匹配并非详尽无遗。
let a: int = 1;
let b: int = 2;

```

与元组类似，模式必须与数据具有相同的长度（这是例外情况）：

```ocaml
# let [| a, b |] = [|1, 2, 3|];
警告：这种模式匹配并非详尽无遗。

异常：Match_failure

```

## 在列表和数组之间转换

这就是你如何在列表和数组之间进行转换：

- 从数组到列表（模块 `ArrayLabels` ）：
```ocaml
let to_list: array('a) => list('a);
```
- 从列表到数组（模块 `ArrayLabels` ）：
```ocaml
let of_list: list('a) => array('a);
```

有时候你的数组中的数据会更容易在列表中处理。然后你可以将它转换为一个列表（如果需要的话，将它转换回数组）。

## 处理数组

标准库仍处于不断变化之中。因此，我现在只会演示一些亮点。

#### ArrayLabels.map()

对于数组，map() 的作用类似于列表的相同函数：

```ocaml
# ArrayLabels.map(s => s ++ "x", [| "a", "b" |]);
- : array(string) = [|"ax", "bx"|]
```

#### ArrayLabels.fold_left()

`fold_left()` 也与其列表版本相似：

```ocaml
let maxOfArray = (arr) => 
    ArrayLabels.fold_left(~f=max, ~init = min_int, arr);
```

这是如何使用 maxOfArray()：

```ocaml
# maxOfArray([||]);
- : int = -4611686018427387904

# maxOfArray([| 3, -1, 5 |]);
- : int = 5

```

再一次，我们使用 `fold` 从二元操作（ `max()` ）转换到 `n` 元操作（ `maxOfArray` ）。除了 `max()` 之外，我们还使用整数常量 `min_int` 。两者都是模块 `Pervasives` 的一部分，因此可以不经鉴定。

`max` 是一个适用于大多数类型的二元函数：

```ocaml
# max(1.0, 1.1);
- : float = 1.1

# max(None, Some(1));
- : option(int) = Some(1)

# max("a", "b");
- : string = "b"

# max(4, 3);
- : int = 4

```

`min_int` 是可能的最小 `int` 值（其确切值取决于您使用的平台）：

```ocaml
# min_int;
- : int = -4611686018427387904
```

## 通过 fold_right() 将数组转换为列表

`fold_right()` 的工作方式与 `fold_left()` 类似，但它始于最后一个元素。它的类型签名是：

```ocaml
let fold_right: (~f: ('b, 'a) => 'a, array('b), ~init: 'a) => 'a;
```

此函数的一个用例是将数组转换为列表。该列表必须如下构造（即，你必须从最后一个数组元素开始）：

```ocaml
[...[x_2nd_last, ...[x_last, ...[]]]]
```

该函数如下所示：

```ocaml
let listFromArray = (arr: array('a)) =>
    ArrayLabels.fold_right(~f=(ele, l) => [ele, ...l], arr, ~init=[]);
```

这是 listFromArray() 的使用：

 ```ocaml
# listFromArray([||]);
- : list('a) = []

# listFromArray([| 1, 2, 3 |]);
- : list(int) = [1, 2, 3]

# listFromArray([|"a", "b", "c"|]);
- : list(string) = ["a", "b", "c"]

 ```

 ## 过滤数组

 所有数组函数都会返回与输入数组长度相同的数组。因此，如果你想删除元素，你必须通过列表绕道：

 ```ocaml
let filterArray = (~f, arr) =>
    arr
    |> ArrayLabels.to_list
    |> ListLabels.filter(~f)
    |> ArrayLabels.of_list;
 ```

`filterArray()` 在使用中：

```ocaml
# filterArray(~f=x => x > 0, [|-2, 3, -4, 1|]);
- : array(int) = [| 3, 1|]
```