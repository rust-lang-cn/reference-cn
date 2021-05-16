{{#include attributes-redirect.html}}
# 属性

>[attributes.md](https://github.com/rust-lang/reference/blob/master/src/attributes.md)\
>commit: eabdf09207bf3563ae96db9d576de0758c413d5d \
>本章译文最后维护日期：2021-1-24

> **<sup>句法</sup>**\
> _InnerAttribute_ :\
> &nbsp;&nbsp; `#` `!` `[` _Attr_ `]`
>
> _OuterAttribute_ :\
> &nbsp;&nbsp; `#` `[` _Attr_ `]`
>
> _Attr_ :\
> &nbsp;&nbsp; [_SimplePath_] _AttrInput_<sup>?</sup>
>
> _AttrInput_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_DelimTokenTree_]\
> &nbsp;&nbsp; | `=` [_LiteralExpression_]<sub>_不带后缀_</sub>

*属性*是一种通用的、格式自由的元数据(free-form metadatum)，这种元数据会（被编译器/解释器）依据名称、约定、语言和编译器版本进行解释。（Rust 语言中的）属性是根据 [ECMA-335] 标准中的属性规范进行建模的，其语法来自 [ECMA-334] \(C#）。

*内部属性(Inner attributes)*以 `#!` 开头的方式编写，应用于它在其中声明的程序项。*外部属性(Outer attributes)*以不后跟感叹号的(`!`)的 `#` 开头的方式编写，应用于属性后面的内容。

属性由指向属性的路径和路径后跟的可选的带定界符的 token树(delimited token tree)（其解释由属性定义）组成。除了宏属性之外，其他属性的输入也允许使用等号(`=`)后跟文字表达式的格式。更多细节请参见下面的[元项属性句法(meta item syntax)](#meta-item-attribute-syntax)。

属性可以分为以下几类：

* [内置属性][Built-in attributes]
* [宏属性][attribute macros]
* [派生宏辅助属性][Derive macro helper attributes]
* [外部工具属性](#tool-attributes)

属性可以应用于语言中的许多场景：

* 所有的[程序项声明][item declarations]都可接受外部属性，同时[外部块][external blocks]、[函数][functions]、[实现][implementations]和[模块][modules]都可接受内部属性。
* 大多数[语句][statements]都可接受外部属性（参见[表达式属性][Expression Attributes]，了解表达式语句的限制）。
* [块表达式][Block expressions]也可接受外部属性和内部属性，但只有当它们是另一个[表达式语句][expression statement]的外层表达式时，或是另一个块表达式的最终表达式(final expression)时才有效。
* [枚举(`enum`)][Enum]变体和[结构体(`struct`)][struct]、[联合体(`union`)][union]的字段可接受外部属性。
* [匹配表达式的匹配臂(arms)][match expressions]可接受外部属性。
* [泛型生存期(Generic lifetime)或类型参数][generics]可接受外部属性。
* 表达式在有限的情况下可接受外部属性，详见[表达式属性][Expression Attributes]。
* [函数][functions]、[闭包][closure]和[函数指针][function pointer]的参数可接受外部属性。这包括函数指针和[外部块][variadic functions]中用 `...` 表示的可变参数上的属性。

属性的一些例子：

```rust
// 应用于当前模块或 crate 的一般性元数据。
#![crate_type = "lib"]

// 标记为单元测试的函数
#[test]
fn test_foo() {
    /* ... */
}

// 一个条件编译模块
#[cfg(target_os = "linux")]
mod bar {
    /* ... */
}

// 用于静音 lint检查后报告的告警和错误提醒
#[allow(non_camel_case_types)]
type int8_t = i8;

// 适用于整个函数的内部属性
fn some_unused_variables() {
  #![allow(unused_variables)]

  let x = ();
  let y = ();
  let z = ();
}
```

## 元项/元程序项属性句法

“元项(meta item)”是遵循 _Attr_ 产生式（见本章头部）的句法，Rust 的大多数[内置属性(built-in attributes)][built-in attributes]都使用了此句法。它有以下文法格式：

> **<sup>句法</sup>**\
> _MetaItem_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_SimplePath_]\
> &nbsp;&nbsp; | [_SimplePath_] `=` [_LiteralExpression_]<sub>_不带后缀_</sub>\
> &nbsp;&nbsp; | [_SimplePath_] `(` _MetaSeq_<sup>?</sup> `)`
>
> _MetaSeq_ :\
> &nbsp;&nbsp; _MetaItemInner_ ( `,` MetaItemInner )<sup>\*</sup> `,`<sup>?</sup>
>
> _MetaItemInner_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _MetaItem_\
> &nbsp;&nbsp; | [_LiteralExpression_]<sub>_不带后缀_</sub>

元项中的字面量表达式不能包含整型或浮点类型的后缀。

各种内置属性使用元项句法的不同子集来指定它们的输入。下面的文法规则展示了一些常用的使用形式：

> **<sup>句法</sup>**\
> _MetaWord_:\
> &nbsp;&nbsp; [IDENTIFIER]
>
> _MetaNameValueStr_:\
> &nbsp;&nbsp; [IDENTIFIER] `=` ([STRING_LITERAL] | [RAW_STRING_LITERAL])
>
> _MetaListPaths_:\
> &nbsp;&nbsp; [IDENTIFIER] `(` ( [_SimplePath_] (`,` [_SimplePath_])* `,`<sup>?</sup> )<sup>?</sup> `)`
>
> _MetaListIdents_:\
> &nbsp;&nbsp; [IDENTIFIER] `(` ( [IDENTIFIER] (`,` [IDENTIFIER])* `,`<sup>?</sup> )<sup>?</sup> `)`
>
> _MetaListNameValueStr_:\
> &nbsp;&nbsp; [IDENTIFIER] `(` ( _MetaNameValueStr_ (`,` _MetaNameValueStr_)* `,`<sup>?</sup> )<sup>?</sup> `)`

元项句法的一些例子是：

形式 | 示例
------|--------
_MetaWord_ | `no_std`
_MetaNameValueStr_ | `doc = "example"`
_MetaListPaths_ | `allow(unused, clippy::inline_always)`
_MetaListIdents_ | `macro_use(foo, bar)`
_MetaListNameValueStr_ | `link(name = "CoreFoundation", kind = "framework")`

## 活跃属性和惰性属性

属性要么是活跃的，要么是惰性的。在属性处理过程中，*活跃属性*将自己从它们所在的对象上移除，而*惰性属性*依然保持原位置不变。

[`cfg`] 和 [`cfg_attr`] 属性是活跃的。[`test`]属性在为测试所做的编译形式中是惰性的，在其他编译形式中是活跃的。[宏属性][Attribute macros]是活跃的。所有其他属性都是惰性的。

## 外部工具属性

编译器可能允许和具体外部工具相关联的属性，但这些工具在编译和检查过程中必须存在并驻留在编译器提供的[工具类预导入包][tool prelude]下对应的命名空间中（才能让这些属性生效）。这种属性的（命名空间）路径的第一段是工具的名称，后跟一个或多个工具自己解释的附加段。

当工具在编译期不可用时，该工具的属性将被静默接受而不提示警告。当工具可用时，该工具负责处理和解释这些属性。

如果使用了 [`no_implicit_prelude`]属性，则外部工具属性不可用。

```rust
// 告诉rustfmt工具不要格式化以下元素。
#[rustfmt::skip]
struct S {
}

// 控制clippy工具的“圈复杂度(cyclomatic complexity)”极限值。
#[clippy::cyclomatic_complexity = "100"]
pub fn f() {}
```

> 注意: `rustc` 目前能识别的工具是 “clippy” 和 “rustfmt”。

## 内置属性的索引表

下面是所有内置属性的索引表：

- 条件编译(Conditional compilation)
  - [`cfg`] — 控制条件编译。
  - [`cfg_attr`] — 选择性包含属性。
- 测试(Testing)
  - [`test`] — 将函数标记为测试函数。
  - [`ignore`] — 禁止测试此函数。
  - [`should_panic`] — 表示测试应该产生 panic。
- 派生(Derive)
  - [`derive`] — 自动部署 trait实现
  - [`automatically_derived`] — 用在由 `derive` 创建的实现上的标记。
- 宏(Macros)
  - [`macro_export`] — 导出声明宏（`macro_rules`宏），用于跨 crate 的使用。
  - [`macro_use`] — 扩展宏可见性，或从其他 crate 导入宏。
  - [`proc_macro`] — 定义类函数宏。
  - [`proc_macro_derive`] — 定义派生宏。
  - [`proc_macro_attribute`] — 定义属性宏。
- 诊断(Diagnostics)
  - [`allow`]、[`warn`]、[`deny`]、[`forbid`] — 更改默认的 lint检查级别。
  - [`deprecated`] — 生成弃用通知。
  - [`must_use`] — 为未使用的值生成 lint 提醒。
- ABI、链接(linking)、符号(symbol)、和 FFI
  - [`link`] — 指定要与外部(`extern`)块链接的本地库。
  - [`link_name`] — 指定外部(`extern`)块中的函数或静态项的符号(symbol)名。
  - [`no_link`] — 防止链接外部crate。
  - [`repr`] — 控制类型的布局。
  - [`crate_type`] — 指定 crate 的类别(库、可执行文件等)。
  - [`no_main`] — 禁止发布 `main`符号(symbol)。
  - [`export_name`] — 指定函数或静态项导出的符号(symbol)名。
  - [`link_section`] — 指定用于函数或静态项的对象文件的部分。
  - [`no_mangle`] — 禁用对符号(symbol)名编码。
  - [`used`] — 强制编译器在输出对象文件中保留静态项。
  - [`crate_name`] — 指定 crate名。
- 代码生成(Code generation)
  - [`inline`] — 内联代码提示。
  - [`cold`] — 提示函数不太可能被调用。
  - [`no_builtins`] — 禁用某些内置函数。
  - [`target_feature`] — 配置特定于平台的代码生成。
  - [`track_caller`] - 将父调用位置传递给 `std::panic::Location::caller()`。
- 文档(Documentation)
  - `doc` — 指定文档。更多信息见 [The Rustdoc Book]。[Doc注释][Doc comments]会被转换为 `doc`属性。
- 预导入包(Preludes)
  - [`no_std`] — 从预导入包中移除 std。
  - [`no_implicit_prelude`] — 禁用模块内的预导入包查找。
- 模块(Modules)
  - [`path`] — 指定模块的源文件名。
- 极限值设置(Limits)
  - [`recursion_limit`] — 设置某些编译时操作的最大递归限制。
  - [`type_length_limit`] — 设置多态类型(polymorphic type)单态化过程中构造具体类型时所做的最大类型替换次数。
- 运行时(Runtime)
  - [`panic_handler`] — 设置处理 panic 的函数。
  - [`global_allocator`] — 设置全局内存分配器。
  - [`windows_subsystem`] — 指定要链接的 windows 子系统。
- 特性(Features)
  - `feature` — 用于启用非稳定的或实验性的编译器特性。参见 [The Unstable Book] 了解在 `rustc` 中实现的特性。
- 类型系统(Type System)
  - [`non_exhaustive`] — 表明一个类型将来会添加更多的字段/变体。

[Doc comments]: comments.md#doc-comments
[ECMA-334]: https://www.ecma-international.org/publications/standards/Ecma-334.htm
[ECMA-335]: https://www.ecma-international.org/publications/standards/Ecma-335.htm
[Expression Attributes]: expressions.md#expression-attributes
[IDENTIFIER]: identifiers.md
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[STRING_LITERAL]: tokens.md#string-literals
[The Rustdoc Book]: https://doc.rust-lang.org/rustdoc/the-doc-attribute.html
[The Unstable Book]: https://doc.rust-lang.org/unstable-book/index.html
[_DelimTokenTree_]: macros.md
[_LiteralExpression_]: expressions/literal-expr.md
[_SimplePath_]: paths.md#simple-paths
[`allow`]: attributes/diagnostics.md#lint-check-attributes
[`automatically_derived`]: attributes/derive.md#the-automatically_derived-attribute
[`cfg_attr`]: conditional-compilation.md#the-cfg_attr-attribute
[`cfg`]: conditional-compilation.md#the-cfg-attribute
[`cold`]: attributes/codegen.md#the-cold-attribute
[`crate_name`]: crates-and-source-files.md#the-crate_name-attribute
[`crate_type`]: linkage.md
[`deny`]: attributes/diagnostics.md#lint-check-attributes
[`deprecated`]: attributes/diagnostics.md#the-deprecated-attribute
[`derive`]: attributes/derive.md
[`export_name`]: abi.md#the-export_name-attribute
[`forbid`]: attributes/diagnostics.md#lint-check-attributes
[`global_allocator`]: runtime.md#the-global_allocator-attribute
[`ignore`]: attributes/testing.md#the-ignore-attribute
[`inline`]: attributes/codegen.md#the-inline-attribute
[`link_name`]: items/external-blocks.md#the-link_name-attribute
[`link_section`]: abi.md#the-link_section-attribute
[`link`]: items/external-blocks.md#the-link-attribute
[`macro_export`]: macros-by-example.md#path-based-scope
[`macro_use`]: macros-by-example.md#the-macro_use-attribute
[`must_use`]: attributes/diagnostics.md#the-must_use-attribute
[`no_builtins`]: attributes/codegen.md#the-no_builtins-attribute
[`no_implicit_prelude`]: names/preludes.md#the-no_implicit_prelude-attribute
[`no_link`]: items/extern-crates.md#the-no_link-attribute
[`no_main`]: crates-and-source-files.md#the-no_main-attribute
[`no_mangle`]: abi.md#the-no_mangle-attribute
[`no_std`]: names/preludes.md#the-no_std-attribute
[`non_exhaustive`]: attributes/type_system.md#the-non_exhaustive-attribute
[`panic_handler`]: runtime.md#the-panic_handler-attribute
[`path`]: items/modules.md#the-path-attribute
[`proc_macro_attribute`]: procedural-macros.md#attribute-macros
[`proc_macro_derive`]: procedural-macros.md#derive-macros
[`proc_macro`]: procedural-macros.md#function-like-procedural-macros
[`recursion_limit`]: attributes/limits.md#the-recursion_limit-attribute
[`repr`]: type-layout.md#representations
[`should_panic`]: attributes/testing.md#the-should_panic-attribute
[`target_feature`]: attributes/codegen.md#the-target_feature-attribute
[`test`]: attributes/testing.md#the-test-attribute
[`track_caller`]: attributes/codegen.md#the-track_caller-attribute
[`type_length_limit`]: attributes/limits.md#the-type_length_limit-attribute
[`used`]: abi.md#the-used-attribute
[`warn`]: attributes/diagnostics.md#lint-check-attributes
[`windows_subsystem`]: runtime.md#the-windows_subsystem-attribute
[attribute macros]: procedural-macros.md#attribute-macros
[block expressions]: expressions/block-expr.md
[built-in attributes]: #built-in-attributes-index
[derive macro helper attributes]: procedural-macros.md#derive-macro-helper-attributes
[enum]: items/enumerations.md
[expression statement]: statements.md#expression-statements
[external blocks]: items/external-blocks.md
[functions]: items/functions.md
[generics]: items/generics.md
[implementations]: items/implementations.md
[item declarations]: items.md
[match expressions]: expressions/match-expr.md
[modules]: items/modules.md
[statements]: statements.md
[struct]: items/structs.md
[tool prelude]: names/preludes.md#tool-prelude
[union]: items/unions.md
[closure]: expressions/closure-expr.md
[function pointer]: types/function-pointer.md
[variadic functions]: items/external-blocks.html#variadic-functions

<!-- 2021-1-24-->
<!-- checked -->
