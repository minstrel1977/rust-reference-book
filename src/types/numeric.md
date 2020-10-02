# Numeric types
# 数字类类型

>[mumeric.md](https://github.com/rust-lang/reference/blob/master/src/types/mumeric.md)\
>commit 73ca198fb3ab52283d67d5fe28c541ee1d169f48

## Integer types
## 整型/整数类型

无符号整数类型：

类型   | 最小值 | 最大值
-------|---------|-------------------
`u8`   | 0       | 2<sup>8</sup>-1
`u16`  | 0       | 2<sup>16</sup>-1
`u32`  | 0       | 2<sup>32</sup>-1
`u64`  | 0       | 2<sup>64</sup>-1
`u128` | 0       | 2<sup>128</sup>-1

有符号整数类型：

类型   | 最小值            | 最大值
-------|--------------------|-------------------
`i8`   | -(2<sup>7</sup>)   | 2<sup>7</sup>-1
`i16`  | -(2<sup>15</sup>)  | 2<sup>15</sup>-1
`i32`  | -(2<sup>31</sup>)  | 2<sup>31</sup>-1
`i64`  | -(2<sup>63</sup>)  | 2<sup>63</sup>-1
`i128` | -(2<sup>127</sup>) | 2<sup>127</sup>-1


## Floating-point types
## 浮点型

Rust 对应 IEEE 754-2008 的“binary32”和“binary64”浮点类型分别是 `f32` 和 `f64`。

## Machine-dependent integer types
## 和计算平台相关的整型

`usize`类型是一种无符号整型，其位数与平台的指针类型相同。它可以表示进程中的每个内存地址。

`isize`类型是一种有符号整型，其位数与平台的指针类型相同。对象的尺寸和数组的长度的理论上限是 `isize`的最大值。这确保了 `isize` 可以用来计算（指向对象或数组的）指针之间的差异，并且可以寻址对象中的每个字节以及结束后的下一个字节。

`usize` 和 `isize` 至少是16位宽。

> **注意**：许多 Rust 代码可能会假设指针、`usize` 和 `isize` 是32位或64位的。因此，16位指针的支持是有限的，可能需要来自库的明确关注和确认才能支持。
