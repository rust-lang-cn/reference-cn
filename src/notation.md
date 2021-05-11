# 标记法

## 语法

下表符号被用于 *Lexer* 和 *Syntax* 的语法片段：

| 符号              | 示例                           | 释义                          |
|-------------------|-------------------------------|-------------------------------|
| CAPITAL           | KW_IF, INTEGER_LITERAL        | 词法相关记号（符号）<sup><strong>待修正</strong></sup> |
| _ItalicCamelCase_ | _LetStatement_, _Item_        | 语法相关定义 <sup><strong>待修正</strong></sup> |
| `string`          | `x`, `while`, `*`             | 字符（串） |
| \\x               | \\n, \\r, \\t, \\0            | 转义字符 |
| x<sup>?</sup>     | `pub`<sup>?</sup>             | 可选项 |
| x<sup>\*</sup>    | _OuterAttribute_<sup>\*</sup> | x 零次或多次重复 |
| x<sup>+</sup>     |  _MacroMatch_<sup>+</sup>     | x 一次或多次重复 |
| x<sup>a..b</sup>  | HEX_DIGIT<sup>1..6</sup>      | x 在 a -> b 范围内重复 |
| \|                | `u8` \| `u16`, Block \| Item  | 或 |
| \[ ]               | \[`b` `B`]                     | 列举的任意字符 |
| \[ - ]             | \[`a`-`z`]                     | a -> z 范围内的任意字符 |
| ~\[ ]              | ~\[`b` `B`]                    | 除列举范围外的任意字符 |
| ~`string`         | ~`\n`, ~`*/`                  | 除此字符序列外的任意字符 |
| ( )               | (`,` _Parameter_)<sup>?</sup> | 分组项 <sup><strong>待修正</strong></sup> |

## 字符串表组合

语法中的一些规则&mdash;尤其是[单目运算符][unary operators]，[双目运算符][binary
operators]和[关键字][keywords]&mdash;以一种简单的形式表现：可打印的字符串表。这些规则是[记号][tokens]相关规则的子集，并且被假定为词法分析阶段的结果。该阶段由一个 DFA（Deterministic Finite
Automaton，确定有限状态自动机）驱动，通过分离所有这些字符串表入口执行。

当语法中出现如 `monospace` 这样的字符串时（译者注：半角双引号 `"` 中的字符串），它是对字符串表组合中的单个成员的隐式引用。查阅[记号][tokens]以获取更多信息。

[binary operators]: expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[keywords]: keywords.md
[tokens]: tokens.md
[unary operators]: expressions/operator-expr.md#borrow-operators
