# Closure types
# 闭包类型

>[closure.md](https://github.com/rust-lang/reference/blob/master/src/types/closure.md)\
>commit 5642af891714145cb2a765f244fff7d6b618a4c7

[闭包表达式][closure expression]生成的闭包值具有唯一性和无法写出的匿名性。闭包类型大约相当于包含捕获变量的结构体。比如以下闭包示例：

```rust
fn f<F : FnOnce() -> String> (g: F) {
    println!("{}", g());
}

let mut s = String::from("foo");
let t = String::from("bar");

f(|| {
    s += &t;
    s
});
// 打印 "foobar".
```

生成大致如下所示的闭包类型：

<!-- ignore: simplified, requires unboxed_closures, fn_traits -->
```rust,ignore
struct Closure<'a> {
    s : String,
    t : &'a String,
}

impl<'a> FnOnce<()> for Closure<'a> {
    type Output = String;
    fn call_once(self) -> String {
        self.s += &*self.t;
        self.s
    }
}
```

所以调用 `f` 相当于：

<!-- ignore: continuation of above -->
```rust,ignore
f(Closure{s: s, t: &t});
```

## Capture modes
## 捕获方式

编译器倾向于优先通过不可变借用，其次唯一不可变借用(见下文)，再其次可变借用，最后移动来捕获一个闭合变量(closed-over variable)。编译器将选择允许闭包编译的第一个选项。这个选择只与闭包表达式的内容有关;编译器不考虑周围的代码，比如所涉及的变量的生存期。
编译器更喜欢通过不可变的借位，然后是唯一的不可变的借位（见下文），通过可变借位，最后通过移动来捕获一个闭合变量。它将选择允许编译闭包的第一个选项。只对闭包表达式的内容做出选择；编译器不考虑周围的代码，例如相关变量的生存期。
The compiler prefers to capture a closed-over variable by immutable borrow, followed by unique immutable borrow (see below), by mutable borrow, and finally by move. It will pick the first choice of these that allows the closure to compile. The choice is made only with regards to the contents of the closure expression; the compiler does not take into account surrounding code, such as the lifetimes of involved variables.

If the `move` keyword is used, then all captures are by move or, for `Copy`
types, by copy, regardless of whether a borrow would work. The `move` keyword is
usually used to allow the closure to outlive the captured values, such as if the
closure is being returned or used to spawn a new thread.

Composite types such as structs, tuples, and enums are always captured entirely,
not by individual fields. It may be necessary to borrow into a local variable in
order to capture a single field:

```rust
# use std::collections::HashSet;
#
struct SetVec {
    set: HashSet<u32>,
    vec: Vec<u32>
}

impl SetVec {
    fn populate(&mut self) {
        let vec = &mut self.vec;
        self.set.iter().for_each(|&n| {
            vec.push(n);
        })
    }
}
```

If, instead, the closure were to use `self.vec` directly, then it would attempt
to capture `self` by mutable reference. But since `self.set` is already
borrowed to iterate over, the code would not compile.

## Unique immutable borrows in captures

Captures can occur by a special kind of borrow called a _unique immutable
borrow_, which cannot be used anywhere else in the language and cannot be
written out explicitly. It occurs when modifying the referent of a mutable
reference, as in the following example:

```rust
let mut b = false;
let x = &mut b;
{
    let mut c = || { *x = true; };
    // The following line is an error:
    // let y = &x;
    c();
}
let z = &x;
```

In this case, borrowing `x` mutably is not possible, because `x` is not `mut`.
But at the same time, borrowing `x` immutably would make the assignment illegal,
because a `& &mut` reference may not be unique, so it cannot safely be used to
modify a value. So a unique immutable borrow is used: it borrows `x` immutably,
but like a mutable borrow, it must be unique. In the above example, uncommenting
the declaration of `y` will produce an error because it would violate the
uniqueness of the closure's borrow of `x`; the declaration of z is valid because
the closure's lifetime has expired at the end of the block, releasing the borrow.

## Call traits and coercions

Closure types all implement [`FnOnce`], indicating that they can be called once
by consuming ownership of the closure. Additionally, some closures implement
more specific call traits:

* A closure which does not move out of any captured variables implements
  [`FnMut`], indicating that it can be called by mutable reference.

* A closure which does not mutate or move out of any captured variables
  implements [`Fn`], indicating that it can be called by shared reference.

> Note: `move` closures may still implement [`Fn`] or [`FnMut`], even though
> they capture variables by move. This is because the traits implemented by a
> closure type are determined by what the closure does with captured values,
> not how it captures them.

*Non-capturing closures* are closures that don't capture anything from their
environment. They can be coerced to function pointers (e.g., `fn()`)
with the matching signature.

```rust
let add = |x, y| x + y;

let mut x = add(5,7);

type Binop = fn(i32, i32) -> i32;
let bo: Binop = add;
x = bo(5,7);
```

## Other traits

All closure types implement [`Sized`]. Additionally, closure types implement the
following traits if allowed to do so by the types of the captures it stores:

* [`Clone`]
* [`Copy`]
* [`Sync`]
* [`Send`]

The rules for [`Send`] and [`Sync`] match those for normal struct types, while
[`Clone`] and [`Copy`] behave as if [derived]. For [`Clone`], the order of
cloning of the captured variables is left unspecified.

Because captures are often by reference, the following general rules arise:

* A closure is [`Sync`] if all captured variables are [`Sync`].
* A closure is [`Send`] if all variables captured by non-unique immutable
  reference are [`Sync`], and all values captured by unique immutable or mutable
  reference, copy, or move are [`Send`].
* A closure is [`Clone`] or [`Copy`] if it does not capture any values by
  unique immutable or mutable reference, and if all values it captures by copy
  or move are [`Clone`] or [`Copy`], respectively.

[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`FnMut`]: https://doc.rust-lang.org/std/ops/trait.FnMut.html
[`FnOnce`]: https://doc.rust-lang.org/std/ops/trait.FnOnce.html
[`Fn`]: https://doc.rust-lang.org/std/ops/trait.Fn.html
[`Send`]: ../special-types-and-traits.md#send
[`Sized`]: ../special-types-and-traits.md#sized
[`Sync`]: ../special-types-and-traits.md#sync
[closure expression]: ../expressions/closure-expr.md
[derived]: ../attributes/derive.md
