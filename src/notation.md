# 表义符/符号

>[notation.md](https://github.com/rust-lang/reference/blob/master/src/notation.md)\
>commit: dd1b9c331eb14ea7047ed6f2b12aaadab51b41d6 \
>本章译文最后维护日期：2020-11-7

## 文法/语法

下表中的各种表义符会在本书中标有 *词法* 和 *句法* 的文法片段中用到：

| 表义符             | 示例                           | 释义                                 
|-------------------|-------------------------------|---------------------------------|
| CAPITAL           | KW_IF, INTEGER_LITERAL        | 由词法分析生成的单一 token          |
| _ItalicCamelCase_ | _LetStatement_, _Item_        | 句法产生式(syntactical production)|
| `string`          | `x`, `while`, `*`             | 确切的字面字符(串)                |
| \\x               | \\n, \\r, \\t, \\0            | 转义字符                         |
| x<sup>?</sup>     | `pub`<sup>?</sup>             | 可选项                           |
| x<sup>\*</sup>    | _OuterAttribute_<sup>\*</sup> | x 重复零次或多次                  |
| x<sup>+</sup>     | _MacroMatch_<sup>+</sup>      | x 重复一次或多次                  |
| x<sup>a..b</sup>  | HEX_DIGIT<sup>1..6</sup>      | x 重复 a 到 b 次                 |
| \|                | `u8` \| `u16`, Block \| Item  | 或                               |
| \[ ]              | \[`b` `B`]                    | 列举的任意字符                    |
| \[ - ]            | \[`a`-`z`]                    | a 到 z 范围内的任意字符(包括 a 和 z)|
| ~\[ ]             | ~\[`b` `B`]                   | 列举范围外的任意字符(序列)          |
| ~`string`         | ~`\n`, ~`*/`                  | 此字符序列外的任意字符(序列)        |
| ( )               | (`,` _Parameter_)<sup>?</sup> | 程序项分组                        |

## 字符串表产生式

文法中的一些规则 &mdash; 比如[一元运算符][unary operators]，[二元运算符][binary operators]和[关键字][keywords] &mdash; 会以简化形式：作为可打印字符串的列表形式（在本书的相关章节的头部的各种产生式定义/句法规则里）给出。这些规则构成了关于 [token][tokens]规则的规则子集，并且它们被假定为源码编译时的词法分析阶段的结果被再次输入给解析器，然后由一个<abbr title="确定性有限自动机(Deterministic Finite Automaton)">DFA</abbr>驱动，对此字符串表里的所有条目(string table entries)进行析取(disjunction)操作（来进行句法分析）。

本书还约定，当文法表中出现如 `monospace` 这样的字符串时，它代表对这些产生式中的单一 token 成员的隐式引用。查阅 [tokens] 以获取更多信息。

[binary operators]: expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[keywords]: keywords.md
[tokens]: tokens.md
[unary operators]: expressions/operator-expr.md#borrow-operators

<!-- 2020-11-12-->
<!-- checked -->
