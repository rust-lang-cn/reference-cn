# 类型自动强转

>[type-coercions.md](https://github.com/rust-lang/reference/blob/master/src/type-coercions.md)\
>commit: d5a5e32d3cda8a297d2a91a85b91ff2629b0e896 \
>本章译文最后维护日期：2020-11-15

**类型自动强转**是改变值的类型的隐式操作。它们在特定的位置自动发生，但实际自动强转的类型也受到很多限制。

任何允许自动强转的转换都可以由[类型强制转换操作符][type cast operator] `as` 来显式执行。

自动强转最初是在 [RFC 401] 中定义的，并在[ RFC 1558] 中进行了扩展。

## 自动强转点

自动强转只能发生在程序中的某些自动强转点(coercion sites)上；典型的位置是那些所需的类型是显式给出了的地方，或者是那些可以从给出的显式类型传播推导(be derived by propagation)出所需的类型（注意这里不是类型推断）的地方。可能的强转点有：

* `let`语句中显式给出了类型。

   例如，下面例子中 `&mut 42` 自动强转成 `&i8` 类型：

   ```rust
   let _: &i8 = &mut 42; // 译者注释：`&i8` 是显示给出的所需类型
   ```

* 静态(`static`)项和常量(`const`)项声明（类似于 `let`语句）。

* 函数调用的参数

  被强制的值是实参(actual parameter)，它的类型被自动强转为形参(formal parameter)的类型。

  例如，下面例子中 `&mut 42` 自动强转成 `&i8` 类型：

  ```rust
  fn bar(_: &i8) { }

  fn main() {
      bar(&mut 42);
  }
  ```

  对于方法调用，接受者（`self`参数）只能使用[非固定尺寸类型自动强转(unsized coercion)](#unsized-coercions)。

* 实例化结构体、联合体或枚举变体的字段。

  例如，下面例子中 `&mut 42` 自动强转成 `&i8` 类型：

  ```rust
  struct Foo<'a> { x: &'a i8 }

  fn main() {
      Foo { x: &mut 42 };
  }
  ```
 
* 函数结果&mdash;块中的最终表达式或者 `return`语句中的任何表达式。

  例如，下面例子中 `x` 将自动强转成 `&dyn Display` 类型：

  ```rust
  use std::fmt::Display;
  fn foo(x: &u32) -> &dyn Display {
      x
  }
  ```

如果一在自动强转点中的表达式是自动强转传播型表达式(coercion-propagating expression)，那么该表达式中的对应子表达式也是自动强转点。传播从这些新的自动强转点开始递归。传播表达式(propagating expressions)及其相关子表达式有：

* 数组字面量，其数组的类型为 `[U; n]`。数组字面量中的每个子表达式都是自动强转到类型 `U` 的自动强转点。

* 重复句法声明的数组字面量，其数组的类型为 `[U; n]`。重复子表达式是用于自动强转到类型 `U` 的自动强转点。

* 元组，其中如果元组是自动强转到类型 `(U_0, U_1, ..., U_n)` 的强转点，则每个子表达式都是相应类型的自动强转点，比如第0个子表达式是到类型 `U_0`的 自动强转点。

* 圆括号括起来的子表达式（`(e)`）：如果整个括号表达式的类型为 `U`，则子表达式 `e` 是自动强转到类型 `U` 的自动强转点。

* 块：如果块的类型是 `U`，那么块中的最后一个表达式（如果它不是以分号结尾的）就是一个自动强转到类型 `U` 的自动强转点。这里的块包括作为控制流语句的一部分的条件分支代码块，比如 `if`/`else`，当然前提是这些块的返回需要有一个已知的类型。

## 自动强转类型

自动强转允许发生在下列类型之间：

* `T` 到 `U` 如果 `T` 是 `U` 的一个[子类型][subtype] (*反射性场景(reflexive case)*)

* `T_1` 到 `T_3` 当 `T_1` 可自动强转到 `T_2` 同时 `T_2` 又能自动强转到 `T_3` (*传递性场景(transitive case)*)

    注意这个还没有得到完全支持。

* `&mut T` 到 `&T`

* `*mut T` 到 `*const T`

* `&T` 到 `*const T`

* `&mut T` 到 `*mut T`

* `&T` 或 `&mut T` 到 `&U` 如果 `T` 实现了 `Deref<Target = U>`。例如：

  ```rust
  use std::ops::Deref;

  struct CharContainer {
      value: char,
  }

  impl Deref for CharContainer {
      type Target = char;

      fn deref<'a>(&'a self) -> &'a char {
          &self.value
      }
  }

  fn foo(arg: &char) {}

  fn main() {
      let x = &mut CharContainer { value: 'y' };
      foo(x); //&mut CharContainer 自动强转成 &char.
  }
  ```

* `&mut T` 到 `&mut U` 如果 `T` 实现了 `DerefMut<Target = U>`.

* TyCtor(`T`) 到 TyCtor(`U`)，其中 TyCtor(`T`) 是下列之一[^译注1]
    - `&T`
    - `&mut T`
    - `*const T`
    - `*mut T`
    - `Box<T>`

    并且 `U` 能够通过[非固定尺寸类型自动强转](#unsized-coercions)得到。

    <!--将来，coerce_inner 会被递归扩展到元组和结构体上。此外从子 trait 到父类trait 的自动强转也会被加进来，更多细节请参见 [RFC 401] 。-->

* 非捕获闭包(Non capturing closures)到函数指针(`fn` pointers)

* `!` 到任意 `T`

### 非固定尺寸类型自动强转

下列自动强转被称为非固定尺寸类型自动强转(`unsized coercions`)，因为它们与将固定尺寸类型(sized types)转换为非固定尺寸类型(unsized types)有关，并且在一些其他自动强转不允许的情况（也就是上面罗列的情况之外的情况）下允许使用。也就是说它们可以发生在任何自动强转发生的地方。

[`Unsize`] 和 [`CoerceUnsized`] 这两个 trait 被用来协助这种转换的发生，并公开给标准库来使用。以下自动强转方式是内置的，并且，如果 `T` 可以用其中任一方式自动强转成 `U`，那么就会为 `T` 提供一个 `Unsize<U>` 的内置实现：

* `[T; n]` 到 `[T]`.

* `T` 到 `dyn U`, 当 `T` 实现 `U + Sized`, 并且 `U` 是[对象安全的][object safe]时。

* `Foo<..., T, ...>` 到 `Foo<..., U, ...>`, 当：
    * `Foo` 是一个结构体。
    * `T` 实现了 `Unsize<U>`。
    * `Foo` 的最后一个字段是和 `T` 相关的类型。
    * 如果这最后一个字段是类型 `Bar<T>`，那么 `Bar<T>` 实现了 `Unsized<Bar<U>>`。
    * `T` 不是任何其他字段的类型的一部分。

此外，当 `T` 实现了 `Unsize<U>` 或 `CoerceUnsized<Foo<U>>` 时，类型 `Foo<T>` 可以实现 `CoerceUnsized<Foo<U>>`。这就允许 `Foo<T>` 提供一个到 `Foo<U>` 的非固定尺寸类型自动强转。

> 注：虽然非固定尺寸类型自动强转的定义及其实现已经稳定下来，但 [`Unsize`] 和 [`CoerceUnsized`] 这两个 trait 本身还没稳定下来，因此还不能直接用于稳定版的 Rust。

## 最小上界自动强转

在某些上下文中，编译器必须将多个类型强制在一起，以尝试找到最通用的类型。这被称为“最小上界(Least Upper Bound,简称 LUB)”自动强转。LUB自动强转只在以下情况中使用：

+ 为一系列的 if分支查找共同的类型。
+ 为一系列的匹配臂查找共同的类型。
+ 为数组元素查找共同的类型。
+ 为带有多个返回项语句的闭包的返回类型查找共同的类型。
+ 检查带有多个返回语句的函数的返回类型。

在这每种情况下，都有一组类型 `T0..Tn` 被共同自动强转到某个未知的目标类型 `T_t`，注意开始时 `T_t` 是未知的。LUB 自动强转的计算过程是不断迭代的。首先把目标类型 `T_t` 定为从类型 `T0` 开始。对于每一种新类型 `Ti`，考虑如下步骤是否成立：

+ 如果 `Ti` 可以自动强转为当前目标类型 `T_t`，则不做任何更改。 
+ 否则，检查 `T_t` 是否可以被自动强转为 `Ti`；如果是这样，`T_t` 就改为 `Ti`。（此检查还取决于到目前为止所考虑的所有源表达式是否带有隐式自动强转。）
+ 如果不是，尝试计算一个 `T_t` 和 `Ti` 的共同的超类型(supertype)，此超类型将成为新的目标类型。

### 示例：

```rust
# let (a, b, c) = (0, 1, 2);
// if分支的情况
let bar = if true {
    a
} else if false {
    b
} else {
    c
};

// 匹配臂的情况
let baw = match 42 {
    0 => a,
    1 => b,
    _ => c,
};

// 数组元素的情况
let bax = [a, b, c];

// 多个返回项语句的闭包的情况
let clo = || {
    if true {
        a
    } else if false {
        b
    } else {
        c
    }
};
let baz = clo();

// 检查带有多个返回语句的函数的情况
fn foo() -> i32 {
    let (a, b, c) = (0, 1, 2);
    match 42 {
        0 => a,
        1 => b,
        _ => c,
    }
}
```

在这些例子中，`ba*` 的类型可以通过 LUB自动强转找到。编译器检查 LUB自动强转在处理函数 `foo` 时，是否把 `a`，`b`，`c` 的结果转为了 `i32`。

### 附加说明

我们这种描述显然是非正式的，但目前使文字描述更精确的工作正作为精细化 Rust 类型检查器的一般性工作的一部分正紧锣密鼓的进行中。

[^译注1]: TyCtor为类型构造器 type constructor 的简写。

[RFC 401]: https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md
[RFC 1558]: https://github.com/rust-lang/rfcs/blob/master/text/1558-closure-to-fn-coercion.md
[subtype]: subtyping.md
[object safe]: items/traits.md#object-safety
[type cast operator]: expressions/operator-expr.md#type-cast-expressions
[`Unsize`]: https://doc.rust-lang.org/std/marker/trait.Unsize.html
[`CoerceUnsized`]: https://doc.rust-lang.org/std/ops/trait.CoerceUnsized.html

<!-- 2020-11-12-->
<!-- checked -->
