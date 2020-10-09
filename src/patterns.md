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

模式基于给定结构去匹配值，并可选地将变量和这些结构中匹配到的值相互绑定。模式还用在函数和闭包的变量声明和参数中。

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

示例：

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

标识符模式将它们匹配的值绑定到一个变量上。此变量的标识符在该模式中必须是唯一的。该变量会在作用域中遮蔽同名的任何变量。这种绑定的作用域取决于使用模式的上下文(例如 `let`绑定或匹配(`match`)的匹配臂上)。

最常见的标识符模式就是函数和闭包的变量声明和参数，这种模式只包含一个标识符(也可能前带一个 `mut`)，能匹配任何值，并将其绑定到该标识符。

```rust
let mut variable = 10;
fn sum(x: i32, y: i32) -> i32 {
#    x + y
# }
```

要将模式的匹配值绑定到变量，也可使用句法 `variable @ subpattern`。例如，下面示例中将值2绑定到 `e` (不是整个区间(range)：这里的区间是一个区间子模式(range subpattern))。

```rust
let x = 2;

match x {
    e @ 1 ..= 5 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

默认情况下，标识符模式里匹配值会用一个拷贝副本或自身移动过来和变量完成绑定，具体是拷贝还是移动取决于匹配值是否实现了 [`Copy`]。也可以通过使用 `ref` 关键字将变量和值的引用绑定，或者使用 `ref mut` 将变量和值的可变引用绑定。示例：

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

在第一个匹配表达式中，值被复制(或移动)。在第二个匹配中，对相同内存位置的引用被绑定到变量上。之所以需要这种句法，是因为在解构子模式(destructuring subpatterns)中，`&`操作符不能应用于值的字段。例如，以下内容无效：

```rust,compile_fail
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person{ name: String::from("John"), age: 23 };
if let Person{name: &person_name, age: 18..=150} = value { }
```

要使其有效，请编写以下代码：

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person{ name: String::from("John"), age: 23 };
if let Person{name: ref person_name, age: 18..=150} = value { }
```

因此，`ref` 不是匹配规则什么的。它唯一的目的就是使变量和匹配值的引用绑定起来，而不是潜在地复制或移动匹配的内容。

