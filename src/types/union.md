# 联合体类型

>[union.md](https://github.com/rust-lang/reference/blob/master/src/types/union.md)\
>commit: cd996a4e6e9b98929c3718fd76988bfee51d757c \
>本章译文最后维护日期：2020-12-24

*联合体类型*是一种标称型(nominal)的、异构的、类似C语言里的 union 的类型，具体的类型名称由[联合体(`union`)程序项][item]的名称表示。

Since transmutes can cause unexpected or undefined behaviour, `unsafe` is required to read from a union field, or to write to a field that doesn't implement [`Copy`] or has a [`ManuallyDrop`] type. See the [item] documentation for further details.
联合体没有“活跃字段(active field)”的概念。相反，每次对联合体的访问都将联合体的部分存储内容转换为被访问字段的类型。由于转换可能会导致意外或未定义行为，所以读取联合体字段，或写入未实现 [`Copy`] 或 [`ManuallyDrop`] 的联合体字段的操作都需要放在 `unsafe`块内进行。有关详细信息，请参阅相应的[程序项][item]文档。

默认情况下，联合体(`union`)的内存布局是未定义的，但是可以使用 `#[repr(...)]`属性来固定为某一类型布局。

[`Copy`]: ../special-types-and-traits.md#copy
[`ManuallyDrop<_>`]: https://doc.rust-lang.org/std/mem/struct.ManuallyDrop.html
[item]: ../items/unions.md

<!-- 2020-12-24-->
<!-- checked -->
