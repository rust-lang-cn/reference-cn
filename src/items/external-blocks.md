# 外部块

>[external-blocks.md](https://github.com/rust-lang/reference/blob/master/src/items/external-blocks.md)\
>commit: 761ad774fcb300f2b506fed7b4dbe753cda88d80 \
>本章译文最后维护日期：2021-1-17

> **<sup>句法</sup>**\
> _ExternBlock_ :\
> &nbsp;&nbsp; `unsafe`<sup>?</sup> `extern` [_Abi_]<sup>?</sup> `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; _ExternalItem_<sup>\*</sup>\
> &nbsp;&nbsp; `}`
>
> _ExternalItem_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_MacroInvocationSemi_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | ( [_Visibility_]<sup>?</sup> ( [_StaticItem_] | [_Function_] ) )\
> &nbsp;&nbsp; )

外部块提供未在当前 crate 中*定义*的程序项的*声明*，外部块是 Rust 外部函数接口的基础。这其实是某种意义上的不受安全检查的导入入口。

外部块里允许存在两种形式的程序项*声明*：[函数][functions]和[静态项][statics]。只有在非安全(`unsafe`)上下文中才能调用在外部块中声明的函数或访问在外部块中声明的静态项。

在句法上，关键字 `unsafe` 允许出现在关键字 `extern` 之前，但是在语义层面却会被弃用。这种设计允许宏在将关键字 `unsafe` 从 token流中移除之前利用此句法来使用此关键字。

## 函数

外部块中的函数与其他 Rust函数的声明方式相同，但这里的函数不能有函数体，取而代之的是直接以分号结尾。外部块中的函数的参数不允许使用模式，只能使用[标识符(IDENTIFIER)][IDENTIFIER] 或 `_` 。函数限定符（`const`、`async`、`unsafe` 和 `extern`）也不允许在这里使用。

外部块中的函数可以被 Rust 代码调用，就跟调用在 Rust 中定义的函数一样。Rust 编译器会自动在 Rust ABI 和外部 ABI 之间进行转换。

在外部块中声明的函数隐式为非安全(`unsafe`)的。当强转为函数指针时，外部块中声明的函数的类型就为 `unsafe extern "abi" for<'l1, ..., 'lm> fn(A1, ..., An) -> R`，其中 `'l1`，…`'lm` 是其生存期参数，`A1`，…，`An` 是该声明的参数的类型，`R` 是该声明的返回类型。

## 静态项

[静态项][statics]在外部块内部与在外部块之外的声明方式相同，只是在外部块内部声明的静态项没有对应的初始化表达式。访问外部块中声明的静态项是 `unsafe` 的，不管它是否可变，因为没有任何保证来确保静态项的内存位模式(bit pattern)对声明它的类型是有效的，因为初始化这些静态项的可能是其他的任意外部代码（例如 C）。

就像外部块之外的[静态项][statics]，外部静态项可以是不可变的，也可以是可变的。在执行任何 Rust 代码之前，不可变外部静态项*必须*被初始化。也就是说，对于外部静态项，仅在 Rust 代码读取它之前对它进行初始化是不够的。

## ABI

不指定 ABI 字符串的默认情况下，外部块会假定使用指定平台上的标准 C ABI 约定来调用当前的库。其他的 ABI 约定可以使用字符串 `abi` 来指定，具体如下所示：

```rust
// 到 Windows API 的接口。（译者注：指定使用 stdcall调用约定去调用 Windows API）
extern "stdcall" { }
```

有三个 ABI 字符串是跨平台的，并且保证所有编译器都支持它们：

* `extern "Rust"` -- 在任何 Rust 语言中编写的普通函数 `fn foo()` 默认使用的 ABI。
* `extern "C"` -- 这等价于 `extern fn foo()`；无论您的 C编译器支持什么默认 ABI。
* `extern "system"` -- 在 Win32 平台之外，中通常等价于 `extern "C"`。在 Win32 平台上，应该使用`"stdcall"`，或者其他应该使用的 ABI 字符串来链接它们自身的 Windows API。

还有一些特定于平台的 ABI 字符串：

* `extern "cdecl"` -- 通过 FFI 调用 x86\_32 C 资源所使用的默认调用约定。
* `extern "stdcall"` -- 通过 FFI 调用 x86\_32架构下的 Win32 API 所使用的默认调用约定 
* `extern "win64"` -- 通过 FFI 调用 x86\_64 Windows 平台下的 C 资源所使用的默认调用约定。
* `extern "sysv64"` -- 通过 FFI 调用 非Windows x86\_64 平台下的 C 资源所使用的默认调用约定。
* `extern "aapcs"` --通过 FFI 调用 ARM 接口所使用的默认调用约定
* `extern "fastcall"` -- `fastcall` ABI——对应于 MSVC 的`__fastcall` 和 GCC 以及 clang 的 `__attribute__((fastcall))`。
* `extern "vectorcall"` -- `vectorcall` ABI ——对应于 MSVC 的 `__vectorcall` 和 clang 的 `__attribute__((vectorcall))`。

## 可变参数函数

可以在外部块内的函数的参数列表中的一个或多个具名参数后通过引入 `...` 来让该函数成为可变参数函数。注意可变参数`...` 前至少有一个具名参数，并且只能位于参数列表的最后。可变参数可以通过标识符来指定：

```rust
extern "C" {
    fn foo(x: i32, ...);
    fn with_name(format: *const u8, args: ...);
}
```

## 外部块上的属性

下面列出的[属性][attributes]可以控制外部块的行为。

### `link`属性

*`link`属性*为外部(`extern`)块中的程序项指定编译器应该链接的本地库的名称。它使用 [_MetaListNameValueStr_]元项属性句法指定其输入参数。`name`键指定要链接的本地库的名称。`kind`键是一个可选值，它指定具有以下可选值的库类型：

- `dylib` — 表示库类型是动态库。如果没有指定 `kind`，这是默认值。
- `static` — 表示库类型是静态库。
- `framework` — 表示库类型是 macOS 框架。这只对 macOS 目标平台有效。

如果指定了 `kind`键，则必须指定 `name`键。

当从主机环境导入 symbols 时，`wasm_import_module`键可用于为外部(`extern`)块中的程序项指定 [WebAssembly模块][WebAssembly module]名称。如果未指定 `wasm_import_module`，则默认模块名为 `env`。

<!-- ignore: requires extern linking -->
```rust,ignore
#[link(name = "crypto")]
extern {
    // …
}