[路径模式(Path pattern)](#path-patterns)优先于标识符模式。如果给某个标识符被限定用上了 `ref` 或 `ref mut`，同时它又遮蔽了某个常量，这会导致错误。

如果 `@`子模式是不可反驳型的或子模式未指定，则标识符模式是不可反驳型的。

### Binding modes
### 绑定方式

（毕竟显式使用 `ref` 或 `ref mut` 绑定有些麻烦，）为了更好地服务于人类工程学，为了让引用(类型的变量)和值的绑定更容易一些，模式会自动选择不同的*绑定方式*。当引用值与非引用模式匹配时，这将自动地被视为 `ref` 或 `ref mut` 绑定。示例：
To service better ergonomics, patterns operate in different binding modes in order to make it easier to bind references to values. When a reference value is matched by a non-reference pattern, it will be automatically treated as a ref or ref mut binding. Example:

```rust
let x: &Option<i32> = &Some(3);
if let Some(y) = x {
    // y 被转换为`ref y` ，其类型为 &i32
}
```

*非引用模式*包括**除**绑定模式、通配符模式(#wildcard-pattern)(`_`)、引用类型的[常量(`const`)模式](#path-patterns)和[引用模式](#reference-patterns)以外的所有模式。
Non-reference patterns include all patterns except bindings, wildcard patterns (_), const patterns of reference types, and reference patterns.

如果绑定模式(binding pattern)没有显式地包含 `ref`、`ref mut`、或 `mut`，那么它将使用*默认绑定方式(the default binding mode)*来确定如何绑定变量。默认绑定方式以使用移动语义的“移动”模式开始。当匹配模式时，编译器对模式从外到内逐层匹配。每次使用非引用模式匹配引用时，它都会自动解引用该值并更新默认绑定方式。引用会将默认绑定模式设置为ref。可变引用会将模式设置为 `ref mut`，除非模式已经是`ref`(在这种情况下它仍然是`ref`)。如果自动解引用的值仍然是引用，则会重复解引用。

## Wildcard pattern
## 通配符模式

> **<sup>句法</sup>**\
> _WildcardPattern_ :\
> &nbsp;&nbsp; `_`

*通配符模式*(下划线符号)能与任何值匹配。常用它来忽略那些无关紧要的值。在其他模式中使用该模式时，它匹配单个数据字段（与和代表和其余字段匹配的 `..` 相对）。与标识符模式不同，它不会复制、移动或借用它匹配的值。

示例：

```rust
# let x = 20;
let (a, _) = (10, x);   // x 一定会被 _ 匹配上
# assert_eq!(a, 10);

// 忽略一个函数/闭包参数
let real_part = |a: f64, _: f64| { a };

// 忽略结构体的一个字段
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

// 能接收带任何值的任何 Some
# let x = Some(10);
if let Some(_) = x {}
```

通配符模式总是不可反驳型的。

## Rest patterns
## 剩余模式

> **<sup>句法</sup>**\
> _RestPattern_ :\
> &nbsp;&nbsp; `..`

*剩余模式*(`..`标记符)充当可变长度模式(variable-length pattern)，它匹配之前之后没有匹配的零个或多个元素。它只能在[元组](#tuple-patterns)模式、[元组结构体](#tuple-struct-patterns)模式和[切片](#slice-patterns)模式中使用，并且在这些模式中只能作为一个元素出现一次。它在[切片模式](#slice-patterns)里也只允许在[标识符模式](#identifier-patterns)中使用。

剩余模式总是不可反驳型的。

示例：

```rust
# let words = vec!["a", "b", "c"];
# let slice = &words[..];
match slice {
    [] => println!("slice is empty"),
    [one] => println!("single element {}", one),
    [head, tail @ ..] => println!("head={} tail={:?}", head, tail),
}

match slice {
    // 忽略除最后一个元素以外的所有元素，并且最后一个元素必须是 "!".
    [.., "!"] => println!("!!!"),

    // `start` 是除最后一个元素之外的所有元素的一个切片，最后一个元素必须是 “z”。
    [start @ .., "z"] => println!("starts with: {:?}", start),

    // `end` 是除第一个元素之外的所有元素的一个切片，第一个元素必须是 “a”
    ["a", end @ ..] => println!("ends with: {:?}", end),

    rest => println!("{:?}", rest),
}

if let [.., penultimate, _] = slice {
    println!("next to last is {}", penultimate);
}

# let tuple = (1, 2, 3, 4, 5);
// 剩余模式也可是在元组和元组结构体模式中使用。
match tuple {
    (1, .., y, z) => println!("y={} z={}", y, z),
    (.., 5) => println!("tail must be 5"),
    (..) => println!("matches everything else"),
}
```

## Range patterns
## 区间模式

> **<sup>句法</sup>**\
> _RangePattern_ :\
> &nbsp;&nbsp; _RangePatternBound_ `..=` _RangePatternBound_
>
> _ObsoleteRangePattern_ :(译者注：废弃的区间模式句法) \ 
> &nbsp;&nbsp; _RangePatternBound_ `...` _RangePatternBound_
>
> _RangePatternBound_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [CHAR_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [INTEGER_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [FLOAT_LITERAL]\
> &nbsp;&nbsp; | [_PathInExpression_]\
> &nbsp;&nbsp; | [_QualifiedPathInExpression_]

区间模式匹配在其上下边界定义的封闭区间内的值。例如，一个模式 `'m'..='p'` 将只匹配值`'m'`，`'n'`，`'o'`和 `'p'`。边界可以是字面量，也可以是指向常量值的路径。

一个模式 a `..=` b 必须总是有 a &le; b。`10..=0` 这样的区间模式是错误的。例如：

保留 `...`句法只是为了向后兼容。

区间模式只适用于标量类型(scalar type)。可接受的类型有：

* 整型 (u8、i8、u16、i16、usize、isize ...)。
* 字符型 (char)。
* 浮点类型( f32 和 f64 )。这已被弃用，将不会在未来版本的 Rust 中可用（参见 [issue #41620](https://github.com/rust-lang/rust/issues/41620)）。

示例：

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

// 使用指向常量值的路径：
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
    println!("这适用并占用{}个字节", size);
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
// 使用限定路径：
println!("{}", match 0xfacade {
    0 ..= <u8 as MaxValue>::MAX => "fits in a u8",
    0 ..= <u16 as MaxValue>::MAX => "fits in a u16",
    0 ..= <u32 as MaxValue>::MAX => "fits in a u32",
    _ => "too big",
});
```

当区间模式跨越(非usize 和非isize)整型和字符型(`char`)整个类型的所有值组成的集合时，此模式是不可反驳型的。例如，`0u8..=255u8` 是不可反驳型的。某类整型的值区间是从该类型的最小值到该类型最大值的闭区间。字符型(`char`)的值的区间就是那些包含所有 Unicode 标量值的区间：`'\u{0000}'..='\u{D7FF}'` 和 `'\u{E000}'..='\u{10FFFF}'`。

## Reference patterns
## 引用模式

> **<sup>句法</sup>**\
> _ReferencePattern_ :\
> &nbsp;&nbsp; (`&`|`&&`) `mut`<sup>?</sup> [_PatternWithoutRange_]

引用模式对当前匹配的指针做解引用，从而能借用它们：

例如，下面 `x: &i32` 上的两个匹配是等效的：

```rust
let int_reference = &3;

let a = match *int_reference { 0 => "zero", _ => "some" };
let b = match int_reference { &0 => "zero", _ => "some" };

assert_eq!(a, b);
```

语法上，引用模式必须使用标记符 `&&` 来匹配引用的引用，因为 `&&` 本身就是一个标记符，而不是两个 `&` 标记符。。

如果为引用模式上添加 `mut` 关键字来解引用一个可变引用，那模式的可变性必须匹配引用对象的可变性
<!-- Adding the `mut` keyword dereferences a mutable reference. The mutability must match the mutability of the reference.TobeModify -->

引用模式总是不可反驳型的。

## Struct patterns
## 结构体模式

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

结构体模式匹配与子模式定义的所有条件匹配的结构体值。它也被用来解构结构体。

在结构体模式中，结构体字段需通过名称、索引(对于元组结构体)来指向(refer to)，或者通过使用 `..` 来忽略：

```rust
# struct Point {
#     x: u32,
#     y: u32,
# }
# let s = Point {x: 1, y: 1};
#
match s {
    Point {x: 10, y: 20} => (),
    Point {y: 10, x: 20} => (),    // 顺序没关系
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
    PointTuple {1: 10, 0: 20} => (),   // 顺序没关系
    PointTuple {0: 10, ..} => (),
    PointTuple {..} => (),
}
```

如果没使用 `..`，需要提供所有字段的详尽匹配：

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

`ref` 和/或 `mut` *标识符*句法匹配任何值，并将其绑定到与给定字段同名的变量上。
The `ref` and/or `mut` _IDENTIFIER_ syntax matches any value and binds it to a variable with the same name as the given field.

```rust
# struct Struct {
#    a: i32,
#    b: char,
#    c: bool,
# }
# let struct_value = Struct{a: 10, b: 'X', c: false};
#
let Struct{a: x, b: y, c: z} = struct_value;          // 解构所有的字段
```

当一个结构体模式的子模式是可反驳型的，那这个结构体模式就是可反驳型的。

## Tuple struct patterns
## 元组结构体模式

> **<sup>句法</sup>**\
> _TupleStructPattern_ :\
> &nbsp;&nbsp; [_PathInExpression_] `(` _TupleStructItems_<sup>?</sup> `)`
>
> _TupleStructItems_ :\
> &nbsp;&nbsp; [_Pattern_]&nbsp;( `,` [_Pattern_] )<sup>\*</sup> `,`<sup>?</sup>

元组结构体模式匹配元组结构体值和枚举值，这些值将与该模式的子模式定义的所有条件进行匹配。它还被用于[析构](#destructuring)元组结构体或枚举值。

当元组结构体模式的一个子模式是可反驳型的，则该元组结构体模式就是可反驳型的。

## Tuple patterns
## 元组模式

> **<sup>句法</sup>**\
> _TuplePattern_ :\
> &nbsp;&nbsp; `(` _TuplePatternItems_<sup>?</sup> `)`
>
> _TuplePatternItems_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_Pattern_] `,`\
> &nbsp;&nbsp; | [_RestPattern_]\
> &nbsp;&nbsp; | [_Pattern_]&nbsp;(`,` [_Pattern_])<sup>+</sup> `,`<sup>?</sup>

元组模式匹配与子模式定义的所有条件匹配的元组值。它们还被用来[解构](#destructuring)元组。

带有单个剩余模式([_RestPattern_])RestPattern的元组列表 `(..)` 是一种不需要逗号的特殊元组列表，它匹配任意大小的元组。

当元组模式的一个子模式是可反驳型的，那该元组模式就是可反驳型的。

## Grouped patterns
## 分组模式

> **<sup>句法</sup>**\
> _GroupedPattern_ :\
> &nbsp;&nbsp; `(` [_Pattern_] `)`

将模式括在圆括号内可用于显式控制复合模式的优先级。例如，在区间模式(如 `&0..=5`)旁边的引用模式会引起歧义，这时可以用圆括号来消除歧义。

```rust
let int_reference = &3;
match int_reference {
    &(0..=5) => (),
    _ => (),
}
```

## Slice patterns
## 切片模式

> **<sup>句法</sup>**\
> _SlicePattern_ :\
> &nbsp;&nbsp; `[` _SlicePatternItems_<sup>?</sup> `]`
>
> _SlicePatternItems_ :\
> &nbsp;&nbsp; [_Pattern_] \(`,` [_Pattern_])<sup>\*</sup> `,`<sup>?</sup>

片模式可以匹配固定长度的数组和动态长度的切片。

```rust
// 固定长度
let arr = [1, 2, 3];
match arr {
    [1, _, _] => "starts with one",
    [a, b, c] => "starts with something else",
};
```
```rust
// 动态长度
let v = vec![1, 2, 3];
match v[..] {
    [a, b] => { /* 这个匹配臂不适用，因为长度不匹配 */ }
    [a, b, c] => { /* 这个匹配臂可以用 */ }
    _ => { /* 这个通配符是必需的，因为长度不是编译时可知的 */ }
};
```

在匹配数组时，只要每个元素是不可反驳型的，切片模式就是不可反驳型的。当匹配切片时，只有单个 `..` [剩余模式](#rest-patterns)或带有 `..` (剩余模式)作为子模式的[标识符模式](#identifier-patterns)的情况才是不可反驳型的。

## Path patterns
## 路径模式

> **<sup>句法</sup>**\
> _PathPattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_PathInExpression_]\
> &nbsp;&nbsp; | [_QualifiedPathInExpression_]

*路径模式*是指向(refer to)常量值或指向没有字段的结构体或没有字段的枚举变体的模式。

非限定路径模式可以指向：

* 枚举变体
* 结构体
* 常量
* 关联常量

限定路径模式只能指向关联常量。

指向的常量不能是联合体类型。结构体和枚举常量必须带有 `#[derive(PartialEq, Eq)]` 属性(不只是实现)。

当路径模式指向结构体或枚举变体(枚举只有一个变体)或不可反驳型的常量时，该路径模式是不可反驳型的。当路径模式指向的是可反驳型常量或带有多个变体的枚举时，该路径模式是可反驳型的。

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
