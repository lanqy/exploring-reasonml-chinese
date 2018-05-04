# 开始使用 ReasonML

> 在本章中，我将提供有关编程语言 [ReasonML](https://reasonml.github.io/) 入门的提示。

## 安装

有两件事要安装：

- bs-platform：安装 BuckleScript 并使您能够将 ReasonML 编译为 JavaScript。 [ReasonML 文档](https://reasonml.github.io/docs/en/quickstart-javascript.html)中描述了安装。

- reason-cli：需要在编辑器中支持 ReasonML ，但也包含各种工具，包括交互式 ReasonML 命令行 `rtop` 。 [ReasonML 文档](https://reasonml.github.io/docs/en/global-installation.html)中描述了安装。编辑器支持由两部分提供：

    - 一方面，所谓的[语言服务器](https://github.com/Microsoft/language-server-protocol)提供了处理 ReasonML 代码的服务。

    - 另一方面，编辑器插件和类似的扩展机制与服务器通信以提供实际的支持。

## 快速尝试 ReasonML

ReasonML在线运行环境（ playground ）

ReasonML 网站包含一个在线运行环境 （ playground ），对于查看语言是如何工作以及相应的 JavaScript 和 OCaml 代码是非常有用的。它也可以从 OCaml 转换为ReasonML（稍后更多）。

[在线运行环境 （ playground ）](https://reasonml.github.io/en/try.html)上的例子让你第一次体验这种语言。

rtop，交互式 ReasonML 命令行

rtop是 ReasonML 的交互式命令行，并通过 shell 从 rtop 启动。与它交互后，它看起来如下：

```ocaml
Reason # 3 + 4;
- : int = 7
```

您已经可以看到 ReasonML 中的所有内容都有静态类型。不要忘记最后的分号 - 它会触发评估！您可以通过 Ctrl-D 或 输入 #quit 退出 rtop;

## 模板项目

有两个模板项目可以帮助你入门。它们是通过bsb（它是bs平台的一部分）创建的：

- [Node.js 代码](https://reasonml.github.io/docs/en/quickstart-javascript.html)：

        ```bsb -init my-first-app -theme basic-reason```

- [Web开发（React）](https://reasonml.github.io/reason-react/docs/en/installation.html)：

        ```bsb -init my-react-app -theme react```

## 重要提示：将 OCaml 转换为 ReasonML

鉴于与 ReasonML 相关的大多数材料都使用 OCaml 的语法，因此能够从 OCaml 的语法转换为 ReasonML 是非常有用的。有两种方法可以这样做：

- 在线运行环境 （ playground ） “[试试 Reason](https://reasonml.github.io/en/try.html)”。

- 工具 refmt（“ ReasonML 格式化工具”）是 reason-cli 的一部分。通过 refmt --help 获取文档。