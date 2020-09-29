# Loops
# 循环

>[loop-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/loop-expr.md)\
>commit 37f61d3347d0813bce53a25c8ee068650d9a025f

> **<sup>句法</sup>**\
> _LoopExpression_ :\
> &nbsp;&nbsp; [_LoopLabel_]<sup>?</sup> (\
> &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; [_InfiniteLoopExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_PredicateLoopExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_PredicatePatternLoopExpression_]\
> &nbsp;&nbsp; &nbsp;&nbsp; | [_IteratorLoopExpression_]\
> &nbsp;&nbsp; )

[_LoopLabel_]: #loop-labels
[_InfiniteLoopExpression_]: #infinite-loops
[_PredicateLoopExpression_]: #predicate-loops
[_PredicatePatternLoopExpression_]: #predicate-pattern-loops
[_IteratorLoopExpression_]: #iterator-loops

Rust支持四种循环表达式：
*   [`loop`表达式](#infinite-loops)表示一个无限循环。
*   [`while`表达式](#predicate-loops)不断循环，直到谓词为假。
*   [`while let`表达式](#predicate-pattern-loops)循环测试给定模式。
*   [`for`表达式](#iterator-loops)从迭代器中循环取值，直到迭代器为空。

所有四种类型的循环都支持 [`break`表达式](#break-expressions)、[`continue`表达式](#continue-expressions)和[设置循环标签](#loop-labels)。只有 `loop`循环支持对循环体[求得有意义的值](#break-and-loop-values)(valuation to non-trivial values)。

## Infinite loops
## 无限循环

> **<sup>句法</sup>**\
> _InfiniteLoopExpression_ :\
> &nbsp;&nbsp; `loop` [_BlockExpression_]

`loop`表达式会不断地重复地执行它代码体内的代码：`loop { println!("I live."); }`。

没有包含关联的 `break`表达式的 `loop`表达式是发散的，并且具有类型 [`!`](../types/never.md)。包含关联的 `break`表达式的 `loop`表达式可以终止，并且其类型必须与 `break`表达式的值兼容。

## Predicate loops
## 谓词循环

> **<sup>句法</sup>**\
> _PredicateLoopExpression_ :\
> &nbsp;&nbsp; `while` [_Expression_]<sub>_except struct expression_</sub> [_BlockExpression_]

`while`循环从对布尔型的循环条件表达式求值开始。如果循环条件表达式的求值结果为 `true`，则执行循环体块，然后控制流程返回到循环条件表达式。如果循环条件表达式的求值结果为 `false`，则 `while`表达式完成。

举个例子：

```rust
let mut i = 0;

while i < 10 {
    println!("hello");
    i = i + 1;
}
```

## Predicate pattern loops
## 谓词模式循环

> **<sup>句法</sup>**\
> [_PredicatePatternLoopExpression_] :\
> &nbsp;&nbsp; `while` `let` [_MatchArmPatterns_] `=` [_Expression_]<sub>_except struct or lazy boolean operator expression_</sub>
>              [_BlockExpression_]

`while let`循环在语义上类似于 `while`循环，但它用 `let`关键字后紧跟着一个模式、一个 `=`、一个[检验(scrutinee)][scrutinee]表达式和一个块表达式，来替代原来的条件表达式。如果检验表达式的值与模式匹配，则执行循环体块，然后控制流程返回到模式匹配语句。否则，`while`表达式完成。

```rust
let mut x = vec![1, 2, 3];

while let Some(y) = x.pop() {
    println!("y = {}", y);
}

while let _ = 5 {
    println!("不可反驳的模式总是会匹配成功的");
    break;
}
```

`while let`循环等价于包含 `match`表达式的 `loop`表达式。如下：

<!-- ignore: expansion example -->
```rust,ignore
'label: while let PATS = EXPR {
    /* loop body */
}
```

等价于

<!-- ignore: expansion example -->
```rust,ignore
'label: loop {
    match EXPR {
        PATS => { /* loop body */ },
        _ => break,
    }
}
```

可以使用 `|`操作符指定多个模式。这与 `match`表达式中的 `|` 具有相同的语义：

```rust
let mut vals = vec![2, 3, 1, 2, 2];
while let Some(v @ 1) | Some(v @ 2) = vals.pop() {
    // 打印 2, 2, 然后 1
    println!("{}", v);
}
```

与 [`if let`表达式][`if let` expressions]的情况一样，检验表达式不能是一个[懒惰布尔运算符表达式][_LazyBooleanOperatorExpression_]。

## Iterator loops
## 迭代器循环

> **<sup>句法</sup>**\
> _IteratorLoopExpression_ :\
> &nbsp;&nbsp; `for` [_Pattern_] `in` [_Expression_]<sub>_except struct expression_</sub>
>              [_BlockExpression_]

`for`表达式是一个语法结构，用于在 `std::iter::IntoIterator` 的实现(implementation)提供的元素上循环。如果迭代器生成一个值，该值将与不可反驳的模式进行匹配，执行循环体，然后控制权返回到`for`循环的头部。如果迭代器为空，则`for`表达式完成。

A `for` expression is a syntactic construct for looping over elements provided
by an implementation of `std::iter::IntoIterator`. If the iterator yields a
value, that value is matched against the irrefutable pattern, the body of the
loop is executed, and then control returns to the head of the `for` loop. If the
iterator is empty, the `for` expression completes.

An example of a `for` loop over the contents of an array:

```rust
let v = &["apples", "cake", "coffee"];

for text in v {
    println!("I like {}.", text);
}
```

An example of a for loop over a series of integers:

```rust
let mut sum = 0;
for n in 1..11 {
    sum += n;
}
assert_eq!(sum, 55);
```

A for loop is equivalent to the following block expression.

<!-- ignore: expansion example -->
```rust,ignore
'label: for PATTERN in iter_expr {
    /* loop body */
}
```

is equivalent to

<!-- ignore: expansion example -->
```rust,ignore
{
    let result = match IntoIterator::into_iter(iter_expr) {
        mut iter => 'label: loop {
            let mut next;
            match Iterator::next(&mut iter) {
                Option::Some(val) => next = val,
                Option::None => break,
            };
            let PATTERN = next;
            let () = { /* loop body */ };
        },
    };
    result
}
```

`IntoIterator`, `Iterator`, and `Option` are always the standard library items
here, not whatever those names resolve to in the current scope. The variable
names `next`, `iter`, and `val` are for exposition only, they do not actually
have names the user can type.

> **Note**: that the outer `match` is used to ensure that any
> [temporary values] in `iter_expr` don't get dropped before the loop is
> finished. `next` is declared before being assigned because it results in
> types being inferred correctly more often.

## Loop labels

> **<sup>句法</sup>**\
> _LoopLabel_ :\
> &nbsp;&nbsp; [LIFETIME_OR_LABEL] `:`

A loop expression may optionally have a _label_. The label is written as
a lifetime preceding the loop expression, as in `'foo: loop { break 'foo; }`,
`'bar: while false {}`, `'humbug: for _ in 0..0 {}`.
If a label is present, then labeled `break` and `continue` expressions nested
within this loop may exit out of this loop or return control to its head.
See [break expressions](#break-expressions) and [continue
expressions](#continue-expressions).

## `break` expressions

> **<sup>句法</sup>**\
> _BreakExpression_ :\
> &nbsp;&nbsp; `break` [LIFETIME_OR_LABEL]<sup>?</sup> [_Expression_]<sup>?</sup>

When `break` is encountered, execution of the associated loop body is
immediately terminated, for example:

```rust
let mut last = 0;
for x in 1..100 {
    if x > 12 {
        break;
    }
    last = x;
}
assert_eq!(last, 12);
```

A `break` expression is normally associated with the innermost `loop`, `for` or
`while` loop enclosing the `break` expression, but a [label](#loop-labels) can
be used to specify which enclosing loop is affected. Example:

```rust
'outer: loop {
    while true {
        break 'outer;
    }
}
```

A `break` expression is only permitted in the body of a loop, and has one of
the forms `break`, `break 'label` or ([see below](#break-and-loop-values))
`break EXPR` or `break 'label EXPR`.

## `continue` expressions

> **<sup>句法</sup>**\
> _ContinueExpression_ :\
> &nbsp;&nbsp; `continue` [LIFETIME_OR_LABEL]<sup>?</sup>

When `continue` is encountered, the current iteration of the associated loop
body is immediately terminated, returning control to the loop *head*. In
the case of a `while` loop, the head is the conditional expression controlling
the loop. In the case of a `for` loop, the head is the call-expression
controlling the loop.

Like `break`, `continue` is normally associated with the innermost enclosing
loop, but `continue 'label` may be used to specify the loop affected.
A `continue` expression is only permitted in the body of a loop.

## `break` and loop values

When associated with a `loop`, a break expression may be used to return a value
from that loop, via one of the forms `break EXPR` or `break 'label EXPR`, where
`EXPR` is an expression whose result is returned from the `loop`. For example:

```rust
let (mut a, mut b) = (1, 1);
let result = loop {
    if b > 10 {
        break b;
    }
    let c = a + b;
    a = b;
    b = c;
};
// first number in Fibonacci sequence over 10:
assert_eq!(result, 13);
```

In the case a `loop` has an associated `break`, it is not considered diverging,
and the `loop` must have a type compatible with each `break` expression.
`break` without an expression is considered identical to `break` with
expression `()`.

[LIFETIME_OR_LABEL]: ../tokens.md#生命周期和循环标签
[_BlockExpression_]: block-expr.md
[_Expression_]: ../expressions.md
[_MatchArmPatterns_]: match-expr.md
[_Pattern_]: ../patterns.md
[`match` expression]: match-expr.md
[scrutinee]: ../glossary.md#scrutinee
[temporary values]: ../expressions.md#临时位置
[_LazyBooleanOperatorExpression_]: operator-expr.md#lazy-boolean-operators
[`if let` expressions]: if-expr.md#if-let-expressions
