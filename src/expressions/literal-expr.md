# 字面量表达式

>[literal-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/literal-expr.md)\
>commit: eb5290329316e96c48c032075f7dbfa56990702b \
>本章译文最后维护日期：2021-02-21

> **<sup>句法</sup>**\
> _LiteralExpression_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [CHAR_LITERAL]\
> &nbsp;&nbsp; | [STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_STRING_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | [BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [INTEGER_LITERAL]\
> &nbsp;&nbsp; | [FLOAT_LITERAL]\
> &nbsp;&nbsp; | [BOOLEAN_LITERAL]

*字面量表达式*由[字面量][literal]里讲过的任一形式组成。
它直接描述一个数字、字符、字符串或布尔值。

```rust
"hello";   // 字符串类型
'5';       // 字符类型
5;         // 整型
```

[literal]: ../tokens.md#literals
<!-- 上面这几个链接从原文来替换时需小心 -->
[CHAR_LITERAL]: ../tokens.md#character-literals
[STRING_LITERAL]: ../tokens.md#string-literals
[RAW_STRING_LITERAL]: ../tokens.md#raw-string-literals
[BYTE_LITERAL]: ../tokens.md#byte-literals
[BYTE_STRING_LITERAL]: ../tokens.md#byte-string-literals
[RAW_BYTE_STRING_LITERAL]: ../tokens.md#raw-byte-string-literals
[INTEGER_LITERAL]: ../tokens.md#integer-literals
[FLOAT_LITERAL]: ../tokens.md#floating-point-literals
[BOOLEAN_LITERAL]: ../tokens.md#boolean-literals

