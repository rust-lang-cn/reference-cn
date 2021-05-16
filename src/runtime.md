# Rust 运行时

>[abi.md.md](https://github.com/rust-lang/reference/blob/master/src/abi.md)\
>commit:  f8e76ee9368f498f7f044c719de68c7d95da9972 \
>本章译文最后维护日期：2020-11-3

本节介绍 Rust运行时的某些方面的特性。

## `panic_handler`属性

*`panic_handler`属性*只能应用于签名为 `fn(&PanicInfo) -> !` 的函数。有此[属性][attribute]标记的函数定义了 panic 的行为。核心库内的结构体 [`PanicInfo`] 可以收集 panic 发生点的一些信息。在二进制、dylib 或 cdylib 类型的 crate 的依赖关系图(dependency graph)中必须有一个`panic_handler` 函数。

下面展示了一个 `panic_handler` 函数，它记录(log) panic 消息，然后终止(halts)线程。

<!-- ignore: test infrastructure can't handle no_std -->
```rust,ignore
#![no_std]

use core::fmt::{self, Write};
use core::panic::PanicInfo;

struct Sink {
    // ..
#    _0: (),
}
#
# impl Sink {
#     fn new() -> Sink { Sink { _0: () }}
# }
#
# impl fmt::Write for Sink {
#     fn write_str(&mut self, _: &str) -> fmt::Result { Ok(()) }
# }

#[panic_handler]
fn panic(info: &PanicInfo) -> ! {
    let mut sink = Sink::new();

    // logs "panicked at '$reason', src/main.rs:27:4" to some `sink`
    let _ = writeln!(sink, "{}", info);

    loop {}
}
```

### 标准行为

标准库提供了 `panic_handler` 的一个实现，它的默认设置是展开堆栈，但也可以[更改为中止(abort)进程][abort]。标准库的 panic 行为可以使用 [set_hook] 函数在运行时里去修改。

## `global_allocator`属性

在实现 [`GlobalAlloc`] trait 的[静态项][static item]上使用 *`global_allocator`属性*来设置全局分配器。

## `windows_subsystem`属性

当为一个 Windows 编译目标配置链接属性时，*`windows_subsystem`属性*可以用来在 crate 级别上配置[子系统(subsystem)][subsystem]类别。它使用 [_MetaNameValueStr_]元项属性句法用 `console` 或 `windows` 两个可行值指定子系统。对于非windows 编译目标和非二进制的 [crate类型][crate types]，该属性将被忽略。

```rust
#![windows_subsystem = "windows"]
```

[_MetaNameValueStr_]: attributes.md#meta-item-attribute-syntax
[`GlobalAlloc`]: https://doc.rust-lang.org/alloc/alloc/trait.GlobalAlloc.html
[`PanicInfo`]: https://doc.rust-lang.org/core/panic/struct.PanicInfo.html
[abort]: https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html
[attribute]: attributes.md
[crate types]: linkage.md
[set_hook]: https://doc.rust-lang.org/std/panic/fn.set_hook.html
[static item]: items/static-items.md
[subsystem]: https://msdn.microsoft.com/en-us/library/fcc1zstk.aspx

<!-- 2020-11-12-->
<!-- checked -->
