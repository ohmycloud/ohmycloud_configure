---
title: Perl 6 中的正则表达式(一)
date: 2016-03-08
tags: 正则表达式
categories: Perl 6
comments: true
---

# 标题

Synopsis 5: Regexes and Rules

# 版本

```
创建于: 2002/06/24

上次修改: 2015/05/12
版本: 180
```

不论何时, 在 `grammar` 中引用递归模式时, 通常更偏好使用 `token` 和 `rule`, 而不是 `regex`.

# 概览

作为常规表达式记法的扩展, Perl 6 原生地实现了 `Parsing Expression Grammars`(PEGs). PEGs 要求你为有歧义的那部分提供一个 `主从秩序`.  Perl 6 的 `主从秩序` 由一个多级的平局择优法测试决定:

```
    1) Most-derived only/proto hides less-derived of the same name
    2) 最长 token 匹配: food\s+ beats foo by 2 or more positions
    3) 最长字面值前缀: food\w* beats foo\w* by 1 position
    4) 对于一个给定的 proto, multis from a more-derived grammar win
    5) 在一个给定的编译单元中, 出现较早的备选分支或 multi 胜出.
```

`#3` 会把任何初始的字面序列当作最长字面值前缀. 如果在最长 token 匹配中有一个嵌入的备选分支, 那些备选分支会扩展字面值前缀,   把备选分支也作为字面值的一部分. 如果所有的备选分支都是字面值的, 那么字面值也能延伸到备选分支的末尾, 当它们重新聚合时.  否则,  备选分支的末尾会终止所有的最长字面值前缀, 即使分支全部是字面值的.  例如:

```perl6
/ a [ 1 | 2 ] b /   # 最长字面值是 'a1b' 和 'a2b'
/ a [ 1 | 2\w ] b / # 最长文字是'a1 和 'a2', \w 不是字面值
/ a <[ 1 2 ]> b /   # 最长字面值是 'a'
```

注意, 这种情况下, 字符类和备选分支被不同地对待. 在字符类中包含一个最长的字面字符串太普遍了.

就像最长 *token* 匹配一样, 最长字面值前缀贯穿于 *subrules* 中. 如果 *subrule* 是一个 *protoregex*, 它会被看作带有 `|` 的备选分支, 后面跟着扩展或终止最长字面值前缀的同一个 *rules*.

除了这个主从秩序以外, 如果任何 *rule* 选择于主从秩序回溯之下, 那么选择下一个最好的 *rule*.  即, 主从秩序决定了候选者列表; 正是因为选择一个候选者并不意味着放弃其它候选者. 然而,  能通过一个恰当的回溯控制显式地放弃其它候选者(有时叫它 `cut` 操作符, 但是  Perl 6 有多个 *cut* , 取决于你想切掉多少)

还有, 任何在 `#1` 下被选中执行的 *rule*  可以选择委托给它的祖先来执行; PEG 不对此回溯.

# 新的匹配结果和捕获变量

附属的匹配对象现在可通过 `$/` 变量获取, 它隐式是词法作用域的. 通过这个变量来访问最近一次的匹配.  单独的捕获变量(例如 `$0`, `$1`等) 正是 `$/` 中的元素.
顺便说一下, 不像 Perl 5, Perl 6 中的捕获变量现在从 `$0` 开始编号而不是 `$1`. 查看下面.
为了检测 Perl 5的不相关的 `$/` 变量的意外使用, Perl 6 的 `$/` 变量不能被直接赋值.

```perl6
$/ = $x;   # 不支持使用  $/ 变量作为输入记录分隔符 (input record separator)
$/ := $x;  # OK, 绑定
$/ RR= $x; # OK, 元操作符
($/) = $x; # OK, 列表赋值
```

# 没变的语法特性

下面的正则特性语法和 Perl 5 是一样的:

[`S05-mass/rx.t lines 100–247`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L100-L247)

