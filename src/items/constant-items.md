# 常量项

>[constant-items.md](https://github.com/rust-lang/reference/blob/master/src/items/constant-items.md)\
>commit: 761ad774fcb300f2b506fed7b4dbe753cda88d80 \
>本章译文最后维护日期：2020-1-17

> **<sup>句法</sup>**\
> _ConstantItem_ :\
> &nbsp;&nbsp; `const` ( [IDENTIFIER] | `_` ) `:` [_Type_] ( `=` [_Expression_] )<sup>?</sup> `;`

*常量项*是一个可选的具名 *[常量值][constant value]*，它与程序中的具体内存位置没有关联。无论常量在哪里使用，它们本质上都是内联的，这意味着当它们被使用时，都是直接被拷贝到相关的上下文中来使用的。这包括使用非拷贝(non-[`Copy`])类型的值和来自外部的 crate 的常量。对相同常量的引用不保证它们引用的是相同的内存地址。

常量必须显式指定数据类型。类型必须具有 `'static`生存期：程序初始化器(initializer)中的任何引用都必须具有 `'static`生存期。

常量可以引用其他常量的地址，在这种情况下，如果适用，该地址将具有省略的生存期，否则（在大多数情况下）默认为 `'static`生存期。（请参阅[静态生存期省略][static lifetime elision]。）但是，编译器仍有权多次调整转移该常量，因此引用的地址可能并不固定。

```rust
const BIT1: u32 = 1 << 0;
const BIT2: u32 = 1 << 1;

const BITS: [u32; 2] = [BIT1, BIT2];
const STRING: &'static str = "bitstring";

struct BitsNStrings<'a> {
    mybits: [u32; 2],
    mystring: &'a str,
}

const BITS_N_STRINGS: BitsNStrings<'static> = BitsNStrings {
    mybits: BITS,
    mystring: STRING,
};
```

常量表达式只能在[trait定义中省略][trait definition]。

## Constants with Destructors
## 常量与析构函数

常量可以包含析构函数。析构函数在值超出作用域时运行。[^译者备注]

```rust
struct TypeWithDestructor(i32);

impl Drop for TypeWithDestructor {
    fn drop(&mut self) {
        println!("Dropped. Held {}.", self.0);
    }
}

const ZERO_WITH_DESTRUCTOR: TypeWithDestructor = TypeWithDestructor(0);

fn create_and_drop_zero_with_destructor() {
    let x = ZERO_WITH_DESTRUCTOR;
    // x 在函数的结尾处通过调用 drop 方法被销毁。
    // 打印出 "Dropped. Held 0.".
}
```

## Unnamed constant
## 未命名常量

不同于[关联][associated]常量(associated constant)，[自由][free]常量(free constant)可以使用下划线来命名。例如:

```rust
const _: () =  { struct _SameNameTwice; };

// OK 尽管名称和上面的一样：
const _: () =  { struct _SameNameTwice; };
```

与[下划线导入][underscore imports]一样，宏可以多次安全地在同一作用域中扩展出相同的未具名常量。例如，以下内容不应该产生错误:

```rust
macro_rules! m {
    ($item: item) => { $item $item }
}

m!(const _: () = (););
// 这会展开出：
// const _: () = ();
// const _: () = ();
```

[^译者备注]: 在程序退出前，析构销毁的只是其中的一份拷贝；这句还有另一层含义是常量在整个程序结束时会调用析构函数。

[associated]: ../glossary.md#associated-item
[constant value]: ../const_eval.md#constant-expressions
[free]: ../glossary.md#free-item
[static lifetime elision]: ../lifetime-elision.md#static-lifetime-elision
[IDENTIFIER]: ../identifiers.md
[trait definition]: traits.md
[underscore imports]: use-declarations.md#underscore-imports
[_Type_]: ../types.md#type-expressions
[_Expression_]: ../expressions.md
[`Copy`]: ../special-types-and-traits.md#copy

<!-- 2021-1-17-->
<!-- checked -->
