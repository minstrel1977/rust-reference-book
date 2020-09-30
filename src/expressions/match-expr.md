# `match` expressions
# `match`表达式

>[match-expr.md](https://github.com/rust-lang/reference/blob/master/src/expressions/match-expr.md)\
>commit 41cf00929903710a9ce9b1f4c5d8b96e6a511614

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
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> _MatchArmPatterns_ _MatchArmGuard_<sup>?</sup>
>
> _MatchArmPatterns_ :\
> &nbsp;&nbsp; `|`<sup>?</sup> [_Pattern_] ( `|` [_Pattern_] )<sup>\*</sup>
>
> _MatchArmGuard_ :\
> &nbsp;&nbsp; `if` [_Expression_]

*`match`表达式*在模式(pattern)上建立控制流程分支(branch)。匹配的具体形式取决于[模式][pattern]。一个 `match`表达式带有一个 *[检验对象][scrutinee](scrutinee)表达式*，它是要与模式进行比较的值。检验对象表达式和模式必须具有相同的类型。

根据检验对象表达式是[位置表达式或值表达式][place expression]，`match` 的行为会有所不同。如果检验对象表达式是一个[值表达式][value expression]，则这个表达式首先会在被求值到一个临时位置，然后将这个返回值按顺序与匹配臂中的模式进行比较，直到找到一个成功的匹配。带有匹配成功的模式的第一个匹配臂会被选中为当前 `match`表达式的分支目标，然后由该模式匹配绑定到的任何变量都会被赋值给该匹配臂的块中的局部变量，然后控制流程进入该块。

当检验对象表达式是一个[位置表达式][place expression]时，此 `match`表达式不用先去内存上分配一个临时位置，但是，按值匹配绑定会复制或移动这个(位置表达式代表的)内存位置里面的值。如果可能，最好是在匹配位置表达式上进行匹配，因为这种匹配的生命周期继承了位置表达式的生命周期，而不会(让其生命周期仅)局限于此匹配的内部。

`match`表达式的一个示例：

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

模式中的变量绑定的作用域是在匹配守卫(match guard)和匹配臂的表达式里。[变量绑定方式][binding mode](移动、复制或引用)取决于模式。

可以使用 `|`操作符连接多个匹配模式。每个模式将按照从左到右的顺序进行测试，直到找到一个成功的匹配。

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

> 注意: `2..=9` 是一个[区间(Range)模式][Range Pattern]，不是一个[区间表达式][Range Expression]。因此，只有区间模式支持的区间类型才能在匹配臂中使用。

每个 `|` 分隔的模式里出现的变量绑定必须出现在匹配臂的所有模式里。相同名称的绑定变量必须具有相同的类型和相同的变量绑定模式。

## Match guards
## 匹配守卫

匹配臂可以接受*匹配守卫*来进一步细化匹配标准。模式守卫(Pattern guard)出现在模式之后，由 `if`关键字后面的布尔类型表达式组成。

当模式匹配成功时，将执行匹配守卫表达式。如果此表达式的计算结果为真，则模式将进一步匹配成功。否则，将匹配将测试下一个模式，包括同一匹配臂中 `|`运算符分割的后续匹配模式。

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

> 注意：使用 `|`操作符的多次匹配可能会导致匹配守卫必须多次执行的副作用。例如：
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

匹配守卫可以引用绑定在它们前面的模式里的变量。在计算匹配守卫之前，将对检验对象内部被模式的变量匹配上的那部分进行共享引用。在计算匹配守卫时，在访问变量时使用这个共享引用。只有当匹配守卫最终计算为真时，值才会从检验对象内部移动或复制到变量中。这使得共享借用可以在守卫内部使用，而不会在守卫不匹配的情况下移出检验对象。此外，通过在评估匹配守卫的同时保留共享引用，也可以防止匹配守卫内部去修改检验对象。

## Attributes on match arms
## 匹配臂上的属性

在匹配臂上允许使用外部属性，但匹配臂上只有 [`cfg`]、[`cold`] 和 [lint检查类属性][lint check attributes]这些属性才有意义。

适用于[块表达式上的属性][attributes on block expressions]的表达式上下文同样适用于匹配表达式上的属性，同样也是允许[内部属性][Inner attributes]直接位于表达式的左括号之后。

[_Expression_]: ../expressions.md
[place expression]: ../expressions.md#位置表达式和值表达式
[value expression]: ../expressions.md#位置表达式和值表达式
[_InnerAttribute_]: ../attributes.md
[_OuterAttribute_]: ../attributes.md
[`cfg`]: ../conditional-compilation.md
[`cold`]: ../attributes/codegen.md#cold属性
[lint check attributes]: ../attributes/diagnostics.md#lint检查类属性
[Range Expression]: range-expr.md

[_Pattern_]: ../patterns.md
[pattern]: ../patterns.md
[Inner attributes]: ../attributes.md
[Range Pattern]: ../patterns.md#range-patterns
[attributes on block expressions]: block-expr.md#块表达式上的属性
[binding mode]: ../patterns.md#binding-modes
[scrutinee]: ../glossary.md#scrutinee