- 捕获: `(…)`
- 重复量词: `*`, `+`, 和  `?`
- 备选分支: `|`
- 反斜线转义: `\`
- 最少匹配后缀: `??`, `*?`, `+?`

虽然 `|` 的语法没有变, 但是默认的语义稍微改变了一点.  我们试图混合一个令人满意的描述性的和程序上的匹配, 以至于我们能拥有其中两者. 简言之, 你不用给 *grammar* 写你自己的 `tokener`了, 因为 Perl 会帮你写好. 查看下面的 `Longest-token` 匹配.

[`S05-metasyntax/longest-alternative.t lines 6–52`](https://github.com/perl6/roast/blob/master/S05-metasyntax/longest-alternative.t#L6-L52)

```perl6
use v6;
use Test;

plan 53;

#L<S05/Unchanged syntactic features/"While the syntax of | does not change">

my $str = 'a' x 7;

{
    ok $str ~~ m:c(0)/a|aa|aaaa/, 'basic sanity with |';
    is ~$/, 'aaaa', 'Longest alternative wins 1';

    ok $str ~~ m:c(4)/a|aa|aaaa/, 'Second match still works';
    is ~$/, 'aa',   'Longest alternative wins 2';

    ok $str ~~ m:c(6)/a|aa|aaaa/, 'Third match still works';
    is ~$/, 'a',    'Only one alternative left';

    ok $str !~~ m:c(7)/a|aa|aaaa/, 'No fourth match';
}

# now test with different order in the regex - it shouldn't matter at all

#?niecza skip 'Regex modifier g not yet implemented'
{
    ok $str ~~ m:c/aa|a|aaaa/, 'basic sanity with |, different order';
    is ~$/, 'aaaa', 'Longest alternative wins 1, different order';

    ok $str ~~ m:c/aa|a|aaaa/, 'Second match still works, different order'; # c -> 从上次匹配结束的位置匹配继续匹配
    is ~$/, 'aa',   'Longest alternative wins 2, different order';

    ok $str ~~ m:c/aa|a|aaaa/, 'Third match still works, different order';
    is ~$/, 'a',    'Only one alternative left, different order';

    ok $str !~~ m:c/aa|a|aaaa/, 'No fourth match, different order';
}

{
    my @list = <a aa aaaa>;
    ok $str ~~ m/ @list /, 'basic sanity with interpolated arrays';
    is ~$/, 'aaaa', 'Longest alternative wins 1';

    ok $str ~~ m:c(4)/ @list /, 'Second match still works';
    is ~$/, 'aa',   'Longest alternative wins 2';

    ok $str ~~ m:c(6)/ @list /, 'Third match still works';
    is ~$/, 'a',    'Only one alternative left';

    ok $str !~~ m:c(7)/ @list /, 'No fourth match';
}
```

# 简化的模式词法解析

不像传统的正则表达式那样, Perl 6 不要求你记住数量众多的元字符.  相反, Perl 6 通过一个简单的 `rule` 将字符进行了分类. 在正则表达式中, 所有根字符为下划线(`_`)或拥有一个以 `L`(例如, 字母) 或 `N`(例如, 数字)开头的 Unicode 类别的字形(字素) 总是字面的.(例如, 自己和自己匹配的).  它们必须使用 `\` 转义以使它们变成元语法的(这时单个字母数字字符本身是元语法的, 但是任何在字母数字后面紧紧跟随的字符不是).

所有其它的字形 — 包括空白符 — 正好与此相反:它们总是被认为是元语法的.(例如, 自身和自身不匹配),  必须被转义或引用以使它们变为字面值. 按照传统, 它们可以使用 `\` 单独转义, 但是在 Perl 6中,  它们也能像下面这样被括起来.

把一个或多个任意类型的字形序列放在单引号中能使它们变为字面值.(如果使用和当前语言相同的插值语义, 双引号也是允许的 )  引号创建了一个能量化的原子,  所以,

[`S05-metasyntax/single-quotes.t lines 16–27`](https://github.com/perl6/roast/blob/master/S05-metasyntax/single-quotes.t#L16-L27)

```perl6
moose*
```

量词只作用在字母 `e` 上, 并匹配 `mooseee`, 而

```perl6
'moose'*
```

量词作用在整个字符串上并匹配  `moosemoose`.

下面有个表格总结了这些区别:

```
                 字母数字的           非字母数字的                 混合的

 Literal glyphs   a    1    _        \*  \$  \.   \\   \'       K\-9\!
 Metasyntax      \a   \1   \_         *   $   .    \    '      \K-\9!
 Quoted glyphs   'a'  '1'  '_'       '*' '$' '.' '\\' '\''     'K-9!'
