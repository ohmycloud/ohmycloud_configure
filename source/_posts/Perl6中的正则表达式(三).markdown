---
title: Perl 6 中的正则表达式(三)
date: 2015-11-05
tags: 正则表达式
categories: Perl 6
comments: true
---

# 预定义 Subrules

下面这些是为任意 grammar 或 regex 预定义好的 `subrules`:

- ident 

  匹配一个标识符.

- upper 

  匹配单个大写字符.

- lower 

  匹配单个小写字符.

- alpha 

  匹配单个字母字符, 或者是一个下划线.

  要匹配不带下划线的 Unicode 字母字符, 使用 `<:alpha>`.

- digit 

  匹配单个数字.

- xdigit 

  匹配单个十六进制数字.

- print 

  匹配单个可打印字符.

- graph 

  匹配单个图形字符.

- cntrl 

  匹配单个控制字符. (等价于 <:Cc> 属性). 控制字符通常不产生输出, 相反, 它们以某种方式控制末端:例如换行符和退格符都是控制字符. 所有使用 `ord()` 之后小于 32 的字符通常归类为控制字符. 就像 ord() 的值为 127 的字符是控制字符(DEL) 一样, 128 到 159 之间的也是控制字符.

- punct 

  匹配单个标点符号字符(即, 任何来自于 Unicode General Category "Punctuation" 的字符).

- alnum 

  匹配单个字母数字字符. 那等价于 `<+alpha +digit>` .

- wb 

  在单词边界匹配成功并返回一个零宽匹配.  一个单词边界是这样一个点, 它的一边是一个 `\w`, 另一边是一个 `\W`. (以任一顺序),  字符串的开头和结尾被看作为匹配 `\W`.

- ww 

  在两个单词字符之间匹配(零宽匹配).

- ws 

  在两个单词字符之间匹配要求的空白, 否则匹配可选的空白. 这等价于 ` \s*` (`ws` 不要求使用 `ww` subrule).

- space 

  匹配单个空白字符character (和 `\s 相同 `).

- blank 

  匹配单个 "blank" 字符 -- 在大部分区域, 这相当于 `space` 和 `tab`.

- before `pattern`

  执行向前查看-- 例如, 检查我们是否在某个能匹配的位置. 如果匹配成功返回一个零宽 Match 对象.

- after `pattern`

  执行向后查看 --例如,检查当前位置之前的字符串是否匹配 `<pattern>`(在结尾锚定). 如果匹配成功就返回一个零宽 Match 对象. 

- <?> 

  匹配一个 null 字符串,即, 总是返回真. 

- <!> 

  `<?>` 的反转, 总是返回假
