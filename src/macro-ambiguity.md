# Appendix：Macro Follow-Set Ambiguity Formal Specification

>[macro-ambiguity.md](https://github.com/rust-lang/reference/blob/master/src/macro-ambiguity.md)\
>commit:  184b056086757e89d68f41c8c0e42721cb50a4a9 \
>本译文最后维护日期：2020-11-3

本页介绍了下述[声明宏][Macros By Example]的规则的正式规范。它们最初是在 [RFC550] 中指定的，本文的大部分内容都是从其中复制过来的，并在后续的RFC中进行展开。

## Definitions & Conventions
## 定义和约定

  - `macro`：宏，源代码中任何可调用的类似 `foo!(...)` 的东西。
  - `MBE`：macro-by-example，声明宏，由 `macro_rules` 定义的宏。
  - `matcher`：匹配器，`macro_rules`调用中一条规则的左侧部分，或其子部分。
  - `macro parser`：宏解释器，Rust 解析器中的一段代码，它收集使用所有匹配器的句法规则来解析输入。 <!-- the bit of code in the Rust parser that will parse the input using a grammar derived from all of the matchers. tobemodify-->
  - `fragment`：匹配段，给定匹配器将接受(或“匹配”)的 Rust 句法对象；<!-- The class of Rust syntax that a given matcher will accept (or "match"). tobemodify-->
  - `repetition` ：重复段，遵循常规性重复模式的匹配段。<!-- a fragment that follows a regular repeating pattern. tobemodify-->
  - `NT`：non-terminal，非终结符，可以出现在匹配器中的各种“元变量”或重复段匹配器，在声明宏(MBE)句法中用前导的 `$` 字符指定。<!-- non-terminal, the various "meta-variables" or repetition matchers that can appear in a matcher, specified in MBE syntax with a leading `$` character. -->
  - `simple NT`：简单NT，“元变量”类型的非终结符(下面会进一步讨论)。
  - `complex NT`：复杂NT，有重复段匹配规则的非终结符，通过重复操作符(`\*`, `+`, `?`)指定。<!-- a repetition matching non-terminal, specified via repetition operators (`\*`, `+`, `?`). -->
  - `token`：标记码，匹配器中不可再细分的元素；例如，标识符、操作符、开/闭定界符、*和*简单NT(simple NT)。
  - `token tree`：标记树，标记树由标记码(叶)、复杂NT和子标记树（标记树的有限序列）的组成的树形数据结构。
  - `delimiter token`：定界符，一种标记，用于划分一个匹配段的结束和下一个匹配段的开始。
  - `separator token`：分隔符，复杂NT中的可选定界符，用于在匹配的重复中，以分隔每个元素对。
  - `separated complex NT`：带分隔符的复杂NT，有自己的分隔符的复杂NT。
  - `delimited sequence`：有界序列，在序列的开始和结束处使用适当的开闭定界符的标记树序列。
  - `empty fragment`：空匹配段，一种不可见的 Rust 句法对象，它分割各种标记码。例如空白符(whitespace)或者(在某些词法上下文中的)空标记序列。<!-- The class of invisible Rust syntax that separates tokens, i.e. whitespace, or (in some lexical contexts), the empty token sequence. -->
  - `fragment specifier`：匹配段类型指示符，简单NT中的后段标识符部分，指定NT接受哪个匹配段。<!-- The identifier in a simple NT that specifies which fragment the NT accepts. tobemodify-->
  - `language`：与上下文无关的语言。

示例：

```rust,compile_fail
macro_rules! i_am_an_mbe {
    (start $foo:expr $($i:ident),* end) => ($foo)
}
```

`(start $foo:expr $($i:ident),\* end)` 是一个匹配器(matcher)。整个匹配器是一个有界序列(使用开闭定界符 `(` 和 `)` 界定)，`$foo` 和 `$i` 是简单NT(simple NT)， `expr` 和 `ident` 是它们各自的匹配段类型指示符(fragment specifiers)。

`$(i:ident),\*` *也*是一个 NT；它是一个复杂NT，匹配标识符类型的以逗号为分隔符的重复段。`,` 是这个复杂NT的分隔符，它出现在匹配段的每对元素（如果有的话）之间。

复杂NT 的另一个例子是 `$(hi $e:expr ;)+`，它匹配 `hi <expr>; hi <expr>; ...` 这种格式的代码，其中 `hi <expr>;` 至少出现一次。注意，这个复杂NT没有专用的分隔符。

(请注意，Rust的解析器确保有界序列始终具有正确的标记树结构嵌套以及开/闭定界符的正确匹配。)

我们倾向于使用变量“M”表示匹配器，变量“t”和“u”表示任意单个标记码，变量“tt”和“uu”表示任意标记树。（使用“tt”确实存在潜在的歧义，因为它的额外角色是一个匹配段类型指示符；但不用太担心，因为从上下文中，可以很清楚地看出哪个解释符合语义）

“SEP”将代表分隔符，“OP”将代表重复运算符 `\*`, `+`, 和 `?` “OPEN”/“CLOSE”覆盖围绕定界序列的匹配标记对（例如 `[` 和 `]` ）。

希腊字母 "α" "β" "γ" "δ" 代表潜在的空标记树序列。（然而，希腊字母 "ε"(epsilon)在表示形式中有特殊的作用，并不代表标记树序列。）

  * 这种希腊字母约定通常只是在需要表现一个序列的技术细节时才被引入；特别是，当我们希望*强调*我们操作的是一个标记树序列时，我们将对该序列使用表义符 "tt ..."，而不是一个希腊字母。

请注意，匹配器仅仅是一个标记树。如前所述，“简单NT”是一个元变量NT；因此，这是一个非重复段。例如，`$foo:ty` 是一个简单NT，而 `$($foo:ty)+` 是一个复杂NT。

还请注意，在这种形式的上下文中，术语“标记码(token)”通常*包括*简单NT。

最后，读者要记住，根据这种形式的定义，没有简单NT 会匹配空匹配段，同样也没有标记码会匹配 Rust句法的空匹配段。(因此，能够匹配空匹配段的*唯一的* NT是复杂NT。)这实际上不是真的，因为 `vis` 匹配器可以匹配空匹配段。因此，为了达到形式统一的目的，我们将把 `$v:vis` 看作是 `$($v:vis)?`，来要求匹配器匹配一个空匹配段。

### The Matcher Invariants
### 匹配器的不变式

为了有效，匹配器必须满足以下三个不变式。注意其中 FIRST 和 FOLLOW 的定义将在后面进行描述。
To be valid, a matcher must meet the following three invariants. The definitions of FIRST and FOLLOW are described later.

1.  对于形式如 `M = ... tt uu ...`，其中 `uu ...` 非空的匹配器 `M` 中的任意两个连续的标记树序列，我们必须遵循 FOLLOW(`... tt`) ∪ {ε} ⊇ FIRST(`uu ...`)
For any two successive token tree sequences in a matcher `M` (i.e. `M = ... tt uu ...`) with `uu ...` nonempty, we must have FOLLOW(`... tt`) ∪ {ε} ⊇ FIRST(`uu ...`).
2.  对于匹配器中任何带分隔符的复杂NT，`M = ... $(tt ...) SEP OP ...`，我们必须遵循 `SEP` ∈ FOLLOW(`tt ...`)
For any separated complex NT in a matcher, `M = ... $(tt ...) SEP OP ...`, we must have `SEP` ∈ FOLLOW(`tt ...`).
3.  对于匹配器中不带分隔符的复杂NT，`M = ... $(tt ...) OP ...`，如果 OP = `\*` 或 `+`，我们必须遵循 FOLLOW(`tt ...`) ⊇ FIRST(`tt ...`)
For an unseparated complex NT in a matcher, `M = ... $(tt ...) OP ...`, if OP = `\*` or `+`, we must have FOLLOW(`tt ...`) ⊇ FIRST(`tt ...`).

第一个不变式表示，无论匹配器后出现什么标记(如果有的话)，它都必须出现在先决随集(predetermined follow set)中的某个地方。这将确保合法的宏观定义将继续对 `... tt` 的结束和 `uu ...` 的开始执行相同的判定(determination)，即使在将来语言中添加了新的语法形式。
The first invariant says that whatever actual token that comes after a matcher, if any, must be somewhere in the predetermined follow set.  This ensures that a legal macro definition will continue to assign the same determination as to where `... tt` ends and `uu ...` begins, even as new syntactic forms are added to the language.

第二个不变式表示一个带分隔符的复杂NT必须使用一个分隔符，它是NT的内部内容的先决随集的一部分。这将确保合法的宏定义将继续将输入匹配段解析成相同的定界序列 `tt ...`，即使在将来语言中添加了新的语法形式。
The second invariant says that a separated complex NT must use a separator token that is part of the predetermined follow set for the internal contents of the NT. This ensures that a legal macro definition will continue to parse an input fragment into the same delimited sequence of `tt ...`'s, even as new syntactic forms are added to the language.

第三个不变式说的是，当我们有一个复杂NT，它可以匹配同一事物的两个或多个副本，并且两者之间没有分隔符，那么根据第一个不变式，它们必须被允许放在一起。这个不变式还要求它们是非空的，这消除了可能的歧义。
The third invariant says that when we have a complex NT that can match two or more copies of the same thing with no separation in between, it must be permissible for them to be placed next to each other as per the first invariant. This invariant also requires they be nonempty, which eliminates a possible ambiguity.

**注意：由于历史疏忽和对行为的严重依赖，第三个不变式目前没有执行。目前还没有决定下一步该怎么做。不尊重这种行为的宏可能会在Rust的未来版本中失效。参见[跟踪问题][tracking issue]**
**NOTE：The third invariant is currently unenforced due to historical oversight and significant reliance on the behaviour. It is currently undecided what to do about this going forward. Macros that do not respect the behaviour may become invalid in a future edition of Rust. See the [tracking issue].**

### FIRST and FOLLOW, informally

给定匹配器 M 映射到三个集合：FIRST(M)，LAST(M) 和 FOLLOW(M)。
A given matcher M maps to three sets：FIRST(M), LAST(M) and FOLLOW(M).

这三个集合中的每一个都是由标记码组成的。FIRST(M) 和 LAST(M) 也可能包含一个可区分的非标记码元素 ε ("epsilon")，这表示 M 可以匹配空匹配段。（但是 FOLLOW(M) 始终只是一组标记码。）
Each of the three sets is made up of tokens. FIRST(M) and LAST(M) may also contain a distinguished non-token element ε ("epsilon"), which indicates that M can match the empty fragment. (But FOLLOW(M) is always just a set of tokens.)

非正式描述(Informally)：

  * FIRST(M)：收集匹配段与 M 匹配时可能首先使用的标记码。collects the tokens potentially used first when matching a fragment to M.

  * LAST(M)：收集匹配段与 M 匹配时可能最后使用的标记码。collects the tokens potentially used last when matching a fragment to M.

  * FOLLOW(M)：允许紧跟在由 M 匹配的某个匹配段之后的标记码集。the set of tokens allowed to follow immediately after some fragment matched by M.

    换言之：t ∈ FOLLOW(M) 当且仅当存在（可能为空的）标记码序列 α、β、γ、δ，其中：
      * M 匹配 β，
      * t 与 γ 匹配，并且
      * 连结 α β γ δ 是一个可解析的 Rust程序。
    In other words：t ∈ FOLLOW(M) if and only if there exists (potentially empty) token sequences α, β, γ, δ where:
      * M matches β,
      * t matches γ, and
      * The concatenation α β γ δ is a parseable Rust program.

我们使用简写的 ANYTOKEN 来表示所有标记码(包括简单NT)的集合。例如，如果任何标记码在匹配器 M 之后是合法的，那么 FOLLOW(M) = ANYTOKEN。
We use the shorthand ANYTOKEN to denote the set of all tokens (including simple NTs). For example, if any token is legal after a matcher M, then FOLLOW(M) = ANYTOKEN.

（为了回顾一下对上述非正式描述的理解，读者在阅读正式定义之前，可以先读一遍 [关于 FIRST 和 LAST 的示例](#examples-of FIRST -and- LAST)。）
(To review one's understanding of the above informal descriptions, the reader at this point may want to jump ahead to the [examples of FIRST/LAST](#examples-of-first-and-last) before reading their formal definitions.)

### FIRST, LAST

下面是对 FIRST 和 LAST 的正式归纳定义(formal inductive definitions)。

“A∪B”表示集合并集，“A∩B”表示集合交集，“A\B”表示集合差集（即存在于A中，且不存在于B中的所有元素的集合）。

#### FIRST

FIRST(M) 是通过对序列 M 及其第一个标记树(如果有的话)的结构进行案例分析来定义的:
FIRST(M) is defined by case analysis on the sequence M and the structure of its first token-tree (if any):

  * 如果 M 为空序列，则 FIRST(M) = { ε }，if M is the empty sequence, then FIRST(M) = { ε },

  * 如果 M 以标记码 t 开始，则 FIRST(M) = { t }，if M starts with a token t, then FIRST(M) = { t },

    （注意:这涵盖了这样一种情况：M 以一个定界的标记树序列开始，`M = OPEN tt ... CLOSE ...`，此时 `t = OPEN`，因此 FIRST(M) = { `OPEN` }。）
    (Note：this covers the case where M starts with a delimited token-tree sequence, `M = OPEN tt ... CLOSE ...`, in which case `t = OPEN` and thus FIRST(M) = { `OPEN` }.)

    （注意：这主要依赖于没有简单NT与空匹配段匹配这一特性。）
    (Note：this critically relies on the property that no simple NT matches the empty fragment.)

  * 否则，M 是一个以复杂NT开始的标记树序列：`M = $( tt ... ) OP α`，或 `M = $( tt ... ) SEP OP α`，(其中 `α` 是匹配器其余部分的标记树序列(可能是空的))。Otherwise, M is a token-tree sequence starting with a complex NT：`M = $( tt ... ) OP α`, or `M = $( tt ... ) SEP OP α`, (where `α` is the (potentially empty) sequence of token trees for the rest of the matcher).

      * Let SEP\_SET(M) = { SEP } 如果存在 SEP 且 ε ∈ FIRST(`tt ...`)；否则 SEP\_SET(M) = {}。

  * Let ALPHA\_SET(M) = FIRST(`α`) if OP = `\*` or `?` and ALPHA\_SET(M) = {} if OP = `+`.
  * FIRST(M) = (FIRST(`tt ...`) \\ {ε}) ∪ SEP\_SET(M) ∪ ALPHA\_SET(M).

复杂NT的定义值得商榷。SEP\_SET(M) 定义了分隔符可能是 M 的第一个有效标记码的可能性，当定义了分隔符且重复匹配段可能为空时，就会发生这种情况。ALPHA\_SET(M)定义了复杂NT可能为空的可能性，这意味着 M 的第一个有效标记码集合是后继标记树序列 `α` 。当使用了操作符 `\*` 或 `?` 时，这种情况下可能没有重复段。理论上，如果 `+` 与一个可能为空的重复匹配段一起使用，也会出现这种情况，但是第三个不变式禁止这样做。
The definition for complex NTs deserves some justification. SEP\_SET(M) defines the possibility that the separator could be a valid first token for M, which happens when there is a separator defined and the repeated fragment could be empty. ALPHA\_SET(M) defines the possibility that the complex NT could be empty, meaning that M's valid first tokens are those of the following token-tree sequences `α`. This occurs when either `\*` or `?` is used, in which case there could be zero repetitions. In theory, this could also occur if `+` was used with a potentially-empty repeating fragment, but this is forbidden by the third invariant.

From there, clearly FIRST(M) can include any token from SEP\_SET(M) or ALPHA\_SET(M), and if the complex NT match is nonempty, then any token starting FIRST(`tt ...`) could work too. The last piece to consider is ε. SEP\_SET(M) and FIRST(`tt ...`) \ {ε} cannot contain ε, but ALPHA\_SET(M) could. Hence, this definition allows M to accept ε if and only if ε ∈ ALPHA\_SET(M) does. This is correct because for M to accept ε in the complex NT case, both the complex NT and α must accept it. If OP = `+`, meaning that the complex NT cannot be empty, then by definition ε ∉ ALPHA\_SET(M). Otherwise, the complex NT can accept zero repetitions, and then ALPHA\_SET(M) = FOLLOW(`α`). So this definition is correct with respect to \varepsilon as well.

#### LAST

LAST(M), defined by case analysis on M itself (a sequence of token-trees):

  * if M is the empty sequence, then LAST(M) = { ε }

  * if M is a singleton token t, then LAST(M) = { t }

  * if M is the singleton complex NT repeating zero or more times, `M = $( tt
    ... ) *`, or `M = $( tt ... ) SEP *`

      * Let sep_set = { SEP } if SEP present; otherwise sep_set = {}.

      * if ε ∈ LAST(`tt ...`) then LAST(M) = LAST(`tt ...`) ∪ sep_set

      * otherwise, the sequence `tt ...` must be non-empty; LAST(M) = LAST(`tt
        ...`) ∪ {ε}.

  * if M is the singleton complex NT repeating one or more times, `M = $( tt ...
    ) +`, or `M = $( tt ... ) SEP +`

      * Let sep_set = { SEP } if SEP present; otherwise sep_set = {}.

      * if ε ∈ LAST(`tt ...`) then LAST(M) = LAST(`tt ...`) ∪ sep_set

      * otherwise, the sequence `tt ...` must be non-empty; LAST(M) = LAST(`tt
        ...`)

  * if M is the singleton complex NT repeating zero or one time, `M = $( tt ...)
    ?`, then LAST(M) = LAST(`tt ...`) ∪ {ε}.

  * if M is a delimited token-tree sequence `OPEN tt ... CLOSE`, then LAST(M) =
    { `CLOSE` }.

  * if M is a non-empty sequence of token-trees `tt uu ...`,

      * If ε ∈ LAST(`uu ...`), then LAST(M) = LAST(`tt`) ∪ (LAST(`uu ...`) \ { ε }).

      * Otherwise, the sequence `uu ...` must be non-empty; then LAST(M) =
        LAST(`uu ...`).

### Examples of FIRST and LAST
### 关于 FIRST 和 LAST 的示例

下面是一些关于 FIRST 和 LAST 的例子。（请特别注意，特殊元素 ε 是如何根据输入匹配段之间的相互作用来引入和消除的。）
Below are some examples of FIRST and LAST. (Note in particular how the special ε element is introduced and eliminated based on the interaction between the pieces of the input.)

我们的第一个例子以树状结构呈现，以详细说明匹配器的分析是如何组成的。（一些较简单的子树已被删除。）
Our first example is presented in a tree structure to elaborate on how the analysis of the matcher composes. (Some of the simpler subtrees have been elided.)

```text
INPUT： $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+   g
            ~~~~~~~~   ~~~~~~~                ~
                |         |                   |
FIRST：  { $d:ident }  { $e:expr }          { h }


INPUT： $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+
            ~~~~~~~~~~~~~~~~~~             ~~~~~~~           ~~~
                        |                      |               |
FIRST：         { $d:ident }               { h, ε }         { f }

INPUT： $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+   g
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~    ~~~~~~~~~~~~~~    ~~~~~~~~~   ~
                        |                       |              |       |
FIRST：       { $d:ident, ε }            {  h, ε, ;  }      { f }   { g }


INPUT： $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+   g
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                                        |
FIRST：                      { $d:ident, h, ;,  f }
```

Thus:

 * FIRST(`$($d:ident $e:expr );* $( $(h)* );* $( f ;)+ g`) = { `$d:ident`, `h`, `;`, `f` }

Note however that:

 * FIRST(`$($d:ident $e:expr );* $( $(h)* );* $($( f ;)+ g)*`) = { `$d:ident`, `h`, `;`, `f`, ε }

Here are similar examples but now for LAST.

 * LAST(`$d:ident $e:expr`) = { `$e:expr` }
 * LAST(`$( $d:ident $e:expr );*`) = { `$e:expr`, ε }
 * LAST(`$( $d:ident $e:expr );* $(h)*`) = { `$e:expr`, ε, `h` }
 * LAST(`$( $d:ident $e:expr );* $(h)* $( f ;)+`) = { `;` }
 * LAST(`$( $d:ident $e:expr );* $(h)* $( f ;)+ g`) = { `g` }

### FOLLOW(M)

Finally, the definition for FOLLOW(M) is built up as follows. pat, expr, etc.
represent simple nonterminals with the given fragment specifier.

  * FOLLOW(pat) = {`=>`, `,`, `=`, `|`, `if`, `in`}`.

  * FOLLOW(expr) = FOLLOW(stmt) =  {`=>`, `,`, `;`}`.

  * FOLLOW(ty) = FOLLOW(path) = {`{`, `[`, `,`, `=>`, `:`, `=`, `>`, `>>`, `;`,
    `|`, `as`, `where`, block nonterminals}.

  * FOLLOW(vis) = {`,`l any keyword or identifier except a non-raw `priv`; any
    token that can begin a type; ident, ty, and path nonterminals}.

  * FOLLOW(t) = ANYTOKEN for any other simple token, including block, ident,
    tt, item, lifetime, literal and meta simple nonterminals, and all terminals.

  * FOLLOW(M), for any other M, is defined as the intersection, as t ranges over
    (LAST(M) \ {ε}), of FOLLOW(t).

The tokens that can begin a type are, as of this writing, {`(`, `[`, `!`, `\*`,
`&`, `&&`, `?`, lifetimes, `>`, `>>`, `::`, any non-keyword identifier, `super`,
`self`, `Self`, `extern`, `crate`, `$crate`, `_`, `for`, `impl`, `fn`, `unsafe`,
`typeof`, `dyn`}, although this list may not be complete because people won't
always remember to update the appendix when new ones are added.

Examples of FOLLOW for complex M:

 * FOLLOW(`$( $d:ident $e:expr )\*`) = FOLLOW(`$e:expr`)
 * FOLLOW(`$( $d:ident $e:expr )\* $(;)\*`) = FOLLOW(`$e:expr`) ∩ ANYTOKEN = FOLLOW(`$e:expr`)
 * FOLLOW(`$( $d:ident $e:expr )\* $(;)\* $( f |)+`) = ANYTOKEN

### Examples of valid and invalid matchers

With the above specification in hand, we can present arguments for
why particular matchers are legal and others are not.

 * `($ty:ty < foo ,)` ：illegal, because FIRST(`< foo ,`) = { `<` } ⊈ FOLLOW(`ty`)

 * `($ty:ty , foo <)` ：legal, because FIRST(`, foo <`) = { `,` }  is ⊆ FOLLOW(`ty`).

 * `($pa:pat $pb:pat $ty:ty ,)` ：illegal, because FIRST(`$pb:pat $ty:ty ,`) = { `$pb:pat` } ⊈ FOLLOW(`pat`), and also FIRST(`$ty:ty ,`) = { `$ty:ty` } ⊈ FOLLOW(`pat`).

 * `( $($a:tt $b:tt)* ; )` ：legal, because FIRST(`$b:tt`) = { `$b:tt` } is ⊆ FOLLOW(`tt`) = ANYTOKEN, as is FIRST(`;`) = { `;` }.

 * `( $($t:tt),* , $(t:tt),* )` ：legal,  (though any attempt to actually use this macro will signal a local ambiguity error during expansion).

 * `($ty:ty $(; not sep)* -)` ：illegal, because FIRST(`$(; not sep)* -`) = { `;`, `-` } is not in FOLLOW(`ty`).

 * `($($ty:ty)-+)` ：illegal, because separator `-` is not in FOLLOW(`ty`).

 * `($($e:expr)*)` ：illegal, because expr NTs are not in FOLLOW(expr NT).

[Macros by Example]: macros-by-example.md
[RFC 550]: https://github.com/rust-lang/rfcs/blob/master/text/0550-macro-future-proofing.md
[tracking issue]: https://github.com/rust-lang/rust/issues/56575

<!-- 2020-11-3 -->
<!-- checked -->
