# 类型布局

>[type-layout.md](https://github.com/rust-lang/reference/blob/master/src/type-layout.md)\
>commit: be4332c6a8f93c425d6f9a8f59b2a6dec63f8ffe \
>本章译文最后维护日期：2020-12-17

类型的布局描述类型的尺寸(size)、对齐量(alignment)和字段(fields)的*相对偏移量(relative offsets)*。对于枚举，其判别值(discriminant)的布局和解释也是类型布局的一部分。

每次编译都有可能更改类型布局。这里我们只阐述当前编译器所保证的内容，而没试图去阐述编译器对此做了什么。

## 尺寸和对齐量

所有值都有一个对齐量和尺寸。

值的*对齐量*指定了哪些地址可以有效地存储该值。对齐量为 `n` 的值只能存储地址为 n 的倍数的内存地址上。例如，对齐量为 2 的值必须存储在偶数地址上，而对齐量为 1 的值可以存储在任何地址上。对齐量是用字节数来度量的，必须至少是 1，并且总是 2 的幂次。值的对齐量可以通过函数 [`align_of_val`] 来检测。

值的*尺寸*是同类型的值组成的数组中连续两个元素之间的字节偏移量，此偏移量包括了为保持程序项类型内部对齐而对此类型做的对齐填充。值的尺寸总是其对齐量的非负整数倍数。值的尺寸可以通过函数 [`size_of_val`] 来检测。

如果某类型的所有的值都具有相同的尺寸和对齐量，并且两者在编译时都是已知的，并且实现了 [`Sized`] trait，则可以使用函数 [`size_of`] 和 [`align_of`] 对此类型进行检测。没有实现 `Sized` trait 的类型被称为[动态尺寸类型][dynamically sized types]。由于实现了 `Sized` trait 的某一类型的所有值共享相同的尺寸和对齐量，所以我们分别将这俩共享值称为该类型的尺寸和该类型的对齐量。

## 原生类型的布局

下表给出了大多数原生类型(primitives)的尺寸。

| 类型              | `size_of::<Type>()`|
|--                 |--                  |
| `bool`            | 1                  |
| `u8` / `i8`       | 1                  |
| `u16` / `i16`     | 2                  |
| `u32` / `i32`     | 4                  |
| `u64` / `i64`     | 8                  |
| `u128` / `i128`   | 16                 |
| `f32`             | 4                  |
| `f64`             | 8                  |
| `char`            | 4                  |

`usize` 和 `isize` 的尺寸足以包含目标平台上的每个内存地址。例如，在 32-bit 目标上，它们是 4 个字节，而在 64-bit 目标上，它们是 8 个字节。

大多数原生类型的对齐量通常与它们的尺寸保持一致，尽管这是特定于平台的行为。比较典型的就是在 x86 平台上，u64 和 f64 都上 32-bit 的对齐量。

## 指针和引用的布局

指针和引用具有相同的布局。指针或引用的可变性不会影响其布局。

指向固定尺寸类型(sized type)的值的指针具有和 `usize` 相同的尺寸和对齐量。

指向非固定尺寸类型(unsized types)的值的指针是固定尺寸的。其尺寸和对齐量至少等于一个指针的尺寸和对齐量

> 注意：虽然不应该依赖于此，但是目前所有指向 <abbr title="Dynamically Sized Types">DST</abbr> 的指针都是 `usize` 的两倍尺寸，并且具有相同的对齐量。

## 数组的布局

数组的布局使得数组的第 n 个(`nth`)元素为从数组开始的位置向后偏移 _n * 元素类型的尺寸(`n * the size of the element's type`)_ 个字节数。数组 `[T; n]` 的尺寸为 `size_of::<T>() * n`，对齐量和 `T` 的对齐量相同。

## 切片的布局

切片的布局与它们所切的那部分数组片段相同。

> 注意：这是关于原生的 `[T]`类型，而不是指向切片的指针（`&[T]`、`Box<[T]>` 等）。

## 字符串切片(`str`)的布局

字符串切片是一种 UTF-8 表示形式(representation)的字符序列，它们与 `[u8]`类型的切片拥有相同的类型布局。

## 元组的布局

元组对于其布局没有任何保证。

一个例外情况是单元结构体(unit tuple)(`()`)类型，它被保证为尺寸为 0，对齐量为 1。

## trait对象的布局

trait对象的布局与 trait对象的值相同。

> 注意：这是关于原生 trait对象类型(raw trait object type)的，而不是指向 trait对象的指针（`&dyn Trait`， `Box<dyn Trait>` 等）。

## 闭包的布局

闭包的布局没有保证。

## 表形/表示形式

所有用户定义的复合类型（结构体(`struct`)、枚举(`enum`)和联合体(`union`)）都有一个*表形(representation)*属性，该属性用于指定该类型的布局。类型的可能表形有：

- [默认(default)表形][Default]
- [`C`表形][`C`]
- [原语表形(primitive representation)][primitive representations]
- [透明表形(`transparent`)][`transparent`]

类型的表形可以通过对其应用 `repr`属性来更改。下面的示例展示了一个 `C`表形的结构体。

```rust
#[repr(C)]
struct ThreeInts {
    first: i16,
    second: i8,
    third: i32
}
```

可以分别使用 `align` 和 `packed` 修饰符增大或缩小对齐量。它们可以更改属性中指定表形的对齐量。如果未指定表形，则更改默认表形的。

```rust
// 默认表形，把对齐量缩小到2。
#[repr(packed(2))]
struct PackedStruct {
    first: i16,
    second: i8,
    third: i32
}

// C表形，把对齐量增大到8
#[repr(C, align(8))]
struct AlignedStruct {
    first: i16,
    second: i8,
    third: i32
}
```

> 注意：由于表形是程序项的属性，因此表形不依赖于泛型参数。具有相同名称的任何两种类型都具有相同的表形。例如，`Foo<Bar>` 和 `Foo<Baz>` 都有相同的表形。

类型的表形可以更改字段之间的填充，但不会更改字段本身的布局。例如一个使用 `C`表形的结构体，如果它包含一个默认表形的字段 `Inner`，那么它不会改变 `Inner` 的布局。

### 默认表形

没有 `repr`属性的标称(nominal)类型具有默认表形。非正式地的情况下，也称这种表形为 `rust`表形。

这种表形不保证每次编译都有统一的数据布局。

### `C`表形

`C`表形被设计用于双重目的：一个目的是创建可以与 C语言互操作的类型；第二个目的是创建可以正确执行依赖于数据布局的操作的类型，比如将值重新解释为其他类型。

因为这种双重目的存在，可以只利用其中的一个目的，如只创建有固定布局的类型，而放弃与 C语言的互操作。

这种表型可以应用于结构体(structs)、联合体(unions)和枚举(enums)。一个例外是[零变体枚举(zero-variant enums)][zero-variant enums]，它的 `C`表形是错误的。

#### `#[repr(C)]`结构体

结构体的对齐量是其*最大对齐量的字段(most-aligned field)*的对齐量。

字段的尺寸和偏移量则由以下算法确定：

1. 把当前偏移量设为从 0 字节开始。

2. 对于结构体中的每个字段，按其声明的先后顺序，首先确定其尺寸和对齐量；如果当前偏移量不是对其齐量的整倍数，则向当前偏移量添加填充字节，直至其对齐量的倍数[^译注1]；至此，当前字段的偏移量就是当前偏移量；下一步再根据当前字段的尺寸增加当前偏移量。

3. 最后，整个结构体的尺寸就是当前偏移量向上取整到结构体对齐量的最小整数倍数。

下面用伪代码描述这个算法：

<!-- ignore: pseudocode -->
```rust,ignore
/// 返回偏移(`offset`)之后需要的填充量，以确保接下来的地址将被安排到可对齐的地址。
fn padding_needed_for(offset: usize, alignment: usize) -> usize {
    let misalignment = offset % alignment;
    if misalignment > 0 {
        // 向上取整到对齐量(`alignment`)的下一个倍数
        alignment - misalignment
    } else {
        // 已经是对齐量(`alignment`)的倍数了
        0
    }
}

struct.alignment = struct.fields().map(|field| field.alignment).max();

let current_offset = 0;

for field in struct.fields_in_declaration_order() {
    // 增加当前字的偏移量段(`current_offset`)，使其成为该字段对齐量的倍数。
    // 对于第一个字段，此值始终为零。
    // 跳过的字节称为填充字节。
    current_offset += padding_needed_for(current_offset, field.alignment);

    struct[field].offset = current_offset;

    current_offset += field.size;
}

struct.size = current_offset + padding_needed_for(current_offset, struct.alignment);
```

<div class="warning">

警告:这个伪代码使用了一个简单粗暴的算法，是为了清晰起见，它忽略了溢出问题。要在实际代码中执行内存布局计算，请使用 [`Layout`]。

</div>

> 注意：此算法可以生成零尺寸的结构体。在 C 语言中，像 `struct Foo { }` 这样的空结构体声明是非法的。然而，gcc 和 clang 都支持启用此类结构体的选项，并将其尺寸指定为零。跟 Rust 不同的是 C++ 给空结构体指定的尺寸为 1，并且除非它们是继承的，否则它们是具有 `[[no_unique_address]]` 属性的字段（在这种情况下，它们不会增大结构体的整体尺寸）。

#### `#[repr(C)]`联合体

使用 `#[repr(C)]` 声明的联合体将与相同目标平台上的 C语言中的 C联合体声明具有相同的尺寸和对齐量。联合体的对齐量等同于其所有字段的最大对齐量，尺寸将为其所有字段的最大尺寸，再对其向上取整到对齐量的最小整数倍。这些最大值可能来自不同的字段。

```rust
#[repr(C)]
union Union {
    f1: u16,
    f2: [u8; 4],
}

assert_eq!(std::mem::size_of::<Union>(), 4);  // 来自于 f2
assert_eq!(std::mem::align_of::<Union>(), 2); // 来自于 f1

#[repr(C)]
union SizeRoundedUp {
   a: u32,
   b: [u16; 5],
}

assert_eq!(std::mem::align_of::<SizeRoundedUp>(), 4); // 来自于 a

assert_eq!(std::mem::size_of::<SizeRoundedUp>(), 12);  // 首先来自于b的尺寸10，然后向上取整到最近的4的整数倍12。

```

#### `#[repr(C)]`无字段枚举

对于[无字段枚举(field-less enums)][field-less enums]，`C`表形的尺寸和对齐量与目标平台的 C ABI 的默认枚举尺寸和对齐量相同。

> 注意：C中的枚举的表形是由枚举的相应实现定义的，所以在 Rust 中，给无字段枚举应用 C表形得到的表型很可能是一个“最佳猜测”。特别是，当使用某些特定命令行参数来编译特定的 C代码时，这可能是不正确的。

<div class="warning">

警告：C语言中的枚举与 Rust 中的那些应用了 `#[repr(C)]`表型的[无字段枚举][field-less enums]之间有着重要的区别。C语言中的枚举主要是 `typedef` 加上一些具名常量；换句话说，C枚举(`enum`)类型的对象可以包含任何整数值。例如，C枚举通常被用做标志位。相比之下，Rust的[无字段枚举][field-less enums]只能合法地[^译注2]保存判别式的值，其他的都是[未定义行为][undefined behavior]。因此，在 FFI 中使用无字段枚举来建模 C语言中的枚举(`enum`)通常是错误的。

</div>

#### `#[repr(C)]`带字段枚举

带字段的 `repr(C)`枚举的表形其实等效于一个带两个字段的 `repr(C)`结构体（这种在 C语言中也被称为“标签联合(tagged union)”），这两个字段：

- 一个为 `repr(C)`表形的枚举（在这个等效结构体内，它也被叫做标签(the tag)字段），它就是原枚举所有的判别值组合成的新枚举，也就是它的变体是原枚举变体移除了它们自身所带的所有字段。
- 一个为 `repr(C)`表形的联合体（在这个等效结构体内，它也被叫做载荷(the payload)字段），它的各个字段就是原枚举的各个变体把自己下面的字段重新组合成的 `repr(C)`表形的结构体。

> 注意：由于等效出的结构体和联合体是 `repr(C)`表形的，因此如果原来某一变体只有单个字段，则直接将该字段放入等效出的联合体中，或将其包装进一个次级结构体后再放入联合体中是没有区别的；因此，任何希望操作此类枚举表形的系统都可以选择使用这两种形式里对它们来说更方便或更一致的形式。

```rust
// 这个枚举的表形等效于 ...
#[repr(C)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ... 这个结构体
#[repr(C)]
struct MyEnumRepr {
    tag: MyEnumDiscriminant,
    payload: MyEnumFields,
}

// 这是原判别式组成的新枚举类型.
#[repr(C)]
enum MyEnumDiscriminant { A, B, C, D }

// 这是原变体的字段组成的联合体.
#[repr(C)]
union MyEnumFields {
    A: MyAFields,   // 译者注：因为原枚举变体A只有一个字段，所以此处的类型标注也可以直接替换为 u32,以省略 MyAFields这层封装
    B: MyBFields,
    C: MyCFields,
    D: MyDFields,
}

#[repr(C)]
#[derive(Copy, Clone)]
struct MyAFields(u32);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyBFields(f32, u64);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyCFields { x: u32, y: u8 }

// 这个结构体可以被省略(它是一个零尺寸类型)，但它必须出现在 C/C++ 头文件中
#[repr(C)]
#[derive(Copy, Clone)]
struct MyDFields;
```

> 注意： 联合体(`union`)可带有未实现 `Copy` 的字段的功能还没有纳入稳定版，具体参见 [55149]。

### 原语表形

*原语表形*是与原生整型具有相同名称的表形。也就是：`u8`，`u16`，`u32`，`u64`，`u128`，`usize`，`i8`，`i16`，`i32`，`i64`，`i128` 和 `isize`。

原语表形只能应用于枚举，此时枚举有没有字段会给原语表形带来不同的表现。给[零变体枚举][zero-variant enums]应用原始表形是错误的。将两个原语表形组合在一起也是错误的

#### 无字段枚举的原语表形

对于[无字段枚举][field-less enums]，原语表形将其尺寸和对齐量设置成与给定表形同名的原生类型的表形的值。例如，一个 `u8`表形的无字段枚举只能有0和255之间的判别值。

#### 带字段枚举的原语表形

带字段枚举的原语表形是一个 `repr(C)`表形的联合体，此联合体的每个字段对应一个和原枚举变体对应的 `repr(C)`表形的结构体。这些结构体的第一个字段是原枚举的变体移除了它们所有的字段组成的原语表形版的无字段枚举（“the tag”），那这些结构体的其余字段是原变体移走的字段。

> 注意：如果在联合体中，直接把标签的成员赋予给标签(“the tag”)，那么这种表形结构仍不变的，并且这样操作对您来说可能会更清晰（尽管遵循 c++ 的标准，标签也应该被包装在结构体中）。

```rust
// 这个枚举的表形效同于 ...
#[repr(u8)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ... 这个联合体.
#[repr(C)]
union MyEnumRepr {
    A: MyVariantA,  //译者注：此字段类型也可直接用 u32 直接替代
    B: MyVariantB,  //译者注：此字段类型也可直接用 (f32, u64) 直接替代
    C: MyVariantC,
    D: MyVariantD,
}

// 这是原判别值组合成的新枚举。
#[repr(u8)]
#[derive(Copy, Clone)]
enum MyEnumDiscriminant { A, B, C, D }

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantA(MyEnumDiscriminant, u32);

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantB(MyEnumDiscriminant, f32, u64);

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantC { tag: MyEnumDiscriminant, x: u32, y: u8 }

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantD(MyEnumDiscriminant);
```

> 注意： 联合体(`union`)带有未实现 `Copy` trait 的字段的功能还没有纳入稳定版，具体参见 [55149]。

#### 带字段枚举的原语表形与`#[repr(C)]`表形的组合使用

对于带字段枚举，还可以将 `repr(C)` 和原语表形（例如，`repr(C, u8)`）结合起来使用。这是通过将判别值组成的枚举的表形改为原语表形来实现的。因此，如果选择组合 `u8`表形，那么组合出的判别值枚举的尺寸和对齐量将为 1 个字节。

那么这个判别值枚举就从[前面][`repr(C)`]示例中的样子变成：

```rust
#[repr(C, u8)] // 这里加上了 `u8`
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ...

#[repr(u8)] // 所以这里就用 `u8` 替代了 `C`
enum MyEnumDiscriminant { A, B, C, D }

// ...
```

例如，对于有 `repr(C, u8)`属性的枚举，不可能有257个唯一的判别值（“tags”），而同一个枚举，如果只有单一 `repr(C)`表形属性，那在编译时就不会出任何问题。

在 `repr(C)` 附加原语表形可以改变 `repr(C)`表形的枚举的尺寸：

```rust
#[repr(C)]
enum EnumC {
    Variant0(u8),
    Variant1,
}

#[repr(C, u8)]
enum Enum8 {
    Variant0(u8),
    Variant1,
}

#[repr(C, u16)]
enum Enum16 {
    Variant0(u8),
    Variant1,
}

// C表形的尺寸依赖于平台
assert_eq!(std::mem::size_of::<EnumC>(), 8);
// 一个字节用于判别值，一个字节用于 Enum8::Variant0 中的值
assert_eq!(std::mem::size_of::<Enum8>(), 2);
// 两个字节用于判别值，一个字节用于Enum16::Variant0中的值，加上一个字节的填充
assert_eq!(std::mem::size_of::<Enum16>(), 4);
```

[`repr(C)`]: #reprc-enums-with-fields

### 对齐量的修饰符

`align` 和 `packed` 修饰符可分别用于增大和减小结构体的和联合体的对齐量。`packed` 也可以改变字段之间的填充。

对齐量被指定为整型参数，形式为 `#[repr(align(x))]` 或 `#[repr(packed(x))]`。对齐量的值必须是从1到2<sup>29</sup>之间的2的次幂数。对于 `packed`，如果没有给出任何值，如 `#[repr(packed)]`，则对齐量的值为1。

对于 `align`，如果类型指定的对齐量比其不带 `align`修饰符时的对齐量小，则该指定的对齐量无效。

对于 `packed`，如果类型指定的对齐量比其不带 `packed`修饰符时的对齐量大，则该指定的对齐量和布局无效。为了定位字段，每个字段的对齐量是指定的对齐量和字段的类型的对齐量中较小的那个对齐量。

`align` 和 `packed` 修饰符不能应用于同一类型，且 `packed` 修饰的类型不能直接或间接地包含另一个 `align` 修饰的类型。`align` 和 `packed` 修饰符只能应用于[默认表形][default]和 [C表形][`C`]中。

`align`修饰符也可以应用在枚举上。如果这样做了，其对枚举对齐量的影响与将此枚举包装在一个新的使用了相同的 `align`修饰符的结构体中的效果相同。

<div class="warning">

***警告：***解引用一个未对齐的指针是[未定义行为][undefined behavior]，但可以[安全地创建指向 `packed`修饰的字段的未对齐指针][27060]。就像在安全(safe) Rust 中所有创建未定义行为的方法一样，这是一个 bug。

</div>

### 透明(`transparent`)表形

透明(`transparent`)表型只能在只有一个字段的[结构体(`struct`)][structs]上或只有一个变体的[枚举(`enum`)][enumerations]上使用，这里只有一个字段/变体的意思是：

- 只能有一个非零尺寸的字段/变体，和
- 任意数量的尺寸为零对齐量为1的字段（例如：[`PhantomData<T>`]）

使用这种表形的结构体和枚举与只有那个非零尺寸的字段具有相同的布局和 ABI。

这与 `C`表形不同，因为带有 `C`表形的结构体将始终拥有 C结构体(`C` `struct`)的ABI，例如，那些只有一个原生类型字段的结构体如果应用了透明表形(`transparent`)，将具有此原生类型字段的ABI。

因为此表形将类型布局委托给另一种类型，所以它不能与任何其他表形一起使用。

[^译注1]: 至此，上一个字段就填充完成，开始计算本字段了。也就是说每一个字段的偏移量是其字段的段首位置；那第一个字段的偏移量就始终为 0。

[^译注2]: 这里合法的意思是变体的判别值受 `repr(u8)` 这样的表形属性约束，像这个例子中，变体的判别值就只能位于 0~255 之间。

[`align_of_val`]: https://doc.rust-lang.org/std/mem/fn.align_of_val.html
[`size_of_val`]: https://doc.rust-lang.org/std/mem/fn.size_of_val.html
[`align_of`]: https://doc.rust-lang.org/std/mem/fn.align_of.html
[`size_of`]: https://doc.rust-lang.org/std/mem/fn.size_of.html
[`Sized`]: https://doc.rust-lang.org/std/marker/trait.Sized.html
[`Copy`]: https://doc.rust-lang.org/std/marker/trait.Copy.html
[dynamically sized types]: dynamically-sized-types.md
[field-less enums]: items/enumerations.md#custom-discriminant-values-for-fieldless-enumerations
[enumerations]: items/enumerations.md
[zero-variant enums]: items/enumerations.md#zero-variant-enums
[undefined behavior]: behavior-considered-undefined.md
[27060]: https://github.com/rust-lang/rust/issues/27060
[55149]: https://github.com/rust-lang/rust/issues/55149
[`PhantomData<T>`]: special-types-and-traits.md#phantomdatat
[Default]: #the-default-representation
[`C`]: #the-c-representation
[primitive representations]: #primitive-representations
[structs]: items/structs.md
[`transparent`]: #the-transparent-representation
[`Layout`]: https://doc.rust-lang.org/std/alloc/struct.Layout.html

<!-- 2020-12-17-->
<!-- checked -->
