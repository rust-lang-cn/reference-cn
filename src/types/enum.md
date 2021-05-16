# 枚举类型

>[enum.md](https://github.com/rust-lang/reference/blob/master/src/types/enum.md)\
>commit: d8cbe4eedb77bae3db9eff87b1238e7e23f6ae92 \
>本章译文最后维护日期：2021-2-21

*枚举类型*是一种标称型(nominal)的、异构的、不相交的类型联合起来组成的类型，它直接用[枚举(`enum`)程序项][`enum` item]的名称来表示。[^enumtype]

[枚举(`enum`)程序项][`enum` item]同时声明了类型和它的各种*变体(variants)*，其中每个变体都独立命名，可使用定义结构体、元组结构体或单元结构体(unit-like struct)的句法来定义它们。

枚举(`enum`)的实例可以通过[结构体体表达式][struct expression]来构造。

任何枚举值消耗的内存和其同类型的其他变体都是相同的，具体都为其枚举(`enum`)类型的最大变体所需的内存再加上存储其判别值(discriminant)所需的内存。

枚举类型不能在*结构上*表示为类型，必须通过对[枚举程序项][`enum` item]的具名引用(named reference)来表示。[^译注1]

[^enumtype]: `enum`类型类似于 ML 中的数据(`data`)构造函数声明，或 Limbo 中的 *pick ADT*。

[^译注1]: 译者理解这句话的意思是：枚举不同于普通结构化的类型，所有的枚举类型都是对枚举程序项的引用；这里引用分两种，一种是类C枚举，就是对程序项的直接具名引用；另一种是带字段的枚举变体，这种其实是类似于 `Box`、`Rc` 这样的具名引用，它通过封装其他类型来指导数据的存储和限定其上可用的操作。

[`enum` item]: ../items/enumerations.md
[struct expression]: ../expressions/struct-expr.md
