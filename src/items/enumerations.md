# Enumerations
# 枚举

>[enumerations.md](https://github.com/rust-lang/reference/blob/master/src/items/enumerations.md)\
>commit 2264855271fae0a915a0fa769e57f5a5d09ff5ef

> **<sup>句法</sup>**\
> _Enumeration_ :\
> &nbsp;&nbsp; `enum`
>    [IDENTIFIER]&nbsp;
>    [_Generics_]<sup>?</sup>
>    [_WhereClause_]<sup>?</sup>
>    `{` _EnumItems_<sup>?</sup> `}`
>
> _EnumItems_ :\
> &nbsp;&nbsp; _EnumItem_ ( `,` _EnumItem_ )<sup>\*</sup> `,`<sup>?</sup>
>
> _EnumItem_ :\
> &nbsp;&nbsp; _OuterAttribute_<sup>\*</sup> [_Visibility_]<sup>?</sup>\
> &nbsp;&nbsp; [IDENTIFIER]&nbsp;( _EnumItemTuple_ | _EnumItemStruct_
>                                | _EnumItemDiscriminant_ )<sup>?</sup>
>
> _EnumItemTuple_ :\
> &nbsp;&nbsp; `(` [_TupleFields_]<sup>?</sup> `)`
>
> _EnumItemStruct_ :\
> &nbsp;&nbsp; `{` [_StructFields_]<sup>?</sup> `}`
>
> _EnumItemDiscriminant_ :\
> &nbsp;&nbsp; `=` [_Expression_]

枚举，英文为 *enumeration*，英文简写为 *enum*，它同时定义了一个标称型(nominal)[枚举类型]和一组*构造器*，这可用于创建相应枚举类型的值或对这些值进行模式匹配。

枚举使用关键字 `enum` 来声明。

`enum` 数据项的一个示例和它的使用演示：

```rust
enum Animal {
    Dog,
    Cat,
}

let mut a: Animal = Animal::Dog;
a = Animal::Cat;
```

枚举构造器可以有命名字段或未命名字段：

```rust
enum Animal {
    Dog(String, f64),
    Cat { name: String, weight: f64 },
}

let mut a: Animal = Animal::Dog("Cocoa".to_string(), 37.2);
a = Animal::Cat { name: "Spotty".to_string(), weight: 2.7 };
```

在这个例子中，`Cat` 是一个*类结构体枚举变体*，而 `Dog` 则被简单地称为枚举变体。每个枚举实例都有一个*判别值*，它是一个与之关联的整数，用来确定它持有哪个变体。可以通过 [`mem::discriminant`] 函数获得对这个判别值的不透明引用。

## 为无字段枚举自定义判别值

如果枚举的*任何*变体都没有附加数据，则可以直接设置和访问判别值。

可以使用 `as` 操作符通过[数值转换]将这些枚举类型转换为整数类型。枚举可以可选地指定每个判别值的具体（证书）值，方法是在变体名后面加上 `=` 和[常量表达式]。如果声明中的第一个变量未指定，则将其判别值设置为零。对于其他未指定的判别值，它比照前一个变体的判别值按 1 递增。

```rust
enum Foo {
    Bar,            // 0
    Baz = 123,      // 123
    Quux,           // 124
}

let baz_discriminant = Foo::Baz as u32;
assert_eq!(baz_discriminant, 123);
```

尽管编译器被允许在实际的内存布局中使用较小的类型，但在[默认表形]下，指定的判别值会被解释为一个 `isize` 值。也可以使用[原语表形]或[`C`表形]来更改成大小可接受的值。

两个变量具有相同的判别值是错误的。

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

当前一个变体的判别值是当前表形允许的的最大值时，再使用默认判别值也是错误的。

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

## 无变体枚举

具有零变体的枚举称为*零变体枚举*。因为它们没有有效的值，所以不能被实例化。

```rust
enum ZeroVariants {}
```

零变体枚举与 [*never 类型*]等效，但它不能强制转换为其他类型。

```rust,compile_fail
# enum ZeroVariants {}
let x: ZeroVariants = panic!();
let y: u32 = x; // 类型不匹配错误
```

## 变体的可见性

语法层面枚举变体是允许有自己的[*可见性*][_Visibility_]注解的，但当枚举被验证时，可见性注解又将被拒绝。这允许对各种类型的数据项的在不同上下文中使用统一的句法对项进行解析。

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

// 这种也行，因为这种属性在验证前会被移除。
#[cfg(FALSE)]
enum E {
    pub U,
    pub(crate) T(u8),
    pub(super) T { f: String }
}
```

[IDENTIFIER]: ../identifiers.md
[_Generics_]: generics.md
[_WhereClause_]: generics.md#where子句
[_Expression_]: ../expressions.md
[_TupleFields_]: structs.md
[_StructFields_]: structs.md
[_Visibility_]: ../visibility-and-privacy.md
[枚举类型]: ../types/enum.md
[`mem::discriminant`]: https://doc.rust-lang.org/std/mem/fn.discriminant.html
[*never 类型*]: ../types/never.md
[数值转换]: ../expressions/operator-expr.md#semantics
[常量表达式]: ../const_eval.md#常量表达式
[默认表形]: ../type-layout.md#the-default-representation
[原语表形]: ../type-layout.md#primitive-representations
[`C`表形]: ../type-layout.md#the-c-representation
