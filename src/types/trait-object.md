# Trait objects
# trait对象

>[trait-object.md](https://github.com/rust-lang/reference/blob/master/src/types/trait-object.md)\
>commit fd10e7043934711ef96b4dd2009db3e4d0182a33

> **<sup>句法</sup>**\
> _TraitObjectType_ :\
> &nbsp;&nbsp; `dyn`<sup>?</sup> [_TypeParamBounds_]
>
> _TraitObjectTypeOneBound_ :\
> &nbsp;&nbsp; `dyn`<sup>?</sup> [_TraitBound_]

*trait对象*是某种类型的不透明的值(opaque value)，该类型实现了一组 trait。这组 trait 是由一个[对象安全的][object safe]*基础trait(base trait)* 加上任意数量的[自动trait][auto traits] 组成的。

trait对象实现基础trait 和它的自动trait 以及基础trait 的任何[超类trait][supertraits]。


trait对象被编写为可选的关键字 `dyn` 后跟一组trait约束，这些trait约束有如此限制：除了第一个 trait 外，所有 trait 都必须是自动trait；生存期不能超过一个；不允许选择退出约束(例如 `?Sized`)。此外，trait 的路径可以用圆括号括起来。

例如，给定一个trait `Trait`，下面是所有 trait对象：
For example, given a trait `Trait`, the following are all trait objects:

* `Trait`
* `dyn Trait`
* `dyn Trait + Send`
* `dyn Trait + Send + Sync`
* `dyn Trait + 'static`
* `dyn Trait + Send + 'static`
* `dyn Trait +`
* `dyn 'static + Trait`.
* `dyn (Trait)`

> **版本差异**：在2015版里，如果 trait对象的第一个约束是以 `::` 开头的路径，那么 `dyn` 会被视为路径的一部分。可以把第一条路径放在圆括号中来解决这个问题。因此，如果希望 trait对象具有 `::your_module::Trait` 路径，那么应该将其写为  `dyn (::your_module::Trait)`。
>
> 从2018版本开始，`dyn` 是一个真正的关键字了，不允许在路径中使用，因此括号就没必要了。
> 
> 
> 注意：为了清晰起见，建议总是在 trait对象上使用关键字 `dyn`，除非你的代码库支持用Rust 1.26 或更低的版本来编译。

如果基础trait 互为别名，如果自动trait 相同且生存期约束相同，则两种 trait对象类型互为别名。例如，`dyn Trait + Send + UnwindSafe` 和 `dyn Trait + UnwindSafe + Send` 是一样的。

由于值的具体类型是不透明的，trait对象的类型是[动态尺寸类型][dynamically sized types]。像所有的 <abbr title="dynamically sized types">DST</abbr> 一样，trait对象被用在某种类型的指针后面；例如 `&dyn SomeTrait` 或 `Box<dyn SomeTrait>`。每个指向 trait对象的指针的实例包括：

 - 一个指向实现 `SomeTrait` 的（那个真实的不透明的）类型 `T` 的实例的指针
 - 一个*虚拟方法表*，它也通常被称为 *虚函数表(vtable)*，它包含了 `T` 实现的 `SomeTrait` 的所有方法，`T` 实现的 `SomeTrait` 的[超类trait][supertraits] 的每个方法，还有一个指向 `T` 的实现的指针（即函数指针）。

trait对象的目的是允许方法的“后期绑定(late binding)”。在 trait对象上调用一个方法会导致运行时的虚拟调度：也就是说，函数指针从 trait对象的虚函数表(vtable)中加载并指示函数被间接调用。每个虚函数表实体的实际实现可能因对象的不同而不同。

一个trait对象的例子：

```rust
trait Printable {
    fn stringify(&self) -> String;
}

impl Printable for i32 {
    fn stringify(&self) -> String { self.to_string() }
}

fn print(a: Box<dyn Printable>) {
    println!("{}", a.stringify());
}

fn main() {
    print(Box::new(10) as Box<dyn Printable>);
}
```

在本例中，trait `Printable` 作为 trait对象出现在 以 `print` 为类型签名函数的参数中，和 `main` 中的 类型转换(cast)表达式中。

## Trait Object Lifetime Bounds
## trait对象的生存期约束

因为 trait对象可以包含引用，所以这些引用的生存期需要表示为 trait对象的一部分。此生存期被写为 `Trait + 'a`。[默认情况][defaults]下，可以通过合理的选择来推断此生存期。

[_TraitBound_]: ../trait-bounds.md
[_TypeParamBounds_]: ../trait-bounds.md
[auto traits]: ../special-types-and-traits.md#auto-traits
[defaults]: ../lifetime-elision.md#default-trait-object-lifetimes
[dynamically sized types]: ../dynamically-sized-types.md
[object safe]: ../items/traits.md#object-safety
[supertraits]: ../items/traits.md#supertraits
