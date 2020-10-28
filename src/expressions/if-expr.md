# `if` and `if let` expressions
# `if`和 `if let`表达式

>[if-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/if-expr.md)\
>commit: f8e76ee9368f498f7f044c719de68c7d95da9972 \
>本译文最后维护日期：2020-10-28

## `if` expressions
## `if`表达式

> **<sup>句法</sup>**\
> _IfExpression_ :\
> &nbsp;&nbsp; `if` [_Expression_]<sub>_排除结构体表达式_</sub> [_BlockExpression_]\
> &nbsp;&nbsp; (`else` (
>   [_BlockExpression_]
> | _IfExpression_
> | _IfLetExpression_ ) )<sup>\?</sup>

`if`表达式是程序控制中的一个条件分支。`if`表达式的形式是一个条件表达式，紧跟一个相应的块，再后面是任意数量的 `else if`条件表达式和块，最后是一个可选的尾部 `else`块。条件表达式的类型必须是布尔型(`bool`)。如果条件表达式的求值结果为 `true`，则执行紧跟的块，并跳过后续的 `else if`块或 `else`块。如果条件表达式的求值结果为 `false`，则跳过紧跟的块，并按顺序求值后续的 `else if`条件表达式。如果所有 `if`条件表达式和 `else if`条件表达式的求值结果均为 `false`，则执行 `else`块。if表达式的求值结果就是所执行的块的返回值，或者如果没有块被求值那 if表达式的求值结果就是 `()`。`if`表达式在所有情况下的类型必须一致。

```rust
# let x = 3;
if x == 4 {
    println!("x is four");
} else if x == 3 {
    println!("x is three");
} else {
    println!("x is something else");
}

let y = if 12 * 15 > 150 {
    "Bigger"
} else {
    "Smaller"
};
assert_eq!(y, "Bigger");
```

## `if let` expressions
## `if let`表达式

> **<sup>句法</sup>**\
> _IfLetExpression_ :\
> &nbsp;&nbsp; `if` `let` [_MatchArmPatterns_] `=` [_Expression_]<sub>_除了结构体表达式和惰性布尔运算符表达式_</sub>
>              [_BlockExpression_]\
> &nbsp;&nbsp; (`else` (
>   [_BlockExpression_]
> | _IfExpression_
> | _IfLetExpression_ ) )<sup>\?</sup>

`if let`表达式在语义上类似于 `if`表达式，但是代替条件表达式的是一个关键字 `let`，再后面是一个模式、一个 `=` 和一个[检验对象(scrutinee)][scrutinee]表达式。如果检验对象表达式的值与模式匹配，则执行相应的块。否则，如果存在 `else`块，则继续处理后面的 `else`块。和 `if`表达式一样，`if let`表达式也可以有返回值，这个返回值是由被求值的块确定。

```rust
let dish = ("Ham", "Eggs");

// 此主体代码将被跳过，因为该模式被反驳
if let ("Bacon", b) = dish {
    println!("Bacon is served with {}", b);
} else {
    // 这个块将被执行。
    println!("No bacon will be served");
}

// 此主体代码将被执行
if let ("Ham", b) = dish {
    println!("Ham is served with {}", b);
}

if let _ = 5 {
    println!("不可反驳型的模式总是会匹配成功的");
}
```

`if`表达式和 `if let`表达式能混合使用:

```rust
let x = Some(3);
let a = if let Some(1) = x {
    1
} else if x == Some(2) {
    2
} else if let Some(y) = x {
    y
} else {
    -1
};
assert_eq!(a, 3);
```

`if let`表达式等价于[match表达式][`match` expression]，例如：

<!-- ignore: expansion example -->
```rust,ignore
if let PATS = EXPR {
    /* body */
} else {
    /*else */
}
```

is equivalent to

<!-- ignore: expansion example -->
```rust,ignore
match EXPR {
    PATS => { /* body */ },
    _ => { /* else */ },    // 如果没有 else块，这相当于 `()`
}
```

可以使用操作符 `|` 指定多个模式。这与匹配(`match`)表达式中的 `|` 具有相同的语义：

```rust
enum E {
    X(u8),
    Y(u8),
    Z(u8),
}
let v = E::Y(12);
if let E::X(n) | E::Y(n) = v {
    assert_eq!(n, 12);
}
```

`if let`表达式不能是[惰性布尔运算符表达式][_LazyBooleanOperatorExpression_]。使用惰性布尔运算符的效果是不明确的，因为 Rust 里一个新特性（if-let执行链(if-let chains)的实现-请参阅[eRFC 2947][_eRFCIfLetChain_]）正被提上日程。当确实需要惰性布尔运算符表达式时，可以像下面一样使用圆括号来实现：

<!-- ignore: psuedo code -->
```rust,ignore
// Before...
if let PAT = EXPR && EXPR { .. }

// After...
if let PAT = ( EXPR && EXPR ) { .. }

// Before...
if let PAT = EXPR || EXPR { .. }

// After...
if let PAT = ( EXPR || EXPR ) { .. }
```

[_BlockExpression_]: block-expr.md
[_Expression_]: ../expressions.md
[_LazyBooleanOperatorExpression_]: operator-expr.md#lazy-boolean-operators
[_MatchArmPatterns_]: match-expr.md
[_eRFCIfLetChain_]: https://github.com/rust-lang/rfcs/blob/master/text/2497-if-let-chains.md#rollout-plan-and-transitioning-to-rust-2018
[`match` expression]: match-expr.md
[scrutinee]: ../glossary.md#scrutinee

<!-- 2020-10-25 -->
<!-- checked -->
