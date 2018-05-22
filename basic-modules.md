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