[`S05-mass/stdrules.t lines 310–326`](https://github.com/perl6/roast/blob/master/S05-mass/stdrules.t#L310-L326)
  ​
# 反斜线改良

- 很多 `\p` 和 `\P` 属性变成诸如 `<alpha>` 和 `<-alpha>` 等内在的 grammar rules. 它们可以使用上面提到的字符类标记法进行组合.`<[-]+alpha+digit>`. 不管高层级字符类的名字, 所有的低层级 Unicode 属性总是可以使用一个前置冒号访问, 即, 在尖括号中使用 pair 标记法.因此 `<+:Lu+:Lt>` 等价于 `<+upper+title>`.

-  `\L...\E`, `\U...\E`, 和 `\Q...\E` 序列被废弃了. 单个字符的大小写修饰符 `\l` 和 `\u` 也被废弃了. 在极少需要使用它们的地方, 你可以使用 `<{ lc $regex }>`, `<{tc $word}>`, 等.

- 就像上面提到的, `\b` 和 `\B` 单词边界断言被废弃了, 并且被 `<|w>` (或 `<wb>`) 和 `<!|w>` (或 `<!wb>`) 零宽断言代替.

- `\G`  序列也没有了. 使用 `:p` 代替. (注意, 虽然, 在模式内使用 `:p` 没有影响, 因为每个内部模式都被隐式的锚定到当前位置) 查看下面的 at 断言.

- 向后引用 (例如. `\1`, `\2`, 等.) 都没有了; 可以使用 `$0`, `$1`, 等代替. 因为正则中变量不再进行插值.
  [`S05-capture/dot.t lines 56–61`](https://github.com/perl6/roast/blob/master/S05-capture/dot.t#L56-L61)
  ​
数字变量被假定是每次都会改变的, 因此被看作是程序化的, 不像普通变量那样.
 ​
-  新的反斜线序列, `\h` 和 `\v`, 分别匹配水平空白和垂直空白, 包括 Unicode. 水平空白被定义为任何匹配 `\s` 并且不匹配 `\v` 的东西. 垂直空白被定义为下面的任一方式:

```
U+000A  LINE FEED (LF)
U+000B  LINE TABULATION
U+000C  FORM FEED (FF)
U+000D  CARRIAGE RETURN (CR)
U+0085  NEXT LINE (NEL)
U+2028  LINE SEPARATOR
U+2029  PARAGRAPH SEPARATOR
```
 ​
注意 `U+000D` (CARRIAGE RETURN) 被认为是垂直空白.
 ​
- `\s` 现在匹配任何 Unicode 空白字符.
- 新的反斜线序列和 `\N` 匹配除逻辑换行符之外的任何字符, 它是 `\n` 的否定.
- 其它新的大写的反斜线序列也都是它们小写身份的反义:

  - `\H` 匹配任何非水平空白字符.
  - `\V` 匹配任何非垂直空白字符.
  - `\T` 匹配任何非 tab 字符.
  - `\R` 匹配任何非 return 字符.
  - `\F` 匹配任何非格式字符.
  - `\E` 匹配任何非转义字符.
  - `\X...` 匹配任何非指定的(指定为十六进制)字符.
  - 在普通字符串中反斜线转义字面字符串在 regexes 中是允许的(\a, \x 等等). 然而, 这个规则的例外是 `\b`., 它被禁用了,为了避免跟之前作为单词边界断言冲突. 要匹配字面反斜线, 使用 `\c8`, `\x8`或双引号引起的 `\b`.

# 便捷的字符类

因为历史原因和使用方便, 下面的字符类可以作为反斜线序列使用:

```
\d      <digit>    A digit
\D      <-digit>   A nondigit
\w      <alnum>    A word character
\W      <-alnum>   A non-word character
\s      <sp>       A whitespace character
\S      <-sp>      A non-whitespace character
\h                 A horizontal whitespace
\H                 A non-horizontal whitespace
\v                 A vertical whitespace
\V                 A non-vertical whitespace
```

# Regexes构成一等语言

而不仅仅是字符串.

- Perl 5 的 `qr/pattern/` 正则构造器滚蛋了.

- Perl 6 中的正则构造:

```perl6
regex { pattern }    # 总是把 {...} 作为模式分隔符
rx    / pattern /    # 几乎能使用任何字符作为模式分割符
```

[`S05-metasyntax/delimiters.t lines 6–20`](https://github.com/perl6/roast/blob/master/S05-metasyntax/delimiters.t#L6-L20)

你不能使用空格或字母数字作为分隔符.你可以使用圆括号作为你的 rx 分隔符, 但是这只有当你插入空格时才行(标识符后面直接跟着圆括号会被认为是函数调用):

```perl6
rx ( pattern )      # okay
rx( 1,2,3 )         # 尝试调用 rx 函数
```

(在 Perl 6 中 这对所有类似 quotelike 的结构都适用.)

 `//` 匹配能使用的地方 `rx` 这种形式也能直接作为模式使用. `regex` 这种形式实际上是一个方法定义, 必须用于 grammar 类中.

- 如果 `regex` 或 `rx` 需要修饰符, 就把修饰符直接放在开放分隔符前面:

```perl6
$regex = regex :s:i { my name is (.*) };
$regex = rx:s:i     / my name is (.*) /;    # same thing
```

如果使用任何括号字符作为分隔符, 那么最后的修饰符之后是需要空格的. ( 否则, 它会被看作修饰符的参数)

-  不能使用冒号作为分隔符. 修饰符之间可以有空格:

```perl6
$regex = rx :s :i / my name is (.*) /;
```

- 正则构建器的名字是从 qr 修改而来, 因为它不再是一个能插值的 quote-like 操作符. `rx` 是 `regex` 的简写形式.(不要对regular expressions 感到困惑, 除了它们是的时候 )

- 像语法指示的那样, 它现在跟 `sub { ... }` 构建器很像. 实际上, 这种类似物深深根植于 Perl 6 中.

- 就像一个原始的 `{...}`现在总是一个闭包 (它仍然能在特定上下文中被立即执行并在其它上下文中作为对象传递), 所以原始的 `/.../` 总是一个 `Regex` 对象(它可能仍然在特定上下文中被立即匹配并在其它上下文中作为对象传递.)

- 特别地, 在 value 上下文(`sink`, `Boolean`, `string` 或 `numeric`),或它是 `~~` 的显式参数时, `/.../`会立即匹配. 否则, 它就是一个跟显式的 regex 形式同一的`Regex` 构建器, 所以这个:

```perl6
$var = /pattern/;
```

不再进行匹配并把结果设置为 `$var`. 相反, 它把一个 `Regex` 对象赋值给 `$var`.

- 这种情况总是可以使用 `m{...}` 或 `rx{...}`进行区分:

```perl6
$match = m{pattern};    # 立刻匹配 regex, 然后把匹配结果赋值给变量
$regex = rx{pattern};   # Assign regex expression itself
```

- 注意前面有个像这样魔法般的用法:

```perl6
@list = split /pattern/, $str;
```

现在来看就是理所当然的了.

- 就是现在, 建立一个像 grep 那样的用户自定义子例程也成为可能:

```perl6
sub my_grep($selector, *@list) {
  given $selector {
    when Regex { ... }
    when Code  { ... }
    when Hash  { ... }
    # etc.
  }
}
```

当你调用 `my_grep` 时, 第一个参数被绑定到 item 上下文, 所以传递 `{...}` 或 `/.../` 产生 `Code` 或 `Regex` 对象,  switch 语句随后选中它作为条件.(正常的 grep 只是让智能匹配操作符做了所有的工作)

- 就像 `rx` 拥有变体一样, `regex` 声明符也有. 特别地, regex 有两个用在 grammars 中的变体: `token` 和 `rule`.

token声明长这样:

```perl6
token ident { [ <alpha> | \- ] \w* }
```

默认**从不回溯**. 即, 它倾向于保留任何目前位置扫描到的东西.所以,上面的代码等价于:

```perl6
regex ident { [ <alpha>: | \-: ]: \w*: }
```

但是相当易读. 在 token 中裸的 `*`, `+` 和 `?` 量词绝不回溯. 在普通的 regexes 中, 使用 `*:`, `+:`, 或 `?:` 阻止量词的回溯. 如果你确实要回溯, 在量词后面添加 `?` 或 `!`.  `?` 像平常一样进行非懒惰匹配, 而 `!` 强制进行贪婪匹配. `token` 声明符就是

```perl6
regex :ratchet { ... }
```
的简写.

另外一个是 `rule` 声明符, 像 token 一样, 它默认也不会回溯. 此外, 一个 `rule` 这样的正则表达式也采取了 `:sigspace` 修饰符. `rule` 实际上是

```perl6
regex :ratchet :sigspace { ... }
```

的简写.  ratchet 这个单词的意思是: (防倒转的)棘齿, 意思它是不能回溯的!

-  Perl 5 的 `?...?` 语法(成功一次)极少使用, 并且现在能使用 `state` 变量以更清晰的方式模拟:

```perl6
$result = do { state $x ||= m/ pattern /; }    # 只在第一次匹配
```

要重置模式, 仅仅让 `$x = 0` 尽管你想要 `$x` 可见, 你还是必须避免使用 block:
```
$result = state $x ||= m/ pattern /;
...
$x = 0;
```

# 回溯控制

在这些被认为是程序的而非陈述性的模式的部分中, 你可以控制回溯行为.

- 默认的, 在 `rx`,`m`, `s` 中的回溯是贪婪的. 在普通的 regex 声明中它们也是贪婪的. 在 rule 和 token 声明中, 回溯必须是显式的.

- 为了强制前面的原子执行节俭回溯(有时也是所谓的"急切的匹配" 或 "最少化的匹配"), 要在原子后面追加一个 `:?` 或 `?`. 如果前面的 token 是一个量词, `:` 就可以被省略, 所以 `*?` 就像在 Perl 5 中那样起作用.

- 为了强制前面的原子执行贪婪回溯, 在原子后面追加一个 `:!`.  如果前面的 token 是一个量词, `:` 可以被省略. (Perl 5 没有对应的结构, 因为在 Perl 5 中回溯默认是贪婪的.)

- 为了强制前面的原子不执行回溯, 使用不带 `?` 或 `!` 的单个 `:`.  单个冒号让正则引擎不再重试前面的原子:
  [`S05-mass/rx.t lines 8–32`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L8-L32)

```perl6
ms/ \( <expr> [ , <expr> ]*: \) /
```

(i.e. there's no point trying fewer `` matches, if there's no closing parenthesis on the horizon)

当修饰一个量词的时候,  可以使用 `+` 代替 `:`, 这种情况下, 量词常常是所谓的占有量词.
```
ms/ \( <expr> [ , <expr> ]*+ \) /  # same thing
```

为了强制表达式中所有的原子默认不去回溯, 要使用 `:ratchet` 或 `rule` 或 `token`.

- 对双冒号进行求值会抛弃当前 LTM 备选分支中所有保存的选择点。
[S05-mass/rx.t lines 33–51](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L33-L51)
​
```perl6
ms/ [ if :: <expr> <block>
   | for :: <list> <block>
   | loop :: <loop_controls>? <block>
   ]
/
```
​
到目前为止 2016.4.29 ，(`::`) 还没有实现。

`::` 还有通过 `|` 从「最长 token」中隐藏右侧的任何陈述性匹配的效果。为了确定性，只有 `::` 左侧的东西被求值。

如果没有当前 LTM 备选分支，那么 `::` 什么也不会做。「当前」是被动态地定义的，而非词法地。*subrule* 中的 `::` 会影响闭合的备选分支。
​  ​
- 对`::>`进行求值会抛弃当前最里面的临时备选分支中所有保存的选择点。它因此表现得像 "then" 那样。

```perl6
ms/ [
    || <?{ $a == 1 }> ::> <foo>
    || <?{ $a == 2 }> ::> <bar>
    || <?{ $a == 3 }> ::> <baz>
    ]
/
```

这儿省略了一部分还没实现的内容。

# Regex 子例程, 具名和匿名

- `sub` 和 `regex` 之间的类推更为相似.

- 就像你可以有匿名子例程和具名子例程...

- 所以你也可以有匿名正则和具名正则(还有 tokens 和 rules)

```perl6
token ident { [<alpha>|\-] \w* }
# 然后...
@ids = grep /<ident>/, @strings;
```

- 就像上面的例子标示的那样, 引用具名正则也是可以的, 例如:

```perl6
regex serial_number { <[A..Z]> \d**8 }
token type { alpha | beta | production | deprecated | legacy }
```

在其它作为具名断言的正则表达式中:

```perl6
rule identification { [soft|hard]ware <type> <serial_number> }
```

这些使用关键字声明的 regexes 官方类型为 `method`, 是从 `Routine` 派生出来的.
​
通常, 任何 subrule 的锚定是由它的调用上下文控制的. 当 regex, token, 或 rule 方法被作为 subrule 调用时, 前面被锚定到当前位置(与 `:p` 一样), 而后面没有被锚定, 因为调用上下文可能会继续解析. 然而, 当这样一个方法被直接智能匹配, 它会自动的锚定两端到字符串的开始和结尾. 因此, 你可以使用一个匿名的 regex 子例程作为单独的模式来直接模式匹配:
  ​
```perl6
$string ~~ regex { \d+ }
$string ~~ token { \d+ }
$string ~~ rule  { \d+ }
```

它们等价于

```perl6
$string ~~ m/^ \d+ $/;
$string ~~ m/^ \d+: $/;
$string ~~ m/^ <.ws> \d+: <.ws> $/;
```

基本经验就是关键字定义的方法绝对不会做 `.*?` 那样的扫描, 而如引号那种形式的 `m//`和`s///` 在缺少显式锚定的时候会做这样的扫描.

`rx//` 和`//` 这两种形式, 怎么扫描都可以: 当被直接用在智能匹配或布尔上下文中时, 但是当它被作为一个 subrule 间接被调用时,就不会扫描. 即, 当直接使用时, `rx//` 返回的对象表现的像 ` m//`, 但是用作 subrule 时, 表现的就像 `regex {}`:

```perl6
$pattern = rx/foo/;
$string ~~ $pattern;                  # 等价于 m/foo/;
$string ~~ /'[' <$pattern> ']'/       # 等价于 /'[foo]'/
```

# 空模式是非法的
[`S05-mass/rx.t lines 2378–2392`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L2378-L2392)
[`S05-metasyntax/null.t lines 17–25`](https://github.com/perl6/roast/blob/master/S05-metasyntax/null.t#L17-L25)

在 Perl 6 中, `Null regex` 是非法的：

```perl6
//
```

要匹配一个零宽字符, 需要显式地表示 null 匹配：

```perl6
 / '' /;
 / <?> /;
```

例如：

```perl6
split /''/, $string
```

 分割字符串。所以这样也行：

```perl6
split '', $string
```

同样地，匹配一个空的分支，使用这个：

```perl6
 /a|b|c|<?>/
 /a|b|c|''/
```

更容易捕获错误：

```perl6
/a|b|c|/
```

 作为一种特殊情况, 匹配中的第一个 null 分支会被忽略：

```perl6
ms/ [
         | if :: <expr> <block>
         | for :: <list> <block>
         | loop :: <loop_controls>? <block>
    ]
  /
```

这在格式化 regex 时会有用。

但是注意, 只有`第一个分支`是特殊的, 如果你这样写：

```perl6
ms/ [
             if :: <expr> <block>              |
             for :: <list> <block>             |
             loop :: <loop_controls>? <block>  |
    ]
  /
```

 就是错的。因为最后一个分支是 null 。
 然而, non-null句法结构有一种退化的情况能匹配 null 字符串:

```perl6
$something = "";
/a|b|c|$something/;
```

特别地,  `<?>` 总是成功地匹配 null 字符串,  并且 `<!>` 总是让任何匹配都会失败.
