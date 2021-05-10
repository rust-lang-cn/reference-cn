# 条件编译

> **<sup>Syntax</sup>**\
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

根据某些条件，*条件编译的源代码* 可能被认为是 crate 源代码的一部分，也可能不被认为是 crate 源代码的一部分。<!-- 定义有点空洞  -->可以使用[属性][attributes] [`cfg`] 和 [`cfg_attr`] 以及内置的 [`cfg` 宏][`cfg` macro]有条件地编译源代码。这些条件基于已编译的 crate 的目标架构、传递给编译器的任意值，以及下面将进一步详细描述的其他一些事项。

每种形式的条件编译都使用一个计算结果为 true 或者 false 的 _配置项_。配置项是以下所述之一：

* 配置选项设置。如果设置了选项，则为 true；如果未设置，则为 false。
* `all()` 使用逗号分隔的配置选项列表。如果至少一个配置选项为 false，则计算结果为 false。如果没有配置选项，则计算结果为 true。
* `any()` 使用逗号分隔的配置选项列表。如果至少一个配置选项为 true，则计算结果为 true。如果没有配置选项，则计算结果为 false。
* `not()` 配置项。如果配置选项为 false，则计算结果为 true。如果配置选项为 true，则计算结果为 false。

*配置选项* 是已设置或未设置的名称和键-值对。名称写为单个标识符，例如 `unix`。键-值对被写为一个标识符，`=`，然后是一个字符串。例如，`target_arch = "x86_64"` 是一个配置选项。

> **注意**：`=` 两边的空格将被忽略。`foo="bar"` 和 `foo = "bar"` 是等效的配置选项。

键在键-值配置选项的集合中不是唯一的。例如，`feature = "std"` 和 `feature = "serde"` 可以同时设置。

## 设置配置选项

设置哪些配置选项是在 crate 编译期间静态确定的。某些选项是根据有关编译的数据由 *编译器设置*。其他选项是根据代码传递给编译器的输入 *主观设置*。无法从正在编译的 crate 源代码中设置配置选项。

> **注意**：对于 `rustc`，使用 [`--cfg`] 标志主观设置配置选项。

> **注意**：带有关键字 `feature` 的配置选项是 [Cargo][cargo-feature] 用于指定编译时选项和可选依赖项的约定。

<div class="warning">

警告：主观设置的配置选项可能与编译器设置的配置选项具有相同的值。例如，可以在编译到 Windows 平台目标时，执行 `rustc --cfg "unix" program.rs`，同时设置 `unix` 和 `windows` 配置选项。实际上，这样做是不明智的。

</div>

### `target_arch`

目标平台的 CPU 架构相关键-值选项，设置一次。该值类似于平台的指标三元组的第一个元素，但不尽相同。

示例值：

* `"x86"`
* `"x86_64"`
* `"mips"`
* `"powerpc"`
* `"powerpc64"`
* `"arm"`
* `"aarch64"`

### `target_feature`

可用于当前编译目标的，各个平台特性相关的键-值选项设置。

示例值：

* `"avx"`
* `"avx2"`
* `"crt-static"`
* `"rdrand"`
* `"sse"`
* `"sse2"`
* `"sse4.1"`

有关可用特性的更多细节，请参阅 [`target_feature` 属性][`target_feature` attribute]。`crt-static` 的附加特性可用于 `target_feature` 选项，在指示[静态 C 运行时][static C runtime]可用。

### `target_os`

目标平台的操作系统相关键-值选项，设置一次。该值类似于平台的指标三元组的第二和第三个元素。

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

目标平台的操作系统家族相关键-值选项，至多设置一次。

示例值：

* `"unix"`
* `"windows"`

### `unix` 和 `windows`

如果设置 `target_family = "unix"`，则操作系统家族设置为 `unix`；如果设置
`target_family = "windows"`，则操作系统家族设置为 `windows`。

### `target_env`

设置键值选项，进一步消除关于目标平台的歧义信息，以及目标平台所使用 ABI 或 `libc` 的相关信息。由于历史原因，仅当在实际需要消除歧义时，才将此值定义为非空字符串。因此，以许多 GNU 平台为例，此值将为空。该值类似于平台的目标三元组的第四个元素。一个不同之处是，`gnueabihf` 等嵌入式 ABIs 将简单地定义为 `target_env` 为 `"gnu"`。

示例值：

* `""`
* `"gnu"`
* `"msvc"`
* `"musl"`
* `"sgx"`

### `target_endian`

目标 CPU 的字节序键-值选项，设置一次，值为 “little” 或 “big”，具体取决于目标 CPU 的字节序。

### `target_pointer_width`

目标平台指针的位长度键-值选项，设置一次。例如，对于具有 32-位 指针的目标，此值设置为 `"32"`。同样，对于具有 64-位 指针的目标，它被设置为 `"64"`。

