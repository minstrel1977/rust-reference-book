# Patterns
# 模式

>[patterns.md](https://github.com/rust-lang/reference/blob/master/src/patterns.md)\
>commit: 5cb05674ee383824cb236a58ec6f75bc75d612e1 \
>本章译文最后维护日期：2024-10-13

> **<sup>句法</sup>**\
> _Pattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `|`<sup>?</sup> _PatternNoTopAlt_  ( `|` _PatternNoTopAlt_ )<sup>\*</sup>
>
> _PatternNoTopAlt_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _PatternWithoutRange_\
> &nbsp;&nbsp; | [_RangePattern_]
>
> _PatternWithoutRange_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_LiteralPattern_]\
> &nbsp;&nbsp; | [_IdentifierPattern_]\
> &nbsp;&nbsp; | [_WildcardPattern_]\
> &nbsp;&nbsp; | [_RestPattern_]\
> &nbsp;&nbsp; | [_ReferencePattern_]\
> &nbsp;&nbsp; | [_StructPattern_]\
> &nbsp;&nbsp; | [_TupleStructPattern_]\
> &nbsp;&nbsp; | [_TuplePattern_]\
> &nbsp;&nbsp; | [_GroupedPattern_]\
> &nbsp;&nbsp; | [_SlicePattern_]\
> &nbsp;&nbsp; | [_PathPattern_]\
> &nbsp;&nbsp; | [_MacroInvocation_]

模式基于给定数据结构去匹配值，并可选地将变量和这些结构中匹配到的值绑定起来。
模式也用在变量声明上和函数（包括闭包）的参数上。

下面示例中的模式完成四件事：

* 测试 `person` 是否在其 `car`字段中填充了内容。
* 测试 `person` 的 `age`字段（的值）是否在 13 到 19 之间，并将其值绑定到给定的变量 `person_age` 上。
* 将对 `name`字段的引用绑定到给定变量 `person_name` 上。
* 忽略 `person` 的其余字段。其余字段可以有任何值，并且不会绑定到任何变量上。

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

模式可用于*解构*[结构体(`struct`)][structs]、[枚举(`enum`)][enums]和[元组][tuples]。
解构将一个值分解成它的组件组成，使用的句法与创建此类值时的几乎相同。
在[检验对象][scrutinee]表达式的类型为结构体(`struct`)、枚举(`enum`)或元组(`tuple`)的模式中，占位符(`_`) 代表*一个*数据字段，而通配符 `..` 代表特定变量/变体(variant)的*所有*剩余字段。当使用字段的名称（而不是字段序号）来解构数据结构时，允许将 `fieldname` 当作 `fieldname: fieldname` 的简写形式书写。

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

