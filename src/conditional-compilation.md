# 条件编译

>[conditional-compilation.md](https://github.com/rust-lang/reference/blob/master/src/conditional-compilation.md)\
>commit: 949726950a4ac5c8674e902dc33c09b48fc7434c \
>本章译文最后维护日期：2021-5-6

> **<sup>句法</sup>**\
> _ConfigurationPredicate_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _ConfigurationOption_\
> &nbsp;&nbsp; | _ConfigurationAll_\
> &nbsp;&nbsp; | _ConfigurationAny_\
> &nbsp;&nbsp; | _ConfigurationNot_
>
> _ConfigurationOption_ :\
> &nbsp;&nbsp; [IDENTIFIER]&nbsp;(`=` ([STRING_LITERAL] | [RAW_STRING_LITERAL]))<sup>?</sup>
>
> _ConfigurationAll_\
> &nbsp;&nbsp; `all` `(` _ConfigurationPredicateList_<sup>?</sup> `)`
>
> _ConfigurationAny_\
> &nbsp;&nbsp; `any` `(` _ConfigurationPredicateList_<sup>?</sup> `)`
>
> _ConfigurationNot_\
> &nbsp;&nbsp; `not` `(` _ConfigurationPredicate_ `)`
>
> _ConfigurationPredicateList_\
> &nbsp;&nbsp; _ConfigurationPredicate_ (`,` _ConfigurationPredicate_)<sup>\*</sup> `,`<sup>?</sup>

根据某些条件，<!-- 这个定义有点空洞 --> *条件性编译的源代码(Conditionally compiled source code)* 可以被认为是 crate 源代码的一部分，也可以不被认为是 crate 源代码的一部分。可以使用[属性][attributes] [`cfg`] 和 [`cfg_attr`] 以及内置的 [`cfg` macro] 来有条件地对源代码进行编译。这些条件可以基于被编译的 crate 的目标架构、传递给编译器的值，以及下面将详细描述的一些其他事项。

每种形式的编译条件都有一个计算结果为真或假的*配置谓词(configuration predicate)*。谓词是以下内容之一：

* 一个配置选项。如果设置了该选项，则为真，如果未设置则为假。
* `all()` 这样的配置谓词列表，列表内的配置谓词以逗号分隔。如果至少有一个谓词为假，则为假。如果没有谓词，则为真。
* `any()` 这样的配置谓词列表，列表内的配置谓词以逗号分隔。如果至少有一个谓词为真，则为真。如果没有谓词，则为假。
* 带一个配置谓词的 `not()` 模式 。如果此谓词为假，整个配置它为真；如果此谓词为真，整个配置为假。

*配置选项*可以是名称，也可以是键值对，它们可以设置，也可以不设置。名称以单个标识符形式写入，例如 `unix`。键值对被写为标识符后跟 `=`，然后再跟一个字符串。例如，`target_arch=“x86_64”` 就是一个配置选项。

> **注意**: `=` 周围的空白符将被忽略。`foo="bar"` 和 `foo = "bar"` 是等价的配置选项。

键在键值对配置选项列表中不是唯一的。例如，`feature = "std"` and `feature = "serde"` 可以同时设置。

## 设置配置选项

设置哪些配置选项是在 crate 编译期时就静态确定的。一些选项属于*编译器设置集(compiler-set)*，这部分选项是编译器根据相关编译数据设置的。其他选项属于*任意设置集(arbitrarily-set)*，这部分设置必须从代码之外传参给编译器来自主设置。无法在正在编译的 crate 的源代码中设置编译配置选项。

> **注意**: 对于 `rustc`，任意配置集的配置选项要使用命令行参数 [`--cfg`] 来设置。

> **注意**: 键名为 `feature` 的配置选项一般被 [Cargo][cargo-feature] 约定用于指定编译期（用到的各种编译）选项和可选依赖项。

<div class="warning">

警告：任意配置集的配置选项可能与编译器设置集的配置选项设置相同的值。例如，在编译一个 Windows 目标时，可以执行命令行 `rustc --cfg "unix" program.rs`，这样就同时设置了 `unix` 和 `windows` 配置选项。但实际上这样做是不明智的。

</div>

### `target_arch`

键值对选项，用于一次性设置编译目标的 CPU 架构。该值类似于平台的目标三元组(target triple)[^target-triple]中的第一个元素，但也不完全相同。

示例值：

* `"x86"`
* `"x86_64"`
* `"mips"`
* `"powerpc"`
* `"powerpc64"`
* `"arm"`
* `"aarch64"`

### `target_feature`

键值对选项，用于设置当前编译目标的可用平台特性。

示例值：

* `"avx"`
* `"avx2"`
* `"crt-static"`
* `"rdrand"`
* `"sse"`
* `"sse2"`
* `"sse4.1"`

有关可用特性的更多细节，请参见 [`target_feature`属性][`target_feature` attribute]。此外，编译器还为 `target_feature`选项提供了一个额外的 `crt-static`特性，它表示需要链接一个可用的[静态C运行时][static C runtime]。

### `target_os`

键值对选项，用于一次性设置编译目标的操作系统类型。该值类似于平台目标三元组中的第二和第三个元素。

示例值：

* `"windows"`
* `"macos"`
* `"ios"`
* `"linux"`
* `"android"`
* `"freebsd"`
* `"dragonfly"`
* `"openbsd"`
* `"netbsd"`

### `target_family`

键值对选项提供了对具体目标平台更通用化的描述，比如编译目标操作系统或架构。可以设置任意数量的键值对。
最多设置一次，用于设置编译目标的操作系统类别。

示例值：

* `"unix"`
* `"windows"`
* `"wasm"`

### `unix` 和 `windows`

如果设置了 `target_family = "unix"` 则谓词 `unix` 为真；如果设置了 `target_family = "windows"` 则谓词 `windows` 为真。

### `target_env`

键值对选项，用来进一步消除编译目标平台信息与所用 ABI 或 `libc` 相关的歧义。由于历史原因，仅当实际需要消除歧义时，才将此值定义为非空字符串。因此，例如在许多 GNU 平台上，此值将为空。该值类似于平台目标三元组的第四个元素，但也有区别。一个区别是在嵌入式 ABI 上，比如在目标为嵌入式系统时，`gnueabihf` 会简单地将 `target_env` 定义为 `"gnu"`。

示例值：

* `""`
* `"gnu"`
* `"msvc"`
* `"musl"`
* `"sgx"`

### `target_endian`

键值对选项，根据编译目标的 CPU 的字节序(endianness)属性一次性设置值为 “little” 或 “big”。

### `target_pointer_width`

键值对选项，用于一次性设置编译目标的指针位宽(pointer width in bits)。

示例值：

* `"16"`
* `"32"`
* `"64"`

### `target_vendor`

键值对选项，用于一次性设置编译目标的供应商。

示例值：

* `"apple"`
* `"fortanix"`
* `"pc"`
* `"unknown"`

### `test`

在编译测试套件时启用。通过在 `rustc` 里使用 [`--test`] 命令行参数来完成此启用。请参阅[测试][Testing]章节来获取更多和测试支持相关的信息。

### `debug_assertions`

在进行非优化编译时默认启用。这可以用于在开发中启用额外的代码调试功能，但不能在生产中启用。例如，它控制着标准库的 [`debug_assert!`]宏（是否可用）。

### `proc_macro`

当须要指定当前 crate 的[编译输出文件类型(crate-type)][crate type]为 `proc_macro` 时设置。

## 条件编译的形式

### `cfg`属性

> **<sup>句法</sup>**\
> _CfgAttrAttribute_ :\
> &nbsp;&nbsp; `cfg` `(` _ConfigurationPredicate_ `)`

<!-- 这里可以用“激活属性”来称呼吗？should we say they're active attributes here? -->

`cfg`[属性][attribute]根据配置谓词有条件地包括它所附加的东西。

它被写成 `cfg` 后跟一个 `(`，再跟一个配置谓词，最后是一个 `)`。

如果谓词为真，则重写该部分代码，使其上没有 `cfg` 属性。如果谓词为假，则从源代码中删除该内容。

在函数上的一些例子:

```rust
// 该函数只会在编译目标为 macOS 时才会包含在构建中
#[cfg(target_os = "macos")]
fn macos_only() {
  // ...
}

// 此函数仅在定义了 foo 或 bar 时才会被包含在构建中
#[cfg(any(foo, bar))]
fn needs_foo_or_bar() {
  // ...
}

// 此函数仅在编译目标是32位体系架构的类unix 系统时才会被包含在构建中
#[cfg(all(unix, target_pointer_width = "32"))]
fn on_32bit_unix() {
  // ...
}

// 此函数仅在没有定义 foo 时才会被包含在构建中
#[cfg(not(foo))]
fn needs_not_foo() {
  // ...
}
```

`cfg`属性允许在任何允许属性的地方上使用。

### `cfg_attr`属性

> **<sup>句法</sup>**\
> _CfgAttrAttribute_ :\
> &nbsp;&nbsp; `cfg_attr` `(` _ConfigurationPredicate_ `,` _CfgAttrs_<sup>?</sup> `)`
>
> _CfgAttrs_ :\
> &nbsp;&nbsp; [_Attr_]&nbsp;(`,` [_Attr_])<sup>\*</sup> `,`<sup>?</sup>

`cfg_attr`[属性][attribute]根据配置谓词有条件地包含[属性][attributes]。

当配置谓词为真时，此属性展开为谓词后列出的属性。例如，下面的模块可以在 `linux.rs` 或 `windows.rs` 中都能找到。

<!-- ignore: `mod` needs multiple files -->
```rust,ignore
#[cfg_attr(target_os = "linux", path = "linux.rs")]
#[cfg_attr(windows, path = "windows.rs")]
mod os;
```

可以列出零个、一个或多个属性。多个属性将各自展开为单独的属性。例如：

<!-- ignore: fake attributes -->
```rust,ignore
#[cfg_attr(feature = "magic", sparkles, crackles)]
fn bewitched() {}

// 当启用了 `magic` 特性时, 上面的代码将会被展开为:
#[sparkles]
#[crackles]
fn bewitched() {}
```

> **注意**: `cfg_attr` 能展开为另一个 `cfg_attr`。比如：\
> `#[cfg_attr(target_os = "linux", cfg_attr(feature = "multithreaded", some_other_attribute))]` \
> 这个是合法的。这个示例等效于：\
> `#[cfg_attr(all(target_os = "linux", feature ="multithreaded"), some_other_attribute)]`.

`cfg_attr` 属性允许在任何允许属性的地方上使用。

### The `cfg` macro
### `cfg`宏

内置的 `cfg`宏接受单个配置谓词，当谓词为真时计算为 `true` 字面量，当谓词为假时计算为 `false` 字面量。

例如：

```rust
let machine_kind = if cfg!(unix) {
  "unix"
} else if cfg!(windows) {
  "windows"
} else {
  "unknown"
};

println!("I'm running on a {} machine!", machine_kind);
```

[^target-triple]: 首先给出 *目标三元组* 的参考资料地址：https://www.bookstack.cn/read/rCore_tutorial_doc/d997e9cbdfeef7d4.md。\
接下来为防止该地址失效，我用自己的理解简单重复一下我对这个名词的理解:\
目标三元组可以理解为我们常说的平台信息，包含这些信息：第一项元素：CPU 架构；第二项元素：供应商；第三项元素：操作系统；第四项元素：ABI。\
Rust 下查看当前平台的三元组属性可以用 `rustc --version --verbose` 命令行。比如我在我工作机下下执行这行命令行的输出的 `host` 信息为：`host: x86_64-unknown-linux-gnu`，那我都工作机的目标三元组的信息就是：CPU 架构为 x86_64 ，供应商为 unknown ，操作系统为 linux ，ABI 为 gnu 。\
我另一台 windows 机器为：`host: x86_64-pc-windows-msvc`，那这台的目标三元组的信息为：CPU 架构为 x86_64 ，供应商为 pc ，操作系统为 windows ，ABI 为 msvc 。\
Rust 官方对一些平台提供了默认的目标三元组，我们可以通过 `rustc --print target-list` 命令来查看完整列表。

[IDENTIFIER]: identifiers.md
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[STRING_LITERAL]: tokens.md#string-literals
[Testing]: attributes/testing.md
[_Attr_]: attributes.md
[`--cfg`]: https://doc.rust-lang.org/rustc/command-line-arguments.html#--cfg-configure-the-compilation-environment
[`--test`]: https://doc.rust-lang.org/rustc/command-line-arguments.html#--test-build-a-test-harness
[`cfg`]: #the-cfg-attribute
[`cfg` macro]: #the-cfg-macro
[`cfg_attr`]: #the-cfg_attr-attribute
[`debug_assert!`]: https://doc.rust-lang.org/std/macro.debug_assert.html
[`target_feature` attribute]: attributes/codegen.md#the-target_feature-attribute
[attribute]: attributes.md
[attributes]: attributes.md
[cargo-feature]: https://doc.rust-lang.org/cargo/reference/features.html
[crate type]: linkage.md
[static C runtime]: linkage.md#static-and-dynamic-c-runtimes
