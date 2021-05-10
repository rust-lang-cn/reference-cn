# 模块

> **<sup>Syntax:</sup>**\
> _Module_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `mod` [IDENTIFIER] `;`\
> &nbsp;&nbsp; | `mod` [IDENTIFIER] `{`\
> &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [_Item_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; `}`

模块是零个或多个[项][items]的容器。

*模块项* 是一个模块，包含在大括号中，需要命名，并以关键字 `mod` 作为前缀。模块项将一个新的命名模块引入到组成 crate 的模块树中。模块可以任意嵌套。

模块示例：

```rust
mod math {
    type Complex = (f64, f64);
    fn sin(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
    fn cos(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
    fn tan(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
}
```

模块和类型共享相同的命名空间。禁止在作用域中声明与模块同名的命名类型，即：类型定义、trait、结构体、枚举、联合体、类型参数或 crate 不能遮蔽作用域中模块的名称，反之亦然。使用 `use` 纳入作用域的项同样有此限制。

## 模块的源文件名字

从外部文件加载没有主体的模块。当模块没有 `path` 属性时，文件的路径将镜像逻辑[模块路径][module
path]。原型模块路径组件是目录，且模块的内容位于一个具有模块名称和 `.rs` 扩展名的文件中。例如，如下模块结构具有相应的文件系统结构：

模块路径                   | 文件系统路径      | 文件内容
------------------------- | ---------------  | -------------
`crate`                   | `lib.rs`         | `mod util;`
`crate::util`             | `util.rs`        | `mod config;`
`crate::util::config`     | `util/config.rs` |

目录也可以作为模块，目录名既是模块文件名，该目录包含一个名为 `mod.rs` 的文件。上面例子中模块路径 `crate::util` 可以将文件系统路径替换表达为 `util/mod.rs`。不允许 `util.rs` 和 `util/mod.rs` 同时存在。

> **注**：在 `rustc` 1.30 之前，使用 `mod.rs` 文件是加载具有嵌套的子级模块的方法。鼓励使用新的命名约定，因为它更加一致，并且避免在一个项目中有许多文件被命名为 `mod.rs`。

### `path` 属性

用于加载外部文件模块的目录和文件可以被 `path` 属性影响。

对于不在内联模块体内的模块上的 `path` 属性，文件路径为相对于源文件所在的目录。例如，以下代码段将使用基于其所在位置的路径：

<!-- ignore: requires external files -->
```rust,ignore
#[path = "foo.rs"]
mod c;
```

源文件    | `c` 的文件位置 | `c` 的模块路径
-------------- | ------------------- | ----------------------
`src/a/b.rs`   | `src/a/foo.rs`      | `crate::a::b::c`
`src/a/mod.rs` | `src/a/foo.rs`      | `crate::a::c`

对于内联模块体内的 `path` 属性，文件路径的相对位置取决于 `path` 属性所在的源文件的类型。“mod-rs” 源文件是根模块（例如 `lib.rs` 或者 `main.rs`）和具有 `mod.rs` 文件的模块，“non-mod-rs” 源文件是其他模块文件。mod-rs 文件中的内联模块体内的 `path` 属性的路径为相对于 mod-rs 文件的目录，包括作为目录的内联模块组件。对于 non-mod-rs 文件，除了路径以具有 non-mod-rs 模块名称的目录开头外，其它都是相同的。例如，以下代码段将使用基于其所在位置的路径：

<!-- ignore: requires external files -->
```rust,ignore
mod inline {
    #[path = "other.rs"]
    mod inner;
}
```

源文件    | `内联`' 的文件位置   | `内联`' 的模块路径
-------------- | --------------------------| ----------------------------
`src/a/b.rs`   | `src/a/b/inline/other.rs` | `crate::a::b::inline::inner`
`src/a/mod.rs` | `src/a/inline/other.rs`   | `crate::a::inline::inner`

在内联模块和内部嵌套模块上组合上述 `path` 属性规则的示例（mod-rs 和 non-mod-rs 文件都适用）：

<!-- ignore: requires external files -->
```rust,ignore
#[path = "thread_files"]
mod thread {
    // Load the `local_data` module from `thread_files/tls.rs` relative to
    // this source file's directory.
    #[path = "tls.rs"]
    mod local_data;
}
```

## prelude 项

模块在作用域中隐式地有一些名称。这些名称是内建类型、宏，在外部 crate 上使用 [`#[macro_use]`][macro_use]，以及 crate 的 [prelude] 导入。这些名称都由一个标识符组成。这些名称不是模块的一部分，因此——比如说——任何名称 `name`，`self::name` 均不是有效路径。通过在模块或其父模块中放置 `no_implicit_prelude` [属性][attribute]，可以移除由 [prelude] 添加的名称。

## 模块的属性

和所有项类似，模块也接受外部属性。模块还接受内部属性：要么在 `{` 之后（具有主体的模块文件中），要么在源文件开头——可选的字节顺序标记（BOM）和释伴（shebang）之后。

对模块有意义的内建属性包括：[`cfg`]、[`deprecated`]、[`doc`]、[lint 检查属性][the lint check attributes]、`path`，以及 `no_implicit_prelude`。模块也接受宏属性。

[_InnerAttribute_]: ../attributes.md
[_Item_]: ../items.md
[macro_use]: ../macros-by-example.md#the-macro_use-attribute
[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../rustdoc/the-doc-attribute.html
[IDENTIFIER]: ../identifiers.md
[attribute]: ../attributes.md
[items]: ../items.md
[module path]: ../paths.md
[prelude]: ../crates-and-source-files.md#preludes-and-no_std
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes

<script>
(function() {
    var fragments = {
        "#prelude-items": "../names/preludes.html",
    };
    var target = fragments[window.location.hash];
    if (target) {
        var url = window.location.toString();
        var base = url.substring(0, url.lastIndexOf('/'));
        window.location.replace(base + "/" + target);
    }
})();
</script>
