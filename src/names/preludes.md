# 预导入包

>[use-declarations.md](https://github.com/rust-lang/reference/blob/master/src/names/preludes.md)\
>commit: 0ce54e64e3c98d99862485c57087d0ab36f40ef0 \
>本章译文最后维护日期：2021-1-24

*预导入包*是一组名称的集合，它会自动把这些名称导入到 crate 中的每个模块的作用域中。

预导入包中的那些名称不是当前模块本身的一部分，它们在[名称解析][name resolution]期间被隐式导入。例如，即使像 [`Box`] 这样在每个模块的作用域中到处使用的名称，你也不能通过 `self::Box` 来引用它，因为它不是当前模块的成员。

有几个不同的预导入包：

- [标准库预导入包][Standard library prelude]
- [外部预导入包][Extern prelude]
- [语言预导入包][Language prelude]
- [`macro_use`预导入包][`macro_use` prelude]
- [工具类预导入包][Tool prelude]

## 标准库预导入包

标准库预导入包包含了 [`std::prelude::v1`]模块中的名称。如果使用[`no_std`属性][`no_std` attribute]，那么它将使用[`core::prelude::v1`]模块中的名称。

## 外部预导入包

在根模块中使用 [`extern crate`] 导入的外部crate 或直接给编译器提供的的外部crate（也就是在 `rustc`命令下使用 `--extern`命令行参数选项）会被添加到*外部预导入包*中。如果使用 `extern crate orig_name as new_name` 这类别名导入，则符号 `new_name` 将被添加到此预导入包。

[`core`] crate 总是会被添加到外部预导入包中。只要 [`no_std`属性][`no_std` attribute]没有在 crate根模块中指定，那么[`std`] crate 就会被添加进来

> **版本差异**：在 2015 版中，在外部预导入包中的 crate 不能通过 [use声明][use declarations]来直接引用，因此通常标准做法是用 `extern crate` 将那它们纳入到当前作用域。
>
> 从 2018 版开始， [use声明][use declarations]可以直接引用外部预导入包里的 crate，所以再在代码里使用 `extern crate` 就会被认为是不规范的。

> **注意**: 随 `rustc` 一起引入的 crate，如 [`alloc`] 和 [`test`]，在使用 Cargo 时不会自动被包含在 `--extern` 命令行参数选项中。即使在 2018 版中，也必须通过外部crate(`extern crate`)声明来把它们引入到当前作用域内。
>
> ```rust
> extern crate alloc;
> use alloc::rc::Rc;
> ```
>
> Cargo却会将 `proc_macro` 带入到编译类型为 proc-macro 的 crate 的外部预导入包中
>
<!--
查看 https://github.com/rust-lang/rust/issues/57288 以了解更多关于 alloc/test 的限制。
-->

### `no_std`属性

默认情况下，标准库自动包含在 crate根模块中。在 [`std`] crate 被添加到根模块中的同时，还会隐式生效一个 [`macro_use`属性][`macro_use` attribute]，它将所有从 `std` 中导出的宏放入到[`macro_use`预导入包][`macro_use` prelude]中。默认情况下，[`core`] 和 [`std`] 都被添加到[外部预导入包][extern prelude]中。[标准库预导入包][standard library prelude]包含了 [`std::prelude::v1`]模块中的所有内容。

*`no_std`[属性][attribute]*可以应用在 crate 级别上，用来防止 [`std`] crate 被自动添加到相关作用域内。此属性作了如下三件事：

* 阻止 `std` crate 被添加进[外部预导入包](#extern-prelude)。
* 使用[标准库预导入包][standard library prelude]中的 [`core::prelude::v1`] 来替代默认情况下导入的 [`std::prelude::v1`]。
* 使用 [`core`] crate 替代 [`std`] crate 来注入到当前 crate 的根模块中，同时把 `core` crate下的所有宏导入到 [`macro_use`预导入包][`macro_use` prelude]中。

> **注意**：当 crate 的目标平台不支持标准库或者故意不使用标准库的功能时，使用核心预导入包而不是标准预导入包是很有用的。此时没有导入的标准库的那些功能主要是动态内存分配（例如：`Box`和' `Vec`）和文件，以及网络功能（例如：`std::fs` 和 `std::io`）。

<div class="warning">

警告：使用 `no_std` 并不会阻止标准库被链接进来。使用 `extern crate std;` 将 `std` crate 导入仍然有效，相关的依赖项也可以被正常链接进来。

</div>

## 语言预导入包

语言预导入包包括语言内置的类型名称和属性名称。语言预导入包总是在当前作用域内有效的。它包括以下内容：

* [类型命名空间][Type namespace]
    * [布尔型][Boolean type] — `bool`
    * [文本型][Textual types] — `char` 和 `str`
    * [整型][Integer types] — `i8`, `i16`, `i32`, `i64`, `i128`, `u8`, `u16`, `u32`, `u64`, `u128`
    * [和机器平台相关的整型][Machine-dependent integer types] — `usize` 和 `isize`
    * [浮点型][floating-point types] — `f32` 和 `f64`
* [宏命名空间][Macro namespace]
    * [内置属性][Built-in attributes]

## `macro_use`预导入包

`macro_use`预导入包包含了外部crate 中的宏，这些宏是通过在当前文档源码内部的 [`extern crate`] 声明语句上应用 [`macro_use`属性][`macro_use` attribute]来导入此声明中的 crate 内部的宏。

## 工具类预导入包

工具类预导入包包含了在[类型命名空间][type namespace]中声明的外部工具的工具名称。请参阅[工具类属性][tool attributes]一节，以了解更多细节。

## `no_implicit_prelude`属性

*`no_implicit_prelude`[属性][attribute]*可以应用在 crate级别或模块上，用以指示它不应该自动将[标准库预导入包][standard library prelude]、[外部预导入包][extern prelude]或[工具类预导入包][tool prelude]引入到当前模块或其任何子模块的作用域中。

此属性不影响[语言预导入包][language prelude]。

> **版本差异**: 在 2015版中，`no_implicit_prelude`属性不会影响[`macro_use`预导入包][`macro_use` prelude]，从标准库导出的所有宏仍然包含在 `macro_use`预导入包中。从 2018版开始，它也会禁止 `macro_use`预导入包生效。


[`alloc`]: ../../alloc/index.html
[`Box`]: ../../std/boxed/struct.Box.html
[`core::prelude::v1`]: ../../core/prelude/index.html
[`core`]: ../../core/index.html
[`extern crate`]: ../items/extern-crates.md
[`macro_use` attribute]: ../macros-by-example.md#the-macro_use-attribute
[`macro_use` prelude]: #macro_use-prelude
[`no_std` attribute]: #the-no_std-attribute
[`no_std` attribute]: #the-no_std-attribute
[`std::prelude::v1`]: ../../std/prelude/index.html
[`std`]: ../../std/index.html
[`test`]: ../../test/index.html
[attribute]: ../attributes.md
[Boolean type]: ../types/boolean.md
[Built-in attributes]: ../attributes.md#built-in-attributes-index
[extern prelude]: #extern-prelude
[floating-point types]: ../types/numeric.md#floating-point-types
[Integer types]: ../types/numeric.md#integer-types
[Language prelude]: #language-prelude
[Machine-dependent integer types]: ../types/numeric.md#machine-dependent-integer-types
[Macro namespace]: namespaces.md
[name resolution]: name-resolution.md
[Standard library prelude]: #standard-library-prelude
[Textual types]: ../types/textual.md
[tool attributes]: ../attributes.md#tool-attributes
[Tool prelude]: #tool-prelude
[Type namespace]: namespaces.md
[use declarations]: ../items/use-declarations.md
