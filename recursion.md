# 递归

译自：http://reasonmlhub.com/exploring-reasonml/ch_recursion.html

> 在本章中，我们看看函数式编程中的一个重要技术：递归。

## 所需的知识

在阅读了关于列表的章节（以及它构建的所有前面的章节）之后，最好阅读本章。

如果您熟悉以下命令性功能（但它们应该相对不言自明），它也会有所帮助：

- 通过 `ref()` 的可变状态：请参阅 ReasonML 文档中的“ [Mutation](https://reasonml.github.io/docs/en/mutation.html) ”（直到本书中的相应章节已完成）。
- `for` 循环：请参阅 ReasonML 文档中的“[Imperative Loops](https://reasonml.github.io/docs/en/imperative-loops.html)”（直到本书中的相应章节已完成）。

## 通过递归函数调用来处理数据

递归函数调用是用于处理递归数据的便捷工具。在进入函数调用之前，我们先来回顾一下如何构建递归数据。

#### 构建递归数据

作为一个例子，考虑下面的递归数据类型：

```ocaml
type intTree = 
    | Empty /* 原子 */
    | Node(int, intTree, intTree); /* 复合 */
```
创建一个intTree：

- 原子：我们从创建一个或多个原子片开始。一般来说，可能有几种原子碎片，每种可能包含也可能不包含数据。在 `intTree` 的情况下，所有原子片段都是空的。例如：
```ocaml
Empty
```
- 复合：我们通过组装原子片和/或其他复合片继续制造复合片。也就是说，复合片是包含一个或多个其他片的复合片。因此，类型定义的相应部分递归地提到了类型。例如，复合节点引用 `intTree` ，两次。以下表达式创建一个具有保存整数 123 的单个节点的树：

```ocaml
Node(123, Empty, Empty)
```

#### 递归数据递归

我们处理递归数据的方式与我们构建它的方式类似。采取以下代码：

```ocaml
let rec len = (myList: list('a)) => 
    switch myList {
        | [] => 0 /* 基础案例 */
        | [_, ...tail] => /* 递归 */
            1 + len(tail)
    };
```

再一次，我们有：

- 处理空列表的基本案例。递归在此停止。
- 处理非空列表的递归案例。递归继续列表的尾部。

#### 例子：自然数

即使你使用自然数字，你也经常会遇到类似的情况：

```ocaml
let rec factorial = (x) => 
    if (x <= 0) { /* 基础案例 */
        1;
    } else { /* 递归案例 */
        x * factorial(x - 1);
    };
```

该函数暗示您可以通过递归数据类型定义自然数：

```ocaml
type nat = Zero | Succ(nat);
```

这种类型表示：自然数 `nat` 是零或自然数的后继数（即，一个加上该数字）。从 `int` 到 `nat` 的转换也可以递归定义：

```ocaml
let rec natFromInt = (x: int) => 
    if (x <= 0) { /* 基础案例 */
        Zero;
    } else { /* 递归案例 */
        Succ(natFromInt(x - 1));
    };

natFromInt(2);
/* Succ((Succ(Zero))) */
```

## 通过递归循环

另一个核心功能技术是使用递归而不是循环。

#### 例子：repeat()

考虑以下以命令式语言使用循环的典型示例：重复一个字符串 `str` 零次或多次。

```ocaml
let repeat = (~times: int, str: string) => {
    let result = ref("");
    for(_count in 1 to times) {
        result := result^ + str;
    };

    return^;
};

repeat(~times = 3, "abc"); /* "abcabcabc" */
```

通过递归可以实现相同的功能，如下所示：

```ocaml
let rec repeat = (~times: int, str: string) => 
    if (times <= 0) {
        "";
    } else {
        str ++ repeat(~times = times - 1, str);
    }
```

再一次，有一个基本案例和一个递归案例。如果你来自命令式语言，以这种方式使用递归最初看起来很奇怪，但你最终会习惯它。

#### 例子：summarizeList()

计算整数列表的总和是命令式语言循环的另一个典型例子：

```ocaml
let summarizeList = (l: list(int)) => {
    let result = ref(0);
    for (index in 0 to List.length(l) - 1) {
        let elem = List.nth(l, index);
        result := result^ + elem;
    };

    result^;
};

summarizeList([1, 2, 3]); /* 6 */
```

旁白：列表不是非常有效的 `w.r.t`. 随机访问操作，如 `List.length()` 和 `List.nth()` 。但是，我只使用列表，因为我希望代码与更多功能的实现类似。在现实世界的代码中，您可以选择使用数组。如果您递归处理列表，则可以通过 `switch` 使用模式匹配：

```ocaml
let rec summarizeList = (l: list(int)) => 
    switch l {
        | [] => 0
        | [head, ...tail] =>
            head + summarizeList(tail)
    };
```

请阅读另一种实现不涉及递归的 `summarizeList()` 方法，但也是有用的。

## 通过更高阶的辅助函数循环

函数式编程中的一个关键技术是用于处理数据结构的高阶辅助函数。它们是使用递归的替代方法。例如，我们可以使用`fold_left()` 来重新实现上一节中的 `summarizeList()` ：

```ocmal
let summarizeList = (theList: list(int)) => 
    ListLabels.fold_left(
        ~f = (sum, elem) => sum + elem,
        ~init = 0,
        theList
    );
```

对列表的每个元素 `elem` 调用 `~f`：

- `sum` 是当前状态（迄今计算的总和）
- `~f` 的结果成为下一个状态。

最终状态由 `fold_left()` 返回。 `~init` 指定了初始状态。

用于处理列表的其他高阶函数是（模块 `ListLabels` ）：

- `map: (~f: ('a) => 'b, list('a)) => list('b)` 通过将 `~f` 应用于列表的每个元素来生成一个新列表。
- `filter: (~f: ('a) => boll, list('a)) => list('a)` 产生一个新的列表，其中只包含那些 `~f` 返回 `true` 的输入列表元素。
- `for_all: (~f: ('a) => bool, list('a)) => bool` 如果 `~f` 对列表的所有元素返回 `true` ，则返回 `true` 。
- `exists: (~f: ('a) => bool, list('a)) => bool` 如果 `~f` 对列表的至少一个元素返回 `true` ，则返回 `true` 。
- `iter: (~f: ('a) => unit, list('a)) => unit` 为列表的每个元素调用 `~f` 。 `iter()` 只有在 `~f` 有副作用（输出，突变等）时才有意义。

除了 filter() ，所有这些函数也可用于数组。

## 递归技术：累加器

有时候，(天真的)递归不是人们思考问题的方式。例如，下面的问题:给定一个ints列表，返回最大的元素。如果我们只使用递归，我们会得到以下代码。唉，它既不是直观的，也不是有效的:

```ocaml
let rec maxList = (l: list(int)) =>
    switch (l) {
        | [] => min_int
        | [head, ...tail] when head > maxList(tail) => head /* A */
        | [_, ...tail] => maxList(tail) /* B */
    };

maxList([-3, 12, 1]); /* 12 */
```

在这段代码中，我们在末尾使用了两个 `switch` 情况（ A 行， B 行）;其中之一有一个 `when` 子句。相反，我们可以使用单个案例和 `if` 表达式，但是扁平代码更易于阅读。

那么这段代码如何工作？对于每个列表，我们计算尾部的最大值并将其与头部进行比较。第一次 A 行中的递归调用直接返回一个值在列表的末尾（当 `l` 是 `[]` 时）。这反过来又使调用者能够返回一个值。然后调用者的调用者。等。

相比之下，`maxList()` 的命令版更直观。它也更有效（除了随机访问）：

```ocaml
let maxList = (l: list(int)) => {
    let max = ref(min_int);
    for (index in 0 to List.length(l) - 1) {
        let elem = List.nth(l, index);
        if (elem > max^) {
            max := elem;
        }
    };
    max^;
};
```

是否可以编写一个直观的函数式版本？是的，通过一种称为累加器的技术。以下代码使用这种技术：

```ocaml
let maxList = (theList: list(int)) => {
    let rec aux = (~max=min_int, l) =>
        switch l {
            | [] => max
            | [head, ...tail] when head > max =>
                aux(~max=head, tail); /* A */
            | [_, ...tail] =>
                aux(~max, tail);
        };
    aux(theList); /* B */
};
```

`~max` 是累加器：一个在递归过程中不断更新并在最后返回的参数。类似于 `fold_left()` 的回调函数的第一个参数，它表示当前状态。在 A 行中，可以看到 `~max` 几乎可以被使用，就好像它是可变的：它被设置为一个新的值，从现在开始使用。

这种技术的一个缺点是你需要管理累加器的附加参数。有两种方法可以这样做。

我们已经看到了第一种方法：创建一个名为 `aux` 或 `helper` 或 `maxListRec` （等）的内部递归辅助函数。我喜欢设置 `aux` 的参数，如下所示：

- 累加器的初始值通过参数默认值提供。
- `maxList` 的一些参数被传递给 `aux` 并在那里改变。我在 `maxList` （B行）的末尾执行此操作。
- 有时候，外部函数的参数不会改变。然后 `aux` 就可以引用它们。

或者，您可以立即引入累加器作为参数并将其设置为默认值。调用者通常会忽略它，但它被递归调用使用。这看起来如下：

```ocaml
 let rec maxList = (~max=min_int, l: list(int)) => {
     switch l {
         | [] => max
         | [head, ...tail] when head > max =>
            maxList(~max=head, tail);
         | [_, ...tail] =>
            maxList(~max, tail);
     };
 };
```

## 尾递归

假定在函数式编程中更喜欢递归循环 - 所有这些函数调用都不会影响性能？事实证明，可以高度优化尾部调用，一种特定类型的函数调用。

#### 什么是尾调用？

函数调用是尾部调用，如果它是周围函数的最后一件事情。在我们看一个例子之前，让我们来看看一个没有 `tail` 调用的函数：

 ```ocaml
 let rec repeat = (~times: int, str: string) =>
    if (times <= 0) { /* 基础案例 */
        ""
    } else { /* 递归案例 */
        str ++ repeat(~times = times - 1, str); /* A */
    };
 ```

A 行中的递归调用不是尾调用：在返回后，其结果被附加到 `str` 并返回。也就是说，追加是 `repeat()` 所做的最后一件事。因此，当我们调用 `repeat()` 时，我们必须保存两种信息：

- 参数和局部变量的值，因为我们在 `repeat()` 返回后需要 `str` 。
- 返回地址，表示函数调用完成后继续计算的位置。

下面的调用树形象化了我们进行函数调用时会发生什么 `repeat(~times=2, "a")`:

 ```ocaml
repeat(~times=2, "a")
  "a" ++ repeat(~times=1, "a")
    "a" ++ repeat(~times=0, "a")
      ""
 ```

每个缩进都表示我们的存储需求增长。参数，变量和返回地址通常存储在堆栈中，这就解释了为什么如果存在太多的嵌套（非尾部）函数调用时会发生堆栈溢出。

我们如何使 `repeat()` 更有效？通过引入累加器（ `~result` ）：

```ocaml
let repeat = (~times: int, str: string) => {
    let rec aux = (~result="", n) =>
        if (n <= 0) { /* 基础案例 */
            result;
        } else { /* 递归案例 */ 
            aux(~result = result ++ str, n - 1); /* A */ 
        };
    aux(times);
}
 ```

现在 A 行中的函数调用是尾部调用：它是周围函数的结果。由于不需要为主叫方保存信息，因此 A 行中的呼叫实际上是一次跳转，并不会增加我们的存储需求。

函数调用 `repeat(~times = 2, "a")` 的执行过程如下：

 ```ocaml
aux(~result="", 2)
aux(~result="a", 1)
aux(~result="aa", 0)
"aa"
 ```

 平坦性表明函数调用更像跳转，而不像递归调用。

## 术语：

- 递归调用是尾调用的函数称为尾递归。
- 尾递归算法也被称为迭代。迭代算法也可以通过循环来实现。

#### 例子：summarizeList()

以下的 `summarizeList()` 实现在 A 行中递归调用，而不是尾调用：

```ocaml
let rec summarizeList = (l: list(int)) =>
    switch l {
        | [] => 0
        | [head, ...tail] => 
            head + summarizeList(tail) /* A */
    };
```

如果我们引入一个累加器，我们在 A 行中得到尾部调用：

```ocaml
let summarizeList = (theList: list(int)) => {
    let aux = (~sum = 0, l) =>
        switch l {
            | [] => sum
            | [head, ...tail] =>
                summarizeList(~sum = sum + head, tail); /* A */
        };
    aux(theList);
};
```

#### 例子：reverse()

下面的函数是我们之前没有看到的代码 - 一个反转列表的递归实现 `l` ：

```ocaml
let rec reverse = (l: list('a)) =>
    switch l {
        | [] => l
        | [head, ...tail] =>
            List.append(reverse(tail), [head]) /* A */
    };
reverse(["a","b", "c"]);
/* ["c", "b", "a"] */
```

行 A 说：反向列表是反向的尾部跟着头部（ `append()` 连接列表）。

`reverse()` 的迭代版本会是什么样子？考虑以下用于反转列表的直观算法 `l = ["a", "b", "c"]`：

- 初始化值是空列表：
```ocaml
let result0 = [];
```
- 我们从 `["a", "b", "c"]` 开始 
```ocaml
let result1 = ["a", ...result0]; /* ["a"] */
```
- 我们继续讨论 `["b", "c"]` 的头部:
```ocaml
let result2 = ["b", ...result1]; /* ["b", "a"] */
```
- 我们继续讨论 `["c"]` 的头部:
```ocaml
let result3 = ["c", ...result2]; /* ["c", "b", "a"] */
```
- 最终结果：result3

事实证明，这个算法很容易通过累加器实现：

```ocaml
let rec reverse = (theList: list('a)) => {
    let rec aux = (~result=[], l) => 
        switch l {
            | [] => result
            | [head, ...tail] => 
                aux(~result= [head, ...result], tail)
        };
    aux(theList);
};
```

当我们调用反转（`["a","b","c"]`）时，将采取以下步骤：

```ocaml
aux(~result=[], ["a", "b", "c"])
aux(~result=["a"], ["b", "c"])
aux(~result=["b", "a"], ["c"])
aux(~result=["c", "b", "a"], [])
["c", "b", "a"]
```

观察两个实现如何以不同的顺序处理数据是很有趣的：

- 递归 `reverse()` 适用于：
    - 当前数据： `head`
    - 未来数据： `reverse(tail)`
- 迭代 `reverse()` 适用于：
    - 过去的数据：  `result`
    - 当前数据：`head`

在这种情况下，累加器版本处理数据的顺序更适合我们对算法的直观理解。

#### 例子：append()

但是，如果我们实现 `append()` ，则递归算法更简单：

```ocaml
let rec append = (l1: list('a), l2: list('a)) => 
    switch l1 {
        | [] => l2
        | [head, ...tail] =>
            [head, ...append(tail, l2)] /* A */
    };
append(["a","b"], ["c","d"]);
/* ["a", "b", "c", "d"] */
```

行 A 表示：两个列表 `l1` 和 `l2` 的连接是 `l1` 的头部，接着是 `l1` 和 `l2` 尾部的连接。

如果我们考虑反复追加，我们得到以下算法：

- 从 `l2` 开始。
- 在前面的结果之前加上 `l1` 的最后一个元素。
- 在前面的结果之前加上 `l1` 的倒数第二个元素。
- 等等。

也就是说，我们必须通过 `List.rev()` （行A）反转 `l1` ，然后才能迭代使用它：

```ocaml
let append = (l1: list('a), l2: list('a)) => {
    let rec aux = (~result, l) =>
        switch l {
            | [] => result
            | [head, ...tail] =>
                aux(~result = [head, ...result], tail)
        };
    aux(~result=12, List.rev(l1)); /* A */
};
```

#### 例子：contains()

以下函数检查 `theList` 是否包含给定值：

```ocaml
let rec contains = (~value: 'a, theList: list('a)) => 
    switch theList {
        | [head, ...tail] when head == value =>
            true /* A */
        | [_, ...tail] => contains(~value, tail)
        | [] => false
    };

contains(~value=1, [1,2,3]); /* true */
contains(~value=5, [1,2,3]); /* false */
```
方便的是，这个递归实现也是迭代的：所有递归调用都是尾调用。因为我们自己正在做递归，所以我们可以在找到价值时立即停止（ A 行）。