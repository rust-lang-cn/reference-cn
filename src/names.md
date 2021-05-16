# 名称

>[keywords.md](https://github.com/rust-lang/reference/blob/master/src/names.md)\
>commit: a989af055ff4fd7e1212754490fff72c3f7cc1be
>本章译文最后维护日期：2020-1-25

*实体(entity)*是一种语言结构，在源程序中可以以某种方式被引用，通常是通过[路径(path)][paths]。实体包括[类型][types]、[程序项][items]、[泛型参数][generic parameters]、[变量绑定][variable bindings]、[循环标签][loop labels]、[生存期][lifetimes]、[字段][fields]、[属性][attributes]和各种[lints]。\

*声明(declaration)*是一种句法结构，它可以引入*名称*来引用实体。实体的名称在相关[*作用域(scope)*][*scope*]内有效。作用域是指可以引用该名称的源码区域。

有些实体是在源码中[显式声明的](#explicitly-declared-entities)，有些则[隐式声明](#implicitly-declared-entities)为语言或编译器扩展的一部分。

[*路径*][*Paths*]用于引用实体，该引用的实体可以在其他的作用域内。生存期和循环标签使用一个带有前导单引号的[专用语法来表达][lifetimes-and-loop-labels]。

名称被分隔成不同的[*命名空间*][*namespaces*]，这样允许不同名称空间中的实体拥有相同的名称，且不会发生冲突。

[*名称解析*][*Name resolution*]是将路径、标识符和标签绑定到实体声明的编译时过程。

对某些名称的访问可能会受到此名称的[*可见性*][*visibility*]的限制。

## 显式声明的实体

在源码中显式引入名称的实体有：

* [程序项][Items]:
    * [模块声明][Module declarations]
    * [外部crate声明][External crate declarations]
    * [Use声明][Use declarations]
    * [函数声明][Function declarations] 和 [函数的参数][function parameters]
    * [类型别名][Type aliases]
    * [结构体][struct]、[联合体][union]、[枚举][enum]、枚举变体声明和它们的字段
    * [常量项声明][Constant item declarations]
    * [静态项声明][Static item declarations]
    * [trait项声明][Trait item declarations]和它们的[关联项][associated items]
    * [外部块][External block items]
    * [`macro_rules`声明][`macro_rules` declarations] 和 [匹配器元变量][matcher metavariables]
    * [实现][Implementation]中的关联项
* [表达式][Expressions]:
    * [闭包][Closure]的参数
    * [`while let`] 模式绑定
    * [`for`] 模式绑定
    * [`if let`] 模式绑定
    * [`match`] 模式绑定
    * [循环标签][Loop labels]
* [泛型参数][Generic parameters]
* [高阶trait约束][Higher ranked trait bounds]
* [`let`语句][`let` statement]中的模式绑定
* [`macro_use`属性][`macro_use` attribute]可以从其他 crate 里引入宏名称。
* [`macro_export`属性][`macro_export` attribute]可以为当前宏引入一个在当前 crate 的根模块下生效的别名

此外，[宏调用][macro invocations]和[属性][attributes]可以通过扩展源代码到上述程序项之一来引入名称。

## 隐式声明的实体

以下实体由语言隐式定义，或由编译器选项和编译器扩展引入：

* [语言预导入包][Language prelude]:
    * [布尔型][Boolean type] — `bool`
    * [文本型][Textual types] — `char` and `str`
    * [整型][Integer types] — `i8`, `i16`, `i32`, `i64`, `i128`, `u8`, `u16`, `u32`, `u64`, `u128`
    * [和机器平台相关的整型][Machine-dependent integer types] — `usize` and `isize`
    * [浮点型][floating-point types] — `f32` and `f64`
* [内置属性][Built-in attributes]
* [标准库预导入包][Standard library prelude]里的程序项、属性和宏
* 在根模块下的[标准库][extern-prelude]里的crate
* 通过编译器链接进的[外部crate][extern-prelude]
* [工具类属性][Tool attributes]
* [Lints] 和 [工具类lint属性][tool lint attributes]
* [派生辅助属性][Derive helper attributes]无需显示引入，就在其程序项内有效
* [`'static`]生存期标签

此外，crate 的根模块没有名称，但可以使用某些[路径限定符][path qualifiers]或别名来引用。

[*Name resolution*]: names/name-resolution.md
[*namespaces*]: names/namespaces.md
[*paths*]: paths.md
[*scope*]: names/scopes.md
[*visibility*]: visibility-and-privacy.md
[`'static`]: keywords.md#weak-keywords
[`for`]: expressions/loop-expr.md#iterator-loops
[`if let`]: expressions/if-expr.md#if-let-expressions
[`let` statement]: statements.md#let-statements
[`macro_export` attribute]: macros-by-example.md#path-based-scope
[`macro_rules` declarations]: macros-by-example.md
[`macro_use` attribute]: macros-by-example.md#the-macro_use-attribute
[`match`]: expressions/match-expr.md
[`while let`]: expressions/loop-expr.md#predicate-pattern-loops
[associated items]: items/associated-items.md
[attributes]: attributes.md
[Boolean type]: types/boolean.md
[Built-in attributes]: attributes.md#built-in-attributes-index
[Closure]: expressions/closure-expr.md
[Constant item declarations]: items/constant-items.md
[Derive helper attributes]: procedural-macros.md#derive-macro-helper-attributes
[enum]: items/enumerations.md
[Expressions]: expressions.md
[extern-prelude]: names/preludes.md#extern-prelude
[External block items]: items/external-blocks.md
[External crate declarations]: items/extern-crates.md
[fields]: expressions/field-expr.md
[floating-point types]: types/numeric.md#floating-point-types
[Function declarations]: items/functions.md
[function parameters]: items/functions.md#function-parameters
[Generic parameters]: items/generics.md
[Higher ranked trait bounds]: trait-bounds.md#higher-ranked-trait-bounds
[Implementation]: items/implementations.md
[Integer types]: types/numeric.md#integer-types
[Items]: items.md
[Language prelude]: names/preludes.md#language-prelude
[lifetimes-and-loop-labels]: tokens.md#lifetimes-and-loop-labels
[lifetimes]: tokens.md#lifetimes-and-loop-labels
[Lints]: attributes/diagnostics.md#lint-check-attributes
[Loop labels]: expressions/loop-expr.md#loop-labels
[Machine-dependent integer types]: types/numeric.md#machine-dependent-integer-types
[macro invocations]: macros.md#macro-invocation
[matcher metavariables]: macros-by-example.md#metavariables
[Module declarations]: items/modules.md
[path]: paths.md
[path qualifiers]: paths.md#path-qualifiers
[Standard library prelude]: names/preludes.md#standard-library-prelude
[Static item declarations]: items/static-items.md
[struct]: items/structs.md
[Textual types]: types/textual.md
[Tool attributes]: attributes.md#tool-attributes
[tool lint attributes]: attributes/diagnostics.md#tool-lint-attributes
[Trait item declarations]: items/traits.md
[Type aliases]: items/type-aliases.md
[types]: types.md
[union]: items/unions.md
[Use declarations]: items/use-declarations.md
[variable bindings]: patterns.md
