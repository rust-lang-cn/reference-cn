# 标识符

>[identifiers.md](https://github.com/rust-lang/reference/blob/master/src/identifiers.md)\
>commit: 28d628166940ffa982a1da552e0acfcc88ff8889 \
>本章译文最后维护日期：2021-04-23

> **<sup>词法分析:<sup>**\
> IDENTIFIER_OR_KEYWORD :\
> &nbsp;&nbsp; &nbsp;&nbsp; XID_start XID_continue<sup>\*</sup>\
> &nbsp;&nbsp; | `_` XID_continue<sup>+</sup>
>
> RAW_IDENTIFIER : `r#` IDENTIFIER_OR_KEYWORD <sub>*排除 `crate`, `self`, `super`, `Self`*</sub>
>
> NON_KEYWORD_IDENTIFIER : IDENTIFIER_OR_KEYWORD <sub>*排除[严格关键字][strict]和[保留关键字][reserved] *</sub>
>
> IDENTIFIER :\
> NON_KEYWORD_IDENTIFIER | RAW_IDENTIFIER

标识符是如下形式的任何非空 Unicode 字符串：

要么是:

* 首字符拥有 [`XID_start`] 字符属性。
* 其余字符拥有 [`XID_continue`] 字符属性。

要么是：

* 首字符是 `_`。
* 整个字符串由多个字符组成。单个 `_` 不是有效标识符。
* 其余字符拥有 [`XID_continue`] 字符属性。

> **注意**：[`XID_start`] 和 [`XID_continue`] 作为字符属性涵盖了用于构成常见的 C 和 Java语言族标识符的字符范围。
 
除了有形式前缀 `r#` 修饰外，原生标识符(raw identifier)与普通标识符类似。（注意形式前缀 `r#` 不包括在实际标识符中。）与普通标识符不同，原生标识符可以是除上面列出的 `RAW_IDENTIFIER` 之外的任何严格关键字或保留关键字。

[strict]: keywords.md#strict-keywords
[reserved]: keywords.md#reserved-keywords
[`XID_start`]:  http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%3AXID_Start%3A%5D&abb=on&g=&i=
[`XID_continue`]: http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%3AXID_Continue%3A%5D&abb=on&g=&i=
