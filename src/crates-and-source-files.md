# crate 和源文件

> **<sup>Syntax</sup>**\
> _Crate_ :\
> &nbsp;&nbsp; UTF8BOM<sup>?</sup>\
> &nbsp;&nbsp; SHEBANG<sup>?</sup>\
> &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; [_Item_]<sup>\*</sup>

> **<sup>Lexer</sup>**\
> UTF8BOM : `\uFEFF`\
> SHEBANG : `#!` ~[`[` `\n`] ~`\n`<sup>\*</sup>

> 注：尽管 Rust 和其他语言一样，可以由解释器和编译器来实现，但现有的唯一实现是编译器，而且 Rust 语言一直都是为编译而设计的。基于这些原因，本章节设定为编译器。

Rust 语言语义遵循编译时和运行时之间的 *阶段区别*。[^phase-distinction] 具有 *静态解释* 的语义规则，以控制编译的成功或失败；而具有 *动态解释* 的语义规则，将控制程序的运行时行为。

编译模型以 _crates_ 的构件为中心。每次编译都会以源代码的形式处理一个单独的 crate。如果编译成功，将生成二进制形式的单个
crate：一个可执行文件或某种类型的库。[^cratesourcefile]

_crate_ 是编译和链接的单元，也是版本控制、发布和运行时加载的单元。crate 包含嵌套[模块][module]作用域的*树*。该树的顶层是一个匿名的模块（从模块内路径的角度来看），crate 中的任何项都有一个规范的[模块路径][module path]，表示其在 crate 的模块树中的位置。

Rust 编译器总是使用单个源文件作为输入来调用，并且总是生成一个输出 crate。对该源文件的处理可能导致其他源文件作为模块加载。源文件的扩展名为 `.rs`。

Rust 源文件描述了一个模块，其名称和位置——在当前 crate 的模块树中——是从源文件外部定义的：要么通过引用源文件中的显式[模块][module]项，要么通过 crate 本身的名称。每个源文件都是一个模块，但并非每个模块都需要自己的源文件：[模块定义][module]可以嵌套在一个文件中。

每个源文件都包含一个序列——由零个或多个[项][_Item_]定义，序列可应用于源文件所包含的模块，并且可选地从任意数量的[属性][attributes]开始，这些属性中的大多数都会影响编译器的行为。crate 作为一个整体，匿名 crate 模块可以具有应用于整个 crate 的附加属性。

```rust
// Specify the crate name.
#![crate_name = "projx"]

// Specify the type of output artifact.
#![crate_type = "lib"]

// Turn on a warning.
// This can be done in any module, not just the anonymous crate module.
#![warn(non_camel_case_types)]
```

## 字节序标记

可选的 [UTF8 字节序标记][_UTF8 byte order mark_]（由 UTF8BOM 生成）表示文件是 UTF8 编码。它只能出现在文件的开头，编译器会忽略它。

## 释伴（Shebang）

源文件可以有一个[释伴][_shebang_]（由 SHEBANG 生成），它指示操作系统使用什么程序来执行此文件。它本质上是将源文件作为可执行脚本处理。释伴只能出现在文件的开头(但是在可选的 _UTF8BOM_ 之后)。它被编译器忽略。例如：

<!-- ignore: tests don't like shebang -->
```rust,ignore
#!/usr/bin/env rustx

fn main() {
    println!("Hello!");
}
```

Rust 中对释伴语法进行了限制，以避免与[属性][attribute]混淆。`#!` 字符后面不能跟随 `[` 记号，忽略中间的[注释][comments]或[空格][whitespace]。如果此限制失败，则它不被视为释伴，而是作为属性的开始。

## Preludes 和 `no_std`

所有 crates 都有一个 *prelude*，它自动将特定模块——*prelude 模块*——的名称插入到每个[模块][module]的作用域，并将 [`extern
crate`] 插入到 crate 根模块。默认情况下，使用 *标准 prelude*，链接的 crate 是 [`std`]，prelude 模块是 [`std::prelude::v1`]。

在根 crate 模块上，通过使用 `no_std`
[属性][attribute]，prelude 可被变更为 *core prelude*。其链接的 crate 是 [`core`]，prelude 模块是 [`core::prelude::v1`]。当 crate 的目标平台不支持标准库或因一些原因不使用标准库的功能时，使用核心 prelude 而不是标准 prelude 会非常有用。这些功能主要是用在动态内存分配（例如 `Box` 和 `Vec`），以及文件和网络功能（
`std::fs` 和 `std::io`）。

<div class="warning">

警告：使用 `no_std` 不会阻止对标准库的链接。`extern crate std;` 在 crate 仍然会有效执行，并且依赖项也可以链接到 crate 中。

</div>

## 主要功能

包含 `main` [函数][function]的 crate 可以被编译为可执行文件。如果存在
`main` 函数，则其必须不带参数，不得声明任何 [trait 或生命周期边界][trait or lifetime bounds]，不得具有任何 [where 子句][where clauses]，并且其返回类型必须为以下类型之一：

* `()`
* `Result<(), E> where E: Error`
<!-- * `!` -->
<!-- * Result<!, E> where E: Error` -->

> 注意：允许哪些返回类型的实现，是由暂未稳定的 [`Termination`] trait 决定的。

<!-- 如果上一节需要更新（自 “必须不带参数” 后），也需要更新 testing.md 文件 -->

### `no_main` 属性

在 crate 层级，可应用 [*`no_main` 属性*][attribute] 来禁止对可执行二进制文件发出 `main` 符号——即禁止 `main` 函数的执行。当链接到的其他对象定义了 `main` 函数时，若要禁止这些连接到的对象 crate 的执行，`no_main` 属性将很有用。

## `crate_name` 属性

[*`crate_name` 属性*][attribute] 可以应用于 crate 层级，以使用 [_MetaNameValueStr_] 语法指定 crate 的名称。

```rust
#![crate_name = "mycrate"]
```

crate 名称不能为空，并且只能包含 [Unicode 字母、数字][Unicode alphanumeric]，或 `-`（U+002D）字符。

[^阶段区别]：这种区别也存在于解释器中。静态检查（如语法分析、类型检查和 lints）应该在程序执行之前进行，而不管它是何时执行的。

[^crate 源文件]：crate 有点类似于 ECMA-335 CLI 模型中的 *汇编*、SML/NJ 编译管理器中的 *库*、Owens 和 Flatt 模块系统中的 *单元*，或 Mesa 中的 *配置*。

[Unicode alphanumeric]: ../std/primitive.char.html#method.is_alphanumeric
[_InnerAttribute_]: attributes.md
[_Item_]: items.md
[_MetaNameValueStr_]: attributes.md#meta-item-attribute-syntax
[_shebang_]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[_utf8 byte order mark_]: https://en.wikipedia.org/wiki/Byte_order_mark#UTF-8
[`Termination`]: ../std/process/trait.Termination.html
[`core`]: ../core/index.html
[`core::prelude::v1`]: ../core/prelude/index.html
[`extern crate`]: items/extern-crates.md
[`std`]: ../std/index.html
[`std::prelude::v1`]: ../std/prelude/index.html
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
