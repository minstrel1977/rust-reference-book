# Enumerations
# 枚举

>[enumerations.md](https://github.com/rust-lang/reference/blob/master/src/items/enumerations.md)\
>commit: 585407f04bba5d154342ff305b3385bc555d1468 \
>本章译文最后维护日期：2024-08-17

> **<sup>句法</sup>**\
> _Enumeration_ :\
> &nbsp;&nbsp; `enum`
>    [IDENTIFIER]&nbsp;
>    [_GenericParams_]<sup>?</sup>
>    [_WhereClause_]<sup>?</sup>
>    `{` _EnumItems_<sup>?</sup> `}`
>
> _EnumItem_ :\
> &nbsp;&nbsp; _OuterAttribute_<sup>\*</sup> [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; [IDENTIFIER]&nbsp;( _EnumItemTuple_ | _EnumItemStruct_ )<sup>?</sup>
>                                _EnumItemDiscriminant_<sup>?</sup>
>
> _EnumItemTuple_ :\
> &nbsp;&nbsp; `(` [_TupleFields_]<sup>?</sup> `)`
>
> _EnumItemStruct_ :\
> &nbsp;&nbsp; `{` [_StructFields_]<sup>?</sup> `}`
>
> _EnumItemDiscriminant_ :\
> &nbsp;&nbsp; `=` [_Expression_]

*枚举*，英文为 *enumeration*，常见其简写形式 *enum*，它同时定义了一个标称型(nominal)[枚举类型][enumerated type]和一组*构造器*，这可用于创建或使用模式来匹配相应枚举类型的值。

枚举使用关键字 `enum` 来声明。
`enum`声明在其所在的模块或块的[类型命名空间][type namespace]中定义枚举类型。

`enum` 程序项的一个示例和它的使用方法：

```rust
enum Animal {
    Dog,
    Cat,
}

let mut a: Animal = Animal::Dog;
a = Animal::Cat;
```

枚举构造器可以带有具名字段或未具名字段：

```rust
enum Animal {
    Dog(String, f64),
    Cat { name: String, weight: f64 },
}

let mut a: Animal = Animal::Dog("Cocoa".to_string(), 37.2);
a = Animal::Cat { name: "Spotty".to_string(), weight: 2.7 };
```

在这个例子中，`Cat` 是一个*类结构体枚举变体(struct-like enum variant)*，而 `Dog` 则被简单地称为枚举变体。
变体都不带字段的枚举被称为*<span id="field-less-enum">无字段枚举</span>*。比如下面这个就是无字段枚举：

```rust
enum Fieldless {
    Tuple(),
    Struct{},
    Unit,
}
```

如果无字段枚举枚举的变体全是单元体变体，这类枚举也被称为*<span id="unit-only-enum">纯单元体枚举</span>*。比如：

```rust
enum Enum {
    Foo = 3,
    Bar = 2,
    Baz = 1,
}
```

枚举变体的构造函数类似于[结构体][struct]定义，并且可以用枚举名称中的路径来引用，这些引用可以直接用在 [use声明][use declarations]中。
每个枚举变体都在[类型命名空间][type namespace]中定义其类型，尽管该类型不能被用作类型识别符。
类元组和类单元变体也在[值命名空间][value namespace]中定义了其构造函数。

类结构体变体可以用[结构体表达式][struct expression]实例化。
类元组的变体可以用[调用表达式][call expression]或[结构体表达式][struct expression]实例化。
类单元变体可以用[路径表达式][path expression]或[结构体表达式][struct expression]实例化。
例如：

```rust
enum Examples {
    UnitLike,
    TupleLike(i32),
    StructLike { value: i32 },
}

use Examples::*; // 为每个变体都创建了别名。
let x = UnitLike; // 常量项的路径表达式
let x = UnitLike {}; // 结构体表达式
let y = TupleLike(123); // 调用表达式
let y = TupleLike { 0: 123 }; // 使用整型数字作为字段名的结构表达式
let z = StructLike { value: 123 }; // 结构体表达式
```

<span id="custom-discriminant-values-for-fieldless-enumerations"></span>
## Discriminants
## 判别值

每个枚举实例都有一个判别值：一个逻辑上与其关联的整数，用于确定它持有的是哪个变体。

在[默认表型][default representation]下，判别值被解释为一个 `isize` 的值。然而，编译器允许在其实际内存布局中使用内存尺寸更小的类型（或其他区分变体的方法）。
### Assigning discriminant values
### 指定判别值的值

#### Explicit discriminants
#### 显式判别值

在两种情况下，变体的判别值可以通过在变量名称后面加上 `=` 和[常量表达式][constant expression]来显式设置：

1. 如果枚举变体全是"[单元体][unit-only]".


2. 如果使用了[原语表型][primitive representation]。例如：

   ```rust
   #[repr(u8)]
   enum Enum {
       Unit = 3,
       Tuple(u16),
       Struct {
           a: u8,
           b: u16,
       } = 1,
   }
   ```

#### Implicit discriminants
#### 隐式判别值

如果未指定变体的判别值，则将其的判别值设置为比当前声明中前一个变体的判别值加一。如果声明中第一个变体的判别值未指定，则将其设置为零。

```rust
enum Foo {
    Bar,            // 0
    Baz = 123,      // 123
    Quux,           // 124
}

let baz_discriminant = Foo::Baz as u32;
assert_eq!(baz_discriminant, 123);
```

同一枚举中，两个变体使用相同的判别值是错误的。

```rust,compile_fail
enum SharedDiscriminantError {
    SharedA = 1,
    SharedB = 1
}

enum SharedDiscriminantError2 {
    Zero,       // 0
    One,        // 1
    OneToo = 1  // 1 (和前值冲突！)
}
```

当前一个变体的判别值是当前表形允许的的最大值时，再使用默认判别值就也是错误的。

```rust,compile_fail
#[repr(u8)]
enum OverflowingDiscriminantError {
    Max = 255,
    MaxPlusOne // 应该是256，但枚举溢出了
}

#[repr(u8)]
enum OverflowingDiscriminantError2 {
    MaxMinusOne = 254, // 254
    Max,               // 255
    MaxPlusOne         // 应该是256，但枚举溢出了。
}
```
### Accessing discriminant
### 读取判别值
#### Via `mem::discriminant`
#### 通过 `mem::discriminant`

[`mem::discriminant`] 返回对枚举值的判别值(可用于比较)的不透明引用。但它不能用于获得判别式的值。

#### 转换

如果枚举是[单元体枚举][unit-only]（没有元组和结构体变体），则可以使用[数字类型转换][numeric cast]直接访问其判别值；例如。：

```rust
enum Enum {
    Foo,
    Bar,
    Baz,
}

assert_eq!(0, Enum::Foo as isize);
assert_eq!(1, Enum::Bar as isize);
assert_eq!(2, Enum::Baz as isize);
```

如果没有显式判别值，或者只有单元体变体是显式指定的，则[无字段枚举][Field-less enums]也可以转换。

```rust
enum Fieldless {
    Tuple(),
    Struct{},
    Unit,
}

assert_eq!(0, Fieldless::Tuple() as isize);
assert_eq!(1, Fieldless::Struct{} as isize);
assert_eq!(2, Fieldless::Unit as isize);

#[repr(u8)]
enum FieldlessWithDiscrimants {
    First = 10,
    Tuple(),
    Second = 20,
    Struct{},
    Unit,
}

assert_eq!(10, FieldlessWithDiscrimants::First as u8);
assert_eq!(11, FieldlessWithDiscrimants::Tuple() as u8);
assert_eq!(20, FieldlessWithDiscrimants::Second as u8);
assert_eq!(21, FieldlessWithDiscrimants::Struct{} as u8);
assert_eq!(22, FieldlessWithDiscrimants::Unit as u8);
```

#### Pointer casting
#### 指针转换

如果枚举指定应用了[原语表型][primitive representation]，则可以通过 unsafe指针转换访问判别值：

```rust
#[repr(u8)]
enum Enum {
    Unit,
    Tuple(bool),
    Struct{a: bool},
}

impl Enum {
    fn discriminant(&self) -> u8 {
        unsafe { *(self as *const Self as *const u8) }
    }
}

let unit_like = Enum::Unit;
let tuple_like = Enum::Tuple(true);
let struct_like = Enum::Struct{a: false};

assert_eq!(0, unit_like.discriminant());
assert_eq!(1, tuple_like.discriminant());
assert_eq!(2, struct_like.discriminant());
```

## Zero-variant enums
## 无变体枚举

没有变体的枚举称为*零变体枚举/无变体枚举*。因为它们没有有效的值，所以不能被实例化。

```rust
enum ZeroVariants {}
```

零变体枚举与 [*never类型*][never type]等效，但它不能被强转为其他类型。

```rust,compile_fail
# enum ZeroVariants {}
let x: ZeroVariants = panic!();
let y: u32 = x; // 类型不匹配错误
```

## Variant visibility
## 变体的可见性

依照句法规则，枚举变体是允许有自己的[*可见性(visibility)*][Visibility]限定/注解(annotation)的，但当枚举被（句法分析程序）验证(validate)通过后，可见性注解又被弃用。因此，在源码解析层面，允许跨不同的上下文对其中不同类型的程序项使用统一的句法规则进行解析。

```rust
macro_rules! mac_variant {
    ($vis:vis $name:ident) => {
        enum $name {
            $vis Unit,

            $vis Tuple(u8, u16),

            $vis Struct { f: u8 },
        }
    }
}

// 允许空 `vis`.
mac_variant! { E }

// 这种也行，因为这段代码在被验证通过前会被移除。
#[cfg(FALSE)]
enum E {
    pub U,
    pub(crate) T(u8),
    pub(super) T { f: String }
}
```

[_Expression_]: ../expressions.md
[_GenericParams_]: generics.md
[_StructFields_]: structs.md
[_TupleFields_]: structs.md
[_Visibility_]: ../visibility-and-privacy.md
[_WhereClause_]: generics.md#where-clauses
[`C` representation]: ../type-layout.md#the-c-representation
[`mem::discriminant`]: https://doc.rust-lang.org/std/mem/fn.discriminant.html
[call expression]: ../expressions/call-expr.md
[constant expression]: ../const_eval.md#constant-expressions
[default representation]: ../type-layout.md#the-default-representation
[enumerated type]: ../types/enum.md
[Field-less enums]: #field-less-enum
[IDENTIFIER]: ../identifiers.md
[never type]: ../types/never.md
[numeric cast]: ../expressions/operator-expr.md#semantics
[path expression]: ../expressions/path-expr.md
[primitive representation]: ../type-layout.md#primitive-representations
[struct expression]: ../expressions/struct-expr.md
[struct]: structs.md
[type namespace]: ../names/namespaces.md
[unit-only]: #unit-only-enum
[use declarations]: use-declarations.md
[value namespace]: ../names/namespaces.md