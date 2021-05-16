# 结构体

>[structs.md](https://github.com/rust-lang/reference/blob/master/src/items/structs.md)\
>commit: 9d0aa172932ed15ec1b13556e6809b74bc58a02b \
>本章译文最后维护日期：2021-1-17

> **<sup>句法</sup>**\
> _Struct_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _StructStruct_\
> &nbsp;&nbsp; | _TupleStruct_
>
> _StructStruct_ :\
> &nbsp;&nbsp; `struct`
>   [IDENTIFIER]&nbsp;
>   [_GenericParams_]<sup>?</sup>
>   [_WhereClause_]<sup>?</sup>
>   ( `{` _StructFields_<sup>?</sup> `}` | `;` )
>
> _TupleStruct_ :\
> &nbsp;&nbsp; `struct`
>   [IDENTIFIER]&nbsp;
>   [_GenericParams_]<sup>?</sup>
>   `(` _TupleFields_<sup>?</sup> `)`
>   [_WhereClause_]<sup>?</sup>
>   `;`
>
> _StructFields_ :\
> &nbsp;&nbsp; _StructField_ (`,` _StructField_)<sup>\*</sup> `,`<sup>?</sup>
>
> _StructField_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; [IDENTIFIER] `:` [_Type_]
>
> _TupleFields_ :\
> &nbsp;&nbsp; _TupleField_ (`,` _TupleField_)<sup>\*</sup> `,`<sup>?</sup>
>
> _TupleField_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup>\
> &nbsp;&nbsp; [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; [_Type_]

*结构体*是一个使用关键字 `struct` 定义的标称型(nominal)[结构体类型][struct type]。

结构体(`struct`)程序项的一个示例和它的使用方法：

```rust
struct Point {x: i32, y: i32}
let p = Point {x: 10, y: 11};
let px: i32 = p.x;
```

*元组结构体*是一个标称型(nominal)[元组类型][tuple type]，也是用关键字 `struct` 定义的。例如：

[struct type]: ../types/struct.md
[tuple type]: ../types/tuple.md

```rust
struct Point(i32, i32);
let p = Point(10, 11);
let px: i32 = match p { Point(x, _) => x };
```

*单元结构体(unit-like struct)*是没有任何字段的结构体，它的定义完全不包含字段(fields)列表。这样的结构体隐式定义了其类型的同名常量（值）。例如:

```rust
struct Cookie;
let c = [Cookie, Cookie {}, Cookie, Cookie {}];
```

等价于

```rust
struct Cookie {}
const Cookie: Cookie = Cookie {};
let c = [Cookie, Cookie {}, Cookie, Cookie {}];
```

结构体的精确内存布局还没有规范下来。目前可以使用 [`repr`属性][`repr` attribute]来指定特定的布局。

[`repr` attribute]: ../type-layout.md#representations

[_OuterAttribute_]: ../attributes.md
[IDENTIFIER]: ../identifiers.md
[_GenericParams_]: generics.md
[_WhereClause_]: generics.md#where-clauses
[_Visibility_]: ../visibility-and-privacy.md
[_Type_]: ../types.md#type-expressions

<!-- 2020-11-12-->
<!-- checked -->
