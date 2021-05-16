# 非安全函数

>[unsafe-functions.md](https://github.com/rust-lang/reference/blob/master/src/unsafe-functions.md)\
>commit:  4a2bdf896cd2df370a91d14cb8ba04e326cd21db \
>本章译文最后维护日期：2020-11-16

非安全函数是指在任何可能的上下文和/或任何可能的输入中可能出现不安全情况的函数。这样的函数必须以关键字 `unsafe` 前缀修饰，并且只能在非安全(`unsafe`)块或其他非安全(`unsafe`)函数中调用此类函数。

<!-- 2020-11-12-->
<!-- checked -->

