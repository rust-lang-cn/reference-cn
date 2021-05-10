# 宏

Rust 语言的功能和语法可以通过自定义宏进行扩展。宏可以被命名，可通过一致的语法调用：`some_extension!(...)`。

定义新宏有两种方式：

* [声明宏][Macros by Example]以更高级别的声明性方式定义新语法。
* [过程宏][Procedural Macros]可用于实现自定义派生。

## 宏调用

> **<sup>Syntax</sup>**\
> _MacroInvocation_ :\
> &nbsp;&nbsp; [_SimplePath_] `!` _DelimTokenTree_
>
> _DelimTokenTree_ :\
> &nbsp;&nbsp; &nbsp;&nbsp;  `(` _TokenTree_<sup>\*</sup> `)`\
> &nbsp;&nbsp; | `[` _TokenTree_<sup>\*</sup> `]`\
> &nbsp;&nbsp; | `{` _TokenTree_<sup>\*</sup> `}`
>
> _TokenTree_ :\
> &nbsp;&nbsp; [_Token_]<sub>_except [delimiters]_</sub> | _DelimTokenTree_
>
> _MacroInvocationSemi_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_SimplePath_] `!` `(` _TokenTree_<sup>\*</sup> `)` `;`\
> &nbsp;&nbsp; | [_SimplePath_] `!` `[` _TokenTree_<sup>\*</sup> `]` `;`\
> &nbsp;&nbsp; | [_SimplePath_] `!` `{` _TokenTree_<sup>\*</sup> `}`

宏调用在编译时执行宏，并用执行结果替换调用。可以在下述情况中调用宏：

* [表达式][Expressions]和[语句][statements]
* [模式][Patterns]
* [类型][Types]
* [项][Items]以及[关联项][associated items]
* [`声明宏`][`macro_rules`]转换
* [外部块][External blocks]

当用作项或语句时，_MacroInvocationSemi_ 形式被使用。_MacroInvocationSemi_ 形式中，如果不使用大括号，则在结尾处需要分号。在宏调用或[`声明宏`][`macro_rules`]定义之前，不允许使用[可见性限定符][Visibility qualifiers]。 

```rust
// Used as an expression.
let x = vec![1,2,3];

// Used as a statement.
println!("Hello!");

// Used in a pattern.
macro_rules! pat {
    ($i:ident) => (Some($i))
}

if let pat!(x) = Some(1) {
    assert_eq!(x, 1);
}

// Used in a type.
macro_rules! Tuple {
    { $A:ty, $B:ty } => { ($A, $B) };
}

type N2 = Tuple!(i32, i32);

// Used as an item.
# use std::cell::RefCell;
thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

// Used as an associated item.
macro_rules! const_maker {
    ($t:ty, $v:tt) => { const CONST: $t = $v; };
}
trait T {
    const_maker!{i32, 7}
}

// Macro calls within macros.
macro_rules! example {
    () => { println!("Macro call in a macro!") };
}
// Outer macro `example` is expanded, then inner macro `println` is expanded.
example!();
```

[Macros by Example]: macros-by-example.md
[Procedural Macros]: procedural-macros.md
[_SimplePath_]: paths.md#simple-paths
[_Token_]: tokens.md
[associated items]: items/associated-items.md
[delimiters]: tokens.md#delimiters
[expressions]: expressions.md
[items]: items.md
[`macro_rules`]: macros-by-example.md
[patterns]: patterns.md
[statements]: statements.md
[types]: types.md
[visibility qualifiers]: visibility-and-privacy.md
[External blocks]: items/external-blocks.md