```

换句话说, 标识符字形是字面值的(或者在被转义时是元语法的), 非标识符字形是元语法的(或者在被转义时是字面值的), 而且单引号包围起来的东西是字面值的。

注意, 目前在 Perl 6 中不是所有的非标识符字形作为元语法都是有意义的(例如 `\1`、`\_`、`-`、`!`)。更准确地说, 所有非转义的非标识符字形都有可能是元语法的, 而且留作将来之用。如果你使用了这样一个序列, 那么编译器会抛出一个有用的编译时错误以标示你要么需要引起这个序列, 要么定义一个新的操作符以识别它。

[`S05-metasyntax/unknown.t lines 6–48`](https://github.com/perl6/roast/blob/master/S05-metasyntax/unknown.t#L6-L48)

分号字符被特别地保留作非意义(non-meaningful)元字符;  如果看到了未被引起的分号, 那么编译器就会抱怨那个正则表达式没有终止符。

[`S05-metasyntax/regex.t lines 73–129`](https://github.com/perl6/roast/blob/master/S05-metasyntax/regex.t#L73-L129)

# 修饰符

- `/x` 语法扩展不在需要了.. 在 Perl 6 中 `/x` 是默认的了.(事实上, 这是强制的 -- 唯一能用回旧语法的方式是使用 `:Perl5/:P5` 修饰符)

[`S05-modifier/perl5_4.t lines 7–119`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_4.t#L7-L119)
[`S05-modifier/perl5_2.t lines 7–119`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_2.t#L7-L119)
[`S05-modifier/perl5_9.t lines 7–112`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_9.t#L7-L112)
[`S05-modifier/perl5_1.t lines 7–119`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_1.t#L7-L119)
[`S05-modifier/perl5_8.t lines 7–128`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_8.t#L7-L128)
[`S05-modifier/perl5_5.t lines 7–126`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_5.t#L7-L126)
[`S05-modifier/perl5_7.t lines 7–119`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_7.t#L7-L119)
[`S05-modifier/perl5_6.t lines 7–130`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_6.t#L7-L130)
[`S05-modifier/perl5_3.t lines 7–119`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_3.t#L7-L119)
[`S05-modifier/perl5_0.t lines 9–113`](https://github.com/perl6/roast/blob/master/S05-modifier/perl5_0.t#L9-L113)

- 没有 `/s` 或 `/m` 修饰符了(变成了元字符来替代它们-看下面.)
- 没有 `/e`求值修饰符用于替换了, 相反, 使用:

```perl6
s/pattern/{ doit() }/
```

或:

```perl6
s[pattern] = doit()
```

代替 `/ee` 的是:

```perl6
s/pattern/{ EVAL doit() }/
```

或:

```perl6
s[pattern] = doit().EVAL
```

- 修饰符现在作为副词放在匹配或替换的开头:

```perl6
m:g:i/\s* (\w*) \s* ,?/;
```

每个修饰符必须以自己的冒号开始. 必须使用空格把模式分隔符和最后一个修饰符隔开, 如果它会被看作为前面那个修饰符的参数.(只有当下一个字符是左圆括号时才为真).

- 单字符修饰符也有更长的版本:

```
:i        :ignorecase   忽略大小写
:m        :ignoremark   忽略记号
:g        :global       全局
:r        :ratchet      回溯
```

- `:i` (或 `:ignorecase`) 修饰符在词法作用域但不是它的动态作用域中忽略大小写. 即, *subrules* 总是使用它们自己的大小写设置. 大小写转换的次数取决于当前上下文. 在字节和代码点模式中, 要求级别为1 的大小写转换. 在字形模式下, 需要级别为 2 的大小写转换.

[`S05-modifier/ignorecase.t lines 24–87`](https://github.com/perl6/roast/blob/master/S05-modifier/ignorecase.t#L24-L87)

- `:ii` (或 `:samecase`) 变体可能被用在替换上以把被替换掉的字符串的大小写模式修改成和匹配字符串的大小写模式一样。它暗示着该修饰符和上面的 `:i` 修饰符的语义拥有相同的模式。所以把 `:i` 和 `:ii` 放在一块没有必要。

[`S05-modifier/ii.t lines 8–30`](https://github.com/perl6/roast/blob/master/S05-modifier/ii.t#L8-L30)

- 如果模式匹配的时候没有使用 `:sigspace` 修饰符, 那么 case info is carried across on a character by character basis. 如果右侧字符串比左侧字符串长, 那么最后一个字符的大小写被重复。 如果可能的话, 不管结果字母是不在单词的开头, 都会执行单词首字母大写; 如果没有可用的单词首字母大写字符, 就会使用对应的大写字符。(该策略可以在本地作用域中通过依赖于语言的 Unicode 声明来根据特定语言的正交规则替换 titlecase。)不携带大小写信息的字符保持它们对应的替换字符串不变。

- 如果模式匹配使用了 `:sigspace` 修饰符, 那么then a slightly smarter algorithm is used which attempts to determine if there is a uniform capitalization policy over each matched word, and applies the same policy to each replacement word. If there doesn't seem to be a uniform policy on the left, the policy for each word is carried over word by word, with the last pattern word replicated if necessary. If a word does not appear to have a recognizable policy, the replacement word is translated character for character as in the non-sigspace case. Recognized policies include:

[`S05-modifier/ii.t lines 31–75`](https://github.com/perl6/roast/blob/master/S05-modifier/ii.t#L31-L75)

```perl6
lc()
uc()
tc()
tclc()
tcuc()
```

任何情况下，只有模式匹配的正式匹配到的字符串部分才被计数，所以任何形式的向前匹配或语境匹配没有包含在分析之内。

- `:m` (或 `:ignoremark`) 修饰符的作用域非常像 `:ignorecase`, 除了它忽略记号(重音和诸如此类) 而非大小写之外。它等价于接收每个字素(in both target and pattern), 把两者都转换为 NFD (最大限度地分解)并比较那两个根字符(Unicode 无记号字符)而忽略任何末尾记号字符。记号字符只在判断断言真假的时候忽略记号字符; 实际匹配的的文本包含了所有的忽略字符, 包含任何跟在最后的根字符后面的东西。
[S05-modifier/ignoremark.t lines 14–39](https://github.com/perl6/roast/blob/master/S05-modifier/ignoremark.t#L14-L39)

[S05-modifier/ignorecase-and-ignoremark.t lines 15–35](https://github.com/perl6/roast/blob/master/S05-modifier/ignorecase-and-ignoremark.t#L15-L35)

- `:mm` (或 `:samemark`) 变体能用于替换以把替换字符串更改为和匹配字符串相同的记号/重音。这暗示着它和上面的 `:m` 的模式语义相同, 所以, 把 `:m` 和 `:mm` 放在一块是没有必要的。
[S05-modifier/samemark.t lines 9–39](https://github.com/perl6/roast/blob/master/S05-modifier/samemark.t#L9-L39)

- 通过字根对字符执行记号信息。如果右侧字符串比左侧字符串长, 那么剩余的字符在被替换时不使用任何修改。 (注意 *NFD*/*NFC* 的区别通常不太重要, 因为 Perl 把它封装在了字素模式中。) 在 `:sigspace` 修饰符下, 会逐字应用之前的规则。

- `:c` (或 `:continue`) 修饰符让 `pattern` 从字符串中指定的位置继续扫描 (默认为 `$/ ?? $/.to !! 0`):
[`S05-modifier/continue.t lines 6–50`](https://github.com/perl6/roast/blob/master/S05-modifier/continue.t#L6-L50)

```perl6
m:c($p)/ pattern /     # 从位置 $p 开始扫描, $p 是字符串中的位置,而非模式中的位置
```

```perl6
  use v6;


  #L<S05/Modifiers/"The :c">

  my regex simple { . a };
  my $string = "1a2a3a";

  {
      $string ~~ m:c/<&simple>/;               # 1a
      $string ~~ m:c/<&simple>/;               # 2a
      $string ~~ m:c/<&simple>/;               # 3a
      $string ~~ m:c/<&simple>/;               # no more 'a's to match

  }

  {
      my $m = $string.match(/.a/);             # 1a
      $m = $string.match(/.a/, :c(2));         # 2a
      $m = $string.match(/.a/, :c(4));         # 3a

  }

  # this batch not starting on the exact point, and out of order
  {
      my $m = $string.match(/.a/, :c(0));      # 1a
      $m = $string.match(/.a/, :c(3));         # 3a
      $m = $string.match(/.a/, :c(1));         # 2a

  }

  {
      my $m = $string.match(/.a/);             # 1a
      $m = $string.match(/.a/, :continue(2));  # 2a
      $m = $string.match(/.a/, :continue(4));  # 3a

  }
