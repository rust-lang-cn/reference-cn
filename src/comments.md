# 注释

>[comments.md](https://github.com/rust-lang/reference/blob/master/src/comments.md)\
>commit: 993393d362cae51584d580f86c4f38d43ae76efc \
>本章译文最后维护日期：2020-10-17

> **<sup>词法分析</sup>**\
> LINE_COMMENT :(译者注：行注释)\
> &nbsp;&nbsp; &nbsp;&nbsp; `/*` (~\[`*` `!`] | `**` | _BlockCommentOrDoc_)
> &nbsp;&nbsp; | `//`
>
> BLOCK_COMMENT :(译者注：块注释)\
> &nbsp;&nbsp; &nbsp;&nbsp; `/*` (~\[`*` `!`] | `**` | _BlockCommentOrDoc_)
>      (_BlockCommentOrDoc_ | ~`*/`)<sup>\*</sup> `*/`\
> &nbsp;&nbsp; | `/**/`\
> &nbsp;&nbsp; | `/***/` 
>
> INNER_LINE_DOC :(译者注：内部行文档型注释)\
> &nbsp;&nbsp; `//!` ~\[`\n` _IsolatedCR_]<sup>\*</sup>
>
> INNER_BLOCK_DOC :(译者注：内部块文档型注释)\
> &nbsp;&nbsp; `/*!` ( _BlockCommentOrDoc_ | ~\[`*/` _IsolatedCR_] )<sup>\*</sup> `*/`
>
> OUTER_LINE_DOC :(译者注：外部行文档型注释)\
> &nbsp;&nbsp; `///` (~`/` ~\[`\n` _IsolatedCR_]<sup>\*</sup>)<sup>?</sup>
>
> OUTER_BLOCK_DOC :(译者注：外部块文档型注释)\
> &nbsp;&nbsp; `/**` (~`*` | _BlockCommentOrDoc_ )
>              (_BlockCommentOrDoc_ | ~\[`*/` _IsolatedCR_])<sup>\*</sup> `*/`
>
> _BlockCommentOrDoc_ :(译者注：块注释或文档型注释)\
> &nbsp;&nbsp; &nbsp;&nbsp; BLOCK_COMMENT\
> &nbsp;&nbsp; | OUTER_BLOCK_DOC\
> &nbsp;&nbsp; | INNER_BLOCK_DOC
>
> _IsolatedCR_ :\
> &nbsp;&nbsp; _后面没有跟 `\n` 的 `\r`_

## 非文档型注释

Rust 代码中的注释一般遵循 C++ 风格的行（`//`）和块（`/* ... */`）注释形式，也支持嵌套的块注释。

非文档型注释(Non-doc comments)被解释为某种形式的空白符。

## 文档型注释

以*三个*斜线（`///`）开始的行文档型注释，以及块文档型注释（`/** ... */`），均为内部文档型注释。它们被当做 [`doc`属性][`doc` attributes]的特殊句法解析。也就是说，它们等同于把注释内容写入 `#[doc="..."]` 里。例如：`/// Foo` 等同于 `#[doc="Foo"]`，`/** Bar */` 等同于 `#[doc="Bar"]`。

以 `//!` 开始的行文档型注释，以及 `/*! ... */` 形式的块文档型注释属于注释体所在对象的文档型注释，而非注释体之后的程序项的。也就是说，它们等同于把注释内容写入 `#![doc="..."]` 里。`//!` 注释通常用于标注模块位于的文件。

孤立的 CRs（`\r`），如果其后没有紧跟有 LF（`\n`），则不能出现在文档型注释中。

## 示例

```rust
//! 应用于此 crate 的隐式匿名模块的文档型注释

pub mod outer_module {

    //!  - 内部行文档型注释
    //!! - 仍是内部行文档型注释 (但是这样开头会更具强调性)

    /*!  - 内部块文档型注释 */
    /*!! - 仍是内部块文档型注释 (但是这样开头会更具强调性) */

    //   - 普通注释
    ///  - 外部行文档型注释 (以 3 个 `///` 开始)
    //// - 普通注释

    /*   - 普通注释 */
    /**  - 外部块文档型注释 (exactly) 2 asterisks */
    /*** - 普通注释 */

    pub mod inner_module {}

    pub mod nested_comments {
        /* 在 Rust 里 /* 我们可以 /* 嵌套注释 */ */ */

        // 所有这三种类型的块注释都可以包含或嵌套在任何其他类型的
        // 注释中：

        /*   /* */  /** */  /*! */  */
        /*!  /* */  /** */  /*! */  */
        /**  /* */  /** */  /*! */  */
        pub mod dummy_item {}
    }

    pub mod degenerate_cases {
        // 空内部行文档型注释
        //!

        // 空内部块文档型注释
        /*!*/

        // 空行注释
        //

        // 空外部行文档型注释
        ///

        // 空块注释
        /**/

        pub mod dummy_item {}

        // 空的两个星号的块注释不是一个文档块，它是一个块注释。
        /***/

    }

    /* 下面这个是不允许的，因为外部文档型注释需要一个
       接收该文档的程序项 */

    /// 我的程序项呢?
#   mod boo {}
}
```

[`doc` 属性]: https://doc.rust-lang.org/rustdoc/the-doc-attribute.html

<!-- 2020-11-12-->
<!-- checked -->
