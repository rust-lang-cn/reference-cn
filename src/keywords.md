# 关键字

>[keywords.md](https://github.com/rust-lang/reference/blob/master/src/keywords.md)\
>commit: 6eb3e87af2c7743d6c7c783154cc380c4b0ea270
>本章译文最后维护日期：2021-04-23

Rust 将关键字分为三类：

  - [严格关键字](#strict-keywords)
  - [保留关键字](#reserved-keywords)
  - [弱关键字](#weak-keywords)

## 严格关键字

这类关键字只能在正确的上下文中使用。它们不能用作以下名称：

* [程序项(item)][Items]
* [变量][Variables]和函数参数
* 字段(field)和[变体][variants]
* [类型参数][Type parameters]
* 生存期参数或者[循环标签][loop labels]
* [宏][Macros]或[属性][attributes]
* [宏占位符][Macro placeholders]
* [crate][Crates]

> **<sup>词法分析:<sup>**\
> KW_AS             : `as`\
> KW_BREAK          : `break`\
> KW_CONST          : `const`\
> KW_CONTINUE       : `continue`\
> KW_CRATE          : `crate`\
> KW_ELSE           : `else`\
> KW_ENUM           : `enum`\
> KW_EXTERN         : `extern`\
> KW_FALSE          : `false`\
> KW_FN             : `fn`\
> KW_FOR            : `for`\
> KW_IF             : `if`\
> KW_IMPL           : `impl`\
> KW_IN             : `in`\
> KW_LET            : `let`\
> KW_LOOP           : `loop`\
> KW_MATCH          : `match`\
> KW_MOD            : `mod`\
> KW_MOVE           : `move`\
> KW_MUT            : `mut`\
> KW_PUB            : `pub`\
> KW_REF            : `ref`\
> KW_RETURN         : `return`\
> KW_SELFVALUE      : `self`\
> KW_SELFTYPE       : `Self`\
> KW_STATIC         : `static`\
> KW_STRUCT         : `struct`\
> KW_SUPER          : `super`\
> KW_TRAIT          : `trait`\
> KW_TRUE           : `true`\
> KW_TYPE           : `type`\
> KW_UNSAFE         : `unsafe`\
> KW_USE            : `use`\
> KW_WHERE          : `where`\
> KW_WHILE          : `while`

以下关键字从 2018 版开始启用。

> **<sup>词法分析 2018+</sup>**\
> KW_ASYNC          : `async`\
> KW_AWAIT          : `await`\
> KW_DYN            : `dyn`

## 保留关键字

这类关键字目前还没有被使用，但是它们被保留以备将来使用。它们具有与严格关键字相同的限制。这样做的原因是通过禁止当前程序使用这些关键字，从而使当前程序能兼容 Rust 的未来版本。

> **<sup>词法分析</sup>**\
> KW_ABSTRACT       : `abstract`\
> KW_BECOME         : `become`\
> KW_BOX            : `box`\
> KW_DO             : `do`\
> KW_FINAL          : `final`\
> KW_MACRO          : `macro`\
> KW_OVERRIDE       : `override`\
> KW_PRIV           : `priv`\
> KW_TYPEOF         : `typeof`\
> KW_UNSIZED        : `unsized`\
> KW_VIRTUAL        : `virtual`\
> KW_YIELD          : `yield`

以下关键字从 2018 版开始成为保留关键字。

> **<sup>词法分析 2018+</sup>**\
> KW_TRY   : `try`

## 弱关键字

这类关键字只有在特定的上下文中才有特殊的意义。例如，可以声明名为 `union` 的变量或方法。

* `macro_rules` 用于创建自定义[宏][macros]。
* `union` 用于声明[联合体(`union`)][union]，它只有在联合体声明中使用时才是关键字。
* `'static` 用于静态生存期，不能用作通用[泛型生存期参数][generic lifetime parameter]和[循环标签][loop label]

  ```compile_fail
  // error[E0262]: invalid lifetime parameter name: `'static`
  fn invalid_lifetime_parameter<'static>(s: &'static str) -> &'static str { s }
  ```
* 在 2015 版本中，当 [`dyn`] 用在非 `::` 开头的路径限定的类型前时，它是关键字。

从 2018 版开始，`dyn` 被提升为一个严格关键字。

> **<sup>词法分析</sup>**\
> KW_UNION          : `union`\
> KW_STATICLIFETIME : `'static`
>
> **<sup>词法分析 2015</sup>**\
> KW_DYN            : `dyn`

[items]: items.md
[Variables]: variables.md
[Type parameters]: types/parameters.md
[loop labels]: expressions/loop-expr.md#loop-labels
[Macros]: macros.md
[attributes]: attributes.md
[Macro placeholders]: macros-by-example.md
[Crates]: crates-and-source-files.md
[union]: items/unions.md
[variants]: items/enumerations.md
[`dyn`]: types/trait-object.md
[loop label]: expressions/loop-expr.md#loop-labels
[generic lifetime parameter]: items/generics.md
