# Comments
# 注释

>[comments.md](https://github.com/rust-lang/reference/blob/master/src/comments.md)\
>commit: bad7ef114cec9c08dda400ea2da988d07696b066 \
>本章译文最后维护日期：2024-10-13

r[comments.syntax]

> **<sup>词法分析</sup>**\
> LINE_COMMENT :(译者注：行注释)\
> &nbsp;&nbsp; &nbsp;&nbsp; `/*` (~\[`*` `!` `\n`] | `**` | _BlockCommentOrDoc_)
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
> &nbsp;&nbsp; \\r

## Non-doc comments
## 非文档型注释

r[comments.normal]

Rust 代码中的注释一般遵循 C++ 风格的行（`//`）和块（`/* ... */`）注释形式，也支持嵌套的块注释。

r[comments.normal.tokenization]
非文档型注释(Non-doc comments)被解释为某种形式的空白符。

## Doc comments
## 文档型注释

r[comments.doc]

r[comments.doc.syntax]
以*三个*斜线（`///`）开始的行文档型注释，以及块文档型注释（`/** ... */`），均为外部文档型注释。它们被当做 [`doc`属性][`doc` attributes]的特殊句法解析。

r[comments.doc.attributes]
也就是说，它们等同于把注释内容写入 `#[doc="..."]` 里。例如：`/// Foo` 等同于 `#[doc="Foo"]`，`/** Bar */` 等同于 `#[doc="Bar"]`。因此，它们必须出现在那些接受外部属性的代码之前。

r[comments.doc.inner-syntax]
以 `//!` 开始的行文档型注释，以及 `/*! ... */` 形式的块文档型注释属于注释体所在对象的文档型注释，而非注释体之后的程序项的。

r[comments.doc.inner-attributes]
也就是说，它们等同于把注释内容写入 `#![doc="..."]` 里。`//!` 注释通常用于标注模块位于的文件。

r[comments.doc.bare-crs]
文档型注释中不允许出现字符`U+000D`(CR)。

> **注**：正如 `rustdoc` 所预期的那样，文档注释通常包含 Markdown。然而，注释语法不考虑任何嵌套的Markdown语法。
> ``/** `glob = "*/*.rs";` */`` 在第一个`*/`处终止注释，因此剩余的代码将导致语法错误。与行文档注释相比，这稍微限制了块文档注释的内容。

> **注意**:  `U+000D`(CR)后直跟`U+000A`(LF) 的字符序列会预先被转换为 `U+000A` (LF)。

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

[`doc` attributes]: https://doc.rust-lang.org/rustdoc/the-doc-attribute.html

<!-- 2020-11-12-->
<!-- checked -->