```

注意, 这不会自动把 `pattern` 锚定到开始位置. (使用 *:p* 可以). 提供给 `split` 的 *pattern* 默认有一个隐式的 `:c` 修饰符.

- `:p` (或 `:pos`) 修饰符让 `pattern` 只在字符串中指定的位置尝试匹配:
  [`S05-modifier/pos.t lines 13–111`](https://github.com/perl6/roast/blob/master/S05-modifier/pos.t#L13-L111)

```perl6
m:pos($p)/ pattern /  # match at position $p
```

如果参数被省略，它就默认为 `($/ ?? $/.to !! 0)`. (Unlike in Perl 5, the string itself has no clue where its last match ended.) All subrule matches are implicitly passed their starting position. Likewise, the pattern you supply to a Perl macro's `is parsed` trait has an implicit `:p` modifier.

注意

```perl6
m:c($p)/pattern/
```

正等价于

```perl6
m:p($p)/.*? <( pattern )> /
```

所有的 `:g`, `:ov`, `:nth`, `:x` 跟 `:p` 都是不兼容的, 并且会失败， 推荐使用 `:c` 代替. 允许使用 `:ex` 修饰符， 但是只会在那个位置产生匹配.

- 新的 `:s` (`:sigspace`) 修饰符让空白序列起作用，这些空白被匹配规则 `<.ws>` 代替. 只有紧跟在匹配结构(原子, 量词化原子)之后的空白序列是合适的。因此, 出现在任何 regex 开头的空白都会被忽略, 为了让能够参与最长 token 匹配的备选分支更容易书写。即： 

[`S05-grammar/ws.t lines 5–34`](https://github.com/perl6/roast/blob/master/S05-grammar/ws.t#L5-L34)

```perl6
m:s/ next cmd '='   <condition>/
```

等价于:

```perl6
m/ next <.ws> cmd <.ws> '=' <.ws> <condition>/
```

同样等价于:

```perl6
m/ next \s+ cmd \s* '=' \s* <condition>/
```

但是在这种情况:

```perl6
m:s{(a|\*) (b|\+)}
```

或等价的:

```perl6
m { (a|\*) <.ws> (b|\+) }
```

`<.ws>` 直到看见数据才能决定要做什么. 它仍旧能做对事情. 不过不能, 定义你自己的 `ws`, 而 `:sigspace` 会使用它.`
仅仅当 rule 可能参与最长 token 匹配时,  rule 前面的空白才会被忽略, 但是在任何显式备选分支的前面这同样适用, 因为同样的原因。如果你想在一组备选分支前面匹配有意义的空格(sigspace), 把你的空白格放在含有备选分支的括号外面。
  ​  ​
