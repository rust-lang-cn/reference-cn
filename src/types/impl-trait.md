# 实现trait

>[impl-object.md](https://github.com/rust-lang/reference/blob/master/src/types/impl-object.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本章译文最后维护日期：2020-11-14

> **<sup>句法</sup>**\
> _ImplTraitType_ : `impl` [_TypeParamBounds_]
>
> _ImplTraitTypeOneBound_ : `impl` [_TraitBound_]

## 匿名类型参数

> 注意：此部分只是一个占位符，预备将来这里会有更全面的参考资料。

> 注意：匿名类型参数通常被称为“参数位置上的实现trait(impl Trait in argument position)”。

函数可以将其参数声明为匿名类型的参数，这种情况下函数内部只能使用匿名类型参数的 trait约束所提供的方法。之后的调用者(callee)在调用此函数时必须提供具体的类型，并且要求提供的具体类型必须拥有此匿名类型参数声明的约束。

它们被写成 `impl` 后跟一组 trait约束。

## 抽象返回类型

> 注意：此部分只是一个占位符，预备将来这里会有更全面的参考资料。

> 注意：抽象返回类型(abstract return types)通常被称为“函数返回位置上的实现trait(impl Trait in return position)”。

除了关联trait函数(associated trait function)外，其他函数都可以返回抽象返回类型。这些类型代表了某种具体类型，它要求在此返回类型的使用点(use-site)上只能使用由该类型的 trait约束声明的 trait方法。

它们被写成 `impl` 后跟一组 trait约束。

[_TraitBound_]: ../trait-bounds.md
[_TypeParamBounds_]: ../trait-bounds.md

<!-- 2020-11-12-->
<!-- checked -->
