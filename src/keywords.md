# 关键字

Rust 将关键字分为三类：

* [规定关键字](#规定关键字)
* [保留关键字](#保留关键字)
* [弱<sup>待修正</sup>关键字](#弱sup待修正sup关键字)

> *译者注*：下文 `KW` 为 keywords 的简写。

## 规定关键字

规定关键字仅能在正确的上下文中使用。它们不能用作：

* [项][Items]
* [变量][Variables]和函数参数
* 字段和[枚举][enumerations]
* [类型参数][Type parameters]
* 生命周期参数或者[循环标签][loop labels]
* [宏][Macros]或[属性][attributes]
* [宏占位符][Macro placeholders]
* [crate 和源文件][Crates]

> **<sup>Lexer:<sup>**\
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

2018 版本增加了下述规定关键字。

> **<sup>Lexer 2018+</sup>**\
> KW_ASYNC          : `async`\
> KW_AWAIT          : `await`\
> KW_DYN            : `dyn`

## 保留关键字

保留关键字尚未使用，但已保留供将来使用。它们与严格的关键字有相同的限制。其背后的原因是，禁止当前程序使用这些关键字，从而使当前程序与未来版本的 Rust 兼容。

> **<sup>Lexer</sup>**\
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

2018 版本增加了下述保留关键字。

> **<sup>Lexer 2018+</sup>**\
> KW_TRY   : `try`

## 弱<sup>待修正</sup>关键字

弱<sup>待修正</sup>关键字仅在特定的上下文中才有特殊的含义。例如，可以声明一个名为 `union` 的变量或方法。

* `union` 用来声明一个[联合体][union]，且仅在联合体声明中是关键字。 
* `'static` 用于静态生命周期，且不能用作泛型生命周期参数。

  ```compile_fail
  // error[E0262]: invalid lifetime parameter name: `'static`
  fn invalid_lifetime_parameter<'static>(s: &'static str) -> &'static str { s }
  ```
* 2015 版本中，[`dyn`] 是一个关键字，当在类型位置使用时，其后跟随的路径不能以 `::` 开头。

  自 2018 版本开始，`dyn` 被提升为规定关键字。

> **<sup>Lexer</sup>**\
> KW_UNION          : `union`\
> KW_STATICLIFETIME : `'static`
>
> **<sup>Lexer 2015</sup>**\
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
[enumerations]: items/enumerations.md
[`dyn`]: types/trait-object.md
