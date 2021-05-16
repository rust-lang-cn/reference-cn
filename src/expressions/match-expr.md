# 匹配(`match`)表达式

>[match-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/match-expr.md)\
>commit: d23f9da8469617e6c81121d9fd123443df70595d \
>本章译文最后维护日期：2021-5-6

> **<sup>句法</sup>**\
> _MatchExpression_ :\
> &nbsp;&nbsp; `match` [_Expression_]<sub>_排除结构体表达式_</sub> `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; [_InnerAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; &nbsp;&nbsp; _MatchArms_<sup>?</sup>\
> &nbsp;&nbsp; `}`
>
> _MatchArms_ :\
> &nbsp;&nbsp; ( _MatchArm_ `=>`
>                             ( [_ExpressionWithoutBlock_][_Expression_] `,`
>                             | [_ExpressionWithBlock_][_Expression_] `,`<sup>?</sup> )
>                           )<sup>\*</sup>\
> &nbsp;&nbsp; _MatchArm_ `=>` [_Expression_] `,`<sup>?</sup>
>
> _MatchArm_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> [_Pattern_] _MatchArmGuard_<sup>?</sup>
> 
> _MatchArmGuard_ :\
> &nbsp;&nbsp; `if` [_Expression_]

*匹配(`match`)表达式*在模式(pattern)上建立代码逻辑分支(branch)。
匹配的确切形式取决于其应用的[模式][pattern]。
一个匹配(`match`)表达式带有一个要与模式进行比较的 *[检验对象][scrutinee](scrutinee)表达式*。
检验对象表达式和模式必须具有相同的类型。

根据检验对象表达式是[位置表达式或值表达式][place expression]，匹配(`match`)的行为表现会有所不同。
如果检验对象表达式是一个[值表达式][value expression]，则这个表达式首先会在被求值到一个临时内存位置，然后将这个结果值按顺序与匹配臂(arms)中的模式进行比较，直到找到一个成功的匹配。
第一个匹配成功的模式所在的匹配臂会被选中为当前匹配(`match`)的分支目标，然后以该模式绑定的变量为中介，（把它从检验对象那里匹配到的变量值）转赋值给该匹配臂的块中的局部变量，然后控制流进入该块。

当检验对象表达式是一个[位置表达式][place expression]时，此匹配(`match`)表达式不用先去内存上分配一个临时位置；但是，按值匹配的绑定方式(by-value binding)会复制或移动这个（位置表达式代表的）内存位置里面的值。
如果可能，最好还是在位置表达式上进行匹配，因为这种匹配的生存期继承了该位置表达式的生存期，而不会（让其生存期仅）局限于此匹配的内部。

匹配(`match`)表达式的一个示例：

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    4 => println!("four"),
    5 => println!("five"),
    _ => println!("something else"),
}
```

模式中绑定到的变量的作用域可以覆盖到匹配守卫(match guard)和匹配臂的表达式里。
[变量绑定方式][binding mode]（移动、复制或引用）取决于使用的具体模式。

可以使用操作符 `|` 连接多个匹配模式。
每个模式将按照从左到右的顺序进行测试，直到找到一个成功的匹配。

```rust
let x = 9;
let message = match x {
    0 | 1  => "not many",
    2 ..= 9 => "a few",
    _      => "lots"
};

assert_eq!(message, "a few");

// 演示模式匹配顺序。
struct S(i32, i32);

match S(1, 2) {
    S(z @ 1, _) | S(_, z @ 2) => assert_eq!(z, 1),
    _ => panic!(),
}
```

> 注意: `2..=9` 是一个[区间(Range)模式][Range Pattern]，不是一个[区间表达式][Range Expression]。
> 因此，只有区间模式支持的区间类型才能在匹配臂中使用。

每个 `|` 分隔的模式里出现的变量绑定必须出现在匹配臂的所有模式里。
相同名称的绑定变量必须具有相同的类型和相同的变量绑定模式。

## 匹配守卫

匹配臂可以接受*匹配守卫(Pattern guard)*来进一步改进匹配标准。
模式守卫出现在模式的后面，由关键字 `if` 后面的布尔类型表达式组成。

当模式匹配成功时，将执行匹配守卫表达式。
如果此表达式的计算结果为真，则此模式将进一步被确认为匹配成功。
否则，匹配将测试下一个模式，包括测试同一匹配臂中运算符 `|` 分割的后续匹配模式。

```rust
# let maybe_digit = Some(0);
# fn process_digit(i: i32) { }
# fn process_other(i: i32) { }
let message = match maybe_digit {
    Some(x) if x < 10 => process_digit(x),
    Some(x) => process_other(x),
    None => panic!(),
};
```

> 注意：使用操作符 `|` 的多次匹配可能会导致后跟的匹配守卫必须多次执行的副作用。
> 例如：
>
> ```rust
> # use std::cell::Cell;
> let i : Cell<i32> = Cell::new(0);
> match 1 {
>     1 | _ if { i.set(i.get() + 1); false } => {}
>     _ => {}
> }
> assert_eq!(i.get(), 2);
> ```

匹配守卫可以引用绑定在它们前面的模式里的变量。
在对匹配守卫进行计算之前，将对检验对象内部被模式的变量匹配上的那部分进行共享引用。
在对匹配守卫进行计算时，访问守卫里的这些变量就会使用这个共享引用。只有当匹配守卫最终计算为真时，此共享引用的值才会从检验对象内部移动或复制到相应的匹配臂的变量中。
这使得共享借用可以在守卫内部使用，还不会在守卫不匹配的情况下将值移出检验对象。
此外，通过在计算匹配守卫的同时持有共享引用，也可以防止匹配守卫内部意外修改检验对象。

## 匹配臂上的属性

在匹配臂上允许使用外部属性，但在匹配臂上只有 [`cfg`]、[`cold`] 和 [lint检查类属性][lint check attributes]这些属性才有意义。

在允许[块表达式上的属性][Inner attributes]存在的那几种表达式上下文中，可以在匹配表达式的左括号后直接使用[内部属性][attributes on block expressions]。

[_Expression_]: ../expressions.md
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[value expression]: ../expressions.md#place-expressions-and-value-expressions
[_InnerAttribute_]: ../attributes.md
[_OuterAttribute_]: ../attributes.md
[`cfg`]: ../conditional-compilation.md
[`cold`]: ../attributes/codegen.md#the-cold-attribute
[lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[Range Expression]: range-expr.md

[_Pattern_]: ../patterns.md
[pattern]: ../patterns.md
[Inner attributes]: ../attributes.md
[Range Pattern]: ../patterns.md#range-patterns
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions
[binding mode]: ../patterns.md#binding-modes
[scrutinee]: ../glossary.md#scrutinee
