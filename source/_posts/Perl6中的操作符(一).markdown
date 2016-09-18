---
title: Perl 6 中的操作符(一)
date: 2015-08-31
tags:
  - 操作符
  - 优先级
categories: Perl 6
comments: true
---

# 操作符优先级

[S03-operators/arith.t lines 46–342](https://github.com/perl6/roast/blob/master/S03-operators/arith.t#L46-L342)

[S03-operators/precedence.t  lines 5–200](https://github.com/perl6/roast/blob/master/S03-operators/precedence.t#L5-L200)

Perl 6 拥有和 Perl 5 同等数量的优先级级别，但是它们散布在不同的地方。这儿，我们列出了从最紧凑到最松散的级别，每一级别还有几个例子：

最高优先级到最低优先级：

```perl6

A  Level             Examples
=  =====             ========
N  Terms             42 3.14 "eek" qq["foo"] $x :!verbose @$array
L  Method postfix    .meth .+ .? .* .() .[] .{} .<> .«» .:: .= .^ .:
N  Autoincrement     ++ --
R  Exponentiation    **
L  Symbolic unary    ! + - ~ ? | || +^ ~^ ?^ ^
L  Multiplicative    * / % %% +& +< +> ~& ~< ~> ?& div mod gcd lcm
L  Additive          + - +| +^ ~| ~^ ?| ?^
L  Replication       x xx
X  Concatenation     ~
X  Junctive and      & (&) ∩
X  Junctive or       | ^ (|) (^) ∪ (-)
L  Named unary       temp let
N  Structural infix  but does <=> leg cmp .. ..^ ^.. ^..^
C  Chaining infix    != == < <= > >= eq ne lt le gt ge ~~ === eqv !eqv (<) (elem)
X  Tight and         &&
X  Tight or          || ^^ // min max
R  Conditional       ?? !! ff fff
R  Item assignment   = => += -= **= xx= .=
L  Loose unary       so not
X  Comma operator    , :
X  List infix        Z minmax X X~ X* Xeqv ...
R  List prefix       print push say die map substr ... [+] [*] any Z=
X  Loose and         and andthen
X  Loose or          or xor orelse
X  Sequencer         <== ==> <<== ==>>
N  Terminator        ; {...} unless extra ) ] }

```

下面使用的两个 `!` 符号通常表示任意一对儿拥有相同优先级的操作符， 上表指定的二元操作符的结合性解释如下(其中 A 代表**结合性**， associativities )：

```perl6

    结合性     Meaning of $a ! $b ! $c
    =====     =========================
L   left      ($a ! $b) ! $c
R   right     $a ! ($b ! $c)
N   non       ILLEGAL
C   chain     ($a ! $b) and ($b ! $c)
X   list      infix:<!>($a; $b; $c)

```

对于一元操作符， 这解释为:

```perl6

    结合性     Meaning of !$a!
    =====     =========================
L   left      (!$a)!
R   right     !($a!)
N   non       ILLEGAL

```

(在标准 Perl 中没有能利用结合性的一元操作符，因为在每一优先级级别中， 标准操作符要么一贯地是前缀，要么是后缀。)

注意列表结合性（X）只在同一操作符之间有效。如果两个拥有不同列表结合性的操作符拥有相同的优先级，它们彼此间就会被认为是非结合性的，必须使用圆括号来消除歧义。

[S03-operators/precedence.t lines 211–245](https://github.com/perl6/roast/blob/master/S03-operators/precedence.t#L211-L245)

例如， `X` 交叉操作符和 `Z` **拉链操作符** 都有 "list infix" 优先级，但是：

```perl6
@a X @b Z @c
```

是非法的，它必须写成下面的任意一种：

```perl6
(@a X @b) Z @c
@a X (@b Z @c)
```

如果仅有的列表结合性操作符的实现是二进制的, 那么它会被当作是右结合性的。
标准的优先级层级尝试和它们的结合性相一致, 但是用户定义的操作符和优先级级别可以在同一优先级级别上混合右结合性和左结合性操作符。如果在同一个表达式中不小心使用了有冲突的操作符, 那么操作符彼此之间会被认为是非结合性的, 并且必须使用圆括号来消除歧义。

如果你没有在上面看见你喜欢的操作符, 下面的章节会包含所有按优先级排列的操作符。这儿描述了基本的操作符。

#### Term precedence

这实际上不真的是优先级, 但是它在这里是因为没有操作符的优先级比 term 高. 查看 S02 获取各种 terms 的更详尽的描述. 这里有一些例子:

- Int 字面量

```perl6
42
```

- Num 字面量

```perl6
3.14
```

- 不能插值的 Str 字面量

```perl6
'$100'
```

- 能插值的 Str 字面量

```perl6
"Answer = $answer\n"
```

- 通用的 Str 字面量

```perl6
q["$100"]
qq["$answer"]
```

- Heredoc

```perl6
qq:to/END/
    Dear $recipient:
    Thanks!
    Sincerely,
    $me
    END
```

- 数组构造器

```perl6
    [1,2,3]
```

 `[ ]` 里面提供了列表上下文. 技术上讲, 它实际上提供了一个  `semilist` 上下文, 即一系列分号分割的语句, 每条语句都在列表上下文中解释, 然后被连接成最终的列表.

- 散列构造器

```perl6

    { }
    { a => 42 }

```

`{ }` 里面要么是空的, 要么是以 pair 或 散列 开头的单个列表, 否则你必须使用 `hash( )` 或 `%( )` 代替.

- Closure

```perl6
{ ... }
```

如果出现在语句那儿, 会立即执行。 否则会延迟内部作用域的求值。

- 捕获构造器

```perl6
\(@a,$b,%c)
```

代表还不知道它的上下文的参数列表的抽取,


- 符号化变量

```perl6
$x
@y
%z
$^a
$?FILE
&func
&div:(Int, Int --> Int)
```

- 符号作为上下文化函数

```perl6
$()
@()
%()
&()
```

- quote-like 记号中的 Regexes

```perl6
/abc/
rx:i[abc]
s/foo/bar/
```

- 转换

```perl6
tr/a..z/A..Z/
```

注意范围使用 `..` 而非 `-`.

- 类型名

```perl6
Num
::Some::Package
```

- 由圆括号环绕的子表达式

```perl6
(1+2)
```

- 带括号的函数调用

```perl6
a(1)
```

一个项后面立即跟着一个圆括号化的表达式总是被当作函数调用，　即使那个标识符也含有前缀意义，　所以那种情况下你从来不用担心优先级。因此：

```perl6
not($x) + 1         # means (not $x) + 1
```

- Pair 构造器

```perl6
:limit(5)
:!verbose
```

- 签名字面量

```perl6
:(Dog $self:)
```

- 使用隐式调用者的方法调用

```perl6
.meth       # call on $_
.=meth      # modify $_
```

注意这只能出现在需要项(term)的地方。需要后缀的地方它就是后缀。如果需要中缀操作符（即, 在项后面, 之间是空格）, .meth 就是语法错误。(.meth 形式在那儿是被允许的因为有一个和方法调用形式在语义上等价但是允许在 = 号和方法名之间于空格存在的特殊 .= 中缀赋值操作符)。

- Listop (leftward)

```perl6
4,3, sort 2,1       # 4,3,1,2
```

就像 Perl 5 中一样, 列表操作符对于它左侧的表达式看起来像一个项(term), 所以它比左侧的逗号绑定的紧凑点, 比右侧的逗号绑定的松散点。-- 查看下面的列表前缀优先级。

#### 方法后缀优先级

所有的方法后缀都以一个点开头, 尽管对于下标来说, 点号是可选的. 因为这些是最紧密的操作符,  你可以看到一系列方法调用作为单独的项, 这个项仅仅要表达一个复杂的名字.

- 标准的单个分发方法调用

```perl6
$obj.meth
```

- 标准单个分发方法调用的变体

```perl6
$obj.+meth
$obj.?meth
$obj.*meth
```

除了普通的 `.` 方法调用之外, 还有 `.*`, `.?`, 和 `.+` 变体来控制如何处理多个同名的相关方法.

- 类限定的方法调用

```perl6
$obj.::Class::meth
$obj.Class::meth    # same thing, 假设预先定义了 Class
```

就跟 Perl 5 一样, 告诉分发器(dispatcher)从哪个类开始搜索, 而不正好是那个被调用的方法。

- 可变方法调用

```perl6
$obj.=meth
```
.= 操作符执行了对左侧对象的就地修改。

- 元方法调用

```perl6
$obj.^meth
```

`.^` 操作符调用了类的元方法(class metamethod); **foo.^bar** 是 `foo.HOW.bar` 的简写。

- 像方法一样的后环缀

```perl6
$routine.()
$array.[]
$hash.{}
$hash.<>
$hash.«»
```

不带点的这些形式有同样的优先级.

- 带点形式的其它后缀操作符

```perl6
$x.++         # postfix:<++>($x)
```

- 带点形式的其它前缀操作符

```perl6
$x.:<++>       # prefix:<++>($x)
```

- 有一个特殊的非中缀操作符 infix:<.> 所以

```perl6
$foo . $bar
```

总是会返回编译时错误来标示用户应该使用中缀操作符 infix<~> 代替。这用于捕获正在学习 Perl 6 的 Perl 5 程序员可能会犯的错误。

#### 自增优先级


就像在 C 中一样，这些操作符增加或减少正在谈论的那个对象的值，根据操作符在前面还是在后面。还是像 C 中一样，在同一个表达式中多个对单个可变对象的引用可能导致未定义行为除非显式地插入了某些序列操作符。请查看 "Sequence points"。

至于 Perl 6 中得所有后缀操作符，项(term) 和它的后缀之间不允许有空格。请查看 S02 来了解为什么，还有怎么使用 "unspace" 来应急这个约束。

和可变方法一样，所有这些操作符被分派为操作符数的类型并返回一个同类型的结果，但是只有（不可变的）值存储在可变容器中时，它们在值类型上才是合法的。然而，为了支持通用的习语，一个裸的未定义值（在一个合适的标量容器中）是被允许把自身修改成 Int 类型的：


```perl6
say $x unless %seen{$x}++;
```

Str (在一个合适的容器中)的增加和 Perl 5 类似，但是被稍微推广了一点。Perl 6 会扫描前面不是 '.' 字符的字符串中得最后的字母数字序列。不像 Perl 5 那样，这个字母数字序列不需要锚定到字符串的开头，也不需要以字母数字符开头；字符串中匹配 `<!after '.'> <rangechar>+` 字母数字的最后的序列被增加而不管它前面是什么。


`<rangechar>` 字符类被定义为那种字符的子集(subset)，Perl 知道怎么在范围(range)中增加它，就像下面定义的那样：
额外的匹配增加了两个好处：对于典型的增长文件名的用法，你不必担心路径名或扩展名：

```perl6
$file = "/tmp/pix000.jpg";
$file++;                 # /tmp/pix001.jpg, 不是 /tmp/pix000.jph
```

也许更重要的是, 如果你恰好增长了一个以小数结尾的字符串，Perl 6 也能应对自如：

```perl6
$num = "123.456";
$num++;             # 124.456, not 123.457
```

字符位置增加自然范围内任何Unicode范围被认为代表了数字0 . .9或被认为是一个完整的周期性字母的(一例)(Unicode)脚本。只在codepoints脚本,代表他们的字母表,形成一个周期独立于其他字母可能使用。(此规范延缓这种脚本的用户,以确定适当的周期的信件)。我们任意定义ASCII字母不相交与其他脚本,使用范围的字符,但是字母点缀ASCII字母是不允许的。

如果在这样一个范围中当前字符是字符串位置中最后的字符,它包装的第一个字符范围和发送一个“携带”的位置了,然后那个位置是增加的范围。当且仅当最左边的位置是筋疲力尽的范围,一个额外的字符相同的范围是插入到持有套利以相同的方式作为Perl 5,所以递增(zz99)“变成”(aaa00)zz和递增(99)“变成”(100 aa)”。

``` perl6
> my $a = "99zz"
> $a++           # 99zz
> $a++           # 100aa

> my $b = 'zz99'
> $b++           # zz99
> $b++           # aaa00
```

下面的 Unicode  范围是某些可能的 rangechar 范围。对于字母我们有这样的范围：

```perl6
A..Z        # ASCII uc
a..z        # ASCII lc
'Α'..'Ω'    # Greek uc
α..ω        # Greek lc (presumably skipping C<U+03C2>, final sigma)
א..ת        # 希伯来
  etc.      # (XXX out of my depth here)
```

```perl6
> my @a =  'Α'..'Ω'  # Α Β Γ Δ Ε Ζ Η Θ Ι Κ Λ Μ Ν Ξ Ο Π Ρ ΢ Σ Τ Υ Φ Χ Ψ Ω
```

对于数字我们有这样的范围：

```perl6
    0..9         # ASCII
    ٠..٩         # 阿拉伯语-印度语
    ०..९         # 天城文
    ০..৯         # 孟加拉语
    '੦'..'੯'    # 古木基文
    ૦..૯        # 古吉拉特文
    ୦..୯        # 奥里亚语
```

等等.

```perl6
> my @b =    '੦'..'੯'   #  ੦ ੧ ੨ ੩ ੪ ੫ ੬ ੭ ੮ ੯
```

某些其它非书写用的 0..9 范围也可以被增长，例如：

```perl6
    ⁰..⁹        # 上标 (note, cycle includes latin-1 chars)
    '₀'..'₉'    # 下标
    ０..９      # 全角数字
```

``` perl6
> my @f = '₀'..'₉' # ₀ ₁ ₂ ₃ ₄ ₅ ₆ ₇ ₈ ₉
```


```perl6
    Ⅰ..Ⅻ        # clock roman numerals uc
    ⅰ..ⅻ        # clock roman numerals lc
    ⓪..㊿       # circled digits/numbers 0..50
    ⒜..⒵        # parenthesized lc
    ⚀..⚅        # die faces 1..6
    '❶'..'❿'        # dingbat negative circled 1..10
```

etc.

注意: 对于 Perl 中实际的范围， 你需要把上面的字符用引号括起来：

```perl6
'⓪'..'㊿'   # circled digits/numbers 0..50
```

```perl6
my @d = '⓪'..'⓾'
```

```perl6
⓪ ⓫ ⓬ ⓭ ⓮ ⓯ ⓰ ⓱ ⓲ ⓳ ⓴ ⓵ ⓶ ⓷ ⓸ ⓹ ⓺ ⓻ ⓼ ⓽ ⓾ 
```

```perl6
my @e = '❶'..'❿' # ❶ ❷ ❸ ❹ ❺ ❻ ❼ ❽ ❾ ❿
```

If you want to future-proof the top end of your range against further Unicode additions, you may specify it as "whatever":

```
'⓪'..*      # circled digits/numbers up to current known Unicode max
```


prefix:<|>,  将对象展开为参数列表

```
| $capture
```

把Capture 值的内容(或 Capture-like)插值进当前的参数列表中, 就像它们被字面地指定那样。

prefix:<||>,  将对象展开为分号列表

```
|| $parcel
```

把 Parcel(或其它有顺序的值)的元素插值到当前参数列表中就像它们被字面地指定一样, 由分号分割, 即, 以多维级别。在列表中的列表上下文之外使用该操作符是错误的; 换句话说, 它必须被绑定到 `**`(slice)参数上而非吞噬参数`*`上。


infix:<div>, 整除

```
$numerator div $denominator
```

``` perl
> 3 div 2     # 1
> 13 div 2    # 6
```

而

``` perl
> 13 div 2.4
```

报错：

``` perl
Cannot call infix:<div>(Int, Rat); none of these signatures match:
    (Int:D \a, Int:D \b)
    (int $a, int $b --> int)
  in block <unit> at <unknown file>:1
```

infix:<%>, modulo

```
$x % $y
```

## 重复操作符

infix:<x>, 字符串/缓冲区 复制(或者重复/拷贝)

```perl6
$string x $count
```

在字符串上下文中计算左边的参数，重复结果字符串值由右侧参数指定的倍数次，然后不管上下文，把结果作为单个连接好的字符串返回。

如果重复次数小于 1，则返回空字符串。重复次数不可以是 `*`，因为 Perl 6 不支持无穷字符串。（至少，还没有...）然而注意，有一天无穷字符串可能使用 cat(`$string xx *`) 来模仿，在这种情况下，`$string x * `  可能是它的简写。

```perl6
'a' x *;              # WhateverCode.new()
my $a = 'a' x *;      # WhateverCode.new()
say $a;               # WhateverCode.new()
say $a(12);           # 可以传递参数！, 结果为 aaaaaaaaaaaa
```

infix:<xx>, 表达式重复操作符

```perl6
@list xx $count  # 如果 $count  是 * ，则返回一个无限列表 （懒惰的，因为列表默认是懒惰的 ）
```

```
rand xx *;                # infinite supply of random numbers
[ 0 xx $cols ] xx $rows   # distinct arrays, not the same row replicated
```

``` perl
> my @random= rand xx *;
> @random[0]                # 0.510689533871727
> @random[0]                # 0.510689533871727
> @random[1]                # 0.993102039714483
> @random[2]                # 0.177400471499773
> @random[12]
```

## 字符串连接

infix:<~>, 字符串 /缓冲 连接

```
$x ~ $y
```

## 范围对象创建

```
$min .. $max
$min ^.. $max
$min ..^ $max
$min ^..^ $max
```

## [逗号操作符优先级](https://desgin.perl6.org/S03.html#Comma_operator_precedence)

- infix:<,>  参数分隔符

```perl6
1, 2, 3, @many
```

不像 Perl5 ，逗号操作符从来不返回最后一个值（在标量上下文中它返回一个列表）

- infix:<:>, 调用者标记

``` perl
say $*OUT: "howdy, world"  # howdy, world
say($*OUT: "howdy, world") # howdy, world
push @array: 1,2,3
push(@array: 1,2,3)
\($object: 1,2,3, :foo, :!bar)
```

冒号操作符就像逗号那样解析，但是它把左边的参数标记为调用者，这会把函数调用转换为方法调用。它只能在参数列表或捕获的第一个参数身上使用，用在其它地方会解析失败。当用在捕获中时，尚不知道捕获会被绑定到哪个签名上；如果绑定到一个非方法的签名身上，调用者仅仅转换成第一个位置参数，就像冒号是一个逗号一样。

为了避免和其它的冒号形式混淆，冒号中缀操作符后面必须跟上空格或终止符。它前面可以有空格也可以没有空格。

注意：在下面把中缀操作符和冒号区别开:

```perl6
@array.push: 1,2,3
@array.push(1,2,3): 4,5,6
push(@array, 1,2,3): 4,5,6
```

这是把普通函数或方法转换为列表操作符的特殊形式。 这种特殊形式只在点语法形式的方法调用后被识别， 或者在方法或函数调用的右圆括号之后被识别。这种特殊形式不允许其间有空格，但是下一个参数的前面要有空格。如果可能的话在所有其它情况下，冒号会被解析为副词的开始，否则会被解析为调用者标记（上面描述的中缀操作符。）

这种特殊形式不允许介于中间的空格， 但是允许在下一个参数之前有空格。 在所有情况下， 冒号会被尽可能地解析为副词的开头，或者调用者标记者（上面描述的中缀）


冒号的另一种特殊方式是, 允许正好在参数列表的右侧圆括号之后为圆括号括住的参数列表添加 listop 参数，附带条件是你被允许把 `.foo(): 1,2,3` 缩短为 `.foo: 1,2,3`.(但是仅限于方法调用， 因为普通的函数不需要把处于第一个位置的冒号转换为 listop， 空格就够了。 如果你尝试使用冒号扩展函数名最好把它看作标签。)

```perl6
foo $obj.bar: 1,2,3     # special, means foo($obj.bar(1,2,3))
foo $obj.bar(): 1,2,3   # special, means foo($obj.bar(1,2,3))
foo $obj.bar(1): 2,3    # special, means foo($obj.bar(1,2,3))
foo $obj.bar(1,2): 3    # special, means foo($obj.bar(1,2,3))
foo($obj.bar): 1,2,3    # special, means foo($obj.bar, 1,2,3)
foo($obj.bar, 1): 2,3   # special, means foo($obj.bar, 1,2,3)
foo($obj.bar, 1,2): 3   # special, means foo($obj.bar, 1,2,3)
foo $obj.bar : 1,2,3    # infix:<:>, means $obj.bar.foo(1,2,3)
foo ($obj.bar): 1,2,3   # infix:<:>, means $obj.bar.foo(1,2,3)
foo $obj.bar:1,2,3      # 语法错误
foo $obj.bar :1,2,3     # 语法错误
foo $obj.bar :baz       # 副词, means foo($obj.bar(:baz))
foo ($obj.bar) :baz     # 副词, means foo($obj.bar, :baz)
foo $obj.bar:baz        # extended identifier, foo( $obj.'bar:baz' )
foo $obj.infix:<+>      # extended identifier, foo( $obj.'infix:<+>' )
foo: 1,2,3              # label at statement start, else infix
```

这个故事的寓意是：如果你不知道冒号是怎样结合的，就使用空格或圆括号让它清晰。

- List infix precedence 列表中缀优先级

列表中缀操作符都有列表结合性，这意味着，同一个中缀操作符是同步起作用的，而不是一个接着一个。不同的操作符被认为是非结合性的，为了明确，必须用括号括起来。

- infix:<Z>,  the zip operator

``` perl
    1,2 Z 3,4   # (1,3),(2,4)
```

``` perl
> 2,5,7 [Zmin] 3,4,5     # 两两比较, 2 4 5
> my @a=3,6,9            # 3 6 9
> my @b=4,5,10           # 4 5 10
> @a [Zmin] @b           # 3 5 9
> my @a = (1,2,9,3,5)    # 1 2 9 3 5
> my @b = (2,3,5,1,9)    # 2 3 5 1 9
> my @c = (2,3,4,5,1)    # 2 3 4 5 1
> @a [Zmin] @b [Zmin] @c # 1 2 4 1 1
> @a [Zmax] @b [Zmax] @c # 2 3 9 5 9
```

- infix:<minmax>,  minmax 操作符

```
@a minmax @b
```

返回@a和@b中最小值和最大值的一个范围。

``` perl
> my @a = 2,4,6,8;
> my @b = 1,3,5,7,9;
> @a minmax @b         # 1..9
```

- infix:<X>,  交叉操作符

S03-metaops/cross.t lines 6–19  

``` perl
1,2 X 3,4          # (1,3), (1,4), (2,3), (2,4)
```

和 zip 操作符相比， X 操作符返回元素交叉后的列表。例如，如果只有 2 个列表，第一个列表中取一个元素和第二个列表中取一个元素组成 pair 对儿，第二个元素变化的最迅速。

最右边的列表先遍历完。因此， 你写：

``` perl
<a b> X <1 2>
```

你会得到：

``` perl
('a', '1'), ('a', '2'), ('b', '1'), ('b', '2')
```

这在平的上下文会变成一个展平的列表，在 list of list 上下文中会变成列表中的列表

``` perl
say flat(<a b> X <1 2>).perl    #  ("a", "1", "a", "2", "b", "1", "b", "2").list
say lol(<a b> X <1 2>).perl     # (("a", "1"), ("a", "2"), ("b", "1"), ("b", "2"))
```

这个操作符是列表结合性的，所以：

``` perl
1,2 X 3,4 X 5,6
```

生成

``` perl
(1,3,5),(1,3,6),(1,4,5),(1,4,6),(2,3,5),(2,3,6),(2,4,5),(2,4,6)
```

另一方面，如果任一列表为空，你会得到一个空列表。

尽管X两边的列表可能是无限的，在操作符 X的右边使用无限列表可能会产生意想不到的结果，例如：

``` perl
<a b> X 0..*
```

会产生

``` perl
('a',0), ('a',1), ('a',2), ('a',3), ('a',4), ('a',5), ...
```

并且你绝对不会到达 'b'。如果你左侧的列表只包含单个元素，然而，这可能有用，尤其是如果 X 用作元操作符时。看下面。

``` perl
say lol(<a b> X <1 2>).perl    # ("a", "1", "a", "2", "b", "1", "b", "2")
```

Cross metaoperators 交叉操作符

``` perl
@files X~ '.' X~ @extensions
1..10 X* 1..10
@x Xeqv @y
```

等等

一个常见的用法是让一个列表只含有单个元素在操作符 X 的一边或另一边：

```Perl6
@vector X* 2;                 # 每个元素都乘以 2
$prefix X~ @infinitelist;     # 在无限列表的每个元素前面前置一个元素
```

``` perl
> my $prefix = ' - '
> my @a =<1 2 3 4 5>
> $prefix X~ @a       #  - 1  - 2  - 3  - 4  - 5
```

这时右边有一个无限列表是可以的。
