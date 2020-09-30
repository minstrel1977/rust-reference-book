# Patterns
# 模式

>[patterns.md](https://github.com/rust-lang/reference/blob/master/src/patterns.md)\
>commit 589c2163d018b408da173325e02c7b59c139c3d1

> **<sup>句法</sup>**\
> _Pattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _PatternWithoutRange_\
> &nbsp;&nbsp; | [_RangePattern_]
>
> _PatternWithoutRange_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_LiteralPattern_]\
> &nbsp;&nbsp; | [_IdentifierPattern_]\
> &nbsp;&nbsp; | [_WildcardPattern_]\
> &nbsp;&nbsp; | [_RestPattern_]\
> &nbsp;&nbsp; | [_ObsoleteRangePattern_]\
> &nbsp;&nbsp; | [_ReferencePattern_]\
> &nbsp;&nbsp; | [_StructPattern_]\
> &nbsp;&nbsp; | [_TupleStructPattern_]\
> &nbsp;&nbsp; | [_TuplePattern_]\
> &nbsp;&nbsp; | [_GroupedPattern_]\
> &nbsp;&nbsp; | [_SlicePattern_]\
> &nbsp;&nbsp; | [_PathPattern_]\
> &nbsp;&nbsp; | [_MacroInvocation_]

模式用于根据给定结构去匹配值，并可选地将变量和这些结构中匹配到的值相互绑定。模式还用在函数和闭包的变量声明和参数中。

下面示例中的模式完成四件事：

* 测试 `person` 是否在 `car`字段中填充了内容。
* 测试变量 `person` 的 `age` 字段(的值)是否在13到19之间，并将其值绑定到变量 `person_age` 上。
* 将对 `name` 字段的引用绑定到变量 `person_name` 上。
* 忽略 `person` 的其余字段。其余字段可以有任何值，并且不绑定到任何变量。

```rust
# struct Car;
# struct Computer;
# struct Person {
#     name: String,
#     car: Option<Car>,
#     computer: Option<Computer>,
#     age: u8,
# }
# let person = Person {
#     name: String::from("John"),
#     car: Some(Car),
#     computer: None,
#     age: 15,
# };
if let
    Person {
        car: Some(_),
        age: person_age @ 13..=19,
        name: ref person_name,
        ..
    } = person
{
    println!("{} has a car and is {} years old.", person_name, person_age);
}
```

模式用于：

