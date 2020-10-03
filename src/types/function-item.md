# Function item types
# 函数项类型

>[function-item.md](https://github.com/rust-lang/reference/blob/master/src/types/function-item.md)\
>commit b0e0ad6490d6517c19546b1023948986578fc378

当引用函数项、元组结构体的构造函数或枚举变量的构造函数时，会产生它们的*函数项类型(function item type)*的值(该值实际是一个内存大小为0的标识符)。该值会显式标识指代该函数——它的名字、它的类型参数，及其早期绑定的生命周期参数(不是后期绑定的生命周期参数,这个生命周期只在函数被调用时才赋值)——所以该值不需要包含一个实际的函数指针，那调用函数时就也不需要一个间接的寻址操作去内存中查找要调用的函数了。

没有直接引用函数项类型的句法，但是编译器会在错误消息中显示类似于 `fn(u32) -> i32 {fn_name}` 这样的“类型”。

因为函数项类型显式地标识了其函数，所以不同函数的数据项类型(不同的数据项或具有不同泛型的相同的数据项)是不同的，混合使用它们将导致类型错误：

```rust,compile_fail,E0308
fn foo<T>() { }
let x = &mut foo::<i32>;
*x = foo::<u32>; //~ 错误：类型不匹配
```

但确实存在从函数项到具有相同签名的函数指针的[强转(coercion)][coercion]。这种强转一般发生在使用函数项却直接预期函数指针；另一种发生情况是 `if`或 `match`表达式的不同分支返回使用了具有相同签名却不同函数项类型的情况：

```rust
# let want_i32 = false;
# fn foo<T>() { }

// 这里 `foo_ptr_1` 标注使用了 `fn()` 这个函数指针类型。
let foo_ptr_1: fn() = foo::<i32>;

// ... `foo_ptr_2` 也行 - 这次是通过类型检查做到的。
let foo_ptr_2 = if want_i32 {
    foo::<i32>
} else {
    foo::<u32>
};
```

所有的函数项都实现了 [`Fn`]、[`FnMut`]、[`FnOnce`]、[`Copy`]、[`Clone`]、[`Send`] 和 [`Sync`]。

[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`FnMut`]: https://doc.rust-lang.org/std/ops/trait.FnMut.html
[`FnOnce`]: https://doc.rust-lang.org/std/ops/trait.FnOnce.html
[`Fn`]: https://doc.rust-lang.org/std/ops/trait.Fn.html
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[coercion]: ../type-coercions.md
[function pointers]: function-pointer.md
