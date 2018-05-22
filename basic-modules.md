# 基本模块

> 在本章中，我们将探讨模块如何在 ReasonML 中工作。

本章的演示存储库可在GitHub上获得：[reasonml-demo-modules](https://github.com/rauschma/reasonml-demo-modules)。请下载并安装它：

```ocaml
cd reasonml-demo-modules/
npm install
```
这就是你需要做的 - 不需要全局安装。

### 您的第一个 ReasonML 程序

这是您的第一个 ReasonML 程序所在的位置：

```ocaml
reasonml-demo-modules/
    src/
        HelloWorld.re
```

在 ReasonML 中，名称具有扩展名的每个文件都是 `.re` 是一个模块。模块名称以大写字母开头，并且是骆驼式的。文件名称定义了它们的模块名称，所以它们遵循相同的规则。

程序只是您从命令行运行的模块。

HelloWorld.re 看起来如下：

```ocaml
/* HelloWorld.re */

let () = {
  print_string("Hello world!");
  print_newline()
};
```

这段代码看起来有点奇怪，所以让我解释一下：我们在花括号内执行两行，并将它们的结果赋给pattern ()。也就是说，不会创建新变量，但该模式可确保结果为 ()。()、unit 的类型与 C风格的语言中的 void 类似。

请注意，我们没有定义函数，我们正在执行 `print_string()` 和 `print_newline()`。

为了编译这段代码，你有两个选择（查看 package.json 以获得更多脚本来运行）：

- 编译一切，一次运行：`npm run build`
- 观察所有文件并逐步编译仅更改的文件：`npm run watch`

因此，我们的下一步是（在单独的终端窗口中运行或执行后台的最后一步）：

```ocaml
cd reasonml-demo-modules/
npm run watch
```

在 HelloWorld.re 旁边，现在有一个文件 HelloWorld.bs.js 。您可以按如下所示运行该文件。

```ocaml
cd reasonml-demo-modules/
node src/HelloWorld.bs.js
```

#### 其他版本的 HelloWorld.re

作为我们方法的替代方案（这是一种常见的 OCaml 惯例），我们也可以简单地将这两条线放入全局作用域：

```ocaml
/* HelloWorld.re */

print_string("Hellow world");
print_newline();
```

我们可以定义一个函数main（），然后我们调用它：

```ocaml
/* HelloWorld.re */

let main = () => {
  print_string("Hello world!");
  print_newline()
};
main();

```

### 两个简单的模块

让我们继续使用另一个模块 Main.re 使用的模块 MathTools.re：

```ocaml
reasonml-demo-modules/
    src/
        Main.re
        MathTools.re
```

MathTools 模块看起来像这样：

```ocaml
/* MathTools.re */

let times = (x, y) => x * y;
let square = (x) => times(x, x);
```

Main 模块看起来像这样：

```ocaml
/* Main.re */

let () = {
  print_string("Result: ");
  print_int(MathTools.square(3));
  print_newline()
};
```

正如您所看到的，在 ReasonML 中，您可以通过简单地引用它们的名称来使用模块。在当前项目的任何地方都可以找到它们。

#### 子模块

你也可以嵌套模块。所以这也适用：

```ocaml
/* Main.re */

module MathTools = {
  let times = (x, y) => x * y;
  let square = (x) => times(x, x);
};

let () = {
  print_string("Result: ");
  print_int(MathTools.square(3));
  print_newline()
};
```

在外部，您可以通过 Main.MathTools 访问 MathTools。

让我们进一步嵌套：

```ocaml
/* Main.re */

module Math = {
  module Tools = {
    let times = (x, y) => x * y;
    let square = (x) => times(x, x);
  };
};

let () = {
  print_string("Result: ");
  print_int(Math.Tools.square(3));
  print_newline()
};

```

### 控制如何从模块中导出值

默认情况下，导出模块的每个模块，类型和值。如果你想隐藏其中一些导出，你必须使用接口。此外，接口支持抽象类型（其内部是隐藏的）。

#### 接口文件

您可以通过所谓的界面控制您导出多少。对于由文件 `Foo.re` 定义的模块，将接口放在文件 `Foo.rei` 中。例如：

```ocaml
/* MathTools.rei */

let times: (int, int) => int;
let square: (int) => int;
```
例如，您省略了来自接口文件的 `times`，它将不会被导出。

模块的接口也称为其签名。

如果接口文件存在，则块级文档注释必须放在那里。否则，你把它们放到 `.re` 文件中。

幸运的是，我们不需要手动编写接口，我们可以从模块中生成接口。如何在 [BuckleScript 文档](https://bucklescript.github.io/docs/en/automatic-interface-generation.html)中进行描述。对于 `MathTools.rei`，我是通过以下方式完成的：

```ocaml
bsc -bs-re-out lib/bs/src/MathTools-ReasonmlDemoModule.cmi
```

#### 定义子模块的接口

假设 MathTools 不在其自己的文件中，而是作为子模块存在：

```ocaml
module MathTools = {
  let times = (x, y) => x * y;
  let square = (x) => times(x, x);
};
```

我们如何为这个模块定义一个接口？我们有两个选择。

首先，我们可以通过模块类型定义和命名一个接口：

```ocaml
module type MathToolsInterface = {
  let times: (int, int) => int;
  let square: (int) => int;
}
```

该接口变成模块 MathTools 的类型：

```ocaml
module MathTools: MathToolsInterface = {
  ...
};
```

其次，我们也可以内联接口：

```ocaml
module MathTools: {
  let times: (int, int) => int;
  let square: (int) => int;
} = {
  ...
};
```

#### 抽象类型：隐藏内部

您可以使用接口来隐藏类型的详细信息。让我们从一个 Log.re 模块开始，让您将字符串“放入”日志。它通过字符串实现日志，并通过直接使用字符串完全公开此实现细节：

```ocaml
/* Log.re */

let make = () => "";
let logStr = (str: string, log: string) => log ++ str ++ "\n";

let print = (log: string) => print_string(log);
```

从这段代码中，不清楚 `make()` 和 `logStr()` 实际上是否返回日志。

这是你如何使用日志。注意管道操作员（|>）在这种情况下的方便程度：

```ocaml
/* LogMain.re */

let () = Log.make()
  |> Log.logStr("Hello")
  |> Log.logStr("everyone")
  |> Log.print;

/* 输出：
Hello
everyone
*/
```

改进 Log 的第一步是引入日志类型。从 OCaml 借用的惯例是使用名称 t 作为模块支持的主类型。例如：Bytes.t

```ocaml
/* Log.re */

type t = string; /* A */
let make = (): t => "";
let logStr = (str: string, log: t): t => log ++ str ++ "\n";

let print = (log: t) => print_string(log);
```

在 A 行中，我们定义了 t 只是字符串的别名。别名很方便，因为您可以简单地开始并在稍后添加更多功能。

然而，别名迫使我们注释 make() 和 logStr() 的结果（否则它们会返回类型字符串）。

完整的接口文件如下所示。

```ocaml
/* Log.rei */

type t = string; /* A */
let make: (unit) => t;
let logStr: (string, t) => t;
let print: (t) => unit;
```

我们可以用下面的代码替换 A 行，并且 t 变得抽象 - 它的细节是隐藏的。这意味着我们可以在将来轻松改变我们的想法，例如通过数组来实现它。

```ocaml
type t;
```

方便的是，我们不必更改 LogMain.re，它仍然适用于新模块。

### 从模块导入值

有几种方法可以从模块中导入值。

#### 通过限定名称导入

我们已经看到，如果用模块名称限定值的名称，则可以自动导入由模块导出的值。例如，在以下代码中，我们从模块 Log 中导入 make， logStr 和 print：

```ocaml
let () = Log.make()
  |> Log.logStr("Hello")
  |> Log.logStr("everyone")
  |> Log.print;
```

#### 全局打开模块

如果您“全局”打开 Log（在当前模块的作用域），则可以省略限定词“ `Log.`”：

```ocaml
open Log;

let () = make()
  |> logStr("Hello")
  |> logStr("everyone")
  |> print;
```

为避免名称冲突，此操作不常使用。大多数模块，如 List，都是通过限定符使用的：List.length()，List.map() 等。

全局打开也可用于选择标准模块的不同实现。例如，模块 Foo 可能有一个子模块 List。然后打开 Foo ;将覆盖标准的 List 模块。

#### 在本地打开模块

通过在本地打开 Log ，我们可以最大限度地减少名称冲突的风险，同时还可以获得开放模块的便利。我们通过在 Log 前添加带括号的表达式来做到这一点。 （即，我们正在限定该表达）。例如：

```ocaml
let () = Log.(
  make()
    |> logStr("Hello")
    |> logStr("everyone")
    |> print
  );
```

#### 重新定义操作符

方便的是，操作符也只是 ReasonML 中的函数。这使我们可以暂时覆盖内置的操作员。例如，我们可能不喜欢使用带点的运算符来进行浮点运算：

```ocaml
let dist = (x, y) =>
  sqrt((x *. x) +. (y *. y));
```

然后我们可以通过模块 FloatOps 覆盖更好的 int 运算符：

```ocaml
module FloatOps = {
  let (+) = (+.);
  let (*) = (*.);
};

let dist = (x, y) =>
  FloatOps.(
    sqrt((x * x) + (y * y))
    );

```

你是否真的应该在生产代码中做到这一点是值得商榷的。

#### 包含模块

导入模块的另一种方式是包含它。然后将其所有输出添加到当前模块的输出中。这类似于面向对象编程中类之间的继承。

在以下示例中，模块 LogWithDate 是模块 Log 的扩展。除了 Log 的所有功能之外，它还具有新的函数 logStrWithDate()。


```ocaml
/* LogWithDateMain.re */

module LogWithDate = {
  include Log;
  let logStrWithDate = (str: string, log: t) => {
    let dateStr = Js.Date.toISOString(Js.Date.make());
    logStr("[" ++ dateStr ++ "]" ++ str, log);
  };
};

let () = LogWithDate.(
  make()
    |> logStrWithDate("Hello")
    |> logStrWithDate("everyone")
    |> print
  );

```

[Js.Date](https://bucklescript.github.io/bucklescript/api/Js.Date.html) 来自 BuckleScript 的标准库，这里不作解释。

你可以包含尽可能多的模块，而不仅仅是一个。

#### 包含接口

接口包含如下（InterfaceB 扩展 InterfaceA）：

```ocaml
module type InterfaceA = {
  ...
};

module type InterfaceB = {
  include InterfaceA;
  ...
}
```

与模块类似，您可以包含多个接口。

我们来为模块 LogWithDate 创建一个接口。唉，我们不能通过名称包含模块 Log 的接口，因为它没有模块类型。但是，我们可以通过它的模块（A行）间接引用它：

```ocaml
module type LogWithDateInterface = {
  include (module type of Log); /* A */
  let logStrWithDate: (t, t) => t;
};


module LogWithDate: LogWithDateInterface = {
  include Log;
  ...
};
```

#### 重命名导入

您不能真正重命名导入，但可以将它们别名。

这就是你的别名模块：

```ocaml
module L = List;
```

这就是你在模块中别名的方法：

```ocaml
let m = List.map;
```

### 命名空间模块

在大型项目中，ReasonML 的识别模块的方式可能会成为问题。由于它有一个全局模块名称空间，因此很容易出现名称冲突。假设有两个模块在不同的目录中调用 Util。

一种技术是使用命名空间模块。以下面的项目为例：

```ocaml
proj/
    foo/
        NamespaceA.re
        NamespaceA_Misc.re
        NamespaceA_Util.re
    bar/
        baz/
            NamespaceB.re
            NamespaceB_Extra.re
            NamespaceB_Tools.re
            NamespaceB_Util.re
```

在这个项目中有两个模块 Util，它们的名称只有不同，因为它们分别以 NamespaceA_ 和 NamespaceB_ 为前缀：

```ocaml
proj/foo/NamespaceA_Util.re
proj/bar/baz/NamespaceB_Util.re
```

为了减少命名的难度，每个命名空间都有一个命名空间模块。第一个看起来像这样：

```ocaml
/* NamespaceA.re */
module Misc = NamespaceA_Misc;
module Util = NamespaceA_Util;
```

NamespaceA的用法如下：

```ocaml
/* Program.re */

open NamespaceA;

let x = Util.func();
```

全局打开让我们使用 Util 而不用加前缀。

这种技术还有两个用例：

- 您可以使用它覆盖模块，甚至可以使用标准库中的模块。例如，NamespaceA.re 可以包含一个自定义 List 实现，它将覆盖 Program.re 中的内置 List 模块：

```ocaml
module List = NamespaceA_List;
```
- 您可以创建嵌套模块，同时将子模块保存在单独的文件中。例如，除了打开 NamespaceA 之外，还可以通过 NamespaceA.Util 访问 Util，因为它嵌套在 NamespaceA 中。当然，NamespaceA_Util 也可以工作，但不鼓励，因为它是一个实现细节。

后者的技术被 BuckleScript 用于 Js.Date ，Js.Promise 等文件中的 [js.ml](https://github.com/BuckleScript/bucklescript/blob/master/jscomp/runtime/js.ml)（这是OCaml语法中）：

```ocaml
...
module Date = Js_date
...
module Promise = Js_promise
...
module Console = Js_console
```

#### OCaml 中的命名空间模块

命名空间模块在 Jane Street 的 OCaml 中广泛使用。他们称它们为打包模块，但我更喜欢命名空间模块，因为它不会与 npm term 软件包冲突。

本节的源代码：Yaron Minsky 为 Jane Street Tech Blog 撰写的“[通过模块别名提高命名空间](https://blog.janestreet.com/better-namespaces-through-module-aliases/)”。

### 探索标准库

ReasonML 的标准库附有两个重要的注意事项：

- 目前正在进行中。
- 它在模块中的值的命名风格将从蛇案（ foo_bar 和 Foo_bar ）变成驼峰案（ fooBar 和 FooBar ）。
- 目前，许多功能仍然缺失。

#### API文档

ReasonML 的标准库被拆分：大多数的核心 ReasonML API 都可以在 native 和 JavaScript 上运行（通过 BuckleScript ）。如果您编译为 JavaScript，则需要在两种情况下使用 BuckleScript 的 API：

- ReasonML 的 API 完全缺少的功能。示例包括对日期的支持，您可以通过 BuckleScript 的 Js.Date 获取日期。
- BuckleScript 不支持的 ReasonML API 功能。示例包括模块Str（由于 JavaScript 的字符串不同于 ReasonML的本机字符串）和 Unix（使用 native API）。

这是两个API的文档：

- [ReasonML API 文档](https://reasonml.github.io/api/)
- [BuckleScript API 文档](https://bucklescript.github.io/bucklescript/api/)

#### Pervasives 模块

[Pervasives 模块](https://reasonml.github.io/api/Pervasives.html)包含核心标准库，并且始终为每个模块自动打开。它包含诸如运算符 ==，+，|> 和诸如 `print_string()` 和 `string_of_int()` 之类的函数。

如果此模块中的某些内容被覆盖，您仍然可以通过例如 Pervasives.(+) 明确地访问它。

如果项目中有一个文件Pervasives.re，它将覆盖内置模块，而是被打开。

#### 具有标签参数的标准函数

以下模块存在两种版本：较旧的版本，其中函数仅具有位置参数，而较新版本的功能也具有标记的参数。

- Array, ArrayLabels
- Bytes, BytesLabels
- List, ListLabels
- String, StringLabels

作为例子，请考虑：

```ocaml
List.map: ('a => 'b, list('a)) => list('b)
ListLabels.map: (~f: 'a => 'b, list('a)) => list('b)
```

另外两个模块提供标签的函数：

- [模块 StdLabels](https://reasonml.github.io/api/StdLabels.html) 具有子模块 `Array，Bytes，List，String`，它们是 ArrayLabels 的别名等。在您的模块中，可以打开 StdLabels 以默认获取带标签的 List 版本。
- [模块 MoreLabels](https://reasonml.github.io/api/MoreLabels.html) 有三个带有标记函数的子模块：`Hashtbl`，`Map` 和 `Set`。

### 安装库

目前，JavaScript 是 ReasonML 的首选平台。因此，安装库的首选方式是通过 npm。这工作如下。作为一个例子，假设我们想为 Jest 安装 BuckleScript 绑定（包括 Jest 本身）。相关的 npm 包叫做 bs-jest。

首先，我们需要安装软件包。在 package.json 中有：

```ocaml
{
  "dependencies": {
    "bs-jest": "^0.1.5"
  },
  ···
}
```

其次，我们需要将包添加到 bsconfig.json 中：

```ocaml
{
  "bs-dependencies": [
    "bs-jest"
  ],
  ···
}
```

之后，我们可以使用 Jest.describe() 等模块 Jest。

有关安装库的更多信息：

- BuckleScript 的构建系统在 Chap 中解释。 “[构建系统支持](https://bucklescript.github.io/bucklescript/Manual.html#_build_system_support)”的 BuckleScript 手册。

- ReasonML 的文档解释了[如何在 npm 上找到 ReasonML 库](https://reasonml.github.io/docs/en/libraries.html)。
  - 有用的 npm 关键字包括：reason，reasonml，bucklescript。
