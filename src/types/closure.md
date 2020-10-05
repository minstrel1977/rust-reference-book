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

编译器倾向于优先通过不可变借用(immutable borrow)，其次唯一不可变借用(unique immutable borrow)(见下文)，再其次可变借用(mutable borrow)，最后移动(move)来捕获一个闭合变量(closed-over variable)。编译器将选择允许闭包编译通过的第一个选项。这个选择只与闭包表达式的内容有关；编译器不考虑闭包之外的代码，比如所涉及的变量的生存期。

如果使用 `move` 关键字，那么所有捕获都是通过移动(move)进行的，当然对于复制类型，则是通过Copy进行的，而不管借用是否可用。`move` 关键字通常用于允许闭包比其捕获的值活得更久，例如要返回闭包或用于生成新线程。

复合类型(如结构体、元组和枚举)始终是全部捕获的，而不是单个字段分开捕获的。如果真要捕获单个字段，那可能需要先借用该字段到本地局部变量中。

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

相反，如果闭包要直接使用 `self.vec`，那么它将尝试通过可变引用捕获 `self`。但是因为 `self.set` 已经被借用用来迭代，代码将无法编译。

## Unique immutable borrows in captures
## 捕获中的唯一不可变借用

捕获方式中有一种被称为*唯一不可变借用*的特殊类型的借用捕获，这种借用不能在语言的其他任何地方使用，也不能显式地写出。它发生在修改可变引用的引用时，如下面的示例所示：

```rust
let mut b = false;
let x = &mut b;
{
    let mut c = || { *x = true; };
    // 下行代码不正确
    // let y = &x;
    c();
}
let z = &x;
```

在这种情况下，不可能可变借用 `x`，因为 `x` 没有标注 `mut`（，所以上面闭包对 `x` 是可变借用）。但如果真能可变借用 `x`，那对其赋值有会非法，因为 `& &mut` 引用可能不是唯一的，因此不能安全地用于修改值。所以这里闭包使用了唯一不可变借用：它不可变地借用 `x`，但是又像可变借用一样，当然前提是此借用必须是唯一的。在上面的例子中，取消注释 `y` 的声明将产生错误，因为这将违反闭包对 `x` 的借用的唯一性；`z` 的声明是有效的，因为闭包的生存期在块结束时已过期，从而已经释放了对 `x` 的借用。

## Call traits and coercions
## 调用trait 和强转

闭包类型都实现了 [`FnOnce`]，这表示它们可以通过消耗掉闭包的所有权来调用执行一次。另外，一些闭包实现了更具体的调用trait：

* 对其捕获变量没采用移动方式(译者注：变量的原始所有者仍拥有该变量的所有权)的闭包实现了 [`FnMut`]，这表明它可以被可变引用调用。

* 对其捕获变量没采用移动方式，也没修改其捕获变量的闭包实现了 [`Fn`]，这表明它可以通过共享引用调用。

> 注意：`move`闭包可能仍然实现 [`Fn`] 或 [`FnMut`]，即便它们通过移动(move)方式来捕获变量。这是因为闭包类型实现的 trait 是由闭包对捕获的值做什么决定的，而不是由它如何捕获值决定的。

*非捕获闭包(Non-capturing closures)*是指不捕获环境中的任何变量的闭包。它们可以通过匹配签名的方式被强转成函数指针(例如 `fn()`)。

```rust
let add = |x, y| x + y;

let mut x = add(5,7);

type Binop = fn(i32, i32) -> i32;
let bo: Binop = add;
x = bo(5,7);
```

## Other traits
## 其他 trait

所有闭包类型都实现了 [`Sized`]。此外，捕获的变量的类型如果实现了如下 trait，闭包类型也可能会自动实现这些 trait：

* [`Clone`]
* [`Copy`]
* [`Sync`]
* [`Send`]

闭包类型实现 [`Send`] 和 [`Sync`] 的规则与普通结构体类型实现这俩 trait 的规则一样，而 [`Clone`] 和 [`Copy`] 就像 [`derived`][derived]属性中的一样。对于 [`Clone`]，捕获变量的克隆顺序目前还没正式规范出来。

由于捕获通常是通过引用进行的，因此会出现以下一般规则：

* 如果所有的捕获变量都实现了 [`Sync`]，则此闭包就也实现了 [`Sync`] 。
* 如果所有非唯一不可变引用捕获的变量都实现了 [`Sync`]，并且所有由唯一不可变、可变引用、复制或移动方式捕获的值都实现了 [`Send`]，则此闭包就也实现了 [`Send`]。
* 如果一个闭包没有通过唯一不可变或可变引用捕获任何值，并且它通过复制或移动捕获的所有值都分别实现了 [`Clone`] 或 [`Copy`]，则此闭包就也实现了 [`Clone`] 或 [`Copy`]。

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
