# Rust 参考手册

![Build Status](https://github.com/rust-lang-cn/reference-cn/workflows/CI/badge.svg)
[![LICENSE-MIT](https://img.shields.io/badge/license-MIT-green)](https://raw.githubusercontent.com/rust-lang-cn/reference-cn/master/LICENSE-MIT)
[![LICENSE-APACHE](https://img.shields.io/badge/license-Apache%202-blue)](https://raw.githubusercontent.com/rust-lang-cn/reference-cn/master/LICENSE-APACHE)
![GitHub last commit](https://img.shields.io/github/last-commit/rust-lang-cn/reference-cn?color=gold)
![rustwiki.org](https://img.shields.io/website?up_message=rustwiki.org&url=https%3A%2F%2Frustwiki.org)

> Chinese translation of [The Rust Language Reference][github-en]
> 
> 更新注：经网友提醒，《Rust 参考手册》中文版已经有由 [minstrel] 翻译[完整的版本][full-version]，故本仓库不再单独翻译，而直接采用 minstrel 翻译的内容，若对翻译内容改正和完善，可到 [minstrel 维护的仓库上提出和 PR][full-version]。在此，*rust-lang-cn* 对译者表示衷心的感谢，您的翻译对我们使用中文资料学习和了解 Rust 起到极大作用。
>
> *rust-lang-cn* 只会对格式和部分内容进行微调，其余内容我们建议大家都反馈到原仓库。中文翻译版（位于 *src* 目录）的版权我们也遵循原仓库的[木兰宽松许可证][mulan]，本仓库下的英文版内容以及其他文件均保持 Rust 官方的原有授权协议（即 MIT 协议和 Apache 2.0 协议）。
>
> 初版注：本仓库翻译内容包括 Rust 中文翻译项目组本身的翻译以及采用网上已有的开源的翻译版本（如：[zzy/rust-reference-cn][zzy] 译本），我们尽可能避免不必要的重复劳动，我们对原译者感激不尽！

[github-en]: https://github.com/rust-lang/reference
[zzy]: https://github.com/zzy/rust-reference-zh-cn
[minstrel]: https://gitee.com/minstrel1977
[full-version]: https://gitee.com/minstrel1977/rust-reference
[mulan]: https://license.coscl.org.cn/MulanPSL2/

This document is the primary reference for the Rust programming language.

This document is not normative. It may include details that are specific
to `rustc` itself, and should not be taken as a specification for the
Rust language. We intend to produce such a document someday, but this is
what we have for now.


本文档是 Rust 编程语言的主要参考。

本文档不是规范性的。它可能包含 `rustc` 自身特定的详细信息，不应视为 Rust 语言的正式规范。我们打算有一天产生这样的正式规范文档，但这是我们目前所拥有的过渡版。

## 依赖

- rustc (the Rust compiler).
- mdbook (use `cargo install mdbook` to install it).
- rust nightly (you would be required to set your Rust version to the nightly version to make sure all tests pass)

- rustc（Rust编译器）。
- mdbook（用于 `cargo install mdbook` 来安装）。
- rust nightly 版（需要将 Rust 版本设置为 nightly 版本，以确保所有测试均通过）。

## 构建步骤

要构建项目，请按照以下步骤操作：

通过从 [GitHub 页面](https://github.com/rust-lang-cn/reference-cn) （或 [英文版参考](https://github.com/rust-lang/reference)）下载 zip 压缩包或运行以下命令 clone 本项目：

```
git clone https://github.com/rust-lang-cn/reference-cn
```

进入到下载或克隆的仓库目录：

```sh
cd reference-cn
```

要运行测试，需要将 Rust 版本设置为 nightly 版本。可以通过执行以下命令来完成此操作：

```shell
rustup override set nightly
```

这将仅为这个当前的项目设置 nightly 版本。

如果希望为所有项目设置为 Rust nightly 版，可以运行以下命令：

```shell
rustup default nightly
```

现在可以运行以下命令来测试代码片段以捕获编译错误：

```shell
mdbook test
```

要生成本书的本地版本，请运行：

```sh
mdbook build
```

生成的 HTML 书籍文件将在 `book` 文件夹中。
