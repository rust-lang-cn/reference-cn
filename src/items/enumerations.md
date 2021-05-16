# 枚举

>[enumerations.md](https://github.com/rust-lang/reference/blob/master/src/items/enumerations.md)\
>commit: d8cbe4eedb77bae3db9eff87b1238e7e23f6ae92 \
>本章译文最后维护日期：2021-2-21

> **<sup>句法</sup>**\
> _Enumeration_ :\
> &nbsp;&nbsp; `enum`
>    [IDENTIFIER]&nbsp;
>    [_GenericParams_]<sup>?</sup>
>    [_WhereClause_]<sup>?</sup>
>    `{` _EnumItems_<sup>?</sup> `}`
>
> _EnumItems_ :\
> &nbsp;&nbsp; _EnumItem_ ( `,` _EnumItem_ )<sup>\*</sup> `,`<sup>?</sup>
>
> _EnumItem_ :\
> &nbsp;&nbsp; _OuterAttribute_<sup>\*</sup> [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; [IDENTIFIER]&nbsp;( _EnumItemTuple_ | _EnumItemStruct_
>                                | _EnumItemDiscriminant_ )<sup>?</sup>
>
> _EnumItemTuple_ :\
> &nbsp;&nbsp; `(` [_TupleFields_]<sup>?</sup> `)`
>
> _EnumItemStruct_ :\
> &nbsp;&nbsp; `{` [_StructFields_]<sup>?</sup> `}`
>
> _EnumItemDiscriminant_ :\
> &nbsp;&nbsp; `=` [_Expression_]

*枚举*，英文为 *enumeration*，常见其简写形式 *enum*，它同时定义了一个标称型(nominal)[枚举类型][enumerated type]和一组*构造器*，这可用于创建或使用模式来匹配相应枚举类型的值。

枚举使用关键字 `enum` 来声明。

`enum` 程序项的一个示例和它的使用方法：

```rust
enum Animal {
    Dog,
    Cat,
}

let mut a: Animal = Animal::Dog;
a = Animal::Cat;
```

枚举构造器可以带有具名字段或未具名字段：

```rust
enum Animal {
    Dog(String, f64),
    Cat { name: String, weight: f64 },
}

let mut a: Animal = Animal::Dog("Cocoa".to_string(), 37.2);
a = Animal::Cat { name: "Spotty".to_string(), weight: 2.7 };
```

在这个例子中，`Cat` 是一个*类结构体枚举变体(struct-like enum variant)*，而 `Dog` 则被简单地称为枚举变体。每个枚举实例都有一个*判别值/判别式(discriminant)*，它是一个与此枚举实例关联的整数，用来确定它持有哪个变体。可以通过 [`mem::discriminant`] 函数来获得对这个判别值的不透明引用。

## 为无字段枚举自定义判别值

如果枚举的*任何*变体都没有附加字段，则可以直接设置和访问判别值。

可以使用操作符 `as` 通过[数值转换][numeric cast]将这些枚举类型转换为整型。枚举可以可选地指定每个判别值的具体值，方法是在变体名后面追加 `=` 和[常量表达式][constant expression]。如果声明中的第一个变体未指定，则将其判别值设置为零。对于其他未指定的判别值，它比照前一个变体的判别值按 1 递增。

```rust
enum Foo {
    Bar,            // 0
    Baz = 123,      // 123
    Quux,           // 124
}

let baz_discriminant = Foo::Baz as u32;
assert_eq!(baz_discriminant, 123);
```

尽管编译器被允许在实际的内存布局中使用较小的类型，但在[默认表形(default representation)][default representation]下，指定的判别值会被解释为一个 `isize` 值。也可以使用[原语表形(primitive representation)]或[`C`表形][`C` representation]来更改成大小可接受的值。

同一枚举中，两个变体使用相同的判别值是错误的。

```rust,compile_fail
enum SharedDiscriminantError {
    SharedA = 1,
    SharedB = 1
}

enum SharedDiscriminantError2 {
    Zero,       // 0
    One,        // 1
    OneToo = 1  // 1 (和前值冲突！)
}
```

当前一个变体的判别值是当前表形允许的的最大值时，再使用默认判别值就也是错误的。

```rust,compile_fail
#[repr(u8)]
enum OverflowingDiscriminantError {
    Max = 255,
    MaxPlusOne // 应该是256，但枚举溢出了
}

#[repr(u8)]
enum OverflowingDiscriminantError2 {
    MaxMinusOne = 254, // 254
    Max,               // 255
    MaxPlusOne         // 应该是256，但枚举溢出了。
}
```

## 无变体枚举

没有变体的枚举称为*零变体枚举/无变体枚举*。因为它们没有有效的值，所以不能被实例化。

```rust
enum ZeroVariants {}
```

零变体枚举与 [*never类型*][never type]等效，但它不能被强转为其他类型。

```rust,compile_fail
# enum ZeroVariants {}
let x: ZeroVariants = panic!();
let y: u32 = x; // 类型不匹配错误
```

## 变体的可见性

依照句法规则，枚举变体是允许有自己的[*可见性(visibility)*][Visibility]限定/注解(annotation)的，但当枚举被（句法分析程序）验证(validate)通过后，可见性注解又被弃用。因此，在源码解析层面，允许跨不同的上下文对其中不同类型的程序项使用统一的句法规则进行解析。

```rust
macro_rules! mac_variant {
    ($vis:vis $name:ident) => {
        enum $name {
            $vis Unit,

            $vis Tuple(u8, u16),

            $vis Struct { f: u8 },
        }
    }
}

// 允许空 `vis`.
mac_variant! { E }

// 这种也行，因为这段代码在被验证通过前会被移除。
#[cfg(FALSE)]
enum E {
    pub U,
    pub(crate) T(u8),
    pub(super) T { f: String }
}
```

[IDENTIFIER]: ../identifiers.md
[_GenericParams_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_Expression_]: ../expressions.md
[_TupleFields_]: structs.md
[_StructFields_]: structs.md
[_Visibility_]: ../visibility-and-privacy.md
[enumerated type]: ../types/enum.md
[`mem::discriminant`]: https://doc.rust-lang.org/std/mem/fn.discriminant.html
[never type]: ../types/never.md
[numeric cast]: ../expressions/operator-expr.md#semantics
[constant expression]: ../const_eval.md#constant-expressions
[default representation]: ../type-layout.md#the-default-representation
[primitive representation]: ../type-layout.md#primitive-representations
[`C` representation]: ../type-layout.md#the-c-representation
