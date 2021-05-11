# 注释

> **<sup>Lexer</sup>**\
> 行注释：\
> &nbsp;&nbsp; &nbsp;&nbsp; `//` (~\[`/` `!`] | `//`) ~`\n`<sup>\*</sup>\
> &nbsp;&nbsp; | `//`
>
> 块注释：\
> &nbsp;&nbsp; &nbsp;&nbsp; `/*` (~\[`*` `!`] | `**` | _BlockCommentOrDoc_)
>      (_BlockCommentOrDoc_ | ~`*/`)<sup>\*</sup> `*/`\
> &nbsp;&nbsp; | `/**/`\
> &nbsp;&nbsp; | `/***/`
>
> 内部行文档注释：
> > *译者注*：此风格文档注释包含注释的项，而不是为注释之后的项增加文档。这通常用于 crate 根文件（通常是 src/lib.rs）或模块的根文件为 crate 或模块整体提供文档。
>
> &nbsp;&nbsp; `//!` ~\[`\n` _IsolatedCR_]<sup>\*</sup>
>
> 内部块文档注释：
> > *译者注*：此风格文档注释包含注释的项，而不是为注释之后的项增加文档。这通常用于 crate 根文件（通常是 src/lib.rs）或模块的根文件为 crate 或模块整体提供文档。
>
> &nbsp;&nbsp; `/*!` ( _BlockCommentOrDoc_ | ~\[`*/` _IsolatedCR_] )<sup>\*</sup> `*/`
>
> 外部行文档注释：
> > *译者注*：此风格文档注释位于需要文档的项之前，为注释之后的项增加文档。
>
> &nbsp;&nbsp; `///` (~`/` ~\[`\n` _IsolatedCR_]<sup>\*</sup>)<sup>?</sup>
>
> 外部块文档注释：
> > *译者注*：此风格文档注释位于需要文档的项之前，为注释之后的项增加文档。
>
> &nbsp;&nbsp; `/**` (~`*` | _BlockCommentOrDoc_ )
>              (_BlockCommentOrDoc_ | ~\[`*/` _IsolatedCR_])<sup>\*</sup> `*/`
>
> _BlockCommentOrDoc_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; BLOCK_COMMENT\
> &nbsp;&nbsp; | OUTER_BLOCK_DOC\
> &nbsp;&nbsp; | INNER_BLOCK_DOC
>
> _IsolatedCR_ :\
> &nbsp;&nbsp; _A `\r` not followed by a `\n`_

## 非文档注释

Rust 代码中的注释大体上遵循 C++ 风格的行（`//`）和块（`/* ... */`）注释形式，嵌套块注释也被支持。

非文档注释被当作空白的形式解析。

## 文档注释

以 *三个斜线（`///`）* 开始的行文档注释，以及以 *一个斜线和两个星号（`/** ... */`）* 开始的块文档注释，均为内部文档注释。它们被作为 [`doc` 属性]的一个特殊语法解析。也就是说，它们等同于在注释体周围写上 `#[doc="..."]`。例如：`/// Foo` 等同于
`#[doc="Foo"]`，`/** Bar */` 等同于 `#[doc="Bar"]`。

以 `//!` 开始的行注释，以及以 `/*! ... */` 开始的块注释，是应用于注释体的父对象的文档注释，而非注释体之后的项。也就是说，它们等同于在注释体周围写上 `#![doc="..."]`。`//!` 注释通常用于说明源文件中的模块。

孤立的 CRs（`\r`）——譬如其后无 LF（`\n`），在文档注释中是不被允许的。

## 示例

```rust
//! A doc comment that applies to the implicit anonymous module of this crate

pub mod outer_module {

    //!  - Inner line doc
    //!! - Still an inner line doc (but with a bang at the beginning)

    /*!  - Inner block doc */
    /*!! - Still an inner block doc (but with a bang at the beginning) */

    //   - Only a comment
    ///  - Outer line doc (exactly 3 slashes)
    //// - Only a comment

    /*   - Only a comment */
    /**  - Outer block doc (exactly) 2 asterisks */
    /*** - Only a comment */

    pub mod inner_module {}

    pub mod nested_comments {
        /* In Rust /* we can /* nest comments */ */ */

        // All three types of block comments can contain or be nested inside
        // any other type:

        /*   /* */  /** */  /*! */  */
        /*!  /* */  /** */  /*! */  */
        /**  /* */  /** */  /*! */  */
        pub mod dummy_item {}
    }

    pub mod degenerate_cases {
        // empty inner line doc
        //!

        // empty inner block doc
        /*!*/

        // empty line comment
        //

        // empty outer line doc
        ///

        // empty block comment
        /**/

        pub mod dummy_item {}

        // empty 2-asterisk block isn't a doc block, it is a block comment
        /***/

    }

    /* The next one isn't allowed because outer doc comments
       require an item that will receive the doc */

    /// Where is my item?
#   mod boo {}
}
```

[`doc` 属性]: attributes.md