当一个模式有可能与它所匹配的值不匹配时，我们就说它是*可反驳型的(refutable)*。
也就是说，*不可反驳型(irrefutable)*模式总是能与它们所匹配的值匹配成功。例如：

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
> &nbsp;&nbsp; &nbsp;&nbsp; `true` | `false`\
> &nbsp;&nbsp; | [CHAR_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | [STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_STRING_LITERAL]\
> &nbsp;&nbsp; | [BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_BYTE_STRING_LITERAL]\
> &nbsp;&nbsp; | [C_STRING_LITERAL]\
> &nbsp;&nbsp; | [RAW_C_STRING_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [INTEGER_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [FLOAT_LITERAL]

*字面量模式*匹配的值与字面量所创建的值完全相同。
由于负数不是[字面量][literals]，（特设定）字面量模式也接受字面量前的可选负号，它的作用类似于否定运算符。

> [!WARNING]

> 字面量模式接受 C语言风格的字符串字面量和原始C语言风格的字符串字面量，但 `&CStr` 没实现结构相等（`#[derive(Eq, PartialEq)]`），因此 `&CStr`上的任何此类 `match` 都将被类型错误所拒绝。

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
> &nbsp;&nbsp; &nbsp;&nbsp; `ref`<sup>?</sup> `mut`<sup>?</sup> [IDENTIFIER] (`@` [_PatternNoTopAlt_] ) <sup>?</sup>

标识符模式将它们匹配的值在[值命名空间][value namespace]中绑定到一个变量上。
此标识符在该模式中必须是唯一的。
该变量会在作用域中遮蔽任何同名的变量。
这种绑定的[作用域][scope]取决于使用模式的上下文（例如 `let`绑定或匹配臂(`match` arm)[^译注1]）。

标识符模式只能包含一个标识符（也可能前带一个 `mut`），能匹配任何值，并将其绑定到该标识符上。
最常见的标识符模式应用场景就是用在变量声明上和用在函数（包括闭包）的参数上。

```rust
let mut variable = 10;
fn sum(x: i32, y: i32) -> i32 {
#    x + y
# }
```

要将模式匹配到的值绑定到变量上，也可使用句法 `variable @ subpattern`。
例如，下面示例中将值 2 绑定到 `e` 上（不是整个区间(range)：这里的区间是一个区间子模式(range subpattern)）。

```rust
let x = 2;

match x {
    e @ 1 ..= 5 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

默认情况下，标识符模式里变量会和匹配到的值的一个拷贝副本绑定，或匹配值自身移动过来和变量完成绑定，具体是使用拷贝语义还是移动语义取决于匹配到的值是否实现了 [`Copy`]。
也可以通过使用关键字 `ref` 将变量和值的引用绑定，或者使用 `ref mut` 将变量和值的可变引用绑定。示例：

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

在第一个匹配表达式中，值被拷贝（或移动）（到变量 `value` 上）。
在第二个匹配中，对相同内存位置的引用被绑定到变量上。
之所以需要这种句法，是因为在解构子模式(destructuring subpatterns)里，操作符 `&` 不能应用在值的字段上。
例如，以下内容无效：

```rust,compile_fail
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person { name: String::from("John"), age: 23 };
if let Person { name: &person_name, age: 18..=150 } = value { }
```

要使其有效，请按如下方式编写代码：

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person { name: String::from("John"), age: 23 };
if let Person { name: ref person_name, age: 18..=150 } = value { }
```

这里，`ref` 不是被匹配的一部分。
这里它唯一的目的就是使变量和匹配值的引用绑定起来，而不是潜在地拷贝或移动匹配到的内容。

[路径模式(Path pattern)](#path-patterns)优先于标识符模式。
如果给某个标识符指定了 `ref` 或 `ref mut`，同时该标识符又遮蔽了某个常量，这会导致错误。

如果 `@`子模式是不可反驳型的或未指定子模式，则标识符模式是不可反驳型的。

### Binding modes
### 绑定方式

基于人类工程学的考虑，为了让引用和匹配值的绑定更容易一些，模式会自动选择不同的*绑定方式*。
当引用值与非引用模式匹配时，这将自动地被视为 `ref` 或 `ref mut` 绑定方式。
示例：

```rust
let x: &Option<i32> = &Some(3);
if let Some(y) = x {
    // y 被转换为`ref y` ，其类型为 &i32
}
```

*非引用模式(Non-reference patterns)*包括**除**上面这种单独的标识符绑定模式和后面会讲到的[通配符模式](#wildcard-pattern)（`_`）、匹配引用类型的[常量(`const`)模式](#path-patterns)和[引用模式](#reference-patterns)这些模式以外的所有模式。

如果绑定模式(binding pattern)中没有显式地包含 `ref`、`ref mut`、`mut`，那么它将使用*默认绑定方式*来确定如何绑定变量。
默认绑定方式以使用移动语义的“移动(move)”方式开始。
当匹配一个模式时，编译器对模式从外到内逐层匹配。每次非引用模式和引用匹配上了时，引用都会自动解引用出最后的值，并更新默认绑定方式，再进行最终的匹配。
此时引用会将默认绑定方式设置为 `ref` 方式。
可变引用会将模式设置为 `ref mut` 方式，除非绑定方式已经是 `ref` 了（在这种情况下它仍然是 `ref` 方式）。
如果自动解引用解出的值仍然是引用，则会重复解引用。[^译注2]

移动语义的绑定方式和引用语义的绑定方式可以在同一个模式中混合使用，这样做会导致绑定对象的部分被移走，并且之后无法再使用该对象。
这只适用于类型无法拷贝的情况下。

下面的示例中，`name` 被移出了 `person`，因此如果再试图把 `person` 作为一个整体使用，或再次使用 `person.name`，将会因为*部分移出(partial move)*的问题而报错。

示例：

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let person = Person{ name: String::from("John"), age: 23 };
// 在 `age` 被引用绑定的情况下，`name` 被从 person 中移出
let Person { name, ref age } = person;
```

## Wildcard pattern
## 通配符模式

> **<sup>句法</sup>**\
> _WildcardPattern_ :\
> &nbsp;&nbsp; `_`

*通配符模式*（下划线符号）能与任何值匹配。
常用它来忽略那些无关紧要的值。
在其他模式中使用该模式时，它匹配单个数据字段（与和匹配所有其余字段的 `..` 相对）。
与标识符模式不同，它不会复制、移动或借用它匹配的值。

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

*剩余模式*（`..` token）充当匹配长度可变的模式(variable-length pattern)，它匹配之前之后没有匹配的零个或多个元素。
它只能在[元组](#tuple-patterns)模式、[元组结构体](#tuple-struct-patterns)模式和[切片](#slice-patterns)模式中使用，并且只能作为这些模式中的一个元素出现一次。
当作为[标识符模式](#identifier-patterns)的子模式时，它也可出现在[切片模式](#slice-patterns)里。

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

    // 'whole' 是整个切片，`last` 是最后一个元素
    whole @ [.., last] => println!("the last element of {:?} is {}", whole, last),

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
> &nbsp;&nbsp; &nbsp;&nbsp; _RangeInclusivePattern_\
> &nbsp;&nbsp; | _RangeFromPattern_\
> &nbsp;&nbsp; | _RangeToInclusivePattern_\
> &nbsp;&nbsp; | _ObsoleteRangePattern_
>
> _RangeExclusivePattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _RangePatternBound_ `..` _RangePatternBound_

>
> _RangeInclusivePattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _RangePatternBound_ `..=` _RangePatternBound_
>
> _RangeFromPattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; _RangePatternBound_ `..`
>
> _RangeToInclusivePattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; `..=` _RangePatternBound_
>
> _ObsoleteRangePattern_ :(译者注：废弃的区间模式句法/产生式) \ 
> &nbsp;&nbsp; _RangePatternBound_ `...` _RangePatternBound_
>
> _RangePatternBound_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [CHAR_LITERAL]\
> &nbsp;&nbsp; | [BYTE_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [INTEGER_LITERAL]\
> &nbsp;&nbsp; | `-`<sup>?</sup> [FLOAT_LITERAL] \
> &nbsp;&nbsp; | [_PathExpression_]

*区间模式*匹配在区间上下边界内界定的标量值。
它们包括一个*符号*（`..`、`..=`或`...`中的一个）和一侧或两侧的绑定。
区间模式符号左侧的界限称为*下限*。
区间模式符号右侧的界限称为*上限*。
区间模式可是闭区间或半开区间。

同时带有下限和上限的区间模式将匹配其两个边界之间的所有值（包括两个边界）。
它被写为其下限，后跟 `..` 来表示本区间为开区间或 `..=` 来表示本区间为闭区间，最后是其上限。
区间模式的类型是其上下限的共同类型。

例如：模式 `'m'..='p'` 只匹配 `'m'`, `'n'`, `'o'` 和 `'p'` 这4个字符。
那 `'m'..'p'` 就只能匹配 `'m'`, `'n'` 和 `'o'`， `'p'` 被有意的排除在外。

下限不能比上限大。
因此在 `a..=b` 里，必须总是有 a &le; b。
比如，`10..=0` 这样的区间模式是错误的。

如果区间模式只有上限或下限，则它们是*半开*的。
它们的类型与其上限或下限相同。

只有下限的区间模式将匹配大于或等于下限的任何值。
它被写为其下限，后跟`..`，并且具本模式有与其下限相同的类型。
比如`1..` 可以匹配 9，或者 9001，或者 9007199254740991（如果此值的内存宽度合适），但是不能匹配到 0，也不能匹配有符号整数的负值。

只有上限的区间模式将匹配小于或等于上限的任何值。
它被写为`..=`后跟上限，并且本模式具有与其上限相同的类型。
比如，`..=10` 将匹配10、1、0；对于有符号整型，除这些外，还包括所有的负值。

只带有一个边界值的区间模式不能用作[切片模式](#slice-patterns)中的子模式的顶层模式。

界值的合法形式如下：

* 字面量中的字符、字节、整形或浮点字面量。
* `-` 后跟一个整型或浮点型字面量。
* [路径][path]

如果界值被书写为路径形式，则在宏解析之后，路径必须解析为一个 `char`、整型或浮点型的常量项。

界值的类型和值取决于它的写入方式。
如果界值是[路径][path]形式，则模式必须能以此路径解析到的此路径所代表的那个[常量][constant]的类型和值。
如果是浮点数区间模式，解析出的常量项不能是一个 `NaN`。
如果界值是一个字面量，则界值具有相应的[字面量表达式][literal expression]的类型和值。
如果字面量前面有一个 `-` 字符，则界值的类型与相应的[字面量表达式][literal expression]的类型相同，且值为相应字面量表达式的值的[相反数][negating] 。

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
    0..7 => "acid",
    7 => "neutral",
    8..=14 => "base",
    _ => unreachable!(),
});

# let uint: u32 = 5;
match uint {
    0 => "zero!",
    1.. => "正数!",
};

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

当区间模式匹配某固定位宽的整型类型和字符型(`char`)的整个值域时，此模式是不可反驳型的。例如，`0u8..=255u8` 是不可反驳型的。某类整型的值区间是从该类型的最小值到该类型最大值的闭区间。字符型(`char`)的值的区间就是那些包含所有 Unicode 标量值的区间，即 `'\u{0000}'..='\u{D7FF}'` 和 `'\u{E000}'..='\u{10FFFF}'`。

> **版次差异**：在2021版之前，在带有上下限的区间模式里，可以使用 `...` 来表达 `..=` 的语义。

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

引用模式的文法产生式(grammar production)要求必须使用 token `&&` 来匹配对引用的引用，因为 `&&` 本身是一个单独的 token，而不是两个 `&` token。

>译者注：举例
>```
>let a = Some(&&10);
>match a {
>    Some( &&value ) => println!("{}", value),
>    None => {}
>}
>```

引用模式中添加关键字 `mut` 可对可变引用做解引用。
引用模式中的可变性标记必须与作为匹配对象的那个引用的可变性匹配。

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

结构体模式匹配与子模式定义的所有条件匹配的结构体值/枚举值/联合体值。它也被用来[解构](#destructuring)结构体值/枚举值/联合体值。

在结构体模式中，结构体字段需通过名称、索引（对于元组结构体来说）来指代，或者通过使用 `..` 来忽略：

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

# enum Message {
#     Quit,
#     Move { x: i32, y: i32 },
# }
# let m = Message::Quit;
#
match m {
    Message::Quit => (),
    Message::Move {x: 10, y: 20} => (),
    Message::Move {..} => (),
}
```

如果没使用 `..`，则需要一个用于匹配结构体的结构体模式来指定所有字段：

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

用于匹配联合体的结构体模式必须且只能指定一个字段（请参阅[在联合体上使用模式匹配][Pattern matching on unions]）。

_`ref` 和/或 `mut` IDENTIFIER_ 这样的句法格式能匹配任意值，并将其绑定到与给定字段同名的变量上。

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

如果 _PathInExpression_ 被解析为带有多个变体的枚举的构造函数，或者它的一个子模式是可反驳型的，则此结构体模式是可反驳型的。

## Tuple struct patterns
## 元组结构体模式

> **<sup>句法</sup>**\
> _TupleStructPattern_ :\
> &nbsp;&nbsp; [_PathInExpression_] `(` _TupleStructItems_<sup>?</sup> `)`
>
> _TupleStructItems_ :\
> &nbsp;&nbsp; [_Pattern_]&nbsp;( `,` [_Pattern_] )<sup>\*</sup> `,`<sup>?</sup>

元组结构体模式匹配元组结构体值和枚举值，这些值将与该模式的子模式定义的所有条件进行匹配。
它还被用于[解构](#destructuring)元组结构体值或枚举值。

当元组结构体模式的一个子模式是可反驳型的，则该元组结构体模式就是可反驳型的。
如果 _PathInExpression_ 被解析为带有多个变体的枚举的构造函数，或者它的一个子模式是可反驳型的，则该元组结构体模式是可反驳型的。

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

元组模式匹配与子模式定义的所有条件匹配的元组值。它们还被用来[解构](#destructuring)元组值。

内部只带有一个[剩余模式][_RestPattern_](_RestPattern_)的元组句法形式 `(..)` 是一种内部不需要逗号分割的特殊匹配形式，它可以匹配任意长度的元组。

当元组模式的一个子模式是可反驳型的，那该元组模式就是可反驳型的。

使用元组模式的示例：

```rust
let pair = (10, "ten");
let (a, b) = pair;

assert_eq!(a, 10);
assert_eq!(b, "ten");
```

## Grouped patterns
## 分组模式

> **<sup>句法</sup>**\
> _GroupedPattern_ :\
> &nbsp;&nbsp; `(` [_Pattern_] `)`

将模式括在圆括号内可用来显式控制复合模式的优先级。例如，像 `&0..=5` 这样的引用模式和区间模式相邻就会引起歧义，这时可以用圆括号来消除歧义。

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

切片模式可以匹配固定长度的数组和动态长度的切片。

```rust
// 固定长度
let arr = [1, 2, 3];
match arr {
    [1, _, _] => "从 1 开始",
    [a, b, c] => "从其他值开始",
};
```
```rust
// 动态长度
let v = vec![1, 2, 3];
match v[..] {
    [a, b] => { /* 这个匹配臂不适用，因为长度不匹配 */ }
    [a, b, c] => { /* 这个匹配臂适用 */ }
    _ => { /* 这个通配符是必需的，因为长度不是编译时可知的 */ }
};
```

在匹配数组时，只要每个元素是不可反驳型的，切片模式就是不可反驳型的。当匹配切片时，只有单个 `..` [剩余模式](#rest-patterns)或带有 `..`（剩余模式）作为子模式的[标识符模式](#identifier-patterns)的情况才是不可反驳型的。

在一个切片中，没有下限和上限的区间模式必须用括号括起来，如`(a..)`中所示，以明确其目的是与单个切片元素匹配。
带有下限和上限的区间模式，如 `a..=b` 不需要用括号括起来。

## Path patterns
## 路径模式

> **<sup>句法</sup>**\
> _PathPattern_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; [_PathExpression_]

*路径模式*是指向(refer to)常量值或指向没有字段的结构体或没有字段的枚举变体的模式。

非限定路径模式可以指向：

* 枚举变体
* 结构体
* 常量
* 关联常量

限定路径模式只能指向关联常量。

当路径模式指向结构体或枚举变体(枚举只有一个变体)或类型为不可反驳型的常量时，该路径模式是不可反驳型的。
当路径模式指向的是可反驳型常量或带有多个变体的枚举时，该路径模式是可反驳型的。

### Constant patterns
### 常量模式

当类型为 `T` 的常量 `C` 被用作模式时，我们首先检查 `T` 是否 `T: PartialEq`。
此外，我们要求 `C` 的值 *具有（递归）结构相等性（structural equality）*，其递归定义如下：

- 整型以及 `str`、`bool` 和 `char` 值总是具有结构相等性的。
- 如果元组、数组和切片的所有字段/元素都具有结构相等性，则它具有结构相等性。
  （特别是，`()`和`[]`总是具有结构相等性。）
- 如果引用指向的值具有结构相等性，则该引用具有结构相等性。
- 如果 `struct` 或 `enum`类型的值通过 `#[derive(PartialEq)]`属性派生的 `PartialEq`实例的所有字段（对于枚举：活动变体的所有字段）都具有结构相等性，则它也具有结构相等性。
- 如果裸指针被定义为常量整数（然后进行转换/转换），则它具有结构相等性。
- 如果浮点值不是 `NaN`，则它具有结构相等性。
- 其他的都不具有结构相等性。

特别需要指出的是，在构建模式时（即单态化之前），必须知道 `C` 的值。
这意味着当涉及泛型参数时，其中的关联常量不能用作模式。

在确保所有条件都满足之后，常量值被转换为模式，现在它的行为就像直接写入了该模式一样。
特别需要指出的是，它还完全参与了详尽性检查。
（对于原始指针，常量是编写此类模式的唯一方式。对于这些类型，只有 `_` 被认为是详尽的。）

## Or-patterns
## or模式

_or模式_是能匹配两个或多个并列子模式（例如：`A | B | C`）中的一个的模式。
此模式可以任意嵌套。
除了 `let`绑定和函数参数（包括闭包参数）中的模式（此时句法上使用 _PatternNoTopAlt_产生式），or模式在句法上允许在任何其他模式出现的地方出现（这些模式句法上使用 _Pattern_产生式）。

### Static semantics
### 静态语义

1. 假定在某个代码深度上给定任意模式 `p` 和 `q`，现假定它们组成模式 `p | q`，则以下情况会导致这种组成的非法：

   + 从 `p` 推断出的类型和从 `q` 推断出的类型不一致，或
   + `p` 和 `q` 引入的绑定标识符不一样，或
   + `p` 和 `q` 中引入的同名绑定标识符的类型和绑定模式中的类型不一致。

   前面提到的所有实例中的类型都必须是精确的，隐式的[类型强转][type coercions]在这里不适用。

2. 当对表达式 `match e_s { a_1 => e_1, ... a_n => e_n }` 做类型检查时，假定在 `e_s` 内部深度为 `d` 的地方存一个表达式片段，那对于此片段，每一个匹配臂 `a_i` 都包含了一个 `p_i | q_i` 来与此段内容进行匹配，但如果表达式片段的类型与 `p_i | q_i` 的类型不一致，则该模式 `p_i | q_i` 被认为是格式错误的。

3. 为了遵从匹配模式的穷尽性检查，模式 `p | q` 被认为同时覆盖了 `p` 和 `q`。对于某些构造器 `c(x, ..)` 来说，此时应用分配律能使 `c(p | q, ..rest)` 与 `c(p, ..rest) | c(q, ..rest)` 覆盖相同的一组匹配值。这个规律可以递归地应用，直到不再有形式为 `p | q`  的嵌套模式。
   
  注意这里的*“构造器”*这个用词，我们并没有特定提到它是元组结构模式，因为它本意是指任何能够生成类型的模式。这包括枚举变量、元组结构、具有命名字段的结构、数组、元组和切片。


### Dynamic semantics
### 动态语义

1. 检查对象表达式(scrutinee expression) `e_s` 与深度为 `d` 的模式 `c(p | q, ..rest)`（这里`c`是某种构造器，`p` 和 `q` 是任意的模式，`rest` 是 `c`构造器的任意的可选因子）进行匹配的动态语义与此表达式与 `c(p, ..rest) | c(q, ..rest)` 进行匹配的语法定义相同。

### Precedence with other undelimited patterns
### 无分解符模式的优先级

如本章其他部分所示，有几种类型的模式在语法上没有定义分界符，它们包括标识符模式、引用模式和 or模式。它们组合在一起时，or模式的优先级总是最低的。这允许我们为将来可能的类型特性保留语法空间，同时也可以减少歧义。例如，`x @ A(..) | B(..)` 将导致一个错误，即 `x` 不是在所有模式中都存在绑定关系； `&A(x) | B(x)`将导致不同子模式中的 `x` 之的类型不匹配。


[^译注1]: 请仔细参研[匹配表达式](expressions/match-expr.md)中的 MatchExpression产生式，搞清楚匹配臂(MatchArm)的位置。

[^译注2]: 文字叙述有些晦涩，译者举个例子：假如 `if let &Some(y) = &&&Some(3) {`，此时会首先剥掉等号两边的第一层 `&`号，然后是 `Some(y)` 和 `&&Some(3)`匹配，此时发现是非引用模式和引用匹配上了，就再对 `&&Some(3)` 做重复解引用，解出 `Some(3)`，然后从外部转向内部，见到最后的变量 `y` 和检验对象 `3`，就更新 `y` 的默认绑定方式为 `ref`，所以 `y` 就匹配为 `&3`；如果我们这个例子的变量 `y` 改为 `ref y`，不影响 `y` 的绑定效果；极端的情况 `if let &Some(y) = &&&Some(x) {`，如果 `x` 是可变的，那么此时 `y` 的绑定方式就是 `ref mut`，再进一步极端 `if let &Some(ref y) = &&&Some(x) {`，此时 `y` 的绑定方式仍是 `ref`。

[_GroupedPattern_]: #grouped-patterns
[_IdentifierPattern_]: #identifier-patterns
[_LiteralPattern_]: #literal-patterns
[_MacroInvocation_]: macros.md#macro-invocation
[_ObsoleteRangePattern_]: #range-patterns
[_PathInExpression_]: paths.md#paths-in-expressions
[_PathExpression_]: expressions/path-expr.md
[_PathPattern_]: #path-patterns
[_Pattern_]: #patterns
[_PatternNoTopAlt_]: #patterns
[_PatternWithoutRange_]: #patterns
[_QualifiedPathInExpression_]: paths.md#qualified-paths
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
[constant]: items/constant-items.md
[enums]: items/enumerations.md
[literals]: expressions/literal-expr.md
[literal expression]: expressions/literal-expr.md
[negating]: expressions/operator-expr.md#negation-operators
[path]: expressions/path-expr.md
[pattern matching on unions]: items/unions.md#pattern-matching-on-unions
[range expressions]: expressions/range-expr.md
[scope]: names/scopes.md
[structs]: items/structs.md
[tuples]: types/tuple.md
[scrutinee]: glossary.md#scrutinee
[type coercions]: type-coercions.md
[value namespace]: names/namespaces.md

[CHAR_LITERAL]: tokens.md#character-literals
[BYTE_LITERAL]: tokens.md#byte-literals
[STRING_LITERAL]: tokens.md#string-literals
[RAW_STRING_LITERAL]: tokens.md#raw-string-literals
[BYTE_STRING_LITERAL]: tokens.md#byte-string-literals
[RAW_BYTE_STRING_LITERAL]: tokens.md#raw-byte-string-literals
[C_STRING_LITERAL]: tokens.md#c-string-literals
[RAW_C_STRING_LITERAL]: tokens.md#raw-c-string-literals
[INTEGER_LITERAL]: tokens.md#integer-literals
[FLOAT_LITERAL]: tokens.md#floating-point-literals