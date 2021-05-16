# 宏

>[macros.md](https://github.com/rust-lang/reference/blob/master/src/macros.md)\
>commit: 012bfafbd995c54a86ebb542bbde5874710cba19 \
>本章译文最后维护日期：2020-1-24

可以使用称被为宏的自定义句法形式来扩展 Rust 的功能和句法。宏需要被命名，并通过一致的句法去调用：`some_extension!(...)`。

定义新宏有两种方式：

* [声明宏(Macros by Example)][Macros by Example]以更高级别的声明性的方式定义了一套新句法规则。
* [过程宏(Procedural Macros)][Procedural Macros]可用于实现自定义派生。

## Macro Invocation
## 宏调用

> **<sup>句法</sup>**\
> _MacroInvocation_ :\
> &nbsp;&nbsp; [_SimplePath_] `!` _DelimTokenTree_
>
> _DelimTokenTree_ :\
> &nbsp;&nbsp; &nbsp;&nbsp;  `(` _TokenTree_<sup>\*</sup> `)`\
> &nbsp;&nbsp; | `[` _TokenTree_<sup>\*</sup> `]`\
> &nbsp;&nbsp; | `{` _TokenTree_<sup>\*</sup> `}`
>
> _TokenTree_ :\
> &nbsp;&nbsp; [_Token_]<sub>_排除 [定界符(delimiters)][delimiters]_</sub> | _DelimTokenTree_
>
> _MacroInvocationSemi_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_SimplePath_] `!` `(` _TokenTree_<sup>\*</sup> `)` `;`\
> &nbsp;&nbsp; | [_SimplePath_] `!` `[` _TokenTree_<sup>\*</sup> `]` `;`\
> &nbsp;&nbsp; | [_SimplePath_] `!` `{` _TokenTree_<sup>\*</sup> `}`

宏调用是在编译时执行宏，并用执行结果替换该调用。可以在下述情况里调用宏：

* [表达式][Expressions]和[语句][statements]
* [模式][Patterns]
* [类型][Types]
* [程序项][Items]，包括[关联程序项][associated items]
* [`macro_rules`] 转码器
* [外部块][External blocks]


当宏调用被用作程序项或语句时，此时它应用的 _MacroInvocationSemi_ 句法规则要求它如果不使用花括号，则在结尾处须添加分号。在宏调用或[宏(`macro_rules`)][`macro_rules`]定义之前不允许使用[可见性限定符][Visibility qualifiers]。

```rust
// 作为表达式使用.
let x = vec![1,2,3];

// 作为语句使用.
println!("Hello!");

// 在模式中使用.
macro_rules! pat {
    ($i:ident) => (Some($i))
}

if let pat!(x) = Some(1) {
    assert_eq!(x, 1);
}

// 在类型中使用.
macro_rules! Tuple {
    { $A:ty, $B:ty } => { ($A, $B) };
}

type N2 = Tuple!(i32, i32);

// 作为程序项使用.
# use std::cell::RefCell;
thread_local!(static FOO: RefCell<u32> = RefCell::new(1));

// 作为关联程序项使用.
macro_rules! const_maker {
    ($t:ty, $v:tt) => { const CONST: $t = $v; };
}
trait T {
    const_maker!{i32, 7}
}

// 宏内调用宏
macro_rules! example {
    () => { println!("Macro call in a macro!") };
}
// 外部宏 `example` 展开后, 内部宏 `println` 才会展开.
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

<!-- 2020-11-12-->
<!-- checked -->
