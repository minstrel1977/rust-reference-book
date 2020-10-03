# Struct types
# 结构体类型

>[struct.md](https://github.com/rust-lang/reference/blob/master/src/types/struct.md)\
>commit b0e0ad6490d6517c19546b1023948986578fc378

结构体(`struct`)*类型*是其由他类型异构组合的产物，这些其他类型被称为结构体类型的字段。[^structtype]

结构体(`struct`)的新实例可以用[结构体表达式][struct expression]来构造。

默认情况下，结构体(`struct`)的内存布局是未定义的（这样就允许进行一些编译器优化，比如字段重排），但也可以使用[`repr`属性][`repr` attribute] [^译者注]来使其布局在定义时就固定下来。在这两种情况下，字段可以在相应的结构体*表达式*中以任何顺序给出，其结果的结构体(`struct`)值将始终具有相同的内存布局。

结构体(`struct`)的字段可以由[可见性修饰符][visibility modifiers]限定，以允许从模块之外来访问结构体中的数据。

*元组结构体*类型与结构类型类似，只是字段是匿名的。

*类单元结构体*类型类似于结构体类型，只是它没有字段。由关联的[结构体表达式][struct expression]构造的值是驻留在此类类型中的惟一值。

[^structtype]: ../`struct` 类型类似于C中的结构体(`struct`)类型、ML家族的*记录(record)*类型或Lisp家族的*结构体(struct)*类型。
[^译者注]: 注意这里是不带参数的 `repr`属性，这种属性只是把在编译阶段才做的布局优化给提前了。

[`repr` attribute]: ../type-layout.md#representations
[struct expression]: ../expressions/struct-expr.md
[visibility modifiers]: ../visibility-and-privacy.md
