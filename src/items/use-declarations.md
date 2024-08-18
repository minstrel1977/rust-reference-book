# Use declarations
# Use声明

>[use-declarations.md](https://github.com/rust-lang/reference/blob/master/src/items/use-declarations.md)\
>commit: 65c20b18bc2bb07c666f58cb1232276f0835fd1f \
>本章译文最后维护日期：2024-06-18

> **<sup>句法:</sup>**\
> _UseDeclaration_ :\
> &nbsp;&nbsp; `use` _UseTree_ `;`
>
> _UseTree_ :\
> &nbsp;&nbsp; &nbsp;&nbsp; ([_SimplePath_]<sup>?</sup> `::`)<sup>?</sup> `*`\8
> &nbsp;&nbsp; | ([_SimplePath_]<sup>?</sup> `::`)<sup>?</sup> `{` (_UseTree_ ( `,`  _UseTree_ )<sup>\*</sup> `,`<sup>?</sup>)<sup>?</sup> `}`\
> &nbsp;&nbsp; | [_SimplePath_]&nbsp;( `as` ( [IDENTIFIER] | `_` ) )<sup>?</sup>

*use声明*用来创建一个或多个与程序项[路径][path]同义的本地名称绑定。通常使用 `use`声明来缩短引用模块所需的路径。这些声明可以出现在[模块][modules]和[块][blocks]中，但通常在作用域顶部。
`use`声明有时也被叫做 _导入(import)_ 或则 _再导入(re-export)_。

[path]: ../paths.md
[modules]: modules.md
[blocks]: ../expressions/block-expr.md

use声明支持多种便捷方法:

* 使用带有花括号的句法格式 `use a::b::{c, d, e::f, g::h::i};` 来同时绑定一个系列有共同前缀的路径。
* 使用关键字 `self`，例如 `use a::b::{self, c, d::e};`，来同时绑定一系列有共同前缀和共同父模块的路径。
* 使用句法 `use p::q::r as x;` 将编译目标名称重新绑定为新的本地名称。这种也可以和上面两种方法一起使用：`use a::b::{self as ab, c as abc}`。
* 使用星号通配符句法 `use a::b::*;` 来绑定与给定前缀匹配的所有路径。
* 将前面的方法嵌套重复使用，例如 `use a::b::{self as ab, c, d::{*, e::f}};`。

`use`声明的一个示例：

```rust
use std::collections::hash_map::{self, HashMap};

fn foo<T>(_: T){}
fn bar(map1: HashMap<String, usize>, map2: hash_map::HashMap<String, usize>){}

fn main() {
    // use声明也可以存在于函数体内
    use std::option::Option::{Some, None};

    // 等价于 'foo(vec![std::option::Option::Some(1.0f64), std::option::Option::None]);'
    foo(vec![Some(1.0f64), None]);

    // `hash_map` 和 `HashMap` 在当前作用域内都有效.
    let map1 = HashMap::new();
    let map2 = hash_map::HashMap::new();
    bar(map1, map2);
}
```

## `use` Visibility
## `use`可见性

与其他程序项一样，默认情况下，`use`声明对包含它的模块来说是私有的。同样的，如果使用关键字 `pub` 进行限定，`use`声明也可以是公有的。`use`声明可用于*重导出(re-expor)*名称。因此，公有的 `use`声明可以将某些公有名称重定向到不同的目标定义中：甚至是位于不同模块内具有私有可见性的规范路径定义中。如果这样的重定向序列形成一个循环或不能明确地解析，则会导致编译期错误。

重导出的一个示例：

```rust
mod quux {
    pub use self::foo::{bar, baz};
    pub mod foo {
        pub fn bar() {}
        pub fn baz() {}
    }
}

fn main() {
    quux::bar();
    quux::baz();
}
```

在本例中，模块 `quux` 重导出了在模块 `foo` 中定义的两个公共名称。

## `use` Paths
## `use`路径

`use`项中允许的 [paths] 遵循 [_SimplePath_] 句法，并且类似于表达式中可以使用的路径。
它们可以为以下对象创建绑定关系：

* 可命名的[程序项][items]
* [枚举变体][Enum variants]
* [内置类型][Built-in types]
* [属性][Attributes]
* [派生宏][Derive macros]

`use`项不能导入[关联项][associated items]、[泛型参数][generic parameters]、[局部变量][local variables]、带有[`Self`]的路径或[工具类属性][tool attributes]。更多限制如下所述。

`use`将从导入的实体中为所有[命名空间][namespaces]名称创建绑定，但 `self`导入将仅从类型命名空间导入（如下所述）。
例如，下面演示了如何在两个命名空间中为相同的名称创建绑定：

```rust
mod stuff {
    pub struct Foo(pub i32);
}

// 导入 `Foo`类型和 `Foo'构造函数。
use stuff::Foo;

fn example() {
    let ctor = Foo; // 使用值命名空间中的 `Foo`。
    let x: Foo = ctor(123); // 使用类型命名空间中的 `Foo`。
}
```

> **版次差异**: 在2015版中，`use`路径是相对于当前 crate 的根层的。
> 例如：
>
> ```rust,edition2015
> mod foo {
>     pub mod example { pub mod iter {} }
>     pub mod baz { pub fn foobaz() {} }
> }
> mod bar {
>     // 解析为根层的 `foo`。
>     use foo::example::iter;
>     // 前缀`::` 显式表明 `foo` 来至于根层
>     use ::foo::baz::foobaz;
> }
>
> # fn main() {}
> ```
>
> 2015版不允许使用声明引用[外部预导入包][extern prelude]。
> 因此，在2015版中，在 `use`声明需要指代外部包时仍然需要 [`extern crate`]声明。
> 从2018版开始，`use`声明可以像 `extern crate` 能做的那样指定外部包的依赖关系。

## `as` renames
## `as`改名

`as`关键字可用于更改导入实体的名称。

```rust
// 为函数`foo` 创建非公有别名 `bar`。
use inner::foo as bar;

mod inner {
    pub fn foo() {}
}
```

## Brace syntax
## 大括号句法

大括号可以用在路径的最后一段上，用以从上一段导入多个实体，或者，如果没有上一段，则从当前作用域内导入多个实体。
可以嵌套大括号，从而创建路径树，其中每一组路径分段都与其父对象进行逻辑组合，以创建完整的路径。

```rust
// 创建了以下绑定:
// - `std::collections::BTreeSet`
// - `std::collections::hash_map`
// - `std::collections::hash_map::HashMap`
use std::collections::{BTreeSet, hash_map::{self, HashMap}};
```

空的大括号不会导入任何内容，尽管前导路径已验证为可访问。
<!-- This is slightly wrong, see: https://github.com/rust-lang/rust/issues/61826 -->

> **版次差异**: 在2015版中，路径是相对于 crate 的根层的，因此像 `use {foo, bar};` 这样的导入将从 crate 的根层导入名称 `foo` 和 `bar`，而从2018版开始，这些名称与当前作用域相关。

## `self` imports
## `self`导入

关键字`self` 可以在[大括号句法](#brace-syntax)中使用，以便用父实体自己的名称创建父实体的绑定。

```rust
mod stuff {
    pub fn foo() {}
    pub fn bar() {}
}
mod example {
    // 为 `stuff` 和 `foo` 创建绑定
    use crate::stuff::{self, foo};
    pub fn baz() {
        foo();
        stuff::bar();
    }
}
# fn main() {}
```

`self`仅能给父实体的[类型命名空间][type namespace]中的实体创建绑定。
例如，在下面的示例中，仅导入 `foo`模块：

```rust,compile_fail
mod bar {
    pub mod foo {}
    pub fn foo() {}
}

// 这仅导入模块`foo`。函数`foo`位于值命名空间中，不会被导入。
use bar::foo::{self};

fn main() {
    foo(); //~ 错误 `foo` 是一个模块
}
```

> **注意**: `self`可以用作路径的第一段。
>  `self`作为第一段和其放在 `use`大括号内在逻辑上是相同的；它表示父段为当前模块，或者如果没有父段，则表示当前模块。
> 有关前导 `self` 含义的详细信息，请参阅路径一章中的[`self`]。

## Glob imports
## `*`导入

字符`*` 可以用作 `use`路径的最后一段，用于从前一段的实体中导入所有可导入的实体。
例如：

```rust
// 给 `bar` 创建了一个非公有的别名。
use foo::*;

mod foo {
    fn i_am_private() {}
    enum Example {
        V1,
        V2,
    }
    pub fn bar() {
        // 为枚举变体 `Example` 的 `V1` 和 `V2` 创建了本地别名。
        use Example::*;
        let x = V1;
    }
}
```

从[命名空间][namespace]中的全局导入允许被本的程序项或从同一命名空间中的命名导入遮蔽。
也就是说，如果同一命名空间中的另一程序项已经定义用掉了该名称，则全局导入将被遮蔽。
例如：

```rust
// 这将创建到 `clashing::Foo`元组结构体构造函数的绑定，但不会导入其类型，因为这将与此处定义的 `Foo`结构体冲突。
//
//注意，这里的定义顺序并不重要。
use clashing::*;
struct Foo {
    field: f32,
}

fn do_stuff() {
    // 使用 `clashing::Foo` 的构造函数。
    let f1 = Foo(123);
    // 结构表达式使用上面定义的 `Foo`结构体中的类型。
    let f2 = Foo { field: 1.0 };
    // 由于全局导入，`Bar`也在当前作用域内。
    let z = Bar {};
}

mod clashing {
    pub struct Foo(pub i32);
    pub struct Bar {}
}
```

`*` cannot be used as the first or intermediate segments.
`*` cannot be used to import a module's contents into itself (such as `use self::*;`).
`*` 不能用作路径的第一段或中间段。
`*` 不能用于将模块的内容导入自身中（例如 `use self::*;`）。

> **版次差异**: 在2015版中，路径是相对于 crate 根层的，因此像 `use *；`这样的导入是有效的，这意味着从 crate 根层导入所有内容。
> 但这不能用于 crate根层本身。

## Underscore Imports
## 下划线导入

通过使用形如为 `use path as _` 的带下划线的 use声明，可以在不绑定名称的情况下导入程序项。这对于导入一个 trait 特别有用，这样就可以在不导入 trait 的 symbol 的情况下使用这个 trait 的方法，例如，如果 trait 的 symbol 可能与另一个 symbol 冲突。再一个例子是链接外部的 crate 而不导入其名称。

使用星号全局导入(Asterisk glob imports,即 `::*`)句法将以 `_` 的形式导入能匹配到的所有程序项，但这些程序项在当前作用域中将处于不可命名的状态。
```rust
mod foo {
    pub trait Zoo {
        fn zoo(&self) {}
    }

    impl<T> Zoo for T {}
}

use self::foo::Zoo as _;
struct Zoo;  // 下划线形式的导入就是为了避免和这个程序项在名字上起冲突

fn main() {
    let z = Zoo;
    z.zoo();
}
```

在宏扩展之后会创建一些惟一的、不可命名的 symbols，这样宏就可以安全地扩展出(emit)对 `_` 导入的多个引用。例如，下面代码不应该产生错误:

```rust
macro_rules! m {
    ($item: item) => { $item $item }
}

m!(use std as _;);
// 这会扩展出:
// use std as _;
// use std as _;
```

## Restrictions
## 限制

以下是对 `use`声明合法性的限制：

* `use crate;` 必须使用 `as`别名定义来将 crate的根层绑定到的别名上。
* `use {self};`是错误的；使用 `self` 时必须有前导段。
* 与任何程序项定义一样，`use`导入不能在模块或代码块中的同一命名空间中创建同名的重复绑定。
* [`macro_rules`]扩展中不允许带有`$crate`的 `use`路径。
* `use`路径不能通过[类型别名][type alias]引用枚举变量。例如：

  ```rust,compile_fail
  enum MyEnum {
      MyVariant
  }
  type TypeAlias = MyEnum;

  use MyEnum::MyVariant; //~ OK
  use TypeAlias::MyVariant; //~ ERROR
  ```

## Ambiguities
## 歧义

> **注意**: 本节还未完成。

当 `use`声明所指的名称不明确时，某些情况会导致错误。当有两个同名的候选名称未解析为同一实体时，会发生这种情况。

只要不使用名称，就允许全局导入导入到同一命名空间中形成的冲突的名称。
例如：

```rust
mod foo {
    pub struct Qux;
}

mod bar {
    pub struct Qux;
}

use foo::*;
use bar::*; //~ OK, 没有名称冲突

fn main() {
    // 因为歧义，这里会报错。
    //let x = Qux;
}
```

允许多个全局导入导入相同的名称，并且如果导入的程序项相同（在重新导出后），则允许使用该名称。名称的可见性是导入的有最大可见性的那一个。例如：

```rust
mod foo {
    pub struct Qux;
}

mod bar {
    pub use super::foo::Qux;
}

// 它们都导入相同的 `Qux`。`Qux`的可见性是`pub`，因为这是这两个 `use`声明之间的最大可见性。
pub use bar::*;
use foo::*;

fn main() {
    let _: Qux = Qux;
}
```

[_SimplePath_]: ../paths.md#simple-paths
[`extern crate`]: extern-crates.md
[`macro_rules`]: ../macros-by-example.md
[`self`]: ../paths.md#self
[associated items]: associated-items.md
[Attributes]: ../attributes.md
[Built-in types]: ../types.md
[Derive macros]: ../procedural-macros.md#derive-macros
[Enum variants]: enumerations.md
[extern prelude]: ../names/preludes.md#extern-prelude
[generic parameters]: generics.md
[IDENTIFIER]: ../identifiers.md
[items]: ../items.md
[local variables]: ../variables.md
[namespace]: ../names/namespaces.md
[namespaces]: ../names/namespaces.md
[paths]: ../paths.md
[tool attributes]: ../attributes.md#tool-attributes
[type alias]: type-aliases.md
[type namespace]: ../names/namespaces.md