* [`let`声明](statements.md#let语句)
* [函数](items/functions.md)和[闭包](expressions/closure-expr.md)的参数。
* [匹配(`match`)表达式](expressions/match-expr.md)
* [`if let`表达式](expressions/if-expr.md)
* [`while let`表达式](expressions/loop-expr.md#predicate-pattern-loops)
* [`for`表达式](expressions/loop-expr.md#iterator-loops)

## Destructuring
## 解构

模式可用于*解构*[结构体][structs]、[枚举][enums]和[元组][tuples]。解构将一个值分解成它的组成部分。使用的句法与创建此类值时的几乎相同。在[检验对象][scrutinee]表达式具有结构体(`struct`)、枚举(`enum`)或元组(`tuple`)类型的模式中，占位符(`_`) 代表*单个*数据字段，而通配符`..` 代表特定变量(variant)的*所有*剩余字段。当使用字段的名称(而不是编号)来解构数据结构时，允许将 `fieldname` 写作 `fieldname: fieldname`的简写形式。

```rust
# enum Message {
#     Quit,
#     WriteString(String),
#     Move { x: i32, y: i32 },
#     ChangeColor(u8, u8, u8),
# }
# let message = Message::Quit;
match message {
    Message::Quit => println!("Quit"),
    Message::WriteString(write) => println!("{}", &write),
    Message::Move{ x, y: 0 } => println!("move {} horizontally", x),
    Message::Move{ .. } => println!("other move"),
    Message::ChangeColor { 0: red, 1: green, 2: _ } => {
        println!("color change, red: {}, green: {}", red, green);
    }
};
```

## Refutability
## 可反驳性

当一个模式有可能与它所匹配的值不匹配时，我们就说它是*可反驳型的(refutable)*。另一方面，*不可反驳型的(irrefutable)*模式总是与它们所匹配的值相匹配。例如：

```rust
let (x, y) = (1, 2);               // "(x, y)" 是一个不可反驳型模式

if let (a, 3) = (1, 2) {           // "(a, 3)" 是可反驳型的, 将不会匹配
    panic!("Shouldn't reach here");
} else if let (a, 4) = (3, 4) {    // "(a, 4)" 是可反驳型的, 将会匹配
    println!("Matched ({}, 4)", a);
}
```

## Literal patterns
## 字面量模式

> **<sup>句法</sup>**\
> _LiteralPattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [BOOLEAN_LITERAL]\
> &nbsp;&nbsp; | [CHAR_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | [STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_STRING_LITERAL]\
> &nbsp;&nbsp; | [BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [INTEGER_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [FLOAT_LITERAL]

[BOOLEAN_LITERAL]: tokens.md#boolean-literals
[CHAR_LITERAL]: tokens.md#character-literals
[BYTE_LITERAL]: tokens.md#byte-literals
[STRING_LITERAL]: tokens.md#string-literals
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[BYTE_STRING_LITERAL]: tokens.md#byte-string-literals
[RAW_BYTE_STRING_LITERAL]: tokens.md#raw-byte-string-literals
[INTEGER_LITERAL]: tokens.md#integer-literals
[FLOAT_LITERAL]: tokens.md#floating-point-literals

*字面量模式*匹配的值与字面量所创建的值完全相同。由于负数不是[字面量][literals]，字面量模式也接受字面量前的可选负号，它的作用类似于否定运算符。

<div class="warning">

浮点字面量目前还可以使用，但是由于它们在数值比较时带来的复杂性，在将来的 Rust 版本中，它们将被禁止用于字面量模式(参见 [issue #41620](https://github.com/rust-lang/rust/issues/41620))。

</div>

字面量模式总是可以反驳型的。

例如：

```rust
for i in -2..5 {
    match i {
        -1 => println!("It's minus one"),
        1 => println!("It's a one"),
        2|4 => println!("It's either a two or a four"),
        _ => println!("Matched none of the arms"),
    }
}
```

## Identifier patterns
## 标识符模式

> **<sup>句法</sup>**\
> _IdentifierPattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `ref`<sup>?</sup> `mut`<sup>?</sup> [IDENTIFIER] (`@` [_Pattern_] ) <sup>?</sup>

标识符模式将它们匹配的值绑定到一个变量上。标识符在模式中必须是唯一的。该变量将在作用域中遮蔽同名的任何变量。这种绑定的作用域取决于使用模式的上下文(例如 `let`绑定或匹配(`match`)的匹配臂)。

只包含标识符(也可能前带一个 `mut`)的模式能匹配任何值并将其绑定到该标识符。这是函数和闭包的变量声明和传参最常用的模式。

```rust
let mut variable = 10;
fn sum(x: i32, y: i32) -> i32 {
#    x + y
# }
```

要将模式的匹配值绑定到变量，可使用句法 `variable @ subpattern`。例如，下面示例中将值2绑定到 `e` (不是整个区间(range)：这里的区间是一个区间子模式(range subpattern))。

```rust
let x = 2;

match x {
    e @ 1 ..= 5 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

默认情况下，标识符模式里变量和匹配值绑定有两种方式，一种是变量和匹配值的副本绑定，一种是将匹配值移动到变量里来完成绑定，具体是拷贝还是移动取决于匹配值是否实现了 [`Copy`]。也可以通过使用 `ref` 关键字将变量和值的引用绑定，或者使用 `ref mut`将变量和值的可变引用绑定。例如：

```rust
# let a = Some(10);
match a {
    None => (),
    Some(value) => (),
}

match a {
    None => (),
    Some(ref value) => (),
}
```

在第一个匹配表达式中，值被复制(或移动)。在第二个匹配中，对相同内存位置的引用被绑定到变量上。之所以需要这种句法，是因为在解构子模式(destructuring subpatterns)中，`&`操作符不能应用于值的字段。例如，以下内容无效:

```rust,compile_fail
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person{ name: String::from("John"), age: 23 };
if let Person{name: &person_name, age: 18..=150} = value { }
```

要使其有效，请编写以下代码:

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person{ name: String::from("John"), age: 23 };
if let Person{name: ref person_name, age: 18..=150} = value { }
```

因此，`ref` 不是被匹配的某个实体。它的目标是使变量和匹配值的引用绑定起来，而不是潜在地复制或移动匹配的内容。

[路径模式(Path pattern)](#path-patterns)优先于标识符模式。如果 `ref` 或 `ref mut` 被指定，同时标识符遮蔽了某个常量，这将导致错误。
<!-- [Path patterns](#path-patterns) take precedence over identifier patterns. It is an error if `ref` or `ref mut` is specified and the identifier shadows a constant. TobeModify-->

如果 `@`子模式是不可反驳型的或子模式未指定，则标识符模式是不可反驳型的。

### Binding modes

To service better ergonomics, patterns operate in different *binding modes* in
order to make it easier to bind references to values. When a reference value is matched by
a non-reference pattern, it will be automatically treated as a `ref` or `ref mut` binding.
Example:

```rust
let x: &Option<i32> = &Some(3);
if let Some(y) = x {
    // y was converted to `ref y` and its type is &i32
}
```

*Non-reference patterns* include all patterns except bindings, [wildcard
patterns](#wildcard-pattern) (`_`), [`const` patterns](#path-patterns) of reference types,
and [reference patterns](#reference-patterns).

If a binding pattern does not explicitly have `ref`, `ref mut`, or `mut`, then it uses the
*default binding mode* to determine how the variable is bound. The default binding
mode starts in "move" mode which uses move semantics. When matching a pattern, the
compiler starts from the outside of the pattern and works inwards. Each time a reference
is matched using a non-reference pattern, it will automatically dereference the value and
update the default binding mode. References will set the default binding mode to `ref`.
Mutable references will set the mode to `ref mut` unless the mode is already `ref` in
which case it remains `ref`. If the automatically dereferenced value is still a reference,
it is dereferenced and this process repeats.

## Wildcard pattern

> **<sup>句法</sup>**\
> _WildcardPattern_ :\
> &nbsp;&nbsp; `_`

The _wildcard pattern_ (an underscore symbol) matches any value. It is used to ignore values when they don't
matter. Inside other patterns it matches a single data field (as opposed to the `..`
which matches the remaining fields). Unlike identifier patterns, it does not copy, move
or borrow the value it matches.

Examples:

```rust
# let x = 20;
let (a, _) = (10, x);   // the x is always matched by _
# assert_eq!(a, 10);

// ignore a function/closure param
let real_part = |a: f64, _: f64| { a };

// ignore a field from a struct
# struct RGBA {
#    r: f32,
#    g: f32,
#    b: f32,
#    a: f32,
# }
# let color = RGBA{r: 0.4, g: 0.1, b: 0.9, a: 0.5};
let RGBA{r: red, g: green, b: blue, a: _} = color;
# assert_eq!(color.r, red);
# assert_eq!(color.g, green);
# assert_eq!(color.b, blue);

// accept any Some, with any value
# let x = Some(10);
if let Some(_) = x {}
```

The wildcard pattern is always irrefutable.

## Rest patterns

> **<sup>句法</sup>**\
> _RestPattern_ :\
> &nbsp;&nbsp; `..`

The _rest pattern_ (the `..` token) acts as a variable-length pattern which
matches zero or more elements that haven't been matched already before and
after. It may only be used in [tuple](#tuple-patterns), [tuple
struct](#tuple-struct-patterns), and [slice](#slice-patterns) patterns, and
may only appear once as one of the elements in those patterns. It is also
allowed in an [identifier pattern](#identifier-patterns) for [slice
patterns](#slice-patterns) only.

The rest pattern is always irrefutable.

Examples:

```rust
# let words = vec!["a", "b", "c"];
# let slice = &words[..];
match slice {
    [] => println!("slice is empty"),
    [one] => println!("single element {}", one),
    [head, tail @ ..] => println!("head={} tail={:?}", head, tail),
}

match slice {
    // Ignore everything but the last element, which must be "!".
    [.., "!"] => println!("!!!"),

    // `start` is a slice of everything except the last element, which must be "z".
    [start @ .., "z"] => println!("starts with: {:?}", start),

    // `end` is a slice of everything but the first element, which must be "a".
    ["a", end @ ..] => println!("ends with: {:?}", end),

    rest => println!("{:?}", rest),
}

if let [.., penultimate, _] = slice {
    println!("next to last is {}", penultimate);
}

# let tuple = (1, 2, 3, 4, 5);
// Rest patterns may also be used in tuple and tuple struct patterns.
match tuple {
    (1, .., y, z) => println!("y={} z={}", y, z),
    (.., 5) => println!("tail must be 5"),
    (..) => println!("matches everything else"),
}
```

## Range patterns

> **<sup>句法</sup>**\
> _RangePattern_ :\
> &nbsp;&nbsp; _RangePatternBound_ `..=` _RangePatternBound_
>
> _ObsoleteRangePattern_ :\
> &nbsp;&nbsp; _RangePatternBound_ `...` _RangePatternBound_
>
> _RangePatternBound_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [CHAR_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [INTEGER_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [FLOAT_LITERAL]\
> &nbsp;&nbsp; | [_PathInExpression_]\
> &nbsp;&nbsp; | [_QualifiedPathInExpression_]

Range patterns match values that are within the closed range defined by its lower and
upper bounds. For example, a pattern `'m'..='p'` will match only the values `'m'`, `'n'`,
`'o'`, and `'p'`. The bounds can be literals or paths that point to constant values.

A pattern a `..=` b must always have a &le; b. It is an error to have a range pattern
`10..=0`, for example.

The `...` syntax is kept for backwards compatibility.

Range patterns only work on scalar types. The accepted types are:

* Integer types (u8, i8, u16, i16, usize, isize, etc.).
* Character types (char).
* Floating point types (f32 and f64). This is being deprecated and will not be available
  in a future version of Rust (see
  [issue #41620](https://github.com/rust-lang/rust/issues/41620)).

Examples:

```rust
# let c = 'f';
let valid_variable = match c {
    'a'..='z' => true,
    'A'..='Z' => true,
    'α'..='ω' => true,
    _ => false,
};

# let ph = 10;
println!("{}", match ph {
    0..=6 => "acid",
    7 => "neutral",
    8..=14 => "base",
    _ => unreachable!(),
});

// using paths to constants:
# const TROPOSPHERE_MIN : u8 = 6;
# const TROPOSPHERE_MAX : u8 = 20;
#
# const STRATOSPHERE_MIN : u8 = TROPOSPHERE_MAX + 1;
# const STRATOSPHERE_MAX : u8 = 50;
#
# const MESOSPHERE_MIN : u8 = STRATOSPHERE_MAX + 1;
# const MESOSPHERE_MAX : u8 = 85;
#
# let altitude = 70;
#
println!("{}", match altitude {
    TROPOSPHERE_MIN..=TROPOSPHERE_MAX => "troposphere",
    STRATOSPHERE_MIN..=STRATOSPHERE_MAX => "stratosphere",
    MESOSPHERE_MIN..=MESOSPHERE_MAX => "mesosphere",
    _ => "outer space, maybe",
});

# pub mod binary {
#     pub const MEGA : u64 = 1024*1024;
#     pub const GIGA : u64 = 1024*1024*1024;
# }
# let n_items = 20_832_425;
# let bytes_per_item = 12;
if let size @ binary::MEGA..=binary::GIGA = n_items * bytes_per_item {
    println!("It fits and occupies {} bytes", size);
}

# trait MaxValue {
#     const MAX: u64;
# }
# impl MaxValue for u8 {
#     const MAX: u64 = (1 << 8) - 1;
# }
# impl MaxValue for u16 {
#     const MAX: u64 = (1 << 16) - 1;
# }
# impl MaxValue for u32 {
#     const MAX: u64 = (1 << 32) - 1;
# }
// using qualified paths:
println!("{}", match 0xfacade {
    0 ..= <u8 as MaxValue>::MAX => "fits in a u8",
    0 ..= <u16 as MaxValue>::MAX => "fits in a u16",
    0 ..= <u32 as MaxValue>::MAX => "fits in a u32",
    _ => "too big",
});
```

Range patterns for (non-`usize` and -`isize`) integer and `char` types are irrefutable
when they span the entire set of possible values of a type. For example, `0u8..=255u8`
is irrefutable. The range of values for an integer type is the closed range from its
minimum to maximum value. The range of values for a `char` type are precisely those
ranges containing all Unicode Scalar Values: `'\u{0000}'..='\u{D7FF}'` and
`'\u{E000}'..='\u{10FFFF}'`.

## Reference patterns

> **<sup>句法</sup>**\
> _ReferencePattern_ :\
> &nbsp;&nbsp; (`&`|`&&`) `mut`<sup>?</sup> [_PatternWithoutRange_]

Reference patterns dereference the pointers that are being matched
and, thus, borrow them.

For example, these two matches on `x: &i32` are equivalent:

```rust
let int_reference = &3;

let a = match *int_reference { 0 => "zero", _ => "some" };
let b = match int_reference { &0 => "zero", _ => "some" };

assert_eq!(a, b);
```

The grammar production for reference patterns has to match the token `&&` to match a
reference to a reference because it is a token by itself, not two `&` tokens.

Adding the `mut` keyword dereferences a mutable reference. The mutability must match the
mutability of the reference.

Reference patterns are always irrefutable.

## Struct patterns

> **<sup>句法</sup>**\
> _StructPattern_ :\
> &nbsp;&nbsp; [_PathInExpression_] `{`\
> &nbsp;&nbsp; &nbsp;&nbsp; _StructPatternElements_ <sup>?</sup>\
> &nbsp;&nbsp; `}`
>
> _StructPatternElements_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _StructPatternFields_ (`,` | `,` _StructPatternEtCetera_)<sup>?</sup>\
> &nbsp;&nbsp; | _StructPatternEtCetera_
>
> _StructPatternFields_ :\
> &nbsp;&nbsp; _StructPatternField_ (`,` _StructPatternField_) <sup>\*</sup>
>
> _StructPatternField_ :\
> &nbsp;&nbsp; [_OuterAttribute_] <sup>\*</sup>\
> &nbsp;&nbsp; (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [TUPLE_INDEX] `:` [_Pattern_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [IDENTIFIER] `:` [_Pattern_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | `ref`<sup>?</sup> `mut`<sup>?</sup> [IDENTIFIER]\
> &nbsp;&nbsp; )
>
> _StructPatternEtCetera_ :\
> &nbsp;&nbsp; [_OuterAttribute_] <sup>\*</sup>\
> &nbsp;&nbsp; `..`

[_OuterAttribute_]: attributes.md
[TUPLE_INDEX]: tokens.md#tuple-index

Struct patterns match struct values that match all criteria defined by its subpatterns.
They are also used to [destructure](#destructuring) a struct.

On a struct pattern, the fields are referenced by name, index (in the case of tuple
structs) or ignored by use of `..`:

```rust
# struct Point {
#     x: u32,
#     y: u32,
# }
# let s = Point {x: 1, y: 1};
#
match s {
    Point {x: 10, y: 20} => (),
    Point {y: 10, x: 20} => (),    // order doesn't matter
    Point {x: 10, ..} => (),
    Point {..} => (),
}

# struct PointTuple (
#     u32,
#     u32,
# );
# let t = PointTuple(1, 2);
#
match t {
    PointTuple {0: 10, 1: 20} => (),
    PointTuple {1: 10, 0: 20} => (),   // order doesn't matter
    PointTuple {0: 10, ..} => (),
    PointTuple {..} => (),
}
```

If `..` is not used, it is required to match all fields:

```rust
# struct Struct {
#    a: i32,
#    b: char,
#    c: bool,
# }
# let mut struct_value = Struct{a: 10, b: 'X', c: false};
#
match struct_value {
    Struct{a: 10, b: 'X', c: false} => (),
    Struct{a: 10, b: 'X', ref c} => (),
    Struct{a: 10, b: 'X', ref mut c} => (),
    Struct{a: 10, b: 'X', c: _} => (),
    Struct{a: _, b: _, c: _} => (),
}
```

The `ref` and/or `mut` _IDENTIFIER_ syntax matches any value and binds it to
a variable with the same name as the given field.

```rust
# struct Struct {
#    a: i32,
#    b: char,
#    c: bool,
# }
# let struct_value = Struct{a: 10, b: 'X', c: false};
#
let Struct{a: x, b: y, c: z} = struct_value;          // destructure all fields
```

A struct pattern is refutable when one of its subpatterns is refutable.

## Tuple struct patterns

> **<sup>句法</sup>**\
> _TupleStructPattern_ :\
> &nbsp;&nbsp; [_PathInExpression_] `(` _TupleStructItems_<sup>?</sup> `)`
>
> _TupleStructItems_ :\
> &nbsp;&nbsp; [_Pattern_]&nbsp;( `,` [_Pattern_] )<sup>\*</sup> `,`<sup>?</sup>

Tuple struct patterns match tuple struct and enum values that match all criteria defined
by its subpatterns. They are also used to [destructure](#destructuring) a tuple struct or
enum value.

A tuple struct pattern is refutable when one of its subpatterns is refutable.

## Tuple patterns

> **<sup>句法</sup>**\
> _TuplePattern_ :\
> &nbsp;&nbsp; `(` _TuplePatternItems_<sup>?</sup> `)`
>
> _TuplePatternItems_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Pattern_] `,`\
> &nbsp;&nbsp; | [_RestPattern_]\
> &nbsp;&nbsp; | [_Pattern_]&nbsp;(`,` [_Pattern_])<sup>+</sup> `,`<sup>?</sup>

Tuple patterns match tuple values that match all criteria defined by its subpatterns.
They are also used to [destructure](#destructuring) a tuple.

The form `(..)` with a single [_RestPattern_] is a special form that does not
require a comma, and matches a tuple of any size.

The tuple pattern is refutable when one of its subpatterns is refutable.

## Grouped patterns

> **<sup>句法</sup>**\
> _GroupedPattern_ :\
> &nbsp;&nbsp; `(` [_Pattern_] `)`

Enclosing a pattern in parentheses can be used to explicitly control the
precedence of compound patterns. For example, a reference pattern next to a
range pattern such as `&0..=5` is ambiguous and is not allowed, but can be
expressed with parentheses.

```rust
let int_reference = &3;
match int_reference {
    &(0..=5) => (),
    _ => (),
}
```

## Slice patterns

> **<sup>句法</sup>**\
> _SlicePattern_ :\
> &nbsp;&nbsp; `[` _SlicePatternItems_<sup>?</sup> `]`
>
> _SlicePatternItems_ :\
> &nbsp;&nbsp; [_Pattern_] \(`,` [_Pattern_])<sup>\*</sup> `,`<sup>?</sup>

Slice patterns can match both arrays of fixed size and slices of dynamic size.
```rust
// Fixed size
let arr = [1, 2, 3];
match arr {
    [1, _, _] => "starts with one",
    [a, b, c] => "starts with something else",
};
```
```rust
// Dynamic size
let v = vec![1, 2, 3];
match v[..] {
    [a, b] => { /* this arm will not apply because the length doesn't match */ }
    [a, b, c] => { /* this arm will apply */ }
    _ => { /* this wildcard is required, since the length is not known statically */ }
};
```

Slice patterns are irrefutable when matching an array as long as each element
is irrefutable. When matching a slice, it is irrefutable only in the form with
a single `..` [rest pattern](#rest-patterns) or [identifier
pattern](#identifier-patterns) with the `..` rest pattern as a subpattern.

## Path patterns

> **<sup>句法</sup>**\
> _PathPattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_PathInExpression_]\
> &nbsp;&nbsp; | [_QualifiedPathInExpression_]

_Path patterns_ are patterns that refer either to constant values or
to structs or enum variants that have no fields.

Unqualified path patterns can refer to:

* enum variants
* structs
* constants
* associated constants

Qualified path patterns can only refer to associated constants.

Constants cannot be a union type. Struct and enum constants must have
`#[derive(PartialEq, Eq)]` (not merely implemented).

Path patterns are irrefutable when they refer to structs or an enum variant when the enum
has only one variant or a constant whose type is irrefutable. They are refutable when they
refer to refutable constants or enum variants for enums with multiple variants.

[_GroupedPattern_]: #grouped-patterns
[_IdentifierPattern_]: #identifier-patterns
[_LiteralPattern_]: #literal-patterns
[_MacroInvocation_]: macros.md#宏调用
[_ObsoleteRangePattern_]: #range-patterns
[_PathInExpression_]: paths.md#表达式中的路径
[_PathPattern_]: #path-patterns
[_Pattern_]: #patterns
[_PatternWithoutRange_]: #patterns
[_QualifiedPathInExpression_]: paths.md#限定路径
[_RangePattern_]: #range-patterns
[_ReferencePattern_]: #reference-patterns
[_RestPattern_]: #rest-patterns
[_SlicePattern_]: #slice-patterns
[_StructPattern_]: #struct-patterns
[_TuplePattern_]: #tuple-patterns
[_TupleStructPattern_]: #tuple-struct-patterns
[_WildcardPattern_]: #wildcard-pattern

[`Copy`]: special-types-and-traits.md#copy
[IDENTIFIER]: identifiers.md
[enums]: items/enumerations.md
[literals]: expressions/literal-expr.md
[structs]: items/structs.md
[tuples]: types/tuple.md
[scrutinee]: glossary.md#scrutinee
