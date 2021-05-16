# 函数指针类型

>[function-pointer.md](https://github.com/rust-lang/reference/blob/master/src/types/function-pointer.md)\
>commit: 761ad774fcb300f2b506fed7b4dbe753cda88d80 \
>本章译文最后维护日期：2021-1-17

> **<sup>句法</sup>**\
> _BareFunctionType_ :\
> &nbsp;&nbsp; [_ForLifetimes_]<sup>?</sup> _FunctionTypeQualifiers_ `fn`\
> &nbsp;&nbsp; &nbsp;&nbsp;  `(` _FunctionParametersMaybeNamedVariadic_<sup>?</sup> `)` _BareFunctionReturnType_<sup>?</sup>
>
> _FunctionTypeQualifiers_:\
> &nbsp;&nbsp; `unsafe`<sup>?</sup> (`extern` [_Abi_]<sup>?</sup>)<sup>?</sup>
>
> _BareFunctionReturnType_:\
> &nbsp;&nbsp; `->` [_TypeNoBounds_]
>
> _FunctionParametersMaybeNamedVariadic_ :\
> &nbsp;&nbsp; _MaybeNamedFunctionParameters_ | _MaybeNamedFunctionParametersVariadic_
>
> _MaybeNamedFunctionParameters_ :\
> &nbsp;&nbsp; _MaybeNamedParam_ ( `,` _MaybeNamedParam_ )<sup>\*</sup> `,`<sup>?</sup>
>
> _MaybeNamedParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> ( ( [IDENTIFIER] | `_` ) `:` )<sup>?</sup> [_Type_]
>
> _MaybeNamedFunctionParametersVariadic_ :\
> &nbsp;&nbsp; ( _MaybeNamedParam_ `,` )<sup>\*</sup> _MaybeNamedParam_ `,` [_OuterAttribute_]<sup>\*</sup> `...`

函数指针类型（使用关键字 `fn` 写出）指向那些在编译时不必知道函数标识符的函数。它们也可以由[函数项][function items]类型或非捕获(non-capturing)[闭包][closures]经过一次自动强转(coercion)来创建。

非安全(`unsafe`)限定符表示类型的值是一个[非安全函数][unsafe function]，而外部(`extern`)限定符表示它是一个[外部函数][extern function]。

可变参数只能通过使用 `"C"` 或 `"cdecl"` 的 ABI调用约定的 [`extern`]函数类型来指定。

下面示例中 `Binop` 被定义为函数指针类型：

```rust
fn add(x: i32, y: i32) -> i32 {
    x + y
}

let mut x = add(5,7);

type Binop = fn(i32, i32) -> i32;
let bo: Binop = add;
x = bo(5,7);
```

## 函数指针参数上的属性

函数指针参数上的属性遵循与[常规函数参数][regular function parameters]相同的规则和限制。

[IDENTIFIER]: ../identifiers.md
[_Abi_]: ../items/functions.md
[_ForLifetimes_]: ../items/generics.md#where-clauses
[_TypeNoBounds_]: ../types.md#type-expressions
[_Type_]: ../types.md#type-expressions
[_OuterAttribute_]: ../attributes.md
[`extern`]: ../items/external-blocks.md
[closures]: closure.md
[extern function]: ../items/functions.md#extern-function-qualifier
[function items]: function-item.md
[unsafe function]: ../unsafe-functions.md
[regular function parameters]: ../items/functions.md#attributes-on-function-parameters

<!-- 2021-1-17-->
<!-- checked -->
