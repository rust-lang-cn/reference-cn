# 语句和表达式

>[statements-and-expressions.md](https://github.com/rust-lang/reference/blob/master/src/statements-and-expressions.md)\
>commit: 4a2bdf896cd2df370a91d14cb8ba04e326cd21db \
>本章译文最后维护日期：2020-11-11

Rust *基本上*是一种表达式语言。这意味着大多数形式的求值或生成表达效果的计算的都是由*表达式*的统一句法类别来指导的。每一种表达式通常都可以*内嵌*到另一种表达式中，表达式的求值规则包括指定表达式产生的值和指定其各个子表达式的求值顺序。

对比之下，Rust 中的语句则*主要*用于包含表达式求值，以及显式地安排表达式的求值顺序。

<!-- 2020-11-12-->
<!-- checked -->