#[link(name = "CoreFoundation", kind = "framework")]
extern {
    // …
}

#[link(wasm_import_module = "foo")]
extern {
    // …
}
```

在空外部块上添加 `link`属性是有效的。可以用这种方式来满足代码中其他地方的外部块的链接需求（包括上游 crate），而不必向每个外部块都添加此属性。

### `link_name`属性

可以在外部(`extern`)块内的程序项声明上指定 `link_name`属性，可以用它来指示要为给定函数或静态项导入的具体 symbol。它使用 [_MetaNameValueStr_]元项属性句法指定 symbol 的名称。

```rust
extern {
    #[link_name = "actual_symbol_name"]
    fn name_in_rust();
}
```

### 函数参数上的属性

外部函数参数上的属性遵循与[常规函数参数][regular function parameters]相同的规则和限制。

[IDENTIFIER]: ../identifiers.md
[WebAssembly module]: https://webassembly.github.io/spec/core/syntax/modules.html
[functions]: functions.md
[statics]: static-items.md
[_Abi_]: functions.md
[_Function_]: functions.md
[_InnerAttribute_]: ../attributes.md
[_MacroInvocationSemi_]: ../macros.md#macro-invocation
[_MetaListNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[_MetaNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[_OuterAttribute_]: ../attributes.md
[_StaticItem_]: static-items.md
[_Visibility_]: ../visibility-and-privacy.md
[attributes]: ../attributes.md
[regular function parameters]: functions.md#attributes-on-function-parameters

<!-- 2021-1-17-->
<!-- checked -->
