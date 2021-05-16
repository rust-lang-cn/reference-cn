# 联合体

>[unions.md](https://github.com/rust-lang/reference/blob/master/src/items/unions.md)\
>commit: 7c6e0c00aaa043c89e0d9f07e78999268e8ac054 \
>本章译文最后维护日期：2021-2-10

> **<sup>句法</sup>**\
> _Union_ :\
> &nbsp;&nbsp; `union` [IDENTIFIER]&nbsp;[_GenericParams_]<sup>?</sup> [_WhereClause_]<sup>?</sup>
>   `{`[_StructFields_] `}`

除了用 `union` 代替 `struct`外，联合体声明使用和结构体声明相同的句法。

```rust
#[repr(C)]
union MyUnion {
    f1: u32,
    f2: f32,
}
```

联合体的关键特性是联合体的所有字段共享同一段存储。因此，对联合体的一个字段的写操作会覆盖其他字段，而联合体的尺寸由其尺寸最大的字段的尺寸所决定。

## 联合体的初始化

可以使用与结构体类型相同的句法创建联合体类型的值，但必须只能指定一个字段：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
let u = MyUnion { f1: 1 };
```

上面的表达式创建了一个类型为 `MyUnion` 的值，并使用字段 `f1` 初始化了其存储。可以使用与结构体字段相同的句法访问联合体：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
# let u = MyUnion { f1: 1 };
let f = unsafe { u.f1 };
```

## 读写联合体字段

联合体没有“活跃字段(active field)”的概念。相反，每次访问联合体只是用访问所指定的字段的类型解释此联合体的存储。读取联合体的字段就是以当前读取字段的类型来解读此联合体的存储位。字段（间）可以有非零的偏移量存在（使用 [C表型][the C representation]的除外）；在这种情况下，读取将从字段的相对偏移量的 bit 开始。程序员有责任确保此数据在当前字段类型下有效。否则会导致[未定义行为(undefined behavior)][undefined behavior]。例如，在 [`bool`类型][boolean type]的字段下读取到数值 `3` 是未定义行为。实际上，对一个 [C表型][the C representation]的联合体进行写操作，然后再从中读取，就好比从用于写入的类型到用于读取的类型的 [`transmute`] 操作。

因此，所有的联合体字段的读取必须放在非安全(`unsafe`)块里：

```rust
# union MyUnion { f1: u32, f2: f32 }
# let u = MyUnion { f1: 1 };
#
unsafe {
    let f = u.f1;
}
```

对实现了 [`Copy`] trait 或 [`ManuallyDrop`][ManuallyDrop] trait 的联合体字段的写操作不需要事先的析构读操作，也因此这些写操作不必放在非安全(`unsafe`)块中。[^译者备注]

```rust
# use std::mem::ManuallyDrop;
union MyUnion { f1: u32, f2: ManuallyDrop<String> }
let mut u = MyUnion { f1: 1 };

// 这些都不是必须要放在 `unsafe` 里的
u.f1 = 2;
u.f2 = ManuallyDrop::new(String::from("example"));
```

通常，那些用到联合体的程序代码会先在非安全的联合体字段访问操作上提供一层安全包装，然后再使用。

## 联合体和 `Drop`

当一个联合体被销毁时，它无法知道需要销毁它的哪些字段。因此，所有联合体的字段都必须实现 `Copy` trait 或被包装进 [`ManuallyDrop<_>`]。这确保了联合体在超出作用域时不需要销毁任何内容。

与结构体和枚举一样，联合体也可以通过 `impl Drop` 手动定义被销毁时的具体动作。

## 联合体上的模式匹配

访问联合体字段的另一种方法是使用模式匹配。联合体字段上的模式匹配与结构体上的模式匹配使用相同的句法，只是这种模式只能一次指定一个字段。因为模式匹配就像使用特定字段来读取联合体，所以它也必须被放在非安全(`unsafe`)块中。

```rust
# union MyUnion { f1: u32, f2: f32 }
#
fn f(u: MyUnion) {
    unsafe {
        match u {
            MyUnion { f1: 10 } => { println!("ten"); }
            MyUnion { f2 } => { println!("{}", f2); }
        }
    }
}
```

模式匹配可以将联合体作为更大的数据结构的一个字段进行匹配。特别是，当使用 Rust 联合体通过 FFI 实现 C标签联合体(C tagged union)时，这允许同时在标签和相应字段上进行匹配：

```rust
#[repr(u32)]
enum Tag { I, F }

#[repr(C)]
union U {
    i: i32,
    f: f32,
}

#[repr(C)]
struct Value {
    tag: Tag,
    u: U,
}

fn is_zero(v: Value) -> bool {
    unsafe {
        match v {
            Value { tag: Tag::I, u: U { i: 0 } } => true,
            Value { tag: Tag::F, u: U { f: num } } if num == 0.0 => true,
            _ => false,
        }
    }
}
```

## 引用联合体字段

由于联合体字段共享存储，因此拥有对联合体一个字段的写访问权就同时拥有了对其所有其他字段的写访问权。因为这一事实，引用的借用检查规则必须调整。因此，如果联合体的一个字段是被出借，那么在相同的生存期内它的所有其他字段也都处于出借状态。

```rust,compile_fail
# union MyUnion { f1: u32, f2: f32 }
// 错误: 不能同时对 `u` (通过 `u.f2`)拥有多余一次的可变借用
fn test() {
    let mut u = MyUnion { f1: 1 };
    unsafe {
        let b1 = &mut u.f1;
//                    ---- 首次可变借用发生在这里 (通过 `u.f1`)
        let b2 = &mut u.f2;
//                    ^^^^ 二次可变借用发生在这里 (通过 `u.f2`)
        *b1 = 5;
    }
//  - 首次借用在这里结束
    assert_eq!(unsafe { u.f1 }, 5);
}
```

如您所见，在许多方面（除了布局、安全性和所有权），联合体的行为与结构体完全相同，这很大程度上是因为联合体继承使用了结构体的句法的结果。对于 Rust 语言未明确提及的许多方面（比如隐私性(privacy)、名称解析、类型推断、泛型、trait实现、固有实现、一致性、模式检查等等）也是如此。

[^译者备注]: 这句译者简单理解就是对已经初始化的变量再去覆写的时候要先去读一下这个变量代表的地址上的值的状态，如果有值，并且允许覆写，那 Rust 为防止内存泄漏就先执行那变量的析构行为（drop()），清空那个地址上的关联堆数据，再写入。我们这里对联合体的预设条件是此联合体值有 Copy特性，有 Copy特性了，对值的直接覆写不会造成内存泄漏，就不必调用析构行为，也不需要事先的非安全读操作了。对于这个问题[nomicon](https://learnku.com/docs/nomicon/2018)的“未初始化内存”章有讲述，博主[CrLF0710](https://www.zhihu.com/people/crlf0710)的两篇“学一点 Rust 内存模型会发生什么呢？”里也都有精彩讲解。

[IDENTIFIER]: ../identifiers.md
[_GenericParams_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_StructFields_]: structs.md
[`transmute`]: https://doc.rust-lang.org/std/mem/fn.transmute.html
[`Copy`]: https://doc.rust-lang.org/std/marker/trait.Copy.html
[boolean type]: ../types/boolean.md
[`ManuallyDrop<_>`]: https://doc.rust-lang.org/std/mem/struct.ManuallyDrop.html
[the C representation]: ../type-layout.md#reprc-unions
[undefined behavior]: ../behavior-considered-undefined.html
