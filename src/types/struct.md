# 结构体类型

>[struct.md](https://github.com/rust-lang/reference/blob/master/src/types/struct.md)\
>commit: 8a98437835db5f3cf57044aedcd1e00bd0d889f9 \
>本章译文最后维护日期：2021-1-17

结构体(`struct`)*类型*是其由他类型异构产生的类型，这些其他类型被称为结构体类型的字段。[^structtype]

结构体(`struct`)的实例可以用[结构体表达式][struct expression]来构造。

默认情况下，结构体(`struct`)的内存布局是未定义的（默认允许进行一些编译器优化，比如字段重排），但也可以使用[`repr`属性][`repr` attribute]来使其布局在定义时就固定下来。在这两种情况下，字段在相应的结构体*表达式*中都可以以任何顺序给出；（但在相同的编译目标中，在）同一布局规则下生成的结构体(`struct`)值将始终具有相同的内存布局。

结构体(`struct`)的字段可以由[可见性修饰符(visibility modifiers)][visibility modifiers]限定，以允许从模块之外来访问结构体中的数据。

*元组结构体(uple struct)*类型与结构体类型类似，只是字段是匿名的。

*单元结构体(unit-like struct)*类型类似于结构体类型，只是它没有字段。由初始[结构体表达式][struct expression]构造的值是驻留在此类类型中惟一的值。

[^structtype]: `struct`类型类似于 C 中的 `struct` 类型、ML家族的 *record* 类型或 Lisp 家族的 *struct* 类型。

[`repr` attribute]: ../type-layout.md#representations
[struct expression]: ../expressions/struct-expr.md
[visibility modifiers]: ../visibility-and-privacy.md

<!-- 2020-11-12-->
<!-- checked -->
