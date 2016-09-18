---
title:  The Awesome Errors of Perl 6
date: 2016-07-20
tags: Errors
categories: Perl 6
comments: true
---

## [The Awesome Errors of Perl 6](http://perl6.party/post/The-Awesome-Errors-of-Perl-6)

如果你一直在读技术相关的东西，你现在可能知道 Rust 里面的[令人惊喜的错误报告能力](https://internals.rust-lang.org/t/new-error-format/3438)。 既然 Perl 6 也因它的绝妙的错误处理而闻名, mst 查询了一些例子来炫耀 rust 的错误处理能力, 但是不幸的是我没有发现。

我尽力避免错误并且我很少完整地读完它。所以我会搜寻一些很酷的关于令人惊叹的错误方面的例子并写出来。虽然我能够用头撞击键盘并把输出粘贴出来, 但是那将会惨不忍读, 所以我会谈论一些对初学者来说没那么明显的棘手的错误, 还有怎样修复那些错误。

让我们开始用头部猛击吧!

## 基础

下面是一段有错误的代码;

```perl6
say "Hello world!;
say "Local time is {DateTime.now}";

# ===SORRY!=== Error while compiling /home/zoffix/test.p6
# Two terms in a row (runaway multi-line "" quote starting at line 1 maybe?)
# at /home/zoffix/test.p6:2
# ------> say "Local time is {DateTime.now}";
#     expecting any of:
#         infix
#         infix stopper
#         postfix
#         statement end
#         statement modifier
#         statement modifier loop
```

第一行丢失了字符串上的闭合引号, 所以直到第二行的开括号之间的所有东西都会被认为是字符串的一部分。一旦推测的闭合引号被找到, Perl 6 就看到单词 "Local", 这个单词被定义为一个项(item)。因为在 Perl 6 中一行中同时存在两个项(item)是不允许的, 所以编译器抛出了错误, 并对它所期望的提供了一些建议, 并且它探测到了我们正处在一个字符串中, 并且建议我们检测, 我们忘记在行 1 中闭合引号了。

===SORRY!=== 部分并不是意味着你运行的是加拿大版本的编译器, 而是意味着该错误是一个编译时错误(和运行时相比)。

## Nom-nom-nom-nom

下面有一个有趣的错误。我们有一个返回东西的子例程, 所以我们调用了它并使用了 for 循环来迭代值:

```perl6
sub things {1 ... ∞}

for things {
    say "Current stuff is $_";
}

# ===SORRY!===
# Function 'things' needs parens to avoid gobbling block
# at /home/zoffix/test.p6:5
# ------> }<EOL>
# Missing block (apparently claimed by 'things')
# at /home/zoffix/test.p6:5
# ------> }<EOL>
```

Perl 6 允许你在调用子例程的时候省略圆括号。上面的错误提到了全局块儿(globbing blocks)。实际发生的是我们希望传给 for 循环的块儿被作为参数传递给了子例程。输出中的第二个错误证实 for 循环丢失了它的块儿(并且给出了一个建议, 它被我们的 things 子例程接收了)。

第一个错误告诉我们怎样修复那个问题: Function 'things' needs parens, 所以我们的循环需要是:

```perl6
for things() {
    say "Current stuff is $_";
}
```

然而, 如果我们的子例程真的期望传递一个块儿, 那么圆括号就不是必须的。两个代码块肩并肩地在一块儿会导致  "two terms in a row" 错误, 所以 Perl 6 知道把第一个 block 传递给子例程并使用第二个 block 作为 for 循环的主体:

```perl6
sub things (&code) { code }

for things { 1 ... ∞ } {
    say "Current stuff is $_";
}
```

## Did You Mean Levestein?

下面有一个很酷的特性, 它不仅告诉你出错了, 还能指出你可能想要的:

```perl6
sub levenshtein {}
levestein;

# ===SORRY!=== Error while compiling /home/zoffix/test.p6
# Undeclared routine:
#     levestein used at line 2. Did you mean 'levenshtein'?
```

当 Perl 6 遇到它不认识的名字时它会为它知道的东西计算[Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance)以尝试提供一个有用的建议。在上面的距离中它遇到了一个它不知道的子例程调用。它注意到我们确实有一个相似的子例程, 所以它把它作为备选提供了出来。不要再盯着屏幕了, 尝试找到你在哪里敲击的键盘！

然而, 这个特性不可能在触发时面面俱到。假设我们把子例程的名字变为大写的 Levenshtein, 我们就不会得到那个建议, 因为对于以大写字母开头的东西, 编译器认为它看起来像一个类型名而非子例程名, 所以它检测这些东西来代替:

```perl6
class Levenshtein {}
Lvnshtein.new;

# ===SORRY!=== Error while compiling /home/zoffix/test.p6
# Undeclared name:
#    Lvnshtein used at line 2. Did you mean 'Levenshtein'?
```

## 一旦你成了 Seq, 你再也变不回来

我们假设你生成了一个短的斐波纳契数字序列。你打印了它然后你想再打印它一次, 但是这一次打印每个成员的平方。发生了什么?

```perl6
my $seq = (1, 1, * + * ... * > 100);
$seq.join(', ').say;
$seq.map({ $_² }).join(', ').say;

# 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144
# This Seq has already been iterated, and its values consumed
# (you might solve this by adding .cache on usages of the Seq, or
# by assigning the Seq into an array)
#   in block <unit> at test.p6 line 3
```

嗷, 运行时错误。我们从[序列操作符](https://docs.perl6.org/language/operators#index-entry-..._operators)得到的 Seq 类型不能保留东西。当你迭代序列的时候, 每次它给你一个值之后就丢弃这个值, 所以一旦你迭代完整个 Seq 序列, 就结束了。

上面的例子中, 我们尝试再次迭代那个序列, 所以 Rakudo 运行时奔溃并抱怨了, 因为它做不了。错误消息的确提供了两种可能的解决方案。我们要么使用 .cache 方法来获得一个我们将要迭代的 List:

```perl6
my $seq = (1, 1, * + * ... * > 100).cache;
$seq             .join(', ').say;
$seq.map({ $_² }).join(', ').say;

# 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144
# 1, 1, 4, 9, 25, 64, 169, 441, 1156, 3025, 7921, 20736
```

或者我们可以从一开始就使用数组 Array:

```perl6
my @seq = 1, 1, * + * … * > 100;
@seq             .join(', ').say;
@seq.map({ $_² }).join(', ').say;

# 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144
# 1, 1, 4, 9, 25, 64, 169, 441, 1156, 3025, 7921, 20736
```


并且即使我们把序列 Seq 存储进了 Array 中, 它不会被具体化直到真正被需要时:

```perl6
my @a = 1 … ∞;
say @a[^10];

# OUTPUT:
# (1 2 3 4 5 6 7 8 9 10)
```

## These Aren't The Attributes You're Looking For

假设你有一个类。在类里面, 你有一些私有属性并且你有一个使用属性的值作为它的一部分的正则匹配方法:

```perl6
class {
    has $!prefix = 'foo';
	method has-prefix ($text) {
	    so $text ~~ /^ $!prefix/;
	}
}.new.has-prefix('foobar').say;

# ===SORRY!=== Error while compiling /home/zoffix/test.p6
# Attribute $!prefix not available inside of a regex, since regexes are methods on Cursor.
# Consider storing the attribute in a lexical, and using that in the regex.
# at /home/zoffix/test.p6:4
# ------>         so $text ~~ /^ $!prefix⏏/;
#     expecting any of:
#         infix stopper
```

糟糕! 发生什么了?

就像编译器所指出的那样, Perl 6 实际上是由几种语言编织而成: Perl 6, Quote, 和 Regex 语言是这个编织物的一部分。这就是为什么像下面这样的东西就能起效:

```perl6
say "foo { "bar" ~ "meow" } ber ";

# OUTPUT:
# foo barmeow ber
```

尽管内插的代码块中使用了同样的引号", 但是没有发生冲突。然而, 同样的机制在正则表达式中有限制, 因为在正则表达式中, 所查询的属性属于 Cursor 对象, 它负责这个正则表达式。

为了避免这个错误, 就像错误信息暗示的那样, 仅仅使用一个临时的变量来存储 $!prefix 好了, 或者使用 given 块儿:

```perl6
class {
    has $!prefix = 'foo';
	method has-prefix ($text) {
	    given $!prefix { so $text ~~ /^ $_/ }
	}
}.new.has-prefix('foobar').say;
```

## De-Ranged

尝试过访问列表中超出范围的元素吗?

```perl6
my @a = <foo bar ber>;
say @a[*-42];

# Effective index out of range. Is: -39, should be in 0..Inf
#  in block <unit> at test.p6 line 2
```

在 Perl 6 中, 如果从列表末端索引一个条目, 要使用时髦的语法: `[*-42]`。它实际上是一个接收一个参数(它是列表中元素的个数)的闭包, 然后减去 42, 然后返回的值作为实际的索引。如果你特别无聊, 你可以使用 `@a[sub ($total) { $total - 42 }]` 代替。

在上面的错误中, 那个索引以 `3 - 42` 结束, 或者说是 `-39`, 这是我们在错误信息中看到的那个值。因为索引不能是负的, 所以我们收到了错误, 这也告诉我们索引必须从 0 到 正无穷大(任何超过列表所包含的索引会在被查询时返回 Any)。

## A Rose By Any Other Name, Would Code As Sweet

如果你是 Perl 6 的姐妹语言, Perl  5 的活跃使用者, 你可能会发现有时候你在 Perl 6 代码中写出 Perl 5 风格的代码:

```perl6
say "foo" . "bar";

# ===SORRY!=== Error while compiling /home/zoffix/test.p6
# Unsupported use of . to concatenate strings; in Perl 6 please use ~
# at /home/zoffix/test.p6:1
# ------> say "foo" .⏏ "bar";
```

在上面, 我们尝试使用 Perl 5 的字符串连接操作符来连接两个字符串。这个错误机制足够聪明地检测到这样的用法并推荐了正确的 `~` 操作符来代替。

这不是这种探测的唯一使用场景。有很多场景。这儿有另外一个例子, 用于探测 Perl 5 的钻石操作符的意外使用, 伴随着几个程序员可能想要的建议:

```perl6
while <> {}

# ===SORRY!=== Error while compiling /home/zoffix/test.p6
# Unsupported use of <>; in Perl 6 please use lines() to read input, ('') to
# represent a null string or () to represent an empty list
# at /home/zoffix/test.p6:1
# ------> while <⏏> {}
```

## Heredoc, Theredoc, Everywheredoc

为了抛出问题, 请先阅读底部的错误, 假装就是你自己写的程序代码:

```perl6
my $stuff = qq:to/END/;
Blah blah blah
END;

for ^10 {
    say 'things';
}

for ^20 {
    say 'moar things';
}

sub foo ($wtf) {
    say 'oh my!';
}

# ===SORRY!=== Error while compiling /home/zoffix/test.p6
# Variable '$wtf' is not declared
# at /home/zoffix/test.p6:13
# ------> sub foo (⏏$wtf) {
```

哈? 编译器哭着说有一个未声明的变量, 但是它指向的却是子例程中的签名。当然它不会被声明。

那些没有发现问题的人: 问题就出在 heredoc 中的闭合 END 后面的分号上。 heredoc 在闭合分隔符单独出现在一行的地方结束。在编译器看来, 我们还没有在 `END;` 这儿看到分隔符, 所以它继续解析就像它仍旧在解析 heredoc 一样。`qq` heredoc 能让你插入变量, 所以当解析器解析到签名中的 `$wtf` 变量时, 解析器并不知道它是一段实际代码中的签名还是某些随机的文本, 所以编译器哭着说变量未找到。

## Won't Someone Think of The Reader?

下面有一个极好的错误能阻止你写出恐怖的代码:

```perl6
my $a;
sub {
    $a.say;
	$^a.say;
}

# ===SORRY!=== Error while compiling /home/zoffix/test.p6
# $a has already been used as a non-placeholder in the surrounding sub block,
#   so you will confuse the reader if you suddenly declare $^a here
# at /home/zoffix/test.p6:4
# ------>         $^a.say;
```

这里有一点背景: 你可以在变量身上使用 `$^ twigil` 来创建一个隐式的签名。为了能在嵌套的块中使用这样的变量, 这个语法实际上创建了不带 twigil 的相同变量, 所以 `$^a` 和 `$a` 是同一个东西, 并且上面的子例程的签名是 `($a)`。

在我们的代码中, 我们还在外部作用域中有个 `$a` 并且推测我们首先打印出它, 在使用 `$^` twigil 在同样一个作用域中创建另外一个 `$a` 之前, 但是这个子例程包含了参数... 真绕脑! 为了避免这样, 就把你的变量重命名为某个不会冲突的东西好了。改成泰文怎么样?

```perl6
my $ความสงบ = 'peace';
sub {
    $ความสงบ.say;
    $^กับตัวแปรของคุณ.say;
}('to your variables');

# OUTPUT:
# peace
# to your variables
```

## Well, Colour Me Errpressed!

如果你的终端支持它, 那么编译器就会发出 ANSI 代码来给输出着点色:

```perl6
for ^5 {
    say meow";
}
```

![img](http://upload-images.jianshu.io/upload_images/326727-dc7e73c2e865eb03.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

那很好也很显眼夺目, 但是假设你想把从编译器中捕获的输出显示到任何地方, 你会原样地得到 ANSI 代码, 就像 `31m===[0mSORRY![31m===[0m`。

这很可怕, 但是幸运的是, 禁用颜色很简单: 仅仅把 `RAKUDO_ERROR_COLOR` 这个环境变量的值设置为 0 就好了：

![img](http://upload-images.jianshu.io/upload_images/326727-3581c5469cbca92e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你也可以在程序中设置它。你不得不足够早地设置它, 所以在任何地方把它放置在程序的开头并使用 [BEGIN phaser](https://docs.perl6.org/language/phasers#BEGIN) 来设置它只要赋值被编译完成:

```perl6
BEGIN %*ENV<RAKUDO_ERROR_COLOR> = 0;

for ^5 {
    say meow";
}
```

## An Exceptional Failure

Perl 6 有一个特殊的异常 -- [Failure](https://docs.perl6.org/type/Failure) -- 直到你把它用作变量它才会被激发, 并且你甚至可以通过在布尔上下文中使用它来彻底地消除它。你可以通过调用 [fail](https://docs.perl6.org/routine/fail) 子例程产生你自己的 Failures 并且 Perl 6 在核心中在尽可能合适的时候使用它。

这儿有一段代码, 其中我们定义了一个前缀操作符用来计算对象的圆周长, 给定一个半径。如果半径是负值, 它就调用 fail, 并返回一个 Failure 对象:

```perl6
sub prefix:<⟳> (\𝐫) {
    𝐫 < 0 and fail 'Your object warps the Universe a new one';
    τ × 𝐫;
}

say 'Calculating the circumference of the mystery object';
my $cₘ = ⟳ −𝑒;

say 'Calculating the circumference of the Earth';
my $cₑ = ⟳ 6.3781 × 10⁶;

say 'Calculating the circumference of the Sun';
my $cₛ = ⟳ 6.957 × 10⁸;

say "The circumference of the largest object is {max $cₘ, $cₑ, $cₛ} metres";

# OUTPUT:
# Calculating the circumference of the mystery object
# Calculating the circumference of the Earth
# Calculating the circumference of the Sun
# Your object warps the Universe a new one
#   in sub prefix:<⟳> at test.p6 line 2
#   in block <unit> at test.p6 line 7
#
# Actually thrown at:
#   in block <unit> at test.p6 line 15
```

在第七行中我们正计算一个半径为负值的圆的周长, 所以如果它仅仅是一个常规的异常, 那么我们的代码会当场挂掉。相反, 通过输出, 我们能够看到我们继续计算了 Earth 和 Sun 的周长, 直到我们到达最后一行。


在那儿我们尝试在 `$cₘ` 变量中使用 Failure 作为 max 程序的一个参数。因为我们在查询真实的值, Failure 被激发并给了我们一个很好的反向追踪。上面的错误信息包含了我们的 Failure 爆发(第十五行)点, 还有我们的接收点(第七行)还有错误来自哪儿(第二行)。真甜!

## 结论

Perl 6 拥有令人惊叹的 Errors!

大西瓜啊。
