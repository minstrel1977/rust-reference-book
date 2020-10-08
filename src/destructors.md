# Destructors
# 析构函数

>[destructors.md](https://github.com/rust-lang/reference/blob/master/src/destructors.md)\
>commit c5648e6303632d99ac96edbc7aee7c032dc28891

当一个[初始化][initialized]的[变量][variable]或[临时变量][temporary]超出作用域时，它的*析构函数*将运行，或者说它将被*销毁(dropped)*。此外[赋值][Assignment]动作也会运行其左操作数的析构函数(如果它已初始化)。如果变量已部分初始化，则只销毁其已初始化的字段。

类型 `T` 的析构函数由以下内容组成：

1. 如果 `T: Drop`, 则调用 [`<T as std::ops::Drop>::drop`]
2. 递归地运行其所有字段的析构函数。
    * [结构体][struct]的字段按照声明顺序被销毁。
    * 活动[枚举变体][enum variant]的字段按声明顺序销毁。
    * [元组][tuple]中的字段按顺序销毁。
    * [数组][array]或拥有所有权的[切片][slice]的元素的销毁顺序是从第一个元素到最后一个元素。
    * [闭包][closure]通过移动(move)方式捕获的变量的销毁顺序未指定。
    * [trait对象][Trait objects]的销毁会运行其非具名基类(underlying type)的析构函数。
    * 其他类型不会导致任何进一步的销毁动作发生。

如果析构函数必须手动运行，比如在实现自定义的智能指针时，可以使用 [`std::ptr::drop_in_place`]

举些例子：

```rust
struct PrintOnDrop(&'static str);

impl Drop for PrintOnDrop {
    fn drop(&mut self) {
        println!("{}", self.0);
    }
}

let mut overwritten = PrintOnDrop("drops when overwritten");
overwritten = PrintOnDrop("drops when scope ends");

let tuple = (PrintOnDrop("Tuple first"), PrintOnDrop("Tuple second"));

let moved;
// 没有析构函数在赋值时运行
moved = PrintOnDrop("Drops when moved");
// 这里执行销毁，但随后变量进入未初始化状态
moved;

// 未初始化不会被销毁
let uninitialized: PrintOnDrop;

// 在部分移动之后，后续销毁动作只销毁剩余字段。
let mut partial_move = (PrintOnDrop("first"), PrintOnDrop("forgotten"));
// 执行部分移出，只留下 `partial_move.0` 处于初始化状态
core::mem::forget(partial_move.1);
// 当 partial_move 的作用域结束时, 这里就只有第一个字段被销毁。
```

## Drop scopes
## 销毁作用域

每个变量或临时变量都与一个*销毁作用域*相关联。当控制流程离开一个销毁作用域时，与该作用域关联的所有变量将按照声明(变量)或创建(临时变量)的相反顺序销毁。

销毁作用域是在使用 [`match`] 将 [`for`]、[`if let`] 和 [`while let`] 这些表达式替换为等效表达式之后确定的。销毁作用域的确定上重载操作符与内置操作符没有区别，[绑定方式(binding modes)][binding modes]也不用考虑。

给定一个函数或闭包，存在以下的销毁作用域：
* 整个函数
* 每个[语句][statement]
* 每个[表达式][expression]
* 每个块，包括函数体
    * 在[块表达式][block expression]上，块和表达式的销毁作用域是相同的
* 匹配(`match`)表达式的每条匹配臂

销毁作用域相互嵌套如下。当同时离开多个作用域时，比如从函数返回时，变量会从内部向外销毁。

* 整个函数作用域是最外层的作用域。
* 函数体块包含在整个函数作用域内。
* 表达式语句中的父表达式是该语句的作用域。
* [`let`语句][`let` statement]的初始化器的父作用域是 `let`语句的作用域
* 语句的父作用域是包含该语句的块的作用域。
* 匹配守卫(`match` guard)表达式的父作用域是该守卫所在的匹配臂的作用域。
* 在匹配表达式(`match` expression)的 =>` 之后的表达式的父作用域是它所在的匹配臂的作用域。
* 匹配臂作用域的父作用域是它所在的匹配表达式(`match` expression)的作用域。
* 所有其他作用域的父作用域都是直接封闭该表达式的作用域。

### Scopes of function parameters
### 函数参数的作用域

所有函数参数都在整个函数体的作用域内，因此在对函数求值时最后销毁。在参数的模式中引入任何绑定之后，实际的每个函数参数都会被销毁
所有函数参数都在整个函数体的作用域内，因此在计算函数时，它们是最后被销毁的。每个实际的函数参数都会在该参数的模式中引入任何绑定之后删除。
All function parameters are in the scope of the entire function body, so are dropped last when evaluating the function. Each actual function parameter is dropped after any bindings introduced in that parameter's pattern.

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
// Drops the second parameter, then `y`, then the first parameter, then `x`
fn patterns_in_parameters(
    (x, _): (PrintOnDrop, PrintOnDrop),
    (_, y): (PrintOnDrop, PrintOnDrop),
) {}

// drop order is 3 2 0 1
patterns_in_parameters(
    (PrintOnDrop("0"), PrintOnDrop("1")),
    (PrintOnDrop("2"), PrintOnDrop("3")),
);
```

### Scopes of local variables

Local variables declared in a `let` statement are associated to the scope of
the block that contains the `let` statement. Local variables declared in a
`match` expression are associated to the arm scope of the `match` arm that they
are declared in.

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let declared_first = PrintOnDrop("Dropped last in outer scope");
{
    let declared_in_block = PrintOnDrop("Dropped in inner scope");
}
let declared_last = PrintOnDrop("Dropped first in outer scope");
```

If multiple patterns are used in the same arm for a `match` expression, then an
unspecified pattern will be used to determine the drop order.

### Temporary scopes

The *temporary scope* of an expression is the scope that is used for the
temporary variable that holds the result of that expression when used in a
[place context], unless it is [promoted].

Apart from lifetime extension, the temporary scope of an expression is the
smallest scope that contains the expression and is for one of the following:

* The entire function body.
* A statement.
* The body of a [`if`], [`while`] or [`loop`] expression.
* The `else` block of an `if` expression.
* The condition expression of an `if` or `while` expression, or a `match`
  guard.
* The expression for a match arm.
* The second operand of a [lazy boolean expression].

> **Notes**:
> 
> Temporaries that are created in the final expression of a function
> body are dropped *after* any named variables bound in the function body, as
> there is no smaller enclosing temporary scope.
>
> The [scrutinee] of a `match` expression is not a temporary scope, so
> temporaries in the scrutinee can be dropped after the `match` expression. For
> example, the temporary for `1` in `match 1 { ref mut z => z };` lives until
> the end of the statement.

Some examples:

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let local_var = PrintOnDrop("local var");

// Dropped once the condition has been evaluated
if PrintOnDrop("If condition").0 == "If condition" {
    // Dropped at the end of the block
    PrintOnDrop("If body").0
} else {
    unreachable!()
};

// Dropped at the end of the statement
(PrintOnDrop("first operand").0 == ""
// Dropped at the )
|| PrintOnDrop("second operand").0 == "")
// Dropped at the end of the expression
|| PrintOnDrop("third operand").0 == "";

// Dropped at the end of the function, after local variables.
// Changing this to a statement containing a return expression would make the
// temporary be dropped before the local variables. Binding to a variable
// which is then returned would also make the temporary be dropped first.
match PrintOnDrop("Matched value in final expression") {
    // Dropped once the condition has been evaluated
    _ if PrintOnDrop("guard condition").0 == "" => (),
    _ => (),
}
```

### Operands

Temporaries are also created to hold the result of operands to an expression
while the other operands are evaluated. The temporaries are associated to the
scope of the expression with that operand. Since the temporaries are moved from
once the expression is evaluated, dropping them has no effect unless one of the
operands to an expression breaks out of the expression, returns, or panics.

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
loop {
    // Tuple expression doesn't finish evaluating so operands drop in reverse order
    (
        PrintOnDrop("Outer tuple first"),
        PrintOnDrop("Outer tuple second"),
        (
            PrintOnDrop("Inner tuple first"),
            PrintOnDrop("Inner tuple second"),
            break,
        ),
        PrintOnDrop("Never created"),
    );
}
```

### Constant promotion

Promotion of a value expression to a `'static` slot occurs when the expression
could be written in a constant, borrowed, and dereferencing that borrow where
the expression was originally written, without changing the runtime behavior.
That is, the promoted expression can be evaluated at compile-time and the
resulting value does not contain [interior mutability] or [destructors] (these
properties are determined based on the value where possible, e.g. `&None`
always has the type `&'static Option<_>`, as it contains nothing disallowed).

### Temporary lifetime extension

> **Note**: The exact rules for temporary lifetime extension are subject to
> change. This is describing the current behavior only.

The temporary scopes for expressions in `let` statements are sometimes
*extended* to the scope of the block containing the `let` statement. This is
done when the usual temporary scope would be too small, based on certain
syntactic rules. For example:

```rust
let x = &mut 0;
// Usually a temporary would be dropped by now, but the temporary for `0` lives
// to the end of the block.
println!("{}", x);
```

If a borrow, dereference, field, or tuple indexing expression has an extended
temporary scope then so does its operand. If an indexing expression has an
extended temporary scope then the indexed expression also has an extended
temporary scope.

#### Extending based on patterns

An *extending pattern* is either

* An [identifier pattern] that binds by reference or mutable reference.
* A [struct][struct pattern], [tuple][tuple pattern], [tuple struct][tuple
  struct pattern], or [slice][slice pattern] pattern where at least one of the
  direct subpatterns is a extending pattern.

So `ref x`, `V(ref x)` and `[ref x, y]` are all extending patterns, but `x`,
`&ref x` and `&(ref x,)` are not.

If the pattern in a `let` statement is an extending pattern then the temporary
scope of the initializer expression is extended.

#### Extending based on expressions

For a let statement with an initializer, an *extending expression* is an
expression which is one of the following:

* The initializer expression.
* The operand of an extending [borrow expression].
* The operand(s) of an extending [array][array expression], [cast][cast
  expression], [braced struct][struct expression], or [tuple][tuple expression]
  expression.
* The final expression of any extending [block expression].

So the borrow expressions in `&mut 0`, `(&1, &mut 2)`, and `Some { 0: &mut 3 }`
are all extending expressions. The borrows in `&0 + &1` and `Some(&mut 0)` are
not: the latter is syntactically a function call expression.

The operand of any extending borrow expression has its temporary scope
extended.

#### Examples

Here are some examples where expressions have extended temporary scopes:

```rust
# fn temp() {}
# trait Use { fn use_temp(&self) -> &Self { self } }
# impl Use for () {}
// The temporary that stores the result of `temp()` lives in the same scope
// as x in these cases.
let x = &temp();
let x = &temp() as &dyn Send;
let x = (&*&temp(),);
let x = { [Some { 0: &temp(), }] };
let ref x = temp();
let ref x = *&temp();
# x;
```

Here are some examples where expressions don't have extended temporary scopes:

```rust,compile_fail
# fn temp() {}
# trait Use { fn use_temp(&self) -> &Self { self } }
# impl Use for () {}
// The temporary that stores the result of `temp()` only lives until the
// end of the let statement in these cases.

let x = Some(&temp());         // ERROR
let x = (&temp()).use_temp();  // ERROR
# x;
```

## Not running destructors

Not running destructors in Rust is safe even if it has a type that isn't
`'static`. [`std::mem::ManuallyDrop`] provides a wrapper to prevent a
variable or field from being dropped automatically.

[Assignment]: expressions/operator-expr.md#assignment-expressions
[binding modes]: patterns.md#binding-modes
[closure]: types/closure.md
[destructors]: destructors.md
[expression]: expressions.md
[identifier pattern]: patterns.md#identifier-patterns
[initialized]: glossary.md#initialized
[interior mutability]: interior-mutability.md
[lazy boolean expression]: expressions/operator-expr.md#lazy-boolean-operators
[place context]: expressions.md#位置表达式和值表达式
[promoted]: destructors.md#constant-promotion
[scrutinee]: glossary.md#scrutinee
[statement]: statements.md
[temporary]: expressions.md#临时位置
[variable]: variables.md

[array]: types/array.md
[enum variant]: types/enum.md
[slice]: types/slice.md
[struct]: types/struct.md
[Trait objects]: types/trait-object.md
[tuple]: types/tuple.md

[slice pattern]: patterns.md#slice-patterns
[struct pattern]: patterns.md#struct-patterns
[tuple pattern]: patterns.md#tuple-patterns
[tuple struct pattern]: patterns.md#tuple-struct-patterns

[array expression]: expressions/array-expr.md#array-expressions
[block expression]: expressions/block-expr.md
[borrow expression]: expressions/operator-expr.md#borrow-operators
[cast expression]: expressions/operator-expr.md#type-cast-expressions
[struct expression]: expressions/struct-expr.md
[tuple expression]: expressions/tuple-expr.md#tuple-expressions

[`for`]: expressions/loop-expr.md#iterator-loops
[`if let`]: expressions/if-expr.md#if-let-expressions
[`if`]: expressions/if-expr.md#if-expressions
[`let` statement]: statements.md#let语句
[`loop`]: expressions/loop-expr.md#infinite-loops
[`match`]: expressions/match-expr.md
[`while let`]: expressions/loop-expr.md#predicate-pattern-loops
[`while`]: expressions/loop-expr.md#predicate-loops

[`<T as std::ops::Drop>::drop`]: ../std/ops/trait.Drop.html#tymethod.drop
[`std::ptr::drop_in_place`]: ../std/ptr/fn.drop_in_place.html
[`std::mem::ManuallyDrop`]: ../std/mem/struct.ManuallyDrop.html
