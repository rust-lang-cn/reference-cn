# Use声明

>[use-declarations.md](https://github.com/rust-lang/reference/blob/master/src/items/use-declarations.md)\
>commit: eabdf09207bf3563ae96db9d576de0758c413d5d \
>本章译文最后维护日期：2021-1-24

> **<sup>句法:</sup>**\
> _UseDeclaration_ :\
> &nbsp;&nbsp; `use` _UseTree_ `;`
>
> _UseTree_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; ([_SimplePath_]<sup>?</sup> `::`)<sup>?</sup> `*`\
> &nbsp;&nbsp; | ([_SimplePath_]<sup>?</sup> `::`)<sup>?</sup> `{` (_UseTree_ ( `,`  _UseTree_ )<sup>\*</sup> `,`<sup>?</sup>)<sup>?</sup> `}`\
> &nbsp;&nbsp; | [_SimplePath_]&nbsp;( `as` ( [IDENTIFIER] | `_` ) )<sup>?</sup>

*use声明*用来创建一个或多个与程序项[路径][path]同义的本地名称绑定。通常使用 `use`声明来缩短引用模块所需的路径。这些声明可以出现在[模块][modules]和[块][blocks]中，但通常在作用域顶部。

[path]: ../paths.md
[modules]: modules.md
[blocks]: ../expressions/block-expr.md

use声明支持多种便捷方法:

* 使用带有花括号的 glob-like(`::`) 句法 `use a::b::{c, d, e::f, g::h::i};` 来同时绑定一个系列有共同前缀的路径。
* 使用关键字 `self`，例如 `use a::b::{self, c, d::e};`，来同时绑定一系列有共同前缀和共同父模块的路径。
* 使用句法 `use p::q::r as x;` 将编译目标名称重新绑定为新的本地名称。这种也可以和上面两种方法一起使用：`use a::b::{self as ab, c as abc}`。
* 使用星号通配符句法 `use a::b::*;` 来绑定与给定前缀匹配的所有路径。
* 将前面的方法嵌套重复使用，例如 `use a::b::{self as ab, c, d::{*, e::f}};`。

`use`声明的一个示例：

```rust
use std::option::Option::{Some, None};
use std::collections::hash_map::{self, HashMap};

fn foo<T>(_: T){}
fn bar(map1: HashMap<String, usize>, map2: hash_map::HashMap<String, usize>){}

fn main() {
    // 等价于 'foo(vec![std::option::Option::Some(1.0f64), std::option::Option::None]);'
    foo(vec![Some(1.0f64), None]);

    // `hash_map` 和 `HashMap` 在当前作用域内都有效.
    let map1 = HashMap::new();
    let map2 = hash_map::HashMap::new();
    bar(map1, map2);
}
```

## `use`可见性

与其他程序项一样，默认情况下，`use`声明对包含它的模块来说是私有的。同样的，如果使用关键字 `pub` 进行限定，`use`声明也可以是公有的。`use`声明可用于*重导出(re-expor)*名称。因此，公有的 `use`声明可以将某些公有名称重定向到不同的目标定义中：甚至是位于不同模块内具有私有可见性的规范路径定义中。如果这样的重定向序列形成一个循环或不能明确地解析，则会导致编译期错误。

重导出的一个示例：

```rust
mod quux {
    pub use self::foo::{bar, baz};
    pub mod foo {
        pub fn bar() {}
        pub fn baz() {}
    }
}

fn main() {
    quux::bar();
    quux::baz();
}
```

在本例中，模块 `quux` 重导出了在模块 `foo` 中定义的两个公共名称。

## `use` 路径

> **注意**：本章节内容还不完整。

一些正常和不正常的使用 `use`程序项的例子：
<!-- 注意: 这个例子在 2015 版或 2018 版都能正常工作。 -->

```rust
# #![allow(unused_imports)]
use std::path::{self, Path, PathBuf};  // good: std 是一个 crate 名称
use crate::foo::baz::foobaz;    // good: foo 在当前 crate 的第一层

mod foo {

    pub mod example {
        pub mod iter {}
    }

    use crate::foo::example::iter; // good: foo 在当前 crate 的第一层
//  use example::iter;      // 在 2015 版里不行，2015 版里相对路径必须以 `self` 开头; 2018 版这样写没问题
    use self::baz::foobaz;  // good: `self` 指的是 'foo' 模块
    use crate::foo::bar::foobar;   // good: foo 在当前 crate 的第一层

    pub mod bar {
        pub fn foobar() { }
    }

    pub mod baz {
        use super::bar::foobar; // good: `super` 指的是 'foo' 模块
        pub fn foobaz() { }
    }
}

fn main() {}
```
：
> **版本差异**: 在 2015 版中，`use`路径也允许访问 crate 根模块中的程序项。继续使用上面的例子，那以下 `use`路径的用法在 2015 版中有效，在 2018 版中就无效了:
>
> ```rust,edition2015
> # mod foo {
> #     pub mod example { pub mod iter {} }
> #     pub mod baz { pub fn foobaz() {} }
> # }
> use foo::example::iter;
> use ::foo::baz::foobaz;
> # fn main() {}
> ```
>
> 2015 版不允许用 use声明来引用[外部预导入包][extern prelude]里的 crate。因此，在2015 版中仍然需要使用 [`extern crate`]声明，以便在 use声明中去引用外部 crate。从 2018 版开始，use声明可以像 `extern crate` 一样指定外部 crate 依赖关系。
>
> 在 2018 版中，如果本地程序项与外部的 crate 名称相同，那么使用该 crate 名称需要一个前导的 `::` 来明确地选择该 crate 名称。这种做法是为了与未来可能发生的更改保持兼容。<!-- uniform_paths future-proofing -->
>
> ```rust,edition2018
> // use std::fs; // 错误, 这样有歧义.
> use ::std::fs;  // 从`std` crate 里导入, 不是下面这个 mod.
> use self::std::fs as self_fs;  // 从下面这个 mod 导入.
>
> mod std {
>     pub mod fs {}
> }
> # fn main() {}
> ```

## 下划线导入

通过使用形如为 `use path as _` 的带下划线的 use声明，可以在不绑定名称的情况下导入程序项。这对于导入一个 trait 特别有用，这样就可以在不导入 trait 的 symbol 的情况下使用这个 trait 的方法，例如，如果 trait 的 symbol 可能与另一个 symbol 冲突。再一个例子是链接外部的 crate 而不导入其名称。

使用星号全局导入(Asterisk glob imports,即 `::*`)句法将以 `_` 的形式导入能匹配到的所有程序项，但这些程序项在当前作用域中将处于不可命名的状态。
```rust
mod foo {
    pub trait Zoo {
        fn zoo(&self) {}
    }

    impl<T> Zoo for T {}
}

use self::foo::Zoo as _;
struct Zoo;  // 下划线形式的导入就是为了避免和这个程序项在名字上起冲突

fn main() {
    let z = Zoo;
    z.zoo();
}
```

在宏扩展之后会创建一些惟一的、不可命名的 symbols，这样宏就可以安全地扩展出(emit)对 `_` 导入的多个引用。例如，下面代码不应该产生错误:

```rust
macro_rules! m {
    ($item: item) => { $item $item }
}

m!(use std as _;);
// 这会扩展出:
// use std as _;
// use std as _;
```

[IDENTIFIER]: ../identifiers.md
[_SimplePath_]: ../paths.md#simple-paths
[`extern crate`]: extern-crates.md
[extern prelude]: ../names/preludes.md#extern-prelude
[path qualifiers]: ../paths.md#path-qualifiers

<!-- 2020-11-12-->
<!-- checked -->