当你这样写:

```perl6
rule TOP { ^ <stuff> $ }
```

这等价于

```perl6
token TOP { ^ <.ws> <stuff> <.ws> $ <.ws> }
```

但是注意最后一个 `<.ws>`  总是匹配空字符串，因为  `$` 锚定字符串的末尾. 如果你的 `TOP` rule 没有使用 `^` 锚定, 它不会匹配开头的空白. 

特别地， 下面的构造会把后面跟着的空白转换为 `sigspace`:

```
任何原子或量词化的原子
$foo @bar
'a' "$b"
^ $ ^^ $$
(...) [...] <...> 作为整个原子
(...)* [...]* <...>* 作为量词化的原子
<( 和 )>
« 和 » (但是不要那样使用 « )
```

然而这些并不会:

```
开口的 ( 或 [
| 或 ||
& 或 &&
** % 或 %%
:foo 声明, 包括 :my 和 :sigspace 自身
{...}
```

当我们说 *sigspace* 能跟在原子或量词化的原子后面时, 我们是说 *sigspace* 能出现在原子和它的量词之间:
  ​
```perl6
ms/ <atom> * /      # 意味着 / [<atom><.ws>]* /
```

(如果每个原子匹配空白, 那么就没有必要匹配后面的量词了)。If each atom matches whitespace, then it doesn't need to match after the quantifier.)