<!-- 这些目标是否有不同的位编号？ -->

### `target_vendor`

目标平台供应商键-值选项，设置一次。

示例值：

* `"apple"`
* `"fortanix"`
* `"pc"`
* `"unknown"`

### `test`

在编译测试代码时启用。通过使用 [`--test`] 参数执行 `rustc` 编译。有关测试支持的更多信息，参阅[测试][Testing]章节。

### `debug_assertions`

在不进行优化的情况下，编译时默认启用。这可以用于在开发中启用额外的调试代码，但不能在生产中使用。例如，它控制标准库的 [`debug_assert!`] 宏。

### `proc_macro`

当正在编译的 crate 使用 `proc_macro`
[crate 类型][crate type]编译时设置。

## 条件编译的形式

### `cfg` 属性

> **<sup>Syntax</sup>**\
> _CfgAttrAttribute_ :\
> &nbsp;&nbsp; `cfg` `(` _ConfigurationPredicate_ `)`

<!-- should we say they're active attributes here? -->

`cfg` [属性][attribute]根据配置选项有条件地包含它所附加到的编译条件。

它被写为：`cfg`，`(`，配置选型键-值对，最后是 `)`。

如果配置选项计算结果为 true，则重写该配置，使其不具有 `cfg` 属性。如果配置选项计算结果为 false，则从源代码中删除该配置。

函数应用 `cfg` 的一些示例：

```rust
// The function is only included in the build when compiling for macOS
#[cfg(target_os = "macos")]
fn macos_only() {
  // ...
}

// This function is only included when either foo or bar is defined
#[cfg(any(foo, bar))]
fn needs_foo_or_bar() {
  // ...
}

// This function is only included when compiling for a unixish OS with a 32-bit
// architecture
#[cfg(all(unix, target_pointer_width = "32"))]
fn on_32bit_unix() {
  // ...
}

// This function is only included when foo is not defined
#[cfg(not(foo))]
fn needs_not_foo() {
  // ...
}
```

`cfg` 属性可用在任何允许其属性使用的位置。

### `cfg_attr` 属性

> **<sup>Syntax</sup>**\
> _CfgAttrAttribute_ :\
> &nbsp;&nbsp; `cfg_attr` `(` _ConfigurationPredicate_ `,` _CfgAttrs_<sup>?</sup> `)`
>
> _CfgAttrs_ :\
> &nbsp;&nbsp; [_Attr_]&nbsp;(`,` [_Attr_])<sup>\*</sup> `,`<sup>?</sup>

`cfg_attr` [属性][attribute]有条件地包含基于配置选项的[属性][attributes]。

当配置选项计算结果为 true，该属性扩展到配置选项中列出的属性。例如，基于目标平台，以下模块具有 `linux.rs` 或者 `windows.rs`。

<!-- ignore: `mod` needs multiple files -->
```rust,ignore
#[cfg_attr(target_os = "linux", path = "linux.rs")]
#[cfg_attr(windows, path = "windows.rs")]
mod os;
```

可以列出零个、一个或多个属性。多重属性将各自扩展为单独的属性。例如：

<!-- ignore: fake attributes -->
```rust,ignore
#[cfg_attr(feature = "magic", sparkles, crackles)]
fn bewitched() {}

// When the `magic` feature flag is enabled, the above will expand to:
#[sparkles]
#[crackles]
fn bewitched() {}
```

> **注意**：`cfg_attr` 可以扩展为另外的 `cfg_attr`。例如，`#[cfg_attr(target_os = "linux", cfg_attr(feature = "multithreaded", some_other_attribute))]` 是有效的。这个例子等价于 `#[cfg_attr(all(target_os = "linux", feature ="multithreaded"), some_other_attribute)]`。

`cfg_attr` 属性可用在任何允许其属性使用的位置。

### `cfg` 宏

内建的 `cfg` 宏接受单个配置选项。当计算结果为 true 时，值为 `true` 字面量；当计算结果为 false 时，值为 `false` 字面量。

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

[IDENTIFIER]: identifiers.md
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[STRING_LITERAL]: tokens.md#string-literals
[Testing]: attributes/testing.md
[_Attr_]: attributes.md
[`--cfg`]: ../rustc/command-line-arguments.html#--cfg-configure-the-compilation-environment
[`--test`]: ../rustc/command-line-arguments.html#--test-build-a-test-harness
[`cfg`]: #the-cfg-attribute
[`cfg` macro]: #the-cfg-macro
[`cfg_attr`]: #the-cfg_attr-attribute
[`debug_assert!`]: ../std/macro.debug_assert.html
[`target_feature` attribute]: attributes/codegen.md#the-target_feature-attribute
[attribute]: attributes.md
[attributes]: attributes.md
[cargo-feature]: ../cargo/reference/features.html
[crate type]: linkage.md
[static C runtime]: linkage.md#static-and-dynamic-c-runtimes
