# 自动推断型类型

>[parameters.md](https://github.com/rust-lang/reference/blob/master/src/types/parameters.md)\
>commit: 43dc1a42f19026f580e34a095e91804c3d6da186 \
>本章译文最后维护日期：2020-11-14

> **<sup>句法</sup>**\
> _InferredType_ : `_`

自动推断型类型要求编译器尽可能根据周围可用的信息推断出实际使用类型。它不能用于程序项的签名中。它经常用于泛型参数中：

```rust
let x: Vec<_> = (0..10).collect();
```

<!--
  这里还有什么要说的？
  我所知道的唯一文档是https://rustc-dev-guide.rust-lang.org/type-inference.html
  有关类型推断应该需要进一步的广泛讨论。
-->

<!-- 2020-11-12-->
<!-- checked -->
