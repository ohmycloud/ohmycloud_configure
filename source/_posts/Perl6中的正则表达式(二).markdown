---
title: Perl 6 中的正则表达式(二)
date: 2016-06-29
tags: 正则表达式
categories: Perl 6
comments: true
---

# 允许的修饰符

有些修饰符能在所有允许的地方出现, 但并非所有的都这样.

通常,  影响 regex 编译的修饰符( 像 `:i` ) 一定要在编译时被知道. 只影响行为而非 regex本身的修饰符(eg. `:pos`, `:overlap`, `:x(4)`) 可能只出现在引用某个调用的结构上(例如 `m//` 和`s///`),  并且不会出现在 `rx//` 上.  最后, 重叠在替换结构中是不被允许的, 而影响修改的副词只允许出现在替中.

这些准则导致了下面的 rules:

- `:ignorecase`, `:ignoremark`, `:sigspace`, `:ratchet`  和 ` :Perl5`  修饰符和它们的便捷形式允许出现在 regex 中的任何地方, 还有 `m//`, `rx//` 和`s///`  结构中. 一个 regex实现可能要求它们的值在编译时是被知晓的, 而如果不是这种情况则给出编译时错误信息.

```
rx:i/ hello /           # OK
rx:i(1) /hello/         # OK
my $i = 1;
rx:i($i) /hello/        # may error out at compile time
```

