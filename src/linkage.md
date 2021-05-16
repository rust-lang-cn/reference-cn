# 链接

>[linkage.md](https://github.com/rust-lang/reference/blob/master/src/linkage.md)\
>commit: 1426474192ecc9c13165c1d4772b26e8445f5734\
>本章译文最后维护日期：2021-4-23

> 注意：本节更多的是从编译器的角度来描述的，而不是语言。

Rust 编译器支持多种将 crate 链接起来使用的方法，这些链接方法可以是静态的，也可以是动态的。本节将聚焦探索这些链接方法，关于本地库的更多信息请参阅 [The Book 中的 FFI 相关章节][ffi]。

[ffi]: https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#using-extern-functions-to-call-external-code

在一个编译会话中，编译器可以通过使用命令行参数或内部 `crate_type`属性来生成多个构件(artifacts)。如果指定了一个或多个命令行参数，则将忽略（源码内部指定的）所有 `crate_type`属性，以便只构建由命令行指定的构件。

* `--crate-type=bin` 或 `#[crate_type = "bin"]` - 将生成一个可执行文件。这就要求在 crate 中有一个 `main`函数，它将在程序开始执行时运行。这将链接所有 Rust 和本地依赖，生成一个单独的可分发的二进制文件。此类型为默认的 crate 类型。

* `--crate-type=lib` 或 `#[crate_type = "lib"]` - 将生成一个 Rust库(library)。但最终会确切输出/生成什么类型的库在未生成之前还不好清晰确定，因为库有多种表现形式。使用 `lib` 这个通用选项的目的是生成“编译器推荐”的类型的库。像种指定输出库类型的选项在 rustc 里始终可用，但是每次实际输出的库的类型可能会随着实际情况的不同而不同。其它的输出（库的）类型选项都指定了不同风格的库类型，而 `lib` 可以看作是那些类型中的某个类型的别名（具体实际的输出的类型是编译器决定的）。

* `--crate-type=dylib` 或 `#[crate_type = "dylib"]` - 将生成一个动态 Rust库。这与 `lib` 选项的输出类型不同，因为这个选项会强制生成动态库。生成的动态库可以用作其他库和/或可执行文件的依赖。这种输出类型将创建依赖于具体平台的库（Linux 上为 `*.so`，macOS 上为 `*.dylib`、Windows 上为 `*.dll`）。

* `--crate-type=staticlib` 或 `#[crate_type = "staticlib"]` - 将生成一个静态系统库。这个选项与其他选项的库输出的不同之处在于——当前编译器永远不会尝试去链接此 `staticlib` 输出[^译注1]。此选项的目的是创建一个包含所有本地 crate 的代码以及所有上游依赖的静态库。此输出类型将在 Linux、macOS 和 Windows(MinGW) 平台上创建 `*.a` 归档文件(archive)，或者在 Windows(MSVC) 平台上创建 `*.lib` 库文件。在这些情况下，例如将 Rust代码链接到现有的非 Rust应用程序中，推荐使用这种类型，因为它不会动态依赖于其他 Rust 代码。

* `--crate-type=cdylib` 或 `#[crate_type = "cdylib"]` - 将生成一个动态系统库。如果编译输出的动态库要被另一种语言加载使用，请使用这种编译选项。这种选项的输出将在 Linux 上创建 `*.so` 文件，在 macOS 上创建 `*.dylib` 文件，在 Windows 上创建 `*.dll` 文件。

* `--crate-type=rlib` 或 `#[crate_type = "rlib"]` - 将生成一个“Rust库”。它被用作一个中间构件，可以被认为是一个“静态 Rust库”。与 `staticlib` 类型的库文件不同，这些 `rlib` 类型的库文件以后会作为其他 Rust代码文件的上游依赖，未来对那些 Rust代码文件进行编译时，那时的编译器会链并解释此 `rlib`文件。这本质上意味着（那时的） `rustc` 将在（此） `rlib` 文件中查找元数据(metadata)，就像在动态库中查找元数据一样。跟 `staticlib` 输出类型类似，这种类型的输出常配合用于生成静态链接的可执行文件(statically linked executable)。

* `--crate-type=proc-macro` 或 `#[crate_type = "proc-macro"]` - 生成的输出类型没有被指定，但是如果通过 `-L` 提供了路径参数，编译器将把输出构件识别为宏，输出的宏可以被其他 Rust 程序加载使用。使用此 crate 类型编译的 crate 只能导出[过程宏][procedural macros]。编译器将自动设置 `proc_macro`[属性配置选项][configuration option]。编译 crate 的目标平台(target)总是和当前编译器所在平台一致。例如，如果在 `x86_64` CPU 的 Linux 平台上执行编译，那么目标将是 `x86_64-unknown-linux-gnu`，即使该 crate 是另一个不同编译目标的 crate 的依赖。

请注意，这些选项是可堆叠使用的，如果同时使用了多个选项，那么编译器将一次生成所有这些选项关联的的输出类型，而不必反复多次编译。但是，命令行和内部 `crate_type` 属性配置不能同时起效。如果只使用了不带属性值的 `crate_type` 属性配置，则将生成所有类型的输出，但如果同时指定了一个或多个 `--crate-type` 命令行参数，则只生成这些指定的输出。

对于所有这些不同类型的输出，如果 crate A 依赖于 crate B，那么整个系统中很可能有多种不同形式的 B，但是，编译器只会查找 `rlib` 类型的和动态库类型的。有了依赖库的这两个选项，编译器在某些时候还是必须在这两种类型之间做出选择。考虑到这一点，编译器在决定使用哪种依赖关系类型时将遵循以下规则：

1. 如果当前生成静态库，则需要所有上游依赖都以 `rlib` 类型可用。这个需求源于不能将动态库转换为静态类型的原因。

   注意，不可能将本地动态依赖链接到静态库，在这种情况下，将打印有关所有未链接的本地动态依赖的警告。

2. 如果当前生成 `rlib` 文件，则对上游依赖的可用类型没有任何限制，仅需要求所有这些文件都可以从其中读出元数据。

   原因是 `rlib` 文件不包含它们的任何上游依赖。但如果所有的 `rlib` 文件都包含一份 `libstd.rlib` 的副本，那编译效率和执行效率将大幅降低。

3. 如果当前生成可执行文件，并且没有指定 `-C prefer-dynamic` 参数，则首先尝试以 `rlib` 类型查找依赖。如果某些依赖在 rlib 类型文件中不可用，则尝试动态链接（见下文）。

4. 如果当前生成动态链接的动态库或可执行文件，则编译器将尝试协调从 rlib 或 dylib 类型的文件里获取可用依赖关系，以创建最终产品。

   编译器的主要目标是确保任何一个库不会在任何构件中出现多次。例如，如果动态库 B 和 C 都静态地去链接了库 A，那么当前 crate 就不能同时链接到 B 和 C，因为 A 有两个副本。编译器允许混合使用 rlib 和 dylib 类型，但这一限制必须被满足。

   编译器目前没有实现任何方法来提示库应该链接到哪种类型的库。当选择动态链接时，编译器将尝试最大化动态依赖，同时仍然允许通过 rlib 类型链接某些依赖。

   对于大多数情况，如果所有的可用库都是 dylib 类型的动态库，则推荐选择动态链接。对于其他情况，如果编译器无法确定一个库到底应该去链接它的哪种类型的版本，则会发布警告。

通常，`--crate-type=bin` 或 `--crate-type=lib` 应该足以满足所有的编译需求，只有在需要对 crate 的输出类型进行更细粒度的控制时，才需要使用其他选项。

## 静态C运行时和动态C运行时

一般来说，标准库会同时尽力支持编译目标的静态链接型C运行时和动态链接型C运行时。例如，目标 `x86_64-pc-windows-msvc` 和 `x86_64-unknown-linux-musl` 通常都带有C运行时，用户可以按自己的偏好去选择静态链接或动态链接到此运行时。编译器中所有的编译目标都有一个链接到 C运行时的默认模式。默认情况下，常见的编译目标都默认是选择动态链接的，但也存在默认情况下是静态链接的情况，例如：

* `arm-unknown-linux-musleabi`
* `arm-unknown-linux-musleabihf`
* `armv7-unknown-linux-musleabihf`
* `i686-unknown-linux-musl`
* `x86_64-unknown-linux-musl`

C运行时的链接类型被配置为通过 `crt-static` 目标特性值来开启。这些目标特性通常是从命令行通过命令行参数传递给编译器来设置的。例如，要启用静态运行时，应该执行：

```sh
rustc -C target-feature=+crt-static foo.rs
```

如果想动态链接到C运行时，应该执行：

```sh
rustc -C target-feature=-crt-static foo.rs
```

不支持在到 C运行时的链接类型之间切换的编译目标将忽略这个标志。建议检查生成的二进制文件，以确保在编译成功之后，如预期的那样链接了 C运行时。

crate 本身也可以检测如何链接 C运行时。例如，MSVC平台上的代码需要根据链接运行时的方式进行差异性的编译（例如选择使用 `/MT` 或 `/MD`）。目前可通过 [`cfg`属性的 `target_feature`选项][`cfg` attribute `target_feature` option]导出检测结果：

```rust
#[cfg(target_feature = "crt-static")]
fn foo() {
    println!("C运行时应该被静态链接");
}

#[cfg(not(target_feature = "crt-static"))]
fn foo() {
    println!("C运行时应该被动态链接");
}
```

还请注意，Cargo构建脚本可以通过[环境变量][cargo]来检测此特性。在构建脚本中，您可以通过如下代码检测链接类型：

```rust
use std::env;

fn main() {
    let linkage = env::var("CARGO_CFG_TARGET_FEATURE").unwrap_or(String::new());

    if linkage.contains("crt-static") {
        println!("C运行时应该被静态链接");
    } else {
        println!("C运行时应该被动态链接");
    }
}
```

[cargo]: https://doc.rust-lang.org/cargo/reference/environment-variables.html#environment-variables-cargo-sets-for-build-scripts

要在本地使用此特性，通常需要使用 `RUSTFLAGS` 环境变量通过 Cargo 来为编译器指定参数。例如，要在 MSVC 平台上编译静态链接的二进制文件，需要执行：

```sh
RUSTFLAGS='-C target-feature=+crt-static' cargo build --target x86_64-pc-windows-msvc
```

[^译注1]: 译者理解此编译选项是将本地 Rust crate 和其上游依赖资源编译输出成一个库，供其他应用程序使用的。所以对当前编译而言，不能依赖还不存在的资源；又因为其他 Rust crate 的编译无法将静态库作为编译时依赖的上游资源，所以编译其他 Rust crate 的编译器也不会来链接此 `staticlib` 输出。

[`cfg` attribute `target_feature` option]: conditional-compilation.md#target_feature
[configuration option]: conditional-compilation.md
[procedural macros]: procedural-macros.md

<!-- 2020-11-12-->
<!-- checked -->
