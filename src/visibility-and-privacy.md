# 可见性与隐私权

>[visibility-and-privacy.md](https://github.com/rust-lang/reference/blob/master/src/visibility-and-privacy.md)\
>commit: f41afd4f69a7996ac66b01f75e333bccf43337f7 \
>本章译文最后维护日期：2020-11-10

> **<sup>句法<sup>**\
> _Visibility_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `pub`\
> &nbsp;&nbsp; | `pub` `(` `crate` `)`\
> &nbsp;&nbsp; | `pub` `(` `self` `)`\
> &nbsp;&nbsp; | `pub` `(` `super` `)`\
> &nbsp;&nbsp; | `pub` `(` `in` [_SimplePath_] `)`

这两个术语经常互换使用，它们试图表达的是对“这个地方能使用这个程序项吗？”这个问题的回答。

Rust 的名称解析是在命名空间的层级结构的全局层（即最顶层）上运行的。此层级结构中的每个级别都可以看作是某个程序项。这些程序项就是我们前面提到的那些程序项之一（注意这也包括外部crate）。声明或定义一个新模块可以被认为是在定义的位置向此层次结构中插入一个新的（层级）树。

为了控制接口是否可以被跨模块使用，Rust 会检查每个程序项的使用，来看看它是否被允许使用。此处就是生成隐私告警的地方，或者说是提示：“you used a private item of another module and weren't allowed to.”的地方。（译者注：这句提示可以翻译为：您使用了另一个模块的私有程序项，但不允许这样做。）

默认情况下，Rust 中的所有内容都是*私有的*，但有两个例外：`pub` trait 中的关联程序项默认为公有的；`pub` 枚举中的枚举变体也默认为公有的。当一个程序项被声明为 `pub` 时，它可以被认为是外部世界能以访问的。例如：

```rust
# fn main() {}
// 声明一个私有结构体
struct Foo;

// 声明一个带有私有字段的公有结构体
pub struct Bar {
    field: i32,
}

// 声明一个带有两个公有变体的公有枚举
pub enum State {
    PubliclyAccessibleState,
    PubliclyAccessibleState2,
}
```

依据程序项是公有的还是私有的，Rust 分两种情况来访问数据：

1. 如果某个程序项是公有的，那么如果可以从外部的某一模块 `m` 访问到该程序项的所有祖先模块，则一定可以从这个模块 `m` 中访问到该程序项。甚至还可以通过重导出来命名该程序项。具体见后文。
   
2. 如果某个程序项是私有的，则当前模块及当前模块的后代模块都可以访问它。

这两种情况在创建能对外暴露公共API 同时又隐藏内部实现细节的模块层次结构时非常好用。为了帮助理解，以下是一些常见案例和它们需要做的事情：

* 库开发人员需要将一些功能暴露给链接了其库的 crate。作为第一种情况的结果，这意味着任何那些在外部可用的程序项自身以及其路径层级结构中的每一层都必须是公有的(`pub`)的。并且此层级链中的任何私有程序项都不允许被外部访问。
 
* crate 需要一个全局可用的“辅助模块(helper module)”，但又不想将辅助模块公开为公共API。为了实现这一点，可以在整个 crate 的根模块（路径层级结构中的最顶层）下建一个私有模块，该模块在内部是“公共API”。因为整个 crate 都是根模块的后代，所以整个本地 crate 里都可以通过第二种情况访问这个私有模块。

* 在为模块编写单元测试时，通常的习惯做法是给要测试的模块加一个命名为 `mod test` 的直接子模块。这个模块可以通过第二种情况访问父模块的任何程序项，这意味着内部实现细节也可以从这个子模块里进行无缝地测试。

在第二种情况下，我们提到了当前模块及其后代“可以访问”私有项，但是访问一个程序项的确切含义取决于该项是什么。例如，访问一个模块意味着要查看它的内部（以导入更多的程序项）；访问一个函数意味着它被调用了。此外，路径表达式和导入语句也被视为访问一个程序项，但只有当访问目标位于当前可见的作用域内时，它们才算是有效的数据访问。

下面是一段示例程序，它例证了上述三种情况：

```rust
// 这个模块是私有的，这意味着没有外部crate 可以访问这个模块。
// 但是，由于它在当前 crate 的根模块下，
// 因此当前 crate 中的任何模块都可以访问该模块中任何公有可见性程序项。
mod crate_helper_module {

    // 这个函数可以被当前 crate 中的任何东西使用
    pub fn crate_helper() {}

    // 此函数*不能*被用于 crate 中的任何其他模块中。它在 `crate_helper_module` 之外不可见，
    // 因此只有当前模块及其后代可以访问它。
    fn implementation_detail() {}
}

// 此函数“对根模块是公有”的，这意味着它可被链接了此crate 的其他crate 使用。
pub fn public_api() {}

// 与 'public_api' 类似，此模块是公有的，因此其他的crate 是能够看到此模块内部的。
pub mod submodule {
    use crate_helper_module;

    pub fn my_method() {
        // 本地crate 中的任何程序项都可以通过上述两个规则的组合来调用辅助模块里的公共接口。
        crate_helper_module::crate_helper();
    }

    // 此函数对任何不是 `submodule` 的后代的模块都是隐藏的
    fn my_implementation() {}

    #[cfg(test)]
    mod test {

        #[test]
        fn test_my_implementation() {
            // 因为此模块是 `submodule` 的后代，因此允许它访问 `submodule` 内部的私有项，而不会侵犯隐私权。
            super::my_implementation();
        }
    }
}

# fn main() {}
```

对于一个 Rust 程序要通过隐私检查，所有的路径都必须满足上述两个访问规则。这里说的路径包括所有的 use语句、表达式、类型等。

## `pub(in path)`, `pub(crate)`, `pub(super)`, and `pub(self)`

除了公有和私有之外，Rust 还允许用户（用关键字 `pub` ）声明仅在给定作用域内可见的程序项。声明形式的限制规则如下：

- `pub(in path)` 使一个程序项在提供的 `path` 中可见。`path` 必须是声明其可见性的程序项的祖先模块。
- `pub(crate)` 使一个程序项在当前 crate 中可见。
- `pub(super)` 使一个程序项对父模块可见。这相当于 `pub(in super)`。
- `pub(self)` 使一个程序项对当前模块可见。这相当于 `pub(in self)` 或者根本不使用 `pub`。

> **版本差异**: 从 2018版开始，`pub(in path)` 的路径必须以 `crate`、`self`或`super`开头。2015版还可以使用以 `::` 开头的路径，或以根模块下的模块名的开头的路径。

这里是一些示例：

```rust
pub mod outer_mod {
    pub mod inner_mod {
        // 此函数在 `outer_mod` 内部可见
        pub(in crate::outer_mod) fn outer_mod_visible_fn() {}
        // 同上，但这只能在2015版中有效
        pub(in outer_mod) fn outer_mod_visible_fn_2015() {}

        // 此函数对整个 crate 都可见
        pub(crate) fn crate_visible_fn() {}

        // 此函数在 `outer_mod` 下可见
        pub(super) fn super_mod_visible_fn() {
            // 此函数之所以可用，是因为我们在同一个模块下
            inner_mod_visible_fn();
        }

        // 这个函数只在 `inner_mod` 中可见，这与它保持私有的效果是一样的。
        pub(self) fn inner_mod_visible_fn() {}
    }
    pub fn foo() {
        inner_mod::outer_mod_visible_fn();
        inner_mod::crate_visible_fn();
        inner_mod::super_mod_visible_fn();

        // 此函数不再可见，因为我们在  `inner_mod` 之外
        // 错误! `inner_mod_visible_fn` 是私有的
        //inner_mod::inner_mod_visible_fn();
    }
}

fn bar() {
    // 此函数仍可见，因为我们在同一个 crate 里
    outer_mod::inner_mod::crate_visible_fn();

    // 此函数不再可见，因为我们在`outer_mod`之外
    // 错误! `super_mod_visible_fn` 是私有的
    //outer_mod::inner_mod::super_mod_visible_fn();

    // 此函数不再可见，因为我们在`outer_mod`之外
    // 错误! `outer_mod_visible_fn` 是私有的
    //outer_mod::inner_mod::outer_mod_visible_fn();

    outer_mod::foo();
}

fn main() { bar() }
```

> **注意:** 此句法仅对程序项的可见性添加了另一个限制。它不能保证该程序项在指定作用域的所有部分都可见。要访问一个程序项，当前作用域内它的所有父项还是必须仍然可见。

## 重导出和可见性

Rust 允许使用指令 `pub use` 公开重导出程序项。因为这是一个公有指令，所以允许通过上面的规则验证后在当前模块中使用该程序项。重导出本质上允许使用公有方式访问重导出的程序项的内部。例如，下面程序是有效的：

```rust
pub use self::implementation::api;

mod implementation {
    pub mod api {
        pub fn f() {}
    }
}

# fn main() {}
```

这意味着任何外部 crate，只要引用 `implementation::api::f` 都将收到违反隐私的错误报告，而使用路径 `api::f` 则被允许。

当重导出私有程序项时，可以认为它允许通过重导出短路了“隐私链(privacy chain)”，而不是像通常那样通过命名空间层次结构来传递“隐私链”。

[_SimplePath_]: paths.md#simple-paths

<!-- 2020-11-12-->
<!-- checked -->