- `:samecase`, `:samespace` 和 `:same mark`  修饰符(还有它们的便捷形式) 只允许出现在替换结构上 (`s///` 和 `s[] = ...`).
- `:overlap` 和 `:exhaustive`修饰符(还有它们的便捷形式) 只允许出现匹配结构上(i.e. `m//`), 不会出现在替换或 regex qoutes 结构上.
-  `:pos`, `:continue`, `:x` 和 `:nth`  修饰符和它们的别名只允许作用在引用即时调用的结构上. (eg. `m//` 和`s///` (but not on `rx//`).
-  `:dba` 副词只在  regex 内部被允许.

# 改变了的元字符

[`S05-metasyntax/changed.t lines 6–38`](https://github.com/perl6/roast/blob/master/S05-metasyntax/changed.t#L6-L38)

- `.` 现在匹配任意字符,包括换行符. (`/s` 修饰符被废弃了).
- `^` 和 `$` 现在总是匹配字符串的开始/末尾, 就像旧的  `\A` 和  `\z`. (`/m` 修饰符被废弃了.)  On the right side of an embedded在内含的  `~~`或 `!~~`  操作符的右侧, `^` 和 `$`  总是匹配指定 submatch  的开头/结尾, 因为那个 submatch  在逻辑上被看作为单独的字符串.
- `$`不再匹配一个可选的前置 `\n`, 所以你还是想要的话, 使用 `\n?$` 代替.
- `\n` 现在匹配一个逻辑(跟平台有关)换行符, 不仅仅是 `\x0a`.

[`S05-metachars/newline.t lines 13–37`](https://github.com/perl6/roast/blob/master/S05-metachars/newline.t#L13-L37)

- `\A`, `\Z`, 和 `\z` 元字符被废弃了.

# 新的元字符

- 因为 `/x` 是默认的:
  - 不加引号的 `#` 现在总是引入一个注释.  如果它后面跟着一个反引号和一个开放的括号字符, 它会引入一个以闭括号终止的嵌入式注释. 否则, 注释会以换行终止.
  - 空白现在总是元语法，即只用于布局，不会再被照字面意义被匹配。（参看上面的描述的`:sigspace`修饰符）

- `^^` 和 `$$` 匹配行的开头和结尾. (`/m` 修饰符不存在了.) 它俩都是零宽断言.  `$$` 匹配任何 `\n` (逻辑换行) 之前的位置, 如果最后一个字符不是 `\n` , 那它也匹配字符串的末尾.  `^^` 总是匹配字符串的开头,  还匹配字符串中任何不是最后一个字符的 `\n` 的后面

[`S05-metachars/line-anchors.t lines 15–43`](https://github.com/perl6/roast/blob/master/S05-metachars/line-anchors.t#L15-L43)

- `.` 匹配任何东西, 而 `\N` 任何除 `\n` 之外的东西 (`/s` 修饰符不存在了.) 特别地, `\N`  既不匹配回车, 也不匹配换行.

  - 新的  `&` 元字符分割连结项.   模式两边必须以同一个开始点和结束点匹配. 注意, 如果你不想两个项以同一个点结尾, 那么你真正需要的是使用向前查看代替.

就像`或`有 `|` 和 `||` 一样, `与`也有 `&` 和 `&&` 两种形式.  `&` 形式的与被认为是描述性的而不是程序性的; 它允许编译器 和/或 运行时系统决定首先计算哪一部分, 一贯的任何顺序都可能发生的假设是错误的. `&&` 保证了从左到右的顺序, 并且回溯使右侧的参数比左侧的参数变化得更快. 换句话说,  `&&` 和 `||` 建立了一连串的点. 左侧可以在回溯允许的时候作为整体回溯到结构中.

  [`S05-metasyntax/sequential-alternation.t lines 5–21`](https://github.com/perl6/roast/blob/master/S05-metasyntax/sequential-alternation.t#L5-L21)

​`&&`and `||`. `&` 像 `|` 那样是列表结合性的, 但是有高一点的优先级. 同样地, `&&` 的优先级比 `||` 的优先级高一点. 就像普通的连接和短路操作符一样, `&` 和 `|` 的结合性比 `&&` 和 `||` 更紧.

- `~~` 和 `!~~` 操作符让在左侧的变量或原子所匹配到的东西身上执行 submatch。所以, 例如, 你可以匹配任何不包含单词"moose"的标识符:

```perl6
<ident> !~~ 'moose'
```

In contrast

```perl6
<ident> !~~ ^ 'moose' $
```

会包含任意标识符 (包括任何含有 "moose" 作为子字符串的标识符)只要该标识符作为一个整体不等于 "moose"。 (注意锚点, 它把子匹配锚定到标识符的开头和结尾, 就像它们是整个匹配一样。) 当用作更长匹配的一部分时, 为了清晰, 使用额外的方括号可能更好:

```perl6
[ <ident> !~~ ^ 'moose' $ ]
```

*~~* 和 `!~~` 的优先级位于逻辑操作符的 `|` 和 `||` 之间, 就像在普通的 Perl 表达式中那样。 (查看 S03). 因此:

```perl6
<ident> !~~ 'moose' | 'squirrel'
```

解析为

```perl6
<ident> !~~ [ 'moose' | 'squirrel' ]
```

而

```perl6
<ident> !~~ 'moose' || 'squirrel'
```

解析为

```perl6
[ <ident> !~~ 'moose' ] || 'squirrel'
```

-  `~` 操作符是一个用于匹配嵌套的带有特定终止符作为目标的 subrules 的助手。 它被设计为放置在开括号和闭括号之间, 就像这样:
  [`S05-metachars/tilde.t lines 6–81`](https://github.com/perl6/roast/blob/master/S05-metachars/tilde.t#L6-L81)

```perl6
'(' ~ ')' <expression>
```

然而, 它通常忽略左边的参数, 并在接下来的那两个原子身上进行操作(原子也可以被量词化)。它对后续的那两个原子的操作就是旋动它们以使它们以反转的顺序匹配。因此上面的表达式, 乍一看, 只是下面这种形式的简写:

```perl6
'(' <expression> ')'
```

  But beyond that, when it rewrites the atoms it also inserts the apparatus that will set up the inner expression to recognize the terminator, and to produce an appropriate error message if the inner expression does not terminate on the required closing atom. So it really does pay attention to the left bracket as well, and it actually rewrites our example to something more like:

```perl6
$<OPEN> = '(' <SETGOAL: ')'> <expression> [ $GOAL || <FAILGOAL> ]
```

注意,  即使在没有开括号时, 你也可以使用这种结构来设置期望一个闭合结构:

```perl6
<?> ~ ')' \d+
```

这儿的  `<?>` 在第一个 null 字符串上返回 true。

例子:

```perl6
my $a = '12)34)';
$a~~ m:g/ <?> ~ ')' \d+ /;
say $/.Str;  # 12) 34)
```
  ​

By default the error message uses the name of the current rule as an indicator of the abstract goal of the parser at that point. However, often this is not terribly informative, especially when rules are named according to an internal scheme that will not make sense to the user. The `:dba("doing business as")` adverb may be used to set up a more informative name for what the following code is trying to parse:

```perl6
token postfix:sym<[ ]> {
    :dba('array subscript')
    '[' ~ ']' <expression>
}
```

那么得到的不是诸如这样的消息：

```
Unable to parse expression in postfix:sym<[ ]>; couldn't find final ']'
```

你会得到像下面这样的消息:
  
```
Unable to parse expression in array subscript; couldn't find final ']'
```

(`:dba` 副词也能用于轮试和候选分支起名字, 这帮助词法分析器给出更好的错误消息。)
  ​
# 括号合理化

- `(...)` 仍然界定一个捕获组. 然而, 这些捕获组的顺序是分等级的, 而不是线性的. 查看 "Nested subpattern captures".

- `[...]` 不再表示字符类. 它现在界定一个`非捕获组`.

[`S05-match/non-capturing.t lines 11–39`](https://github.com/perl6/roast/blob/master/S05-match/non-capturing.t#L11-L39)

字符类现在使用 `<[...]>` 指定. 查看 "Extensible metasyntax (<...>)".

- `{...}` 不再是重复量词. 它现在界定一个嵌入的`闭包`. 它总是被认为是过程式的而非声明性的;  它在之前和之后之间建立了一系列的点. (为了避免这个, 使用 `<?{…}>` 断言语法代替. ). regex 中的闭包建立了它自己的词法作用域 
  [`S05-metachars/closure.t lines 15–53`](https://github.com/perl6/roast/blob/master/S05-metachars/closure.t#L15-L53)
```
{
my $x = 3;
my $y = 2;
'a' ~~ /. { $y = $x; 0 }/;  # can match and execute a closure'
say $y;                     # 3, 'could access and update outer lexicals';
}
```

测试中的 `#?rakudo skip 'assignment to match variables (dubious) RT #124946'` 表示跳过该测试, 功能尚未实现。

```perl6
my $caught = "oops!";
"abc" ~~ m/a(bc){$caught = $0}/;  # Outer match
say $caught;                      # bc, Outer caught
```

- 你可以使用闭包调用 Perl 代码作为正则表达式匹配的一部分. 嵌入的代码不经常影响匹配 -- 它只用作副作用(比如保存捕获的值):
```
/ (\S+) { print "string not blank\n"; $text = $0; }
  \s+   { print "but does contain whitespace\n"   }
/
```
  ​
在匹配过程中使用 `{...}` 闭包的一个例子就是 `make` 函数。
一个使用 make 函数的显式换算生成了这个匹配(match)的抽象语法树(简写成抽象对象或 ast):

[`S05-grammar/action-stubs.t lines 37–182`](https://github.com/perl6/roast/blob/master/S05-grammar/action-stubs.t#L37-L182)
[`S05-match/make.t lines 7–8`](https://github.com/perl6/roast/blob/master/S05-match/make.t#L7-L8)

```perl6
/ (\d) { make $0.sqrt } Remainder /;
```

  ​
这捕获了数字化字符串的平方根, 而不是字符串的平方根。   如果前面的 `\d` 匹配成功, 那么 `Remainder`(剩余的) 部分继续被匹配并作为 `Match` 对象的一部分返回, 但是没有作为`抽象对象`的一部分返回。因为抽象对象通常代表抽象语法树的顶层节点, 所以抽象对象可以通过使用 `.made` 方法从 `Match` 对象中提取出来。

  This has the effect of capturing the square root of the numified string, instead of the string. The `Remainder`part is matched and returned as part of the `Match` object but is not returned as part of the abstract object. Since the abstract object usually represents the top node of an abstract syntax tree, the abstract object may be extracted from the `Match` object by use of the `.made` method.
  ​
`make` 的二次调用会重写之前的 `make` 调用。 每个匹配对象上都可以有 `make`方法。

  A second call to `make` overrides any previous call to `make`. `make` is also available as a method on each match object.
  ​
在闭包里面, 搜索的实时位置是由  `$¢.pos` 方法指示的。就像所有的字符串位置一样, 你不能把它当作一个数字除非你很清楚你正处理的单元是哪一个。
  Within a closure, the instantaneous position within the search is denoted by the `$¢.pos` method. As with all string positions, you must not treat it as a number unless you are very careful about which units you are dealing with.

`Cursor`  对象也能返回我们匹配的原始项; 这能从 `.orig` 方法中得到。
  ​
  The `Cursor` object can also return the original item that we are matching against; this is available from the `.orig` method.

  到目前为止闭包也保证了开启一个 `$/` Match 对象代表了匹配到的东西。 然而, 如果闭包自身内部做了匹配, 那么它的 `$/` 变量会被绑定到那个匹配的结果上直到嵌入闭包的结束。在闭包的后面, 匹配实际会使用 `$¢` 的当前值继续。 在你的闭包中 `$/` 和 `$¢` 是一样的。

- 闭包能影响匹配如果它调用了 `fail`:

```perl6
/ (\d+) { $0 < 256 or fail } /
```

因为闭包建立了一个序列点,  它们保证会在规定的时间被调用即使优化器能证明它们后面的某些东西不能匹配。(任何之前的都是公平游戏。 特别地, 闭包常常用作最长 token 模式的终结者。)​

- 普通的重复分类符现在是 `**`, 用于贪婪匹配,  使用对应的 `**?` 用于非贪婪匹配.(所有这样的量词修饰符现在直接跟在 `**` 后面).整个量词的两边都允许有空格, 但是只有 `**` 前面的空格在 `:sigspace` 下和重复之间的匹配被认为是有意义的.

[`S05-metasyntax/repeat.t lines 19–88`](https://github.com/perl6/roast/blob/master/S05-metasyntax/repeat.t#L19-L88)

下一个 token 限制了左边的 pattern 必须被匹配多少次。 如果下一个东西是整数, 那么它会被解析为一个精确的计数或一个范围:

```perl6
. ** 42                  # match exactly 42 times
<item> ** 3..*           # match 3 or more times
```

这种形式被认为是陈述性的。

如果你提供一个闭包, 它应该返回一个 Int 或 Range 对象.

```perl6
'x' ** {$m}              # 从闭包中返回精确计数
<foo> ** {$m..$n}        # 从闭包中返回范围
 / value was (\d **? {1..6}) with ([ <alpha>\w* ]**{$m..$n}) /
```

从闭包返回一个列表是非法的, 所以这种简单的错误会导致失败:

```perl6
/ [foo] ** {1,3} /
```

这种形式的闭包总被看作是​过程式的, 所以它所修饰的项绝不会被当作是最长 token 的一部分。
  ​
为了和之前的 Perl 6 版本保持向后兼容, 如果一个 token 后面跟着的既不是闭包也不是整数字面值, 那么它会被解释为 `+%`, 并带有一个警告：

```perl6
/ x ** y /                # same as / x+ % y /
/ x ** $y /               # same as / x [$y x]* /
```

这并不会检查 $y 是否包含一个整数或范围值。这个兼容功能也不能保证会永远存在。
  ​
- 负数范围值也是允许的, 但是只有当模式是可逆的时候(例如 after 能匹配的). 例如, 搜索元素周围 200 个定义为点的字符, 可以写为:

```
/ . ** -100..100 <element> /
```

类似地, 你可以后退 50 个字符:

```perl6
/ . ** -50 <element> /
```

- 任何量词化的原子都能通过添加一个额外的约束, 来指定重复左侧的两个原子之间的分隔符.  在量词和分割符之间添加一个 `%`号. 只要两个 item 之间有分隔符就重复初始的 item:

```perl6
<alt>+    % '|'            # repetition controlled by presence of character
<addend>+ % <addop>        # repetition controlled by presence of subrule
<item>+   % [ \!?'==' ]    # repetition controlled by presence of operator
<file>+   % \h+            # repetition controlled by presence of whitespace
```

任何量词都可以这样修改:

```perl6
<a>* % ','              # 0 or more comma-separated elements
<a>+ % ','              # 1 or more
<a>? % ','              # 0 or 1 (but ',' never used!?!)
<a> ** 2..* % ','       # 2 or more
```

`%` 修饰符只能用在量词上;  把 `%` 用在裸 item 上的任何尝试都会导致解析失败.

```perl6
/ <ident>+ % ',' /
```

能匹配

```perl6
foo
foo,bar
foo,bar,baz
```

但是不会匹配

```
foo,
foo,bar,
```

```perl6
'' ~~ / <ident>* % ',' /  # matches because of the *
```

使用 `%%` 能匹配末尾的分隔符. 因此

```perl6
/ <ident>+ %% ',' /
```

能匹配

```
foo
foo,
foo,bar
foo,bar,
foo,bar,baz
foo,bar,baz,
```

If you wish to quantify each match on the left individually, you must place it in brackets:

```perl6
[<a>*]+ % ','
```

零宽的分隔符也是合法的, 只要左侧的模式每次能够重复:

```perl6
.+ % <?same>   # 匹配同一字符的序列
```

The separator never matches independently of the next item; if the separator matches but the next item fails, it backtracks all the way back through the separator. Likewise, this matching of the separator does not count as "progress" under `:ratchet` semantics unless the next item succeeds.

When significant space is used under `:sigspace`, each matching element enables the immediately following whitespace to be considered significant. Space after the `%` does nothing. 当在 `:sigspace` 下使用了有意义的空格, 每个匹配元素使后面跟着的空格变得有意义. % 后面的空格什么也不做. 如果你这样写:

```perl6
ms/ <element> +  %  ',' /
  #1        #2 #3 #4  #5
```

它会忽略 `#1` 和 `#4` 位置的空白, 并把剩下的重写为:

```perl6
/ [ <element> <.ws> ]+ % [ ',' <.ws> ] <.ws> /
                #2               #5      #3
```

因为 `#3` 对于 `#2` 来说是多余的(因为 `+` 要求一个元素), `#2` 或 `#3` 都可以满足:

```perl6
ms/ <element>+ % ',' /    # ws after comma and at end
ms/ <element> +% ',' /    # ws after comma and any element
```

所以第一个

```perl6
ms/ <element>+ % ',' /    # ws after comma and at end
```

就像

```perl6
/ <element>[','<.ws><element>]*<.ws> /
```

而第二个

```perl6
ms/ <element> +% ',' /    # ws after comma and any element
```

就像

```perl6
/ <element><.ws>[','<.ws><element><.ws>]* /
```

并且

```perl6
ms/ <element>+% ','/
```

排除了所有有意义的空格,就像这样:

```perl6
/ <element>[','<element>]* /
```
  ​
注意, 使用 `*` 而非 `+`, 空格 `#3` 对于 `#2` 来说并不是多余的, 因为如果匹配了 0 个元素, 那么跟它有关的(#2) 空格就不会匹配。 那种情况下, 在 `*` 两边都放上空格是有意义的:

```perl6
ms/ <element> * % ',' /
```

- `<...>` 现在是可扩展的元语法分隔符或*断言*(例如, 它们代替 Perl‘5 的`(?...)` 语法)。

# 变量(non-)插值

[`S05-interpolation/regex-in-variable.t lines 13–81`](https://github.com/perl6/roast/blob/master/S05-interpolation/regex-in-variable.t#L13-L81)

- 在 Perl 6 的 regexes 中, 变量不会进行插值. 

- 相反, 它们被原原本本地传递给正则引擎, 然后正则引擎决定怎样处理它们.

- 在正则引擎中处理字符串标量的默认方式是把它作为 `"..."` 字面量匹配 (i.e. 它不会把插值字符串作为 subpattern). 换句话说, 一个 Perl 6 的:
  [`S05-metasyntax/litvar.t lines 17–39`](https://github.com/perl6/roast/blob/master/S05-metasyntax/litvar.t#L17-L39)

```perl6
/ $var /
```

就像 Perl 5 的:

```perl6
/ \Q$var\E /
```

为了插值一个 Regex 对象, 使用  `<$var>` 代替. 如果 `$var` 未定义, 会出现一个警告并匹配失败.` 

[`S05-interpolation/regex-in-variable.t lines 85–119`](https://github.com/perl6/roast/blob/master/S05-interpolation/regex-in-variable.t#L85-L119)

当匹配一个不是 Str 类型的字符串化的类型时, 那个变量必须被作为那个字符串化类型的值被插值(或者是能强制转换成那个类型的相关类型) 例如: 当 regex 匹配一个 Buf 类型时, 变量将会在 Buf 类型的语义下被匹配, 而非 Str 语义.

[猜想: 当我们允许匹配非字符串类型时, 在当前节点上做类型匹配会要求一个内含的签名的语法,  不仅仅是一个裸的变量, 所以没有必要对包含一个类型对象的变量作出解释, 它明显是未定义的, 因此对上面的 rule 会匹配失败]

然而,  一个在等号左边用作别名的变量或 submatch 操作符是不用于匹配的:

```perl6
$x = <.ident>
$0 ~~ <.ident>
```

如果你想再次匹配 `$0` 然后把它用作 submatch, 你可以强制这个匹配使用双引号:

```perl6
"$0" ~~ <.ident>
```

另一方面,  如果别名不是一个变量的话就没有意义:

```perl6
"$0" = <.ident>     # ERROR
$0 = <.ident>       # okay
$x = <.ident>       # okay, 临时捕获
$<x> = <.ident>     # okay, 持久捕获
<x=.ident>          # 同上
```

在捕获别名中声明的变量的作用域是词法作用域,一直到 regex 的剩余部分. 你不能把这种 = 号的用法和普通赋值或普通绑定操作混淆。 你更应该把这种 = 号读作声明符的伪赋值, 而非普通赋值。 它更像普通的 `:=` 操作符, 因为在 regexes 的工作级别, 字符串是不可变的, 所以捕获正是预先计算好的 substr 值.  尽管如此, 当你最终独立地使用这些值时, 那个 substr 就会被复制, 然后它就更像原来的赋值操作.

`$<ident>` 形式的捕获变量能在词法作用域之外持久; 如果匹配成功的话, 它们会被记忆在 Match 对象的散列中, 散列的键对应于变量名的标识符. 同样地, 绑定的数字变量保存在 `$0` 那样的变量中, 等等.

 你可以把捕获保存到已经存在的词法变量中; 这样的变量可能已经能从外部的作用域中可见, 或者可能在 regex 中通过一个 `:my` 声明符来声明。

```perl6
my $x; / $x = [...] /            # capture to outer lexical $x
/ :my $x; $x = [...] /           # capture to our own lexical $x
```

- 一个插值的数组:
[`S05-metasyntax/litvar.t lines 40–102`](https://github.com/perl6/roast/blob/master/S05-metasyntax/litvar.t#L40-L102)
[`S05-metasyntax/sequential-alternation.t lines 22–39`](https://github.com/perl6/roast/blob/master/S05-metasyntax/sequential-alternation.t#L22-L39)

```perl6
/ @cmds /
```

被匹配为好像它是它的字面元素的一个备选分支. 通常地, 它使用 junctive 语义来匹配:

```perl6
/ [ $(@cmds[0]) | $(@cmds[1]) | $(@cmds[2]) | ... ] /
```

然而, 如果它是 `||` 列表中的一个直接成员, 它会使用相继的匹配语义, 即使它是列表中的唯一成员. 方便地, 你可以把 `||` 放在备选分支的第一个成员之前, 因此

```perl6
/ || @cmds /
```

等价于

```perl6
/ [ $(@cmds[0]) || $(@cmds[1]) || $(@cmds[2]) || ... ] /
```

当然, 你也可以:

```perl6
/ | @cmds /
```

需要明确的是, 你想要 junctive 语义.

注意, ` $(...)` 的用法是为了阻止下标被解析为 regex 语法而非真正的下标.

因为 `$x` 被插值为好像你说了 `"$x"` 一样, 如果 $x 包含了一个列表, 它会先被字符串化. 为了获取备选分支, 你必须使用 `@$x` 或  `@($x)` 形式来标示你想要把那个标量变量当作一个列表。

只有当它在regex被编译时为常量所熟知, 一个使用 junctive 语义的插值数组才是陈述性的(参与外部的最长token匹配)。

像标量变量那样, 每个元素被作为字面量匹配. 所有这样的值负责当前的 `:ignorecase` 和 `:ignoremark` 设置.

当你写烦了:

```perl6
token sigil { '$' | '@' | '%' | '&' | '::' }
```

你可以这样写:

```perl6
token sigil { < $ @ % & :: > }
```

只要你细心地在起始的尖括号后面放上一个空格, 以至于它不会被解释为 subrule. 有了空格, 它会像普通 Perl 6 中的尖括号引号那样被解析, 并被当作一个字面数组值。

- 要不, 如果你预先声明一个 proto regex, 你可以给同一个类别写多个正则表达式, 区别仅仅在于它们所匹配的符号. 符号被指定为"长名字" 的一部分. 也可以在 rule 中使用 `<sym>` 进行匹配, 就像这样:
[`S05-grammar/protos.t lines 7–31`](https://github.com/perl6/roast/blob/master/S05-grammar/protos.t#L7-L31)
```
proto token sigil {*}
multi token sigil:sym<$>  { <sym> }
multi token sigil:sym<@>  { <sym> }
multi token sigil:sym<%>  { <sym> }
multi token sigil:sym<&>  { <sym> }
multi token sigil:sym<::> { <sym> }
```

(multi 是可选的, 并且通常在 grammar 中被省略)

这可以被看作多重分发的一种形式, 除了它是基于最长 token 匹配而非签名匹配之外。 这种写法的好处就是在一个派生的 grammar 中, 给同一个类别添加额外的 rules 很容易.  当你尝试匹配 `/<sigil>/` 时, 它们中的所有 rules 都会被并行地匹配.
  ​
如果 multi regex 方法中有形参, 仍然首先通过最长 token 继续匹配。如果那导致了绑定, 使用剩下的变体的参数来产生一个普通的多重分发, 假设它们能通过类型进行区分的话。

当 `proto` 看见一个不为量词的 `*` 并且在包含 * 号的 block 中只有这个`*` 时, `proto` 就会进入 subdispatcher 调用。因此, 通过把这个 `*` 号放进花括号中, 你就能在这个 subdispatcher 的前面和后面放上 items 了:

```perl6
proto token foo { <prestuff> {*} <poststuff> }
```
  ​
这只在 proto 中有效。查看 [S06](http://design.perl6.org/S06.html) 关于 `{*}` 语法的讨论。(不像 proto sub 那样, proto regex 会自动记忆从 `{*}` 中返回的值, 因为它们伴随着匹配光标)。

- 模式中散列变量的用法被保留了.

[`S05-interpolation/regex-in-variable.t lines 82–84`](https://github.com/perl6/roast/blob/master/S05-interpolation/regex-in-variable.t#L82-L84)

- 只有当变量代表一个常量时, 变量的匹配才被认为是声明性的，否则它们是程序性的。注意，role 参数（如果ReadOnly）被认为是用于此目的的常量声明,尽管没有显式的 `constant` 声明符 , 因为 roles 本身是不变的，当组合的时候,可能会使用一个常量值来替换那个参数（如果传递的值是一个常量）。使用常量的宏也会使那些常量在声明时更适合。

# 可扩展的 `<...>` 元语法

[`S05-metasyntax/angle-brackets.t lines 16–139`](https://github.com/perl6/roast/blob/master/S05-metasyntax/angle-brackets.t#L16-L139)
[`S05-mass/recursive.t lines 14–48`](https://github.com/perl6/roast/blob/master/S05-mass/recursive.t#L14-L48)

`<` 和 `>` 都是元字符, 并且经常(但不总是) 用于 matched pairs. (有些元字符函数组合成独立的 tokens, 并且这些可能包含尖括号). 对于 matched pairs, `<` 后面的**第一个字符**决定了断言的性质:

- 如果 `<` 后面的第一个字符是`空格`, 尖括号会被看作普通的引号单词数组字面量

```
< adam & eve >   # 等价于 [ 'adam' | '&' | 'eve' ]
```

注意末尾的 `>` 之前的空格是可选的, 因此, `< adam & eve>` 也可以。

```perl6
"even" ~~ /< odd & eve >/
"even" ~~ /< adam & eve> {say ~$/}/ # eve
```

- `<` 后面的第一个字符如果是字母, 那么它就是一个符合语法规范的捕获断言(例如: subrule 或字符类 - 看下面):

```
/ <sign>? <mantissa> <exponent>? /
```

标识符(例如下面的 foo)后面的第一个字符决定了闭合尖括号之前剩余文本的处理。它的底层语义是**函数**或**方法调用**, 所以, 如果标识符后面的第一个字符是*左圆括号*, 那么它要么是方法调用, 要么是函数调用:

```perl6
<foo('bar')>
```

如果标识符后面的第一个字符是 `=`, 那么该标识符就是等号后面跟着的另一个**标识符的别名**。 特别地,

```perl6
<foo=bar>
```

是下面这种形式的简写:

```perl6
$<foo> = <bar>
```

注意这种别名不会修改原来的 `<bar>` 捕获. 要重命名一个继承而来的方法而不使用它原来的名字,  就在你想要抑制的捕获名前面加上一个点, 即

```perl6
<foo=.bar>
```

等价于

```perl6
$<foo> = <.bar>
```

同样地, 要显式的重命名一个本地作用域的 regex, 就在 `=` 号后面的标识符前面添加一个 `&`,

```perl6
<foo=&bar>
```

等价于

```perl6
$<foo> = <&bar>
```

多个别名也是允许的, 所以:

```perl6
<foo=pub=bar>
```

是下面这种形式的简写

```perl6
$<foo> = $<pub> = <bar>
```


类似地, 你也能给其它断言起别名, 例如:

```perl6
<foo=[abc]>    # 字符类, 等同于      $<foo>=<[abc]>
<foo=:Letter>  # unicode 属性,等同于 $<foo>=<:Letter>
<foo=:!Letter> # a negated unicode property lookup
```

例子:

```perl6
$_ = "aabdc";
m/<foo=[abc]>/;
say ~$<foo>;     # a
say $<foo>.WHAT; # Match

m/<foo=[abc]>+/;
say $<foo>.WHAT    # Array
say $<foo>[0].WHAT # Match
say ~$<foo>;       # a a b
```

如果标识符后面的第一个字符是**空格**, 则随后的文本(跟着任意空格)被解析为 *regex*, 所以:

```perl6
<foo bar>
```

或多或少,等价于

```perl6
<foo(/bar/)>  # 方法调用
```

要传递一个带有前置空格的 regex, 你必须使用加上括弧的形式。

如果标识符后面的第一个字符是一个`冒号后再跟着空格`, 那么闭合尖括号之前的剩余文本会被当作方法的**参数列表**, 就像普通 Perl 语法中的那样。所以这些意味着相同的东西:  ​

[S05-grammar/signatures.t lines 7–24](https://github.com/perl6/roast/blob/master/S05-grammar/signatures.t#L7-L24)

```perl6
<foo('foo', $bar, 42)>  # 函数调用, 果然, 标识符后面紧跟着圆括号一般就是函数调用
<foo: 'foo', $bar, 42>  # 冒号形式的函数调用
```

起始标识符的后面不再允许有其它字符。

例如, subrule 匹配在某种程度上是陈述性的, subrule 自身de 前面被认为是陈述性的。如果 subrule 包含了一个序列点, 那么 subrule 匹配也是。Longest-token 匹配不继续通过这样的 subrule。
​
这种形式总是给词法作用域的正则表达式声明以优先, 直接分派它, 就好像它是函数一样。如果作用域中没有这样的词法正则表达式(或词法方法),那么调用会分派给当前 grammar,假设有的话。
即, 如果在当前本地作用域有一个可见的 `my regex foo` 声明, 那么:

```perl6
<foo(1,2,3)>
```

等价于:

```perl6
<foo=&foo(1,2,3)>
```

然而, 如果没有这样的词法作用域的 regex (并且注意在 gramamr 中, regexes 被作为方法安装, 默认没有词法别名), 那么该调用被在当前 `Cursor` 上作为普通方法被分派。(这会失败, 如果当前你不在 grammar 中的话)。 所以在那种情况下:

```perl6
<foo(1,2,3)>
```

等价于:

```perl6
<foo=.foo(1,2,3)>
```
​
如果既没有任何它能调用的那个名字的词法作用域的子例程, 又没有任何以通过方法分发获得的那个名字的方法, 那么调用 `<foo>` 会失败。(决定使用哪个分派器是在编译时做出的; 方法调用不是回调机制。)
​
- 一个前置的 `.` 显式地把方法作为 subrule 调用; 实际上如果初始的字符不是字母数字的话会引起该具名断言不捕获它匹配到的东西。(查看 "Subrule captures") 例如:

[`S05-metasyntax/angle-brackets.t lines 140–242`](https://github.com/perl6/roast/blob/master/S05-metasyntax/angle-brackets.t#L140-L242)

```perl6
/ <ident>  <ws>  /      # $/<ident> 和 $/<ws> 都被捕获了
/ <.ident> <ws>  /      # 只有 $/<ws> 被捕获了
/ <.ident> <.ws> /      # 什么也没有捕获
```

该断言然后被恒等地解析为以一个标识符开始的断言，假设点号之后的下一个东西是一个标识符的话。至于标识符的形式，任何跟匹配引擎有关的额外参数都被自动地通过隐式的 `Cursor` 调用者提供给参数列表。如果没有当前类/grammar， 或者当前类不是派生于 `Cursor`， 那么该调用很可能会失败。
  ​  ​
如果点号后面跟着的不是标识符，那么它被解析为某种类型的 "dotty" 后缀，例如一个间接的方法调用:

```perl6
<.$indirect(@args)>
```

对于所有的正则匹配，当前的匹配状态（某些 `Cursor` 的衍生物）被作为第一个参数传递，它在这种情况下就是该方法的调用者。这个方法期望返回一个新匹配状态对象的惰性列表，或者返回 `Nil` 如果匹配彻底失败的话。像棘轮般转动的程序通常会返回一个只含有一个匹配的列表。
  ​
- 鉴于一个前置的 `.` 明确无误地调用一个方法，一个前置的 `&` 明确无误地调用了子例程代替。这样的一个正则表达式程序必须使用 `my` 或 `our` 作用域声明(或导入)以让它的名字在本地作用域中可见，因为默认地正则表达式的名字仅被安装到当前类的元对象实例中，就像普通方法那样。 那个程序充当着一种私有的 submethod，
并且调用时不用考虑任何继承。它仍旧必须接收一个 **Cursor** 作为它的第一个参数(它能把它当做一个调用者如果它喜欢的话)，并且必须返回那个新的匹配状态作为游标对象。因此,

[`S05-metasyntax/interpolating-closure.t lines 17–39`](https://github.com/perl6/roast/blob/master/S05-metasyntax/interpolating-closure.t#L17-L39)

```perl6
<&foo(1,2,3)>
```

对于某些像如下这样的东四是一种语法糖:  is sugar for something like:

```perl6
<.gather { take foo($¢,1,2,3) }>
```

其中 `$c` 代表当前传入的匹配状态，并且程序必须在失败时返回 `Nil`，如果匹配成功则返回一个含有一个或多个匹配状态(`Cursor` 驱动的对象)的惰性列表。
正如 `.` 形式一样， 一个显式的 `&` 抑制了捕获。

注意所有正常的 `Regex` 对象实际上是伪装成这样的程序。当你说:

```perl6
rx/stuff/
```

你实际上正在声明一个匿名的方法，某些像这样的东西：

```perl6
my $internal = anon regex :: ($¢: ) { stuff }
```

- 一个前置的 `$` 标示着一个间接的 subrule 调用。那个变量要么包含一个 `Regex` 对象(实际上是一个匿名的方法 -- 看上面), 要买包含一个将被编译为正则表达式的字符串。那个字符串绝对不会按照字面值匹配。

如果字符串形式的编译失败，那么错误信息会被转换为警告并且那个断言会失败。
那个间接的 subrule 断言没有被捕获。（默认地没有带有前置的标点符号的断言会被捕获）当然，你总是可以显式地捕获它:
  ​
```perl6
/ <name=$rx> /
```

间接的 subrule 总是被看作是过程式的，并且可能不会参与最长 token 匹配。

- 一个前置的 `::` 标示着一个符号的间接 subrule:

```perl6
/ <::($somename)> /
```
  ​
那个变量必须包含 subrule 的名字。按照单个方法分发的规则，在当前 grammar 和它的祖先中这被首先搜索。如果这个搜索失败了，那么会尝试通过 MMD 进行分发，在这种情况下，它可以找到定义为 *multis* 的 subrules 而非 方法。默认地这种形式不会被捕获。它总是被当作是过程式的，不是声明式的。

- 一个前置的 `@` 像一个裸的数组那样匹配除了每个元素被看做 subrule 之外（字符串或 `Regex` 对象）而非被当做字面值。即，字符串被强制编译为 subrule 而不是按照字面值匹配。(对于 `Regex` 对象没有区别。)

这个断言不会被自动捕获。  
​
- 散列作为断言的用法被保留了。
- 一个前置的 `{` 标示在那个位置上产生要被插值到模式中作为 subrule 的 regex 的代码：

```perl6
/ (<.ident>)  <{ %cache{$0} //= get_body_for($0) }> /
```

​那个闭包确保会在标准时运行；它声明了一个序列点，并且别看作是过程式的。

- 在任何正则表达式插值情况下，如果那个值碰巧已经是一个  `Regex` 对象，那么它不会被编译。如果它是一个字符串，那么带有字符串的编译形式被缓存，以至于下次你使用它的时候它不会重新编译除非字符串发生更改。（然而任何外部词法变量名必须被重新装订。）带有不平衡括号的 Subrules 可能不被插值。插值过的 subrule 把它自己内部的结果保存为单个标量，所以它的括号绝对不会计算到外部的正则表达式分组中。（换句话说，圆括号编号总是本地作用域的。）

- 在 `<...>` 中, 一个前置的 `?{` 或 `!{` 标示着代码断言:
[`S05-metasyntax/assertions.t lines 7–25`](https://github.com/perl6/roast/blob/master/S05-metasyntax/assertions.t#L7-L25)
```
/ (\d**1..3) <?{ $0 < 256 }> /
/ (\d**1..3) <!{ $0 < 256 }> /
```

类似于:
```
/ (\d**1..3) { $0 < 256 or fail } /
/ (\d**1..3) { $0 < 256 and fail } /
```
  ​
不像闭包那样, 代码断言被认为是陈述性质的; 
  ​
```perl6
my $str = "foo123bar";
$str  ~~ token { foo .* <?{ do { say "Got here!" } or 1 }> .* bar } # Got here!
```

`do` block 不太可能运行,除非字符串以 "bar" 结尾.

- 一个前置的 `[` 标示着可枚举的字符类. 在枚举字符类中, 范围是由 `..` 而非 `-` 来标示的.
[`S05-metasyntax/charset.t lines 18–22`](https://github.com/perl6/roast/blob/master/S05-metasyntax/charset.t#L18-L22)
[`S05-mass/rx.t lines 248–262`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L248-L262)
[`S05-mass/rx.t lines 283–428`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L283-L428)
  ​
```perl6
/ <[a..z_]>* /
```

方括号内的`空白`被忽略:

```perl6
/ <[ a .. z _ ]>* /
/ <[ . _ ]>* /
```

反转的范围是非法的. 在直接编译的代码中会报错如果你写成这样:

```perl6
/ <[ z .. a ]> /  # 反转的范围是不允许的
```

在间接编译的代码中, 出现类似的问题并使断言失败:

```perl6
$rx = '<[ z .. a ]>';
/ <$rx> /;  # warns and never matches
```

-  前置的 `-` 标示互补字符类:
[`S05-metasyntax/charset.t lines 23–27`](https://github.com/perl6/roast/blob/master/S05-metasyntax/charset.t#L23-L27)
[`S05-mass/rx.t lines 263–282`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L263-L282)

```perl6
/ <-[a..z_]> <-alpha> /
/ <- [a..z_]> <- alpha> /  #  - 后面允许有空格
```

这在本质上与使用 `否定向前查看` 和 `点` 是相同的.

```perl6
/ <![a..z_]> . <!alpha> . / # `!`标示前面不是什么
```

初始的 `-` 之后的空白被忽略.

- 一个前置的 `+` 也能标示后面的字符类会以肯定的意义匹配:
[`S05-mass/rx.t lines 511–2140`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L511-L2140)

```perl6
/ <+[a..z_]>* /
/ <+[ a..z _ ]>* /
/ <+ [ a .. z _ ] >* /      # whitespace allowed after +
```

- 在单个尖括号集中, 字符类可以组合(相加或相减). 空白被忽略. 例如:
[`S05-metasyntax/charset.t lines 28–96`](https://github.com/perl6/roast/blob/master/S05-metasyntax/charset.t#L28-L96)

```perl6
/ <[a..z] - [aeiou] + xdigit> /      # 辅音或十六进制数
```

一个具名的字符类可以使用它自己:

```perl6
<alpha>
```

然而, 为了组合字符类, 必须在`具名`字符类前面前置一个 `+` 或 `-`。在任何 `-` 可能会被误解析为一个标识符扩展器的前面需要有空格。

- 使用 pair 记法代替一个正常的 rule 的名字来标记 Unicode 属性:

[`S05-metasyntax/unicode-property-pair.t lines 6–21`](https://github.com/perl6/roast/blob/master/S05-metasyntax/unicode-property-pair.t#L6-L21)

```perl6
<:Letter>   # a letter
<:!Letter>  # a non-letter
```

带参数的属性作为参数传递给 pair:

```perl6
<:East_Asian_Width<Narrow>>
<:!Blk<ASCII>>
```

这个 pair 值与 Unicode 数据库中的值相智能匹配。 

```perl6
<:Nv(0 ^..^ 1)>     # Nv 代表数值, 该正则匹配特有分数值的字符

'flubber¼½worms' ~~ /<:NumericValue(0 ^..^ 1)>+/;  # ~$/ => ¼½
```

作为智能匹配的一种特殊情况, TR18 的第 2.6 章节也允许使用模式作为参数:

```perl6
<:name(/^LATIN LETTER.*P$/)>

'FooBar' ~~ /<:name(/:s LATIN SMALL LETTER/)>+/;     #  'oo', 'match character names';
'FooBar' ~~ /<:Name(/:s LATIN CAPITAL LETTER/)>+/;   #  'F',  'match character names';
```

- 多个这样的项(terms)可以使用加号和减号组合到一块儿:

```perl6
<+ :HexDigit - :Upper >
```

项之间也能使用 `&` 组合起来用于集合交集，或使用 `|` 用于集合并集，还有使用 `^` 用于对称的集合的差集。括号能用于分组。（方括号总是括起字面的字符(包括反斜线字面形式)，并且可能没有嵌套，不像 TR18 章节 1.3 中建议的记号那样。）操作符的优先级和["Operator precedence" in S03](http://design.perl6.org/S03.html#Operator_precedence)中对应的具名操作符的优先级相同，即使它们的语义稍微有点不同。

- 额外的长字符可以通过引用来键入并通过交叉来包含。合适的时候,任何引起的字符都会被当作"最长 tokens"。 这儿 'll' 会在 'l' 之前被识别:

```perl6
/ <[ a..z ] | 'ñ' | 'ch' | 'll' | 'rr'> /
```

​注意包含"长字符"的否定字符类总是提前单个字符。

- 当任何诸如 `\c`, `\x`, or `\o` 的字符构造结构包含多个由逗号分割的值时，这些值就被看做"长字符"。所以你可以把 `\c[13,10]` 添加到上面的列表中以把 CRLF 作为一个长字符匹配。
  ​
当那个长字符没有作为一个整体被匹配时，那么否定形式的就会提前一个单个字符(像 `.` 那样匹配)。因此，这个会匹配：
  
```perl6
"\c[13,13,10,10]" ~~ /\C[13,10]* \c[13,10] \C[13,10]/;
```

如果你想要的是 \C13\C10，那么你就那样写好了。
​
- 一个前置的 `!` 标示否定的意思(总是一个零宽断言):

[`S05-metasyntax/angle-brackets.t lines 243–321`](https://github.com/perl6/roast/blob/master/S05-metasyntax/angle-brackets.t#L243-L321)
[`S05-mass/rx.t lines 2141–2377`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L2141-L2377)
​
```perl6
/ <!before _ > /    # 我们不在 _ 前面
```

注意 `<!alpha>` 和 `<-alpha>` 是不同的.  `/<-alpha>/` 是一个互补字符类 , 它等价于 `/<!before <alpha>> ./`,  而 `<!alpha>` 是一个零宽断言, 它等价于  `/<!before <alpha>>/` .

还要注意作为一个元字符, `!`不改变它后面所跟的任何东西的解析规则(这点与 + 或 - 不同)

- 一个前置的 `?` 标示正向的零宽断言, 并且像 `!` 只是重新递归解析剩下的断言, 就像 `?` 不存在那一样. 此外, 要强制零宽断言, 它也能抑制任何具名捕获:
[`S05-mass/rx.t lines 429–451`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L429-L451)

```perl6
<alpha>     # 匹配一个字母,并捕获到 `$alpha` (最终是捕获到 $<alpha>)
<.alpha>    # 匹配一个字母,不捕获
<?alpha>    # match null before a letter, 不捕获
```

特殊的具名断言包括:
[`S05-metasyntax/lookaround.t lines 13–27`](https://github.com/perl6/roast/blob/master/S05-metasyntax/lookaround.t#L13-L27)
[`S05-mass/rx.t lines 452–510`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L452-L510)
[`S05-mass/charsets.t lines 5–59`](https://github.com/perl6/roast/blob/master/S05-mass/charsets.t#L5-L59)
[`S05-mass/stdrules.t lines 15–309`](https://github.com/perl6/roast/blob/master/S05-mass/stdrules.t#L15-L309)

```perl6
/ <?before pattern> /    # lookahead  向前查看
/ <?after pattern> /     # lookbehind 向后查看
```

```perl6
/ <?same> /              # true between two identical characters
```

```perl6
/ <.ws> /                # match "whitespace":
                         # \s+ if it's between two \w characters,
                         # \s* otherwise
```

```perl6
/ <?at($pos)> /          # 只在特定位置匹配
                         # 是 <?{ .pos === $pos }> 的简写形式
                         # (considered declarative until $pos changes)
```

​通过省略前面的标点符号把这些断言作为具名捕获使用是合法的。然而, 捕获需要一些内存和计算消耗, 所以你一般会抑制捕获不感兴趣的数据。
`after` 断言通过反转语法树并以与左相反的顺序查找东西来实现向后查看。在不能反转的模式上做向后查看是违反规则的。  ​
注意：向前扫描向后查看的效果在顶层可以使用这个达成：

```perl6
/ .*? prestuff <( mainpat )> /
```

- 一个前置的 `*` 标示后面的模式允许部分匹配. 匹配尽可能多的字符之后,它总是能成功. (它不是零宽的除非它匹配了0个字符). 例如, 要匹配一些缩写词, 你可以写下面任意一个:

```perl6
s/ ^ G<*n|enesis>     $ /gen/  or
s/ ^ Ex<*odus>        $ /ex/   or
s/ ^ L<*v|eviticus>   $ /lev/  or
s/ ^ N<*m|umbers>     $ /num/  or
s/ ^ D<*t|euteronomy> $ /deut/ or

...
```

```perl6
/ (<* < foo bar baz > >) /
```

```perl6
/ <short=*@abbrev> / and return %long{$<short>} || $<short>;
```

这儿省略了一堆尚未实现的内容。​

- 一个前置的 `|` 标示某种零宽边界. 使用这个语法你可以引用反引号序列; `<|h>` 会在 \h 和  \H 之间匹配, 例如. 一些例子:

```perl6
<|w> 单词边界
<|g> 字素边界 (总是在字素模式下匹配)
<|c> 代码点边界 (总是在 `字素/代码点` 模式下匹配)
```

下面的 tokens 包含尖号但平衡不是必须的:

- `<(` token 标示匹配的全部捕获的开头, 而对应的 `)>` token 标示它的终点。当匹配后, 这些表现像断言的总为真, 但是有设置匹配对象的 `.from` 和 `.to` 属性的副作用。即：

```perl6
/ foo <( \d+ )> bar /
```

等价于:

```perl6
/ <?after foo> \d+ <?before bar> /
```


- `«` 或 `<<` token 标示左单词边界。 `»` 或 `>>` token 标示右单词边界。(作为单独的 tokens, 这些 tokens 不需要保持平衡)。Perl 5'的 `\b` 被 `<|w>` "单词边界" 断言代替, 而 `\B` 变为 `丢失了`.
