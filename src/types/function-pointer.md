# Function pointer types
# 函数指针类型

>[function-pointer.md](https://github.com/rust-lang/reference/blob/master/src/types/function-pointer.md)\
>commit 3d1b9ae5e7a61da43ac83cc42815e29b34b350ba

> **<sup>句法</sup>**\
> _BareFunctionType_ :\
> &nbsp;&nbsp; [_ForLifetimes_]<sup>?</sup> [_FunctionQualifiers_] `fn`\
> &nbsp;&nbsp; &nbsp;&nbsp;  `(` _FunctionParametersMaybeNamedVariadic_<sup>?</sup> `)` _BareFunctionReturnType_<sup>?</sup>
>
> _BareFunctionReturnType_:\
> &nbsp;&nbsp; `->` [_TypeNoBounds_]
>
> _FunctionParametersMaybeNamedVariadic_ :\
> &nbsp;&nbsp; _MaybeNamedFunctionParameters_ | _MaybeNamedFunctionParametersVariadic_
>
> _MaybeNamedFunctionParameters_ :\
> &nbsp;&nbsp; _MaybeNamedParam_ ( `,` _MaybeNamedParam_ )<sup>\*</sup> `,`<sup>?</sup>
>
> _MaybeNamedParam_ :\
> &nbsp;&nbsp; [_OuterAttribute_]<sup>\*</sup> ( ( [IDENTIFIER] | `_` ) `:` )<sup>?</sup> [_Type_]
>
> _MaybeNamedFunctionParametersVariadic_ :\
> &nbsp;&nbsp; ( _MaybeNamedParam_ `,` )<sup>\*</sup> _MaybeNamedParam_ `,` [_OuterAttribute_]<sup>\*</sup> `...`

函数指针类型（使用关键字 `fn` 编写）指的是在编译时不必知道其标识符的函数。它们可以由[函数项][function items]或非捕获( non-capturing)[闭包][closures]经过一次自动强转来创建。

非安全(`unsafe`)限定符表示类型的值是一个[非安全函数][unsafe function]，而 `extern` 限定符表示它是一个[外部函数][extern function]。

可变参数只能通过 [`extern`]函数类型使用 `"C"` 或 `"cdecl"` 的调用约定来指定。

下面示例中 `Binop` 被定义为函数指针类型：

```rust
fn add(x: i32, y: i32) -> i32 {
    x + y
}

let mut x = add(5,7);

type Binop = fn(i32, i32) -> i32;
let bo: Binop = add;
x = bo(5,7);
```

## Attributes on function pointer parameters
## 函数指针参数上的属性

函数指针参数上的属性遵循与[常规函数参数][regular function parameters]相同的规则和限制

[IDENTIFIER]: ../identifiers.md
[_ForLifetimes_]: ../items/generics.md#where子句
[_FunctionQualifiers_]: ../items/functions.md
[_TypeNoBounds_]: ../types.md#type-expressions
[_Type_]: ../types.md#type-expressions
[_OuterAttribute_]: ../attributes.md
[`extern`]: ../items/external-blocks.md
[closures]: closure.md
[extern function]: ../items/functions.md#函数参数上的属性
[function items]: function-item.md
[unsafe function]: ../unsafe-functions.md
[regular function parameters]: ../items/functions.md#函数参数上的属性
<!-- 2020-10-16 -->