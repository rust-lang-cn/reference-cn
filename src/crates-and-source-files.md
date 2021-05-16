# crate 和源文件

>[crates-and-source-files.md](https://github.com/rust-lang/reference/blob/master/src/crates-and-source-files.md)\
>commit: eabdf09207bf3563ae96db9d576de0758c413d5d \
>本章译文最后维护日期：2021-1-24

> **<sup>句法</sup>**\
> _Crate_ :\
> &nbsp;&nbsp; UTF8BOM<sup>?</sup>\
> &nbsp;&nbsp; SHEBANG<sup>?</sup>\
> &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; [_Item_]<sup>\*</sup>

> **<sup>词法</sup>**\
> UTF8BOM : `\uFEFF`\
> SHEBANG : `#!` \~`\n`<sup>\+</sup>[†](#shebang)


> 注意：尽管像任何其他语言一样，Rust 也都可以通过解释器和编译器实现，但现在唯一存在的实现是编译器，并且该语言也是一直被设计为可编译的。因为这些原因，所以本章节所有的讨论都是基于编译器这条路径的。

Rust 的语义有编译时和运行时之间的*阶段差异(phase distinction)*。[^phase-distinction] 其中*静态解释*的语义规则控制编译的成败，而*动态解释*的语义规则控制程序在运行时的行为。

编译模型以 _crate_ 为中心。每次编译都以源码的形式处理单个的 crate，如果成功，将生成二进制形式的单个 crate：可执行文件或某种类型的库文件。[^cratesourcefile]

crate 是编译和链接的单元，也是版本控制、版本分发和运行时加载的基本单元。一个 crate 包含一个嵌套的带作用域的[模块][module]*树*。这个树的顶层是一个匿名的模块（从模块内部路径的角度来看），并且一个 crate 中的任何程序项都有一个规范的[模块路径][module path]来表示它在 crate 的模块树中的位置。

Rust 编译器总是使用单个源文件作为输入来开启编译过程的，并且总是生成单个输出 crate。对输入源文件的处理可能导致其他源文件作为模块被加载进来。源文件的扩展名为 `.rs`。

Rust 源文件描述了一个模块，其名称和（在当前 crate 的模块树中的）位置是从源文件外部定义的：要么通过引用源文件中的显式[模块(_Module_)][module]项，要么由 crate 本身的名称定义。每个源文件都是一个模块，但并非每个模块都需要自己的源文件：多个[模块定义][module]可以嵌套在同一个文件中。

每个源文件包含一个由零个或多个[程序项][_Item_]定义组成的代码序列，并且这些源文件都可选地从应用于其内部模块的任意数量的[属性][attributes]开始，大部分这些属性都会会影响编译器行为。匿名的 crate 根模块可附带一些应用于整个 crate 的属性。

```rust
// 指定 crate 名称.
#![crate_name = "projx"]

// 指定编译输出文件的类型
#![crate_type = "lib"]

// 打开一种警告
// 这句可以放在任何模块中, 而不是只能放在匿名 crate 模块里。
#![warn(non_camel_case_types)]
```

## 字节顺序标记(BOM)

可选的[_UTF8字节序标记_][_UTF8 byte order mark_]（UTF8BOM产生式）表示该文件是用 UTF8 编码的。它只能出现在文件的开头，并且编译器会忽略它。

## Shebang

源文件可以有一个[_shebang_]（SHEBANG产生式），它指示操作系统使用什么程序来执行此文件。它本质上是将源文件作为可执行脚本处理。shebang 只能出现在文件的开头（但是要在可选的 _UTF8BOM_ 生产式之后）。它会被编译器忽略。例如：

<!-- ignore: tests don't like shebang -->
```rust,ignore
#!/usr/bin/env rustx

fn main() {
    println!("Hello!");
}
```

为了避免与[属性][attribute]混淆， Rust 对 shebang 句法做了一个限制：是 `#!` 字符不能后跟 token `[`，忽略中间的[注释][comments]或[空白符][whitespace]。如果违反此限制，则不会将其视为 shebang，而会将其视为属性的开始。

## 预导入包和 `no_std`

本节内容已经移入[预导入包那章里](names/preludes.md)了。
<!-- 本节是为了让其他资料的链入链接不止于立即失效，一旦其他链接被更新，本节就会删除 -->

## main函数

包含 `main`[函数][function]的 crate 可以被编译成可执行文件。如果一个 `main`函数存在，它必须不能有参数，不能对其声明任何 [trait约束或生存期约束][trait or lifetime bounds]，不能有任何 [where子句][where clauses]，并且它的返回类型必须是以下类型之一:

* `()`
* `Result<(), E> where E: Error`
<!-- * `!` -->
<!-- * Result<!, E> where E: Error` -->

> 注意: 允许哪些返回类型的实现是由暂未稳定的 [`Termination`] trait 决定的。

<!-- 如果前面这节需要更新 (从 "必须不能有参数" 开始, 同时需要修改 attributes/testing.md 文件 -->

### `no_main`属性

可在 crate 层级使用 *`no_main`[属性][attribute]*来禁止对可执行二进制文件发布 `main` symbol，即禁止当前 crate 的 main 函数的执行。当链接的其他对象定义了 `main`函数时，这很有用。

## `crate_name`属性

可在 crate 层级应用 *`crate_name`[属性][attribute]*，并通过使用 [_MetaNameValueStr_]元项属性句法来指定 crate 的名称。

```rust
#![crate_name = "mycrate"]
```

crate 名称不能为空，且只能包含 [Unicode字母数字]或字符 `-`(U+002D)。

[^phase-distinction]: 这种区别也存在于解释器中。静态检查，如语法分析、类型检查和 lint检查，都应该在程序执行之前进行，而不要去管程序何时执行。

[^cratesourcefile]: crate 有点类似于 ECMA-335 CLI 模型中的 *assembly*、SML/NJ 编译管理器中的 *library*、Owens 和 Flatt 模块系统中的 *unit*， 或 Mesa 中的 *configuration*。

[Unicode alphanumeric]: https://doc.rust-lang.org/std/primitive.char.html#method.is_alphanumeric
[_InnerAttribute_]: attributes.md
[_Item_]: items.md
[_MetaNameValueStr_]: attributes.md#meta-item-attribute-syntax
[_shebang_]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[_utf8 byte order mark_]: https://en.wikipedia.org/wiki/Byte_order_mark#UTF-8
[`Termination`]: https://doc.rust-lang.org/std/process/trait.Termination.html
[attribute]: attributes.md
[attributes]: attributes.md
[comments]: comments.md
[function]: items/functions.md
[module]: items/modules.md
[module path]: paths.md
[trait or lifetime bounds]: trait-bounds.md
[where clauses]: items/generics.md#where-clauses
[whitespace]: whitespace.md

<script>
(function() {
    var fragments = {
        "#preludes-and-no_std": "names/preludes.html",
    };
    var target = fragments[window.location.hash];
    if (target) {
        var url = window.location.toString();
        var base = url.substring(0, url.lastIndexOf('/'));
        window.location.replace(base + "/" + target);
    }
})();
</script>

<!-- 2020-11-12-->
<!-- checked -->
