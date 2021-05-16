# 影响来源

>[influences.md](https://github.com/rust-lang/reference/blob/master/src/influences.md)\
>commit:  dc3808468e37ff4c1f663d26c491a3549a42c201 \
>本章译文最后维护日期：2020-11-3

Rust 并不是一种极端原创的语言，它的设计元素来源广泛。下面列出了其中一些（包括已经删除的）：

* SML，OCaml：代数数据类型、模式匹配、类型推断、分号语句分隔
* C++：引用，RAII，智能指针，移动语义，单态化(monomorphization)，内存模型
* ML Kit, Cyclone：基于区域的内存管理(region based memory management)
* Haskell (GHC)：类型属性(typeclasses), 类型簇(type families)
* Newsqueak, Alef, Limbo：通道，并发
* Erlang：消息传递，线程失败，<strike>链接线程失败</strike>，<strike>轻量级并发</strike>
* Swift：可选绑定(optional bindings)
* Scheme：卫生宏(hygienic macros)
* C#：属性
* Ruby：闭包句法，<strike>块句法</strike>
* NIL, Hermes：<strike>typestate</strike>
* [Unicode Annex #31](http://www.unicode.org/reports/tr31/)：标识符和模式句法

<!-- 2020-11-12-->
<!-- checked -->