一般地, 你不需要在 grammars 中使用 `:sigspace`, 因为解析规则会为你自动处理空白策略。在该上下文中, 空白常会包含注释, 根据 grammar 如何去定义它的空白规则。尽管默认的 `<.ws>` subrule 识别不了注释结构, 任何 grammar 能随意重写 `<.ws>` 规则。`<.ws>` *rule* 并不意味着在哪儿都是一样。

[`S05-grammar/ws.t lines 6–34`](https://github.com/perl6/roast/blob/master/S05-grammar/ws.t#L6-L34)

给 `:sigspace` 传递一个参数也是可能的, 指定一个完全不同的 *subrule* 以应用。 这可以是任何 *rule*, 它不需要必须匹配空白。当谈论这个修饰符的时候, 把模式中的有意义的空白和正被匹配的"空白"区分开很重要, 所以我们会把模式中的空白叫做 *sigspace*, 而通常保留  *whitespace* 来标示当前 *grammar* 中 `<.ws>` 所匹配到的任何东西。  *sigspace* 和 *whitespace* 之间的一致性是隐喻性的, 这就是为什么一致性既有用又令人迷惑。

除了智能空白匹配, *:ss* (或 *:samespace*) 变体还可以用在替换上做智能空白映射。(即, *:ss* 暗示了 *:s* )。 对于左侧每个包含 *sigspace* 的 `<ws>` 调用,  所匹配到的空白被复制到右侧对应的坑(slot)里, 在想要空白替换的替换字符中由单个空白符表示。如果右侧比左侧的空白坑多, 则那些右手边的字符保持自身不变。如果右侧没有足够的空白坑映射所有从匹配中得到的空白坑, 那么算法会尝试通过从空白列表中随机地拼接"普通"空白符来使信息丢失最小化。 从最不贵重(valuable)到最贵重, 排列顺序为:
  ​
```
spaces
tabs
所有其他水平空白, 包括 Unicode
换行符 (包括作为一个单元的 crlf)
所有其他垂直空白, 包括 Unicode
```

这些规则的主要意图是发生在行边界这样的替换时最小化格式分裂。也就是说, 当然, 不保证结果正是人们想要的。
  ​
`:s` 修饰符很重要, 以至于为它们定义了匹配变体:
[`S05-modifier/sigspace.t lines 27–48`](https://github.com/perl6/roast/blob/master/S05-modifier/sigspace.t#L27-L48)
[`S05-substitution/subst.t lines 212–222`](https://github.com/perl6/roast/blob/master/S05-substitution/subst.t#L212-L222)
  ​
```perl6
ms/match some words/                        # 和 m:sigspace  相同
ss/match some words/replace those words/    # 和 s:samespace 相同
```

注意 `ss///` 是就 `:ss` 而言所定义的, 所以:

```perl6
$_ = "a b\nc\td";
ss/b c d/x y z/;
```

结果就是  `a x\ny\tz`.

- 新修饰符可指定 Unicode 级别:

```perl6
m:bytes  / .**2 /       # 匹配两个字节
m:codes  / .**2 /       # match two codepoints
m:graphs / .**2 /       # match two language-independent graphemes
m:chars  / .**2 /       # match two characters at current max level
```

- 新的 `:Perl5`/`:P5` 修饰符允许使用 Perl 5 正则语法 (现在不允许你把修饰符放在后面). 例如,

```perl6
m:P5/(?mi)^(?:[a-z]|\d){1,2}(?=\s)/
```

等价于 Perl 6 语法:

```perl6
m/ :i ^^ [ <[a..z]> || \d ] ** 1..2 <?before \s> /
```

- 任何一个整数修饰符指定一个计数. 数字后面的字符决定了计数的种类:
- 如果后面跟着一个 `x`, 则意味着重复. 一般使用 `:x(4)` 这种形式:

[`S05-modifier/repetition-exhaustive.t lines 16–30`](https://github.com/perl6/roast/blob/master/S05-modifier/repetition-exhaustive.t#L16-L30)
[`S05-modifier/repetition.t lines 6–29`](https://github.com/perl6/roast/blob/master/S05-modifier/repetition.t#L6-L29)

```perl6
s:4x [ (<.ident>) '=' (\N+) $$] = "$0 => $1";
```

等同于:

```perl6
s:x(4) [ (<.ident>) '=' (\N+) $$] = "$0 => $1";
```

这几乎等同于:

```perl6
s:c[ (<.ident>) '=' (\N+) $$] = "$0 => $1" for 1..4;
```

except that the string is unchanged unless all four matches are found. However, ranges are allowed, so you can say `:x(1..4)` to change anywhere from one to four matches.

- 如果数字后面跟着 `st`, `nd`, `rd`, 或 `th`, 它意味着查找第`*N*th` 次出现.  一般使用 `:nth(3)` 这种形式, 所以
[`S05-modifier/counted.t lines 13–252`](https://github.com/perl6/roast/blob/master/S05-modifier/counted.t#L13-L252)

```perl6
s:3rd/(\d+)/@data[$0]/;
```

等同于

```perl6
s:nth(3)/(\d+)/@data[$0]/;
```

它等价于

```perl6
m/(\d+)/ && m:c/(\d+)/ && s:c/(\d+)/@data[$0]/;
```

​例如:

```perl6
$_ =  "123abc456def789hij"
s:3rd/(\d+)/"PERL"/;  # $_ => 123abc456def"PERL"hij
s:2rd/(\d+)/PHP/;     # $_ => 123abcPHPdef"PERL"hij
```

```perl6
$_ = "0abc1DEF2cfg3OOP";
my @data =('-', '_', ':');
s:3rd/(\d+)/@data[$0]/;    # $_ => 0abc1DEF:cfg3OOP
```

`:nth` 的参数允许是一个整数列表, 但是这样的列表应该是单调递增的.(任何小于等于前一个值的值都会被忽略.) 所以:
  ​

```perl6
:nth(2,4,6...*)    # return only even matches
:nth(1,1,*+*...*)  # match only at 1,2,3,5,8,13...
```

例子:

```perl6
$_ = "1 2 3 4 5 6 7 8 9";
my @a = m:nth(2,4,6,8)/(\d+)/; # 返回 2 4 6 8
```

该选项不再要求支持智能匹配。你可以 grep 一个整数列表如果你真的需要那个能力:

```perl6
:nth(grep *.oracle, 1..*)
```
  
如果 `:nth` 和 `:x` 都出现了, 那么相匹配的子例程查找使用 `:nth` 匹配的子匹配。如果第后 nth 个匹配的数量与 `:x` 中的约束相容, 那么整个匹配成功, 并且子匹配的可能的数量最高。如果 `:nth` 不是单个标量, 则 `:nth` 和 `:x` 通常才有意义。

- 使用新的 `:ov` (`:overlap`) 修饰符, 当前的 regex 会在所有可能的字符位置上(包括重叠)匹配, 并在列表上下文中返回所有的匹配, 或在 item 上下文中返回间断的匹配。任一位子的第一个匹配会被返回。匹配保证了相对于开始位置, 会以从左到右的顺序返回。

[`S05-modifier/overlapping.t lines 16–67`](https://github.com/perl6/roast/blob/master/S05-modifier/overlapping.t#L16-L67)

```perl6
$str = "abracadabra";
if $str ~~ m:overlap/ a (.*) a / {
     @substrings = slice @();    # bracadabr cadabr dabr br
 }
```

- 使用新的 `:ex` (`:exhaustive`) 修饰符, 当前正则会匹配所有可能的路径(包括重叠)并返回所有 `matches`的一个列表.
[`S05-modifier/exhaustive.t lines 10–147`](https://github.com/perl6/roast/blob/master/S05-modifier/exhaustive.t#L10-L147)
[`S05-modifier/repetition-exhaustive.t lines 17–30`](https://github.com/perl6/roast/blob/master/S05-modifier/repetition-exhaustive.t#L17-L30)


匹配确保会相对于起始位置以从左到右的顺序返回。每个起始位置之内的顺序不能收到保证, 并且可能依赖于模式和匹配引擎两者。(推测: 或者我们能强制回溯引擎语义。或者我们可以保证一点也没有顺序, 除非模式以 `::` 开始或者某种这样的东西抑制了 DFAish 方法。)
  ​
```perl6
$str = "abracadabra";
if $str ~~ m:exhaustive/ a (.*?) a / {
   say "@()";    # br brac bracad bracadabr c cad cadabr d dabr br
}
```

注意一旦发现第一个匹配, 上面的 `~~` 就返回, 而匹配的剩余部分可能通过 `@()` 来懒惰地执行。
  ​
- 新的 `:rw` 修饰符让该 regex 要求当前的字符串可以修改而非假设写时复制语义。 `$/` 中所有的捕获变成了左值字符串, 这样如果你修改, 例如, `$1`, 那么原来的字符串在那个位置被修改, 并且所有其它字段的的位置也相应地修改(不管它是什么意思)。如果没有该修饰符, (特别是如果它尚未实现或永远不会被实现), 那么 `$/` 的所有片段都被当作是写时复制的, 如果不是只读的话。

[猜测: 这应该真正把模式和字符串变量联合在一起, 而不是一个字符串值(可能是不可变的)]

- 新的 `:r` 或 `:ratchet` 修饰符让这个 regex 默认不回溯。 (通常你不直接使用这个修饰符, 因为在 `token` 和 `rule` 声明符中已经隐式地包含了这个修饰符, 也就是说 **token** 和 **rule** 默认也是不回溯的。) 这个修饰符的作用是在每个原子(atom)后面暗指一个 `:`, 包括但不限于  `*`, `+`, 和 `?` 量词, 还有备选分支。量词化原子上的显式回溯修饰符, 例如 `**`, 会重写这个修饰符。 

[`S05-modifier/ratchet.t lines 5–27`](https://github.com/perl6/roast/blob/master/S05-modifier/ratchet.t#L5-L27)
[`S05-mass/rx.t lines 92–99`](https://github.com/perl6/roast/blob/master/S05-mass/rx.t#L92-L99)

- `:i`, `:m`, `:r`, `:s`, `:dba`, `:Perl5`, 和  Unicode 级别的修饰符能被放在 regex 里面  (并且是词法作用域的):
[`S05-modifier/ignorecase.t lines 18–23`](https://github.com/perl6/roast/blob/master/S05-modifier/ignorecase.t#L18-L23)

```perl6
m/:s alignment '=' [:i left|right|cent[er|re]] /
```

就像外面的修饰符一样,  只有圆括号能够作为副词的参数被识别为合法的括号. 特别地:

```perl6
m/:foo[xxx]/        Parses as :foo [xxx]
m/:foo{xxx}/        Parses as :foo {xxx}
m/:foo<xxx>/        Parses as :foo <xxx>
```

- 用户自定义修饰符成为可能:

```perl6
m:fuzzy/pattern/;
```

- 用户自定义修饰符也能接收参数, 但是只能在圆括号中:

```perl6
m:fuzzy('bare')/pattern/;
```

- 要使用括号作为你的分隔符你必须隔开:

```perl6
m:fuzzy (pattern);
```

或者把 pattern 放在最后:

```perl6
m:fuzzy(fuzzyargs); pattern ;
```

- 任何 grammar regex 实际上是一种`方法`, 并且你可以在这样一个子例程中使用一个冒号跟着任何作用域声明符来声明一个变量, 这些声明符包括 `my`, `our`, `state` 和 `constant` (作为类似的声明符, temp 和 let 也能被识别). 单个语句(直到结尾的分号或行末尾的闭括号为止) 被解析为普通的 Perl 6 代码:

[`S05-modifier/my.t lines 5–87`](https://github.com/perl6/roast/blob/master/S05-modifier/my.t#L5-L87)

```perl6
token prove-nondeterministic-parsing {
    :my $threshold = rand;
    'maybe' \s+ <it($threshold)>
}
```

这种作用域声明符不会终止最长 token 匹配, 所以无效的声明符可以作为作为一个挂钩来挂起副作用而不改变随后的模式匹配:

```perl6
rule breaker {
    :state $ = say "got here at least once";
    ...
}
```
