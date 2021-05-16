# 生命周期(类型参数)省略

>[lifetime-elision.md](https://github.com/rust-lang/reference/blob/master/src/lifetime-elision.md)\
>commit: f8e76ee9368f498f7f044c719de68c7d95da9972 \
>本章译文最后维护日期：2020-11-16

Rust 拥有一套允许在多种位置省略生命周期的规则，但前提是编译器在这些位置上能推断出合理的默认生命周期。

## 函数上的生命周期(类型参数)省略

为了使常用模式使用起来更方便，可以在[函数项][function item]、[函数指针][function pointer]和[闭包trait][closure trait][^译注1]的签名中*省略*生命周期类型参数。以下规则用于推断出被省略的生命周期类型参数。省略不能被推断出的生命周期类型参数是错误的。占位符形式的生命周期，`'_`，也可以用这一套规则来推断出。对于路径中的生命周期，首选使用 `'_`。trait对象的生命周期类型参数遵循不同的规则，具体[这里](#default-trait-object-lifetimes)讨论。

* 参数中省略的每个生命周期类型参数都会（被推断）成为一个独立的生命周期类型参数。
* 如果参数中只使用了一个生命周期（省略或不省略都行），则将该生命周期作为*所有*省略的输出生命周期类型参数。

在方法签名中有另一条规则

* 如果接受者(receiver)类型为 `&Self` 或 `&mut Self`，那么对 `Self` 的引用的生命周期会被作为所有省略的输出生命周期类型参数。

示例：

```rust
# trait T {}
# trait ToCStr {}
# struct Thing<'a> {f: &'a i32}
# struct Command;
#
# trait Example {
fn print1(s: &str);                                   // 省略
fn print2(s: &'_ str);                                // 也省略
fn print3<'a>(s: &'a str);                            // 未省略

fn debug1(lvl: usize, s: &str);                       // 省略
fn debug2<'a>(lvl: usize, s: &'a str);                // 未省略

fn substr1(s: &str, until: usize) -> &str;            // 省略
fn substr2<'a>(s: &'a str, until: usize) -> &'a str;  // 未省略

fn get_mut1(&mut self) -> &mut dyn T;                 // 省略
fn get_mut2<'a>(&'a mut self) -> &'a mut dyn T;       // 未省略

fn args1<T: ToCStr>(&mut self, args: &[T]) -> &mut Command;                  // 省略
fn args2<'a, 'b, T: ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command; // 未省略

fn new1(buf: &mut [u8]) -> Thing<'_>;                 // 省略 - 首选的
fn new2(buf: &mut [u8]) -> Thing;                     // 省略
fn new3<'a>(buf: &'a mut [u8]) -> Thing<'a>;          // 未省略
# }

type FunPtr1 = fn(&str) -> &str;                      // 省略
type FunPtr2 = for<'a> fn(&'a str) -> &'a str;        // 未省略

type FunTrait1 = dyn Fn(&str) -> &str;                // 省略
type FunTrait2 = dyn for<'a> Fn(&'a str) -> &'a str;  // 未省略
```

```rust,compile_fail
// 下面的示例展示了不允许省略生命周期类型参数的情况。

# trait Example {
// 无法推断，因为没有可以推断的起始参数。
fn get_str() -> &str;                                 // 非法

// 无法推断，这里无法确认输出的生命周期类型参数该遵从从第一个还是第二个参数的。
fn frob(s: &str, t: &str) -> &str;                    // 非法
# }
```

## 默认的 trait对象的生命周期

假定存在于（代表） [trait对象][trait object]（的那个胖指针）上的生命周期(assumed lifetime)类型参数称为此 trait对象的*默认对象生命周期约束(default object lifetime bound)*。这些在 [RFC 599] 中定义，在 [RFC 1156] 中修定增补。

当 trait对象的生命周期约束被完全省略时，会使用默认对象生命周期约束来替代上面定义的生命周期类型参数省略规则。但如果使用 `'_` 作为生命周期约束，则该约束仍遵循上面通常的省略规则。

如果将 trait对象用作泛型类型的类型参数，则首先使用此容器泛型来尝试为此 trait对象推断一个约束（来替代那个假定的生命周期）。

* 如果存在来自此容器泛型的唯一约束，则该约束就为此 trait对象的默认约束
* 如果此容器泛型有多个约束，则必须指定一个约显式束为此 trait对象的默认约束

如果这两个规则都不适用，则使用该 trait对象的声明时的 trait约束：

* 如果原 trait 声明为单生命周期*约束*，则此 trait对象使用该约束作为默认约束。
* 如果 `'static` 被用做原 trait声明的任一一个生命周期约束，则此 trait对象使用 `'static` 作为默认约束。
* 如果原 trait声明没有生命周期约束，那么此 trait对象的生命周期会在表达式中根据上下文被推断出来，在表达式之外直接用 `'static`。

```rust
// 对下面的 trait 来说，...
trait Foo { }

// 这两个是等价的，就如 `Box<T>` 对 `T` 没有生命周期约束一样
// These two are the same as Box<T> has no lifetime bound on T
type T1 = Box<dyn Foo>;  //译者注：此处的 `T1` 和 上行备注中提到的 `Box<T>` 都是本节规则中所说的泛型类型，即容器泛型
type T2 = Box<dyn Foo + 'static>;

// ...这也是等价的
impl dyn Foo {}
impl dyn Foo + 'static {}

// ...这也是等价的, 因为 `&'a T` 需要 `T: 'a`
type T3<'a> = &'a dyn Foo;
type T4<'a> = &'a (dyn Foo + 'a);

// `std::cell::Ref<'a, T>` 也需要 `T: 'a`, 所以这俩也是等价的
type T5<'a> = std::cell::Ref<'a, dyn Foo>;
type T6<'a> = std::cell::Ref<'a, dyn Foo + 'a>;
```

```rust,compile_fail
// 这是一个反面示例：
# trait Foo { }
struct TwoBounds<'a, 'b, T: ?Sized + 'a + 'b> {
    f1: &'a i32,
    f2: &'b i32,
    f3: T,
}
type T7<'a, 'b> = TwoBounds<'a, 'b, dyn Foo>;
//                                  ^^^^^^^
// 错误: 不能从上下文推导出此对象类型的生命周期约束
```

注意，像 `&'a Box<dyn Foo>` 这样多层包装的，只需要看最内层包装 `dyn Foo` 的那层，所以扩展后仍然为 `&'a Box<dyn Foo + 'static>`

```rust
// 对下面的 trait 来说，...
trait Bar<'a>: 'a { }

// ...这两个是等价的：
type T1<'a> = Box<dyn Bar<'a>>;
type T2<'a> = Box<dyn Bar<'a> + 'a>;

// ...这俩也是等价的:
impl<'a> dyn Bar<'a> {}
impl<'a> dyn Bar<'a> + 'a {}
```

## 静态(`'static`)生命周期省略

除非指定了显式的生命周期，引用类型的[常量项][constant]声明和[静态项][static]声明都具有*隐式的*静态(`'static`)生命周期。因此，有 `'static`生命周期的常量项声明在编写时可以略去其生命周期。

```rust
// STRING: &'static str
const STRING: &str = "bitstring";

struct BitsNStrings<'a> {
    mybits: [u32; 2],
    mystring: &'a str,
}

// BITS_N_STRINGS: BitsNStrings<'static>
const BITS_N_STRINGS: BitsNStrings<'_> = BitsNStrings {
    mybits: [1, 2],
    mystring: STRING,
};
```

注意，如果静态项(`static`)或常量项(`const`)包含对函数或闭包的引用，而这些函数或闭包本身也包含引用，此时编译器将首先尝试使用标准的省略规则来推断生命周期类型参数。如果它不能通过通常的生命周期省略规则来推断出生命周期类型参数，那么它将报错。举个例子：

```rust
# struct Foo;
# struct Bar;
# struct Baz;
# fn somefunc(a: &Foo, b: &Bar, c: &Baz) -> usize {42}
// 解析为 `fn<'a>(&'a str) -> &'a str`.
const RESOLVED_SINGLE: fn(&str) -> &str = |x| x;

// 解析为 `Fn<'a, 'b, 'c>(&'a Foo, &'b Bar, &'c Baz) -> usize`.
const RESOLVED_MULTIPLE: &dyn Fn(&Foo, &Bar, &Baz) -> usize = &somefunc;
```

```rust,compile_fail
# struct Foo;
# struct Bar;
# struct Baz;
# fn somefunc<'a,'b>(a: &'a Foo, b: &'b Bar) -> &'a Baz {unimplemented!()}
// 没有足够的信息将返回值的生命周期与参数的生命周期绑定起来，因此这是一个错误
const RESOLVED_STATIC: &dyn Fn(&Foo, &Bar) -> &Baz = &somefunc;
//                                            ^
// 这个函数的返回类型包含一个借用来的值，但是签名没有说明它是从参数1还是从参数2借用来的
```

[^译注1]: 指 Fn、FnMute 和 FnOnce 这三个 trait。

[closure trait]: types/closure.md
[constant]: items/constant-items.md
[function item]: types/function-item.md
[function pointer]: types/function-pointer.md
[RFC 599]: https://github.com/rust-lang/rfcs/blob/master/text/0599-default-object-bound.md
[RFC 1156]: https://github.com/rust-lang/rfcs/blob/master/text/1156-adjust-default-object-bounds.md
[static]: items/static-items.md
[trait object]: types/trait-object.md

<!-- 2020-11-12-->
<!-- checked -->
