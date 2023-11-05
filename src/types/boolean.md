# Boolean type
# 布尔型

>[boolean.md](https://github.com/rust-lang/reference/blob/master/src/types/boolean.md)\
>commit: d3a66eb69892b25794b2d82a1249ec01d8ead9f1 \
>本章译文最后维护日期：2023-11-05

```rust
let b: bool = true;
```

*布尔型*或*布尔数*是一种可以为*真(`true`)*或*假(`false`)*的原语数据类型。

这种类型的值可以使用[字面量表达式][literal expression]创建，使用关键字 `true` 和 `false` 来表达对应名称的值。

该类型是此[语言的预导入包][language prelude]的一部分，使用[名称][name] `bool` 来表示。

布尔型的对象[内存宽度和对齐量][size and alignment]均为1。false 的位模式为 `0x00`， true 的位模式为 `0x01`。其他的任何其他位模式的布尔型的象都是[未定义的行为][undefined behavior]。

布尔型是多种[表达式][expressions]的操作数的类型:

* [if表达式][if expressions]和 [while表达式][while expressions]中的条件操作数
* [惰性布尔运算表达式][lazy]的操作数

> **注意**：布尔型的行为类似于[枚举类型][enumerated type]，但它确实不是枚举类型。在实践中，这主要意味着构造函数不与类型相关联（例如没有 `bool::true` 这种写法）。

和其他所有的原语类型一样，布尔型[实现][p-impl]了 [`Clone`][p-clone]、[`Copy`][p-copy]、[`Sized`][p-sized]、[`Send`][p-send] 和 [`Sync`][p-sync] 这些 [traits][p-traits]。

> **注意**: 参见[标准库文档][std]中的相关操作运算。

## Operations on boolean values
## 布尔运算

<!-- 这是一种模棱两可的措辞 --> 当使用带有布尔型的操作数的特定操作符表达式时，它们使用[布尔逻辑规则][boolean logic]进行计算。

### Logical not
### 逻辑非

| `b` | [`!b`][op-not] |
|- | - |
| `true` | `false` |
| `false` | `true` |

### Logical or
### 逻辑或

| `a` | `b` | [<code>a &#124; b</code>][op-or] |
|- | - | - |
| `true` | `true` | `true` |
| `true` | `false` | `true` |
| `false` | `true` | `true` |
| `false` | `false` | `false` |

### Logical and
### 逻辑与

| `a` | `b` | [`a & b`][op-and] |
|- | - | - |
| `true` | `true` | `true` |
| `true` | `false` | `false` |
| `false` | `true` | `false` |
| `false` | `false` | `false` |

### Logical xor
### 逻辑异或

| `a` | `b` | [`a ^ b`][op-xor] |
|- | - | - |
| `true` | `true` | `false` |
| `true` | `false` | `true` |
| `false` | `true` | `true` |
| `false` | `false` | `false` |

### Comparisons
### 比较

| `a` | `b` | [`a == b`][op-compare] |
|- | - | - |
| `true` | `true` | `true` |
| `true` | `false` | `false` |
| `false` | `true` | `false` |
| `false` | `false` | `true` |

| `a` | `b` | [`a > b`][op-compare] |
|- | - | - |
| `true` | `true` | `false` |
| `true` | `false` | `true` |
| `false` | `true` | `false` |
| `false` | `false` | `false` |

* `a != b` 等同于 `!(a == b)`
* `a >= b` 等同于 `a == b | a > b`
* `a < b` 等同于 `!(a >= b)`
* `a <= b` 等同于 `a == b | a < b`

## Bit validity
## 位有效性

Rust会保证 `bool`类型的单个字节被初始化（换句话说，`transmute::<bool, u8>(...)` 总是健全(sound)的——但由于一些位模式是无效的 `bool`，因此相反的操作并不总是健全的）。

[boolean logic]: https://en.wikipedia.org/wiki/Boolean_algebra
[enumerated type]: enum.md
[expressions]: ../expressions.md
[if expressions]: ../expressions/if-expr.md#if-expressions
[language prelude]: ../names/preludes.md#language-prelude
[lazy]: ../expressions/operator-expr.md#lazy-boolean-operators
[literal expression]: ../expressions/literal-expr.md
[name]: ../names.md
[op-and]: ../expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[op-compare]: ../expressions/operator-expr.md#comparison-operators
[op-not]: ../expressions/operator-expr.md#negation-operators
[op-or]: ../expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[op-xor]: ../expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[p-clone]: ../special-types-and-traits.md#clone
[p-copy]: ../special-types-and-traits.md#copy
[p-impl]: ../items/implementations.md
[p-send]: ../special-types-and-traits.md#send
[p-sized]: ../special-types-and-traits.md#sized
[p-sync]: ../special-types-and-traits.md#sync
[p-traits]: ../items/traits.md
[size and alignment]: ../type-layout.md#size-and-alignment
[std]: ../../std/primitive.bool.html
[undefined behavior]: ../behavior-considered-undefined.md
[while expressions]: ../expressions/loop-expr.md#predicate-loops
