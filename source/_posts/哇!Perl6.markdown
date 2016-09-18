---
title: 哇, Perl 6!
date: 2016-08-14 13:14
categories: Perl 6
tags:
  - 惰性列表
  - Junctions
comments: true
---

## **前言**
[Slides 地址](http://tpm2016.zoffix.com/#/1)
[视频地址](https://www.youtube.com/watch?v=paa3niF72Nw)

[答案地址](http://blogs.perl.org/users/zoffix_znet/2016/03/wow-perl-6-talk-slides-recording-and-answers-to-questions.html)

## **注意** Unicode 

Perl 6 允许你使用 Unicode 的项和操作符。
这些 Unicode 都有对应的只使用 ASCII 字符的 Texas 变体, 如果你更喜欢使用它们, 到 [http://docs.perl6.org/language/unicode_texas](http://docs.perl6.org/language/unicode_texas) 找到它们。

## **惰性列表和它们的使用**

![img](http://upload-images.jianshu.io/upload_images/326727-b8abf1b05dbb2b7e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##  **惰性列表**

我们来做点儿疯狂的事情吧...例如创建一个无限列表(INIFINITE LIST)!

```perl6
my @to-infinity-and-beyond = 0, 2 ... ∞;
say @to-infinity-and-beyond[1008]; # 2016
```

来点更有用的东西: 处理大文件

```perl6
for '/tmp/4GB-file.txt'.IO.words {
    .say;
    last if ++$ == 3;
}
say "Code took {now - INIT now} seconds to run";

# OUTPUT:
# foo
# bar
# ber
# Code took 0.01111143 seconds to run
```

```perl6
.say for '/tmp/4GB-file.txt'.IO.words[0..2];
say "Code took {now - INIT now} seconds to run";

# OUTPUT:
# foo
# bar
# ber
# Code took 0.01111143 seconds to run
```

## **塑造你自己的** Subsets, **自定义操作符**, Muti-dispatch

![img](http://upload-images.jianshu.io/upload_images/326727-be2bf4d1cd691330.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Subsets

类型的 subset 使你可以约束它能接收的值:

```perl6
subset BigPrime of Int where { $_ > 10_000 and .is-prime }

sub MAIN ( BigPrime $num ) {
    say "That's a nice-looking prime number you got there!";
}
```

```
$ perl6 test.p6 3
Usage:
  test.p6 <num>
$ perl6 test.p6 31337
That's a nice-looking prime number you got there!
$ perl6 test.p6 100000
Usage:
  test.p6 <num>
```

### Multi-Dispatch

多个同名的 subs 或 方法, 但是拥有不同的参数:

```perl6
subset Prime      of Int   where *.is-prime;
subset BigPrime   of Prime where * >  10_000;
subset SmallPrime of Prime where * <= 10_000;

multi MAIN ( BigPrime   $num ) { say "Prime number! Nice and big"; }
multi MAIN ( SmallPrime $num ) { say "Puny prime number";          }
multi MANI (            $num ) { say "Gimme primes";               }
```

```
$ perl6 test.p6 42
Gimme primes!
$ perl6 test.p6 7
Puny prime number
$ perl6 test.p6 31337
Prime number! Nice and big
```

```perl6
class Numbers {
    multi method id ( Numeric $num ) { say "$num is a number"       }
    multi method id (         $num ) { say "$num is something else" }
}

Numeric.new.id:  π;
Numbers.new.id: 'blah';

# OUTPUT:
# 3.14159265358979 is a number
# blah is something else
```

扩展方法的功能性:

```perl6
class Numbers {
    multi method id ( Numeric $num ) { say "$num is a number"       }
    multi method id (        $num  ) { say "$num is something else" }
}

class SmarterNumbers is Numbers {
    multi method id (Numeric $num where * ==  π ) { say "Mmmm yummy pie!" }
}

SmarterNumbers.new.id: 42;
SmarterNumbers.new.id: π;
SmarterNumbers.new.id: 'blah';

# OUTPUT:
# 42 is a number
# Mmmm yummy pie!
# blah is something else
```

### Custom Terms and Operators

![img](http://upload-images.jianshu.io/upload_images/326727-b310dac691db4f59.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```perl6
sub infix:<¯\(°_o)/¯> {
     ($^a, $^b).pick
 }

say 'Coke' ¯\(°_o)/¯ 'Pepsi';

# OUTPUT:
# Pepsi
```

操作符分类: `infix`, `prefix`, `postfix`,`circumfix`, `postcircumfix`  并且你还能把  `term` 用于项中。

```perl6
sub prefix:<∑> (*@els) { @els.sum }
say ∑ 1, 2, 3, 4;

# OUTPUT:
# 1234
```

看起来并没有很好地起作用...

```perl6
sub prefix:<∑> (*@els) is looser(&infix:<,>) { @els.sum }
say ∑ 1, 2, 3, 4;

# OUTPUT:
# 10
```

使用 `is looser` 或 `is tighter` 来更改优先级。默认的优先级和那个位置上的 `+`/`++` 操作符的优先级相同。(查看 [docs](https://doc.perl6.org/language/functions#Associativity) 以使用 `is assoc` trait 更改结合性。)

One More Example:

```perl6
 sub term:<ξ> { (^10 + 1).pick; }
 sub postcircumfix:<❨  ❩> ($before, $inside) is rw {
     $before{$inside};
 }

 my %hash = :foo<bar>, :meow<moo>;
 %hash❨'foo'❩  = ξ;
 %hash❨'meow'❩ = ξ;

say %hash;

# OUTPUT:
# foo => 6, meow => 8
```

重写已经存在的操作符:

```perl6
sub infix:<+> (Int $a, Int $b) { $a - $b };
say 2 + 2;

# OUTPUT:
# 0
```

![img](http://upload-images.jianshu.io/upload_images/326727-71e3d5f44f434c2c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不要担心… 这个效果是本地作用域的(lexically scoped!)

![img](http://upload-images.jianshu.io/upload_images/326727-815a894031734d6c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```perl6
class Thingamagig { has $.value };

multi infix:<~> (Thingamagig $a, Str $b) {
    $a.value ~ $b
}

my $thing = Thingamagig.new: value => 'thingamagig';
say 'foo' ~ 'bar';
say $thing ~ 'bar';
say 'bar' ~ $thing;

# OUTPUT:
# foobar
# thingamagigbar
# barThingamagig<139870715547680>
```

查看 [Color::Operators](https://github.com/zoffixznet/perl6-Color/blob/master/lib/Color/Operators.pm6) 模块获取更多例子。

## Hyperspace

### Multi-core processing at a touch of a button

![img](http://upload-images.jianshu.io/upload_images/326727-be2f92ec80152cf1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 超运算符

它们看起来像 `«` 和 `»` 你能在这儿看到它们的 [解释](http://perl6maven.com/tutorial/perl6-hyper-operators) :

```perl6
      (1, 2) «+« (3)
      (1, 2) »+» 1
(1, 2, 3, 4) «+» (1, 2)
```

那些操作符暂时还不是多线程的, if at all。相反, 我要谈论的变体是这个:

```perl6
@foo».bar
```

假设你想把数组中的每个字符串都转为大写然后把那个数组分割为子列表, 每个子列表含有 3 个元素:

```perl6
my @a = <one two three four five six seven eight nine>;
say @a.map({ .uc}).rotor: 3;

# OUTPUT:
# ((ONE TWO THREE) (FOUR FIVE SIX) (SEVEN EIGHT NINE))
```

很好很简短, 但是假设你想在上千个元素身上调用耗时更多的方法呢?

那么就在方法调用前使用一个 hyper 操作符好了:

```perl6
my @a =  <one two three four five six  seven eight nine>;
say @a».uc.rotor: 3;

# OUTPUT:
# ((ONE TWO THREE) (FOUR FIVE SIX) (SEVEN EIGHT NINE))
```

在点号方法调用前面放上一个 » 那么你所调用的方法会在每个元素身上单独调用。链式调用中更远的那个方法会在数组(列表等)身上调用, 除非它们也是 hypered。

### Hyper Seqs

假如你想在多核上对一堆东西上"做事情"呢?  遍历一个 `HyperSeq`。你可以调用下面的两者之一得到一个 HyperSeq:

- `.hyper`— 保留元素顺序
- `.race`—   不保留元素顺序

遍历一个 4-元素的序列, 每个元素睡眠 1 秒钟:

```perl6
for (1..4).race( batch => 1 ) {
    say "Doing $_";
    sleep 1;
}
say "Code took {now - INIT now} seconds to run";

# OUTPUT:
# Doing 1
# Doing 3
# Doing 2
# Doing 4
# Code took 1.0090415 seconds to run
```

代码只是运行了 1秒多!

`.hyper`  同样, 但是它保留了结果序列中元素的顺序。

### Autothreaded junctions

- Logical superposition of values

某些逻辑检查的代码:

```perl6
my @valid = <foo bar baz>;
my $what = 'ber';
say "$what is not valid" if not @valid.grep: { $what eq $_ };
say "A ber or a bar"     if $what eq 'ber' or $what eq 'bar';

# OUTPUT:
# ber is not valid
# A ber or a bar
```

- Autothreaded junctions

```perl6
my @valid = <foo bar baz>;
my $what = 'ber';
say "$what is not valid" if $what eq none @valid;
say "A ber or a bar"     if $what eq 'ber' | 'bar';

# OUTPUT:
# ber is not valid
# A ber or a bar
```

| type | constructor | operator |  True if ...  |
| :--: | :---------: | :------: | :-----------: |
| all  |     all     |    &     |  没有值为 False   |
| any  |     any     |    \|    | 至少一个值为 True  |
| one  |     one     |    ^     | 完全只有一个值为 True |
| none |    none     |          |   没有值为 True   |

最好的部分?  Junctions 是自动线程化的(autothreaded), 这意味着编译器能在多个线程上对它们进行求值!

![](http://upload-images.jianshu.io/upload_images/326727-0494c5bd718ac532.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Promises

我不经常写并行化的代码, 但是当我写的时候, 它是这样的简单:

```perl6
 start { sleep 3; say "two" };
 say "one";
 sleep 5;
 say "three";

# OUTPUT:
# one
# two
# three
```

并发 / 异步 代码:

```perl6
my @promises = ^3 .map: {
    start {
        .say; sleep 1;
        $_ * 4;
    }
};
say "Started! {now - INIT now}";
say await @promises;
say "All done! {now - INIT now}";

# OUTPUT:
# 0
# 1
# 2
# Started! 0.0196113
# (0 4 8)
# All done! 1.0188611
```

Start later:

```perl6
Promise.in(5).then: -> $v { say "It's been {now - INIT now} seconds!" };
sleep 7;
say "Ran for {now - INIT now} seconds"

# OUTPUT:
# It's been 5.031918 seconds!
# Ran for 7.0160562 seconds
```

## Supplies

异步数据流:

```perl6
my $supplier = Supplier.new;

$supplier.Supply              .tap: -> $v { say "Original: $v" };
$supplier.Supply.map(  * × 2 ).tap: -> $v { say "  Double: $v" };
$supplier.Supply.grep( * % 2 ).tap: -> $v { say "  Odd: $v"    };

$supplier.emit: $_ for ^3;

# OUTPUT:
# Original: 0
#   Double: 0
# Original: 1
#   Double: 2
#   Odd: 1
# Original: 2
#   Double: 4
```

Events at interval inside an event loop (`react`):

```perl6
react {
    whenever Supply.interval: 2 -> $v {
        say "$v: {now - INIT now}";
        done if $v == 2;
    }
    whenever Supply.interval: 1 -> $v { say "  1 sec: {now - INIT now}"; }
}

 # OUTPUT:
 # 0: 0.026734
 #   1 sec: 0.0333274
 #   1 sec: 1.02325708
 # 1: 2.027192
 #   1 sec: 2.0276854
 #   1 sec: 3.0234109
 # 2: 4.0324349
```

## Channels

一个线程安全的队列:

```perl6
my $c = Channel.new;
start {
    loop { say "$c.receive() at {now - INIT now}" }
}
await ^10 .map: -> $r {
    start {
         sleep $r;
         $c.send: $r;
    }
}
$c.close;

# OUTPUT:
# 0 at 0.01036314
# 1 at 1.0152853
# 2 at 2.0174991
# 3 at 3.020067105
```

## Grammars

更容易的解析东西的方式。

![img](http://upload-images.jianshu.io/upload_images/326727-efdc5ad20795b4dd.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```perl6
grammar MyGrammar {
    token TOP     { <sign> <digits> <decimal>? }
    token sign    { <[+-]>?                    }
    token digits  { \d+                        }
    token decimal { \. <digits>                }
 }

say MyGrammar.parse: '250.42';

# OUTPUT:
# ｢250.42｣
#  sign => ｢｣
#  digits => ｢250｣
#  decimal => ｢.42｣
#   digits => ｢42｣
```

actions:

```perl6
grammar MyGrammar {
    token TOP     { <sign> <digits> <decimal>? }
    token sign    { <[+-]>?                    }
    token digits  { \d+                        }
    token decimal { \. <digits>                }
}

class MyActions {
    method sign ($/) { $/.make: $/.chars ?? ~$/ !! '+'                        }
    method TOP  ($/) { $/.make: $<sign>.made ~ ($<digits> + 42 ) ~ $<decimal> }
}

say MyGrammar.parse('250.42', actions => MyActions).made;

# OUTPUT:
# +292.42
```

### 有用的模块:

- `Grammar::Debugger` 和 `Grammar::Tracer`—  使用两者之一来调试你的  grammars 的输出。
- `Grammar::BNF`— 自定地把 BNF 转换为 Perl 6 grammars !!

## *Whatever, man!*

### Whatever Code, Meta operators, Model6 Object Model, Sets, bags, and mixes

![img](http://upload-images.jianshu.io/upload_images/326727-4b889b4217fec764.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Whatever Code

使用 WhateverStar 作为一种快速获取一个带有参数的闭包的方式:

```perl6
 say (* + 2)(2);
 say <1 25 3 100>.grep: * > 5;
 subset Primes of Int where *.is-prime;
 
 # same as
 
 say sub { $^a + 2 }(2);
 say <1 25 3 100>.grep: { $_ > 5 };
 subset Primes of Int where { $_.is-prime };
```

每个 WhateverStar 代表着下一个位置参数。你不能使用 WhateverStar 引用同一个参数多于 1 次:

```perl6
say ( * + * + * )(2, 3, 4);

# OUTPUT:
# 9
```

n-at-a-time(一次 n 个) `.map` 从未像这样简单:

```perl6
say ^12 .map: * + * + *;

# OUTPUT:
# (3 12 21 30)
```

看, 整个 Fibonacci 序列存在于一个惰性列表中!

```perl6
my  @fibonacci = 0, 1, * + * … *;
say @fibonacci[42];

# OUTPUT:
# 267914296
```

## **喂**, **还有** HyperWhatever!

Just sayin'...

```perl6
say ( ** + 2 )(^10);
say (  * + 2 )(^10);

# OUTPUT:
# (2 3 4 5 6 7 8 9 10 11)
# 2..^12
```

## Meta Operators

和你把方括号里面的操作符放在列表中的元素之间的效果一样:

```
say [+] 1, 2, 3;   # 6
say [*] 1..5;      # 120

my @numbers = (2, 4, 3);
say [<] @numbers;  # False
```

*Triangle Reduce*: 在两个相继的元素之间使用操作符, 在它的结果和下一个元素之间再次使用操作符:

```
say [\+] 1, 2, 3;
say [\*] 1..5;

 # OUTPUT:
 # (1 3 6)  ## Breaking it down: 1 (1), 1 + 2 (3), 3 + 3 (6)
 # (1 2 6 24 120)
```

## Perl 6 Object Model

```perl6
class Foo {
    has $.attr;
    has $.attr-rw is rw;
    has $.attr-required is required;
    has $!attr-private = 42;

    method public   { say $!attr + $!attr-private; }
    method !private { say 'secret' }
}

my $foo = Foo.new: :attr<public>,
                   :attr-required<yup-here-it-is>;

say $foo.attr;
say $foo.public;
$foo.attr-rw = 42;
```

你也可以指定类型约束和 subset 约束:

```perl6
class Foo {
    subset Primes of IntStr where *.is-prime;

    has Int    $.attr;
    has Str    $.attr-rw is rw where *.chars < 5;
    has Primes $.attr-required is required;
}
```

Roles— 安全地为你的类添加行为:

```
role Drawable {
    method draw { say "Line from $.x to $.y" };
}
class Line does Drawable {
    has $.x;
    has $.y;
}
my $line = Line.new: :42x, :75y;
$line.draw;

# OUTPUT:
# Line from 42 to 75
```

```perl6
role Drawable {
    method draw { say "Line from $.x to $.y" };
}
role OtherDrawable {
    method draw { say "It's a draw!" };
}
class Line does Drawable does OtherDrawable {
    has $.x;
    has $.y;
}
my $line = Line.new: :42x, :75y;
$line.draw;

# OUTPUT:
# ===SORRY!=== Error while compiling /home/zoffix/CPAN/TPM-2016/test.p6
# Method 'draw' must be resolved by class Line because it exists in multiple
# roles (OtherDrawable, Drawable) at /home/zoffix/CPAN/TPM-2016/test.p6:7
```

## MOP: Meta Object Protocol

自省和 Perl 6 对象系统

![img](http://upload-images.jianshu.io/upload_images/326727-9562eb8b97578bd9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
#| Just 'cause
class TestStuff {};
my $x = TestStuff.new;
say $x.WHAT;     # (TestStuff)  # The type object of the type
say $x.WHICH;    # TestStuff|179689128 # The object's identity value
say $x.WHO;      # TestStuff # The package supporting the object
say $x.WHERE;    # -1225903068 # The memory address of the object (not stable)
say $x.HOW;      # Perl6::Metamodel::ClassHOW.new # The metaclass object
say $x.WHY;      # Just 'cause # The attached Pod value.
say $x.DEFINITE; # True # Returns True for instances and False for type objects
```

为已存在的类添加方法:

```
Int.^add_method('x', -> $class, $v { say $v });
constant A := Metamodel::ClassHOW.new_type( name => 'A' );
A.^add_method('x', -> $class, $v { say $v });
#A.^compose;

A.x: 'A class';
Int.x: 'Int class';
```

## Sets, Bags, and Mixes

**Immutable:**

- `Set`—不同对象的集合
- `Bag`—带有整数系数(权重)的不同对象的集合
- `Mix`—带有实际系数(权重)的不同对象的集合

**Mutable:**

- `SetHash`—不同对象的集合
- `BagHash`—带有整数系数(权重)的不同对象的集合
- `MixHash`—带有实际系数(权重)的不同对象的集合

对东西进行计数? 一个 `Bag`  就做到了:

```perl6
.say for 'This is just a sentence'.comb.Bag;

# OUTPUT:
# n => 2
# a => 1
#   => 4
# c => 1
# j => 1
# s => 4
# T => 1
# e => 3
# t => 2
# i => 2
# u => 1
# h => 1
```

使用集合操作符写简明代码:

```perl6
my $valid = set <foo bar ber boor>;
my @given = <foo meow bar ber>;

say ‘Something's wrong’ unless @given ⊆ $valid;
```

并集和差集:

```perl6
say <foo bar> ∪ <bar meow>;  # Union operator
say <foo bar> ⊖ <bar meow>;  # Symmetric set difference operator

# OUTPUT:
# set(foo, bar, meow)
# set(foo, meow)
```

查看 [the docs](http://docs.perl6.org/language/setbagmix) 获取完整的操作符列表。

## Polyglot

在 Perl 6 使用其它语言!

![](http://upload-images.jianshu.io/upload_images/326727-5c9d882e4102b06a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## NativeCall

不用写任何 C 代码就可以使用  C libraries!!

(或多或少)

标准 C library:

```perl6
use NativeCall;
sub fork is native {};
fork;
say $*PID;

# OUTPUT:
# 11274
# 11275
```

## BTW: A Safety Tip

在标准 C 库中有一个 `system` 

![](http://upload-images.jianshu.io/upload_images/326727-3dece01d75627a4d.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用 C library:

```perl6
class STMT is repr('CPointer') { };
sub sqlite3_column_text(STMT, int32)
    returns Str
    is native('sqlite3', v0) { };
```

```
int8           (int8_t in C, also used for char)
int16          (int16_t in C, also used for short)
int32          (int32_t in C, also used for int)
int64          (int64_t in C)
uint8          (uint8_t in C, also used for unsigned char)
uint16         (uint16_t in C, also used for unsigned short)
uint32         (uint32_t in C, also used for unsigned int)
uint64         (uint64_t in C)
long           (long in C)
longlong       (long long in C, at least 64-bit)
num32          (float in C)
num64          (double in C)
Str            (C string)
CArray[int32]  (int* in C, an array of ints)
Pointer[void]  (void* in C, can point to all other types)
bool           (bool from C99)
size_t         (size_t in C)
```

使用 [App::GPTrixie](http://modules.perl6.org/dist/App::GPTrixie) 把 C 头文件转换为 Perl 6 子例程定义:

```perl6
$ gptrixie --all /usr/local/include/gumbo.h

## Enumerations
enum GumboNamespaceEnum is export (
   GUMBO_NAMESPACE_HTML => 0,
   GUMBO_NAMESPACE_SVG => 1,
   GUMBO_NAMESPACE_MATHML => 2
);
enum GumboParseFlags is export (
   GUMBO_INSERTION_NORMAL => 0,
   GUMBO_INSERTION_BY_PARSER => 1,

   ...
```

## [Inline::Perl5](http://modules.perl6.org/dist/Inline::Perl5)

在 Perl 6中使用任意 Perl 5 模块!

```perl6
use Inline::Perl5;
use Mojo::DOM:from<Perl5>;
 
my $dom = Mojo::DOM.new: '<p><b>This is awesome</b>, trust me</p>';
 
say $dom.at('b').all_text;

# OUTPUT:
# This is awesome
```

- 支持 Perl 5 模块, 包括 XS 模块。
- 允许在 Perl 5 和 Perl 6 之间传递整数、字符串、数组、散列、代码引用、文件句柄和对象。
- 支持从 Perl 6 中在 Perl 5 对象上调用方法并且支持从 Perl 5 中在 Perl 6 对象上调用方法。
- 在 Perl 6中子类化 Perl 5 的类。

## **使用一堆其它语言**...

你和我展示的可能有点不同:

![img](http://upload-images.jianshu.io/upload_images/326727-fc3ec03dfe5240f3.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Simple Things:Hacking on the Perl 6 Compiler

![](http://upload-images.jianshu.io/upload_images/326727-0c0668c970b6cc1b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## **听起来很疯狂**...

Perl 6 大部分是用 Perl 6 写的!

## **这意味着**...

一般的 Perl 6 使用者可以使 Perl 6 变得更好!

![](http://upload-images.jianshu.io/upload_images/326727-1cb84f014ede918b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Perl 6 is Written in Perl 6

### How??

基础是用 NQP 写的 (Not Quite Perl):[https://github.com/perl6/nqp](https://github.com/perl6/nqp)

![](http://upload-images.jianshu.io/upload_images/326727-28231953961eec03.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其它的是用 Perl 6 写的: [https://github.com/rakudo/rakudo](https://github.com/rakudo/rakudo)

![](http://upload-images.jianshu.io/upload_images/326727-7f43059cb5b74117.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 那么解析呢...?

是的, 仅仅是一个 Perl 6 Grammar:

![](http://upload-images.jianshu.io/upload_images/326727-e54d22969bf5ef49.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Bonus Slides

## **并发代码中的错误**

错误在哪一行?

```perl6
my $promise = start {
    say 0/0;
    # A whole
    # bunch
    # of other code
};
 
say await $promise;

# OUTPUT:
# Attempt to divide by zero using div
#   in block <unit> at test.p6 line 8
```

包含一个花括号:

```
 1: my $promise = start {
 2:     say 0/0;
 3:     # A whole
 4:     # bunch
 5:     # of other code
 6:     CATCH { warn .backtrace };
 7: };
 8:
 9: say await $promise;

# OUTPUT:
#   in block  at test.p6 line 2
#   in block  at test.p6 line 6
# Attempt to divide by zero using div
#   in block <unit> at test.p6 line 9
```

## **有理数**!

`Rat` 是一种 Perl 6 类型, 它由分子和分母表示:

```
my $x = 0/0;
say 'Universe is still unimploded';
say sin $x;

# OUTPUT:
# Universe is still unimploded
# NaN
```

实际的数字只在需要时计算:

```
my $x = 0/0;
say 'Universe is still unimploded';
say $x;

# OUTPUT:
# Universe is still unimploded
# Attempt to divide by zero using div
#   in block <unit> at test.p6 line 3
#
# Actually thrown at:
#   in block <unit> at test.p6 line 3
```

## [Proc::Async](http://docs.perl6.org/language/concurrency#Proc%3A%3AAsync)

Async 和其它程序通信:

```
my $proc = Proc::Async.new: :w, 'grep', 'foo';
$proc.stdout.tap: -> $v { print "Output: $v" };

say 'Starting grep process...';
my $promise = $proc.start;
    $proc.say: 'this line has foo';
    $proc.say: "this one doesn't";
    $proc.close-stdin;
await $promise;
say 'Done.';

# OUTPUT:
# Starting grep process...
# Output: this line has foo
# Done.
```

## **不要** `say` **太多**!

`say` 函数/方法用于展示人类可读的简明信息并调用对象的 `gist` 方法。如果你想输出一大堆东西, 那么使用 `put` 函数/方法:

```
^101 .say;
(0..100).list.say;
say '----------';
^101 .put;
(0..100).put;

# OUTPUT:
# ^101
# (0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 ...)
# ----------
# 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100
# 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100
```

## **更有用的对象**

 当使用`say`-ed, `put`-ed, 或其它任何东西时, 让你的对象更加准确地工作:

```
class Thingamagig {
    method gist    { 'Brief info'        }
    method Str     { 'Not so brief info' }
    method Numeric { 42                  }
}

my $thing = Thingamagig.new;
say $thing;
put $thing;
say $thing + 2;

# OUTPUT:
# Brief info
# Not so brief info
# 44
```

## **内置的性能分析器**

仅仅在命令行上指定 `--profile` 标记, 就会为你生成 HTML 格式的结果文件。

```
$ perl6 --profile test.p6
```

![](http://upload-images.jianshu.io/upload_images/326727-8a66f7a51366f797.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## **谢谢**!

Any Questions?
