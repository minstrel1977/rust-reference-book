# 常量项

>[constant-items.md](https://github.com/rust-lang/reference/blob/master/src/items/constant-items.md)\
>commit da910b725a59ba9bb32c6954074f377589a2a689

> **<sup>句法</sup>**\
> _ConstantItem_ :\
> &nbsp;&nbsp; `const` ( [IDENTIFIER] | `_` ) `:` [_Type_] `=` [_Expression_] `;`

*常量项*是一个可选的命名[*常量值*]，它与程序中的具体内存位置没有关联。常量本质上是内联的，无论它们在哪里使用，这意味着它们在使用时是直接复制到相关的上下文中的。这包括使用来自外部 crate 的常量和非 [`Copy`] 类型值。对相同常量的引用不保证引用相同的内存地址。

常量必须显式指定数据类型。类型必须具有 `'static` 生命周期：程序初始化器(initializer)中的任何引用都必须具有 `'static` 生命周期。

常量可以引用其他常量的地址，在这种情况下，该地址将具有省略（如果适用）的生存期，否则（在大多数情况下）默认为 `'static` 生命周期。（请参阅[静态生命周期省略]）。但是，编译器仍然可以自由地任意次数地转换该常量，因此引用的地址可能不稳定。

```rust
const BIT1: u32 = 1 << 0;
const BIT2: u32 = 1 << 1;

const BITS: [u32; 2] = [BIT1, BIT2];
const STRING: &'static str = "bitstring";

struct BitsNStrings<'a> {
    mybits: [u32; 2],
    mystring: &'a str,
}

const BITS_N_STRINGS: BitsNStrings<'static> = BitsNStrings {
    mybits: BITS,
    mystring: STRING,
};
```

## 常量与析构函数

常量可以包含析构函数。析构函数在值超出作用域时运行。

```rust
struct TypeWithDestructor(i32);

impl Drop for TypeWithDestructor {
    fn drop(&mut self) {
        println!("Dropped. Held {}.", self.0);
    }
}

const ZERO_WITH_DESTRUCTOR: TypeWithDestructor = TypeWithDestructor(0);

fn create_and_drop_zero_with_destructor() {
    let x = ZERO_WITH_DESTRUCTOR;
    // x 在函数的结尾处通过调用 drop 方法被销毁。
    // 打印出 "Dropped. Held 0.".
}
```

## 未命名常量

不同于[关联]常量，[自由]常量可以使用下划线而不是名称来命名。例如:

```rust
const _: () =  { struct _SameNameTwice; };

// OK 尽管名称和上面的一样：
const _: () =  { struct _SameNameTwice; };
```

与[下划线导入]一样，宏可以多次安全地在同一作用域中发出相同的未命名常量。例如，以下内容不应该产生错误:

```rust
macro_rules! m {
    ($item: item) => { $item $item }
}

m!(const _: () = (););
// 这会展开出：
// const _: () = ();
// const _: () = ();
```

[关联]: ../glossary.md#associated-item
[*常量值*]: ../const_eval.md#常量表达式
[自由]: ../glossary.md#free-item
[静态生命周期省略]: ../lifetime-elision.md#static-lifetime-elision
[IDENTIFIER]: ../identifiers.md
[下划线的导入]: use-declarations.md#underscore-imports
[_Type_]: ../types.md#type-expressions
[_Expression_]: ../expressions.md
[`Copy`]: ../special-types-and-traits.md#copy
