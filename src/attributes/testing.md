# 测试类属性

>[testing.md](https://github.com/rust-lang/reference/blob/master/src/attributes/testing.md)\
>commit: b0e0ad6490d6517c19546b1023948986578fc378 \
>本章译文最后维护日期：2020-11-10

以下[属性][attributes]用于指定函数来执行测试。在“测试(test)”模式下编译 crate 可以构建测试函数以及构建用于执行测试（函数）的测试套件(test harness)。启用测试模式还会启用 [`test`条件编译选项][`test` conditional compilation option]。

## `test`属性

*`test`属性*标记一个用来执行测试的函数。这些函数只在测试模式下编译。测试函数必须是自由函数和单态函数，不能有参数，返回类型必须是以下类型之一：

* `()`
* `Result<(), E> where E: Error`
<!-- * `!` -->
<!-- * Result<!, E> where E: Error` -->

> 注意：允许哪些返回类型是由暂未稳定的 [`Termination`] trait 决定的。

<!-- 如果前面这节需要更新(从 "不能有参数" 开始, 同时需要修改 crates-and-source-files.md 文件 -->

> 注意：测试模式是通过将 `--test` 参数选项传递给 `rustc` 或使用 `cargo test` 来启用的。

返回 `()` 的测试只要结束(terminate)且没有触发 panic 就会通过。返回 `Result<(), E>` 的测试只要它们返回 `Ok(())` 就算通过。不结束的测试既不（计为）通过也不（计为）失败。

```rust
# use std::io;
# fn setup_the_thing() -> io::Result<i32> { Ok(1) }
# fn do_the_thing(s: &i32) -> io::Result<()> { Ok(()) }
#[test]
fn test_the_thing() -> io::Result<()> {
    let state = setup_the_thing()?; // 预期成功
    do_the_thing(&state)?;          // 预期成功
    Ok(())
}
```

## `ignore`属性

被 `test`属性标注的(annotated with)函数也可以被 `ignore`属性标注。*`ignore`属性*告诉测试套件不要将该函数作为测试执行。但在测试模式下，这类函数仍然会被编译。

`ignore`属性可以选择使用 [_MetaNameValueStr_]元项属性句法来说明测试被忽略的原因。

```rust
#[test]
#[ignore = "not yet implemented"]
fn mytest() {
    // …
}
```

> **注意**：`rustc` 的测试套件支持使用 `--include-ignored` 参数选项来强制运行那些被忽略测试的函数。

## `should_panic`属性

被 `test`属性标注并返回 `()` 的函数也可以被 `should_panic`属性标注。*`should_panic`属性*使测试函数只有在实际发生 panic 时才算通过。

`should_panic`属性可选输入一条出现在 panic消息中的字符串。如果在 panic消息中找不到该字符串，则测试将失败。可以使用 [_MetaNameValueStr_]元项属性句法或带有 `expected`字段的 [_MetaListNameValueStr_]元项属性句法来传递字符串。

```rust
#[test]
#[should_panic(expected = "值未匹配上")]
fn mytest() {
    assert_eq!(1, 2, "值未匹配上");
}
```

[_MetaListNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[_MetaNameValueStr_]: ../attributes.md#meta-item-attribute-syntax
[`Termination`]: https://doc.rust-lang.org/std/process/trait.Termination.html
[`test` conditional compilation option]: ../conditional-compilation.md#test
[attributes]: ../attributes.md

<!-- 2020-11-12-->
<!-- checked -->

