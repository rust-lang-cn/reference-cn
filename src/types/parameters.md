# 类型参数

>[parameters.md](https://github.com/rust-lang/reference/blob/master/src/types/parameters.md)\
>commit: eb02dd5194a747277bfa46b0185d1f5c248f177b \
>本章译文最后维护日期：2020-11-14

在带有类型参数声明的程序项的代码体内，这些类型参数的名称可以直接当做类型使用：

```rust
fn to_vec<A: Clone>(xs: &[A]) -> Vec<A> {
    if xs.is_empty() {
        return vec![];
    }
    let first: A = xs[0].clone();
    let mut rest: Vec<A> = to_vec(&xs[1..]);
    rest.insert(0, first);
    rest
}
```

这里，`first` 的类型为 `A`，援引的是 `to_vec` 的类型参数 `A`；`rest` 的类型为 `Vec<A>`，它是一个元素类型为 `A` 向量(vector)。

<!-- 2020-11-12-->
<!-- checked -->
