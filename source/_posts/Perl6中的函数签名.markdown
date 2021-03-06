---
title: Perl 6 中的函数签名
date: 2016-04-12
tags: 签名
categories: Perl 6
comments: true
---

## [签名](http://doc.perl6.org/type/Signature)也是对象
---

```perl6
> sub a($a, $b) {};
> &a.signature.perl.say
:($a, $b)
> my $b = -> $a, $b {};
> $b.signature.perl.say
:($a, $b)
```

签名是一个对象, 就像 Perl 6 中的任何其它东西一样。 任何 **Callable** 类型中都有签名, 并且它能使用 `.signature`方法获取到。

```perl6
class Signature { ... }
```

签名是代码对象参数列表的静态描述。即, 签名描述了你需要什么参数和多少参数传递给代码或函数以调用它们。

传递参数给签名把包含在 **Capture** 中的参数绑定到了签名上。

## 签名字面量
---

签名出现在子例程和方法名后面的圆括号中, 还出现在 blocks 里面的 `->`或 `<->`后面, 或者作为变量声明符(例如 `my` )的输入, 或者以冒号开头作为单独的项。

```perl6
sub f($x) { }
#    ^^^^ sub f 的签名
method x() { }
#       ^^ 方法 x 的签名
my $s = sub (*@a) { }
#           ^^^^^ 匿名函数的签名

for @list -> $x { }
#            ^^    block 的签名

my ($a, @b) = 5, (6,7,8);
#  ^^^^^^^^ 变量声明符的签名

my $sig = :($a, $b);
#          ^^^^^^^^ 独立的签名对象
```

签名字面量可以用于定义回调或闭包的签名。

```perl6
sub f(&c:(Int)){}
sub will-work(Int){}
sub won't-work(Str){}
f(&will-work);
f(&won't-work); # fails at runtime
f(-> Int { 'this works too' } );
```

### 参数分隔符
---

签名由逗号分割的0个或多个参数组成。

```perl6
:($a, @b, %c)
sub add ($a, $b) { $a + $b }
```

作为一个例外, 签名中的第一个参数后面可以跟着一个冒号而非逗号来标记方法的调用者。调用者是用于调用方法的东西, 它通常通过在签名中指定它来绑定给 **self**, 你可以更改所绑定的变量的名字。

```perl6
:($a: @b, %c)  # 第一个参数是调用者

class Foo {
    method whoami ($me:) {
        "Well I'm class $me.^name(), of course!"
    }
}

say Foo.whoami; # Well I'm class Foo, of course!
```

### 类型约束
---

参数可以可选地拥有一个类型约束(默认为 `Any`)。这些能用于限制函数允许的输入。

```perl6
:(Int $a, Str $b)
sub divisors (Int $n) { $_ if $n %% $_ for 1..$n }
divisors 2.5; # !!! Calling 'divisors' will never work with argument types (Rat)
```

匿名的参数也行, 如果参数只需要它的类型约束的话。

```perl6
:($, @, %a)         # 两个匿名参数和一个 "正常的(有名字的)"参数
:(Int, Positional)  # 只有类型也行(两个参数)
sub baz (Str) {"Got a String"}
baz("hello");
```

类型约束也可以是类型捕获([type captures](http://doc.perl6.org/type/Signature#Type_Captures))。

除了这些名义上的类型之外, 额外的约束可以以代码块的形式加到参数上, 代码块必须返回一个真值以通过类型检测。

```perl6
sub f(Real $x where { $x > 0 }, Real $y where { $y >= $x }) { }
```

事实上, where 后面不需要是一个代码块, `where`-block右侧的任何东西都会被用于和参数智能匹配。所以你也可以这样写:

```perl6
multi factorial(Int $ where 0) { 1 }
multi factorial(Int $x) { $x * factorial($x - 1) }
```

第一个还能简化为

```perl6
multi factorial(0) { 1 }
```

你可以直接把字面量用作类型而值约束到匿名参数上。

#### 约束定义值和未定义值
---

通常, 类型约束只检查传递的值是否是正确的**类型**。

```perl6
sub limit-lines (Str $s, Int $limit) {
    my @lines = $s.lines;
    @lines[0 ..^ min @lines.elems, $limit].join("\n")
}
say (limit-lines "a \n b \n c \n d \n", 3).perl; # "a \n b \n c "
say limit-lines Str,      3;  # Uh-oh. Dies with "Cannot call 'lines';"
say limit-lines "a \n b", Int # Always returns the max number of lines
```

这样的情况, 我们其实只想处理定义了的字符串。要这样做, 我们使用 `:D`类型约束。

```perl6
sub limit-lines (Str:D $s, Int $limit) {
    ...
}

say limit-lines Str, 3;
# Dies with "参数 '$s' 需要一个实例, 但是函数 limit-lines 中却传递了一个类型对象。

```

如果传递一个诸如 **Str** 这样的类型对象进去, 那么就会报错。这样的失败方式比以前更好了, 因为失败的原因更清晰了。

也有可能未定义的类型是子例程唯一有意义的接收值。这可以使用 `:U`类型约束来约束它。例如, 我们可以把 `&limit-lines`转换成 multi 函数以使用 `:U`约束。

```perl6
multi  limit-lines (Str $s, Int:D $limit) {
    my @lines = $s.lines;
    @lines[0 ..^ min @lines.elems, $limit].join("\n");
}

multi limit-lines (Str $s, Int:U $) {$s} # 如果传递给我一个未定义的类型对象, 就返回整个字符串

say limit-lines "a \n b \n c", Int;      # "a \n b \n c"
```

为了显式地标示常规的行为,  可以使用`:_`,  但这不是必须的。 `:(Num:_ $)` 和 `Num $`相同。

#### 约束返回类型
---

`-->`标记后面跟着一个类型会强制在子例程执行成功时进行类型检测。返回类型箭头必须放在参数列表的后面。跟在签名声明后面的 `returns` 关键字有同样的功能。`Nil`在类型检测中被认为是定义了的。

```perl6
sub foo(--> Int) { 1 };
sub foo() returns Int { 1 };        # 同上
sub does-not-work(--> Int) { " " }; # throws X::TypeCheck::Return
```

如果类型约束是一个常量表达式, 那么它被用于子例程的**返回值**。那个子例程中的任何**return**语句必须是不含参数的。

```perl6
sub foo(--> 123) { return }
```

`Nil`和 `Failure`总是被允许作为返回**类型**, 不管类型约束是什么。

```perl6
sub foo(--> Int) { Nil };
say foo.perl; # Nil
```

不支持类型捕获和强制类型。

### 吞噬参数(或长度可变参数)
---

数组或散列参数可以通过前置一个星号(s)被标记为吞噬参数, 这意味着它可以被绑定给任意数量的参数(0 个或 多个)。

它们被叫做吞噬参数, 因为它们吞完函数中的任何剩余参数, 就像有些人吞吃面条那样。

```perl6
:($a, @b)  # 正好两个参数, 而第二个参数必须是 Positional 的
:($a, *@b) # 至少一个参数, @b 吞噬完任何剩余的参数
:(*%h)     # 没有位置参数, 除了任意数量的具名参数
```

```perl6
sub one-arg (@)  { };
sub slury   (*@) { };

one-arg(5, 6, 7);  # !!! 参数个数太多
one-arg (5, 6, 7); # ok, 和 one-arg((5,6,7))相同, 传递的是一个数组

slurp (5, 6, 7);   # ok
one-arg 5, 6, 7;   # 调用 one-arg(Int, Int, Int) 绝对不会工作, 使用声明的签名 (@), 参数个数太多
slurp 5, 6, 7;     # ok

one-arg (5);       # Calling one-arg(Int) will never work with declared signature (@)
one-arg (5,);      # ok
```

one-arg 函数需要的参数是**一个**列表(或数组), 而不是多个参数。

```perl6
> (5).WHAT.say
(Int)
> (5,).WHAT.say
(List)
```

```perl6
sub named-names (*%named-args) { %named-args.keys };
say named-names :foo(42) :bar<hahaha>  # => foo bar
```

注意位置参数不允许出现在吞噬参数的后面:

```perl6
:(*@args, $last) # !!! 不能把必要参数放在可变长度参数的后面
```

带有一个星号的吞噬参数会通过消融一层或多层裸的可迭代对象来展平参数。 带有两个星号的吞噬参数不会展平参数：

```perl6
sub a (*@a)  { @a.join("|").say };
sub b (**@b) { @b.join("|").say };

a(1,[1,2],([3,4],5));    #  1|1|2|3|4|5
b(1,[1,2],([3,4],5));    # 1|1 2|3 4 5

```

通常, 吞噬参数会创建一个数组, 为每个 argument 创建一个标量容器, 并且把每个参数的值赋值给那些标量。如果在该过程中原参数也有一个中间的标量分量, 那么它在调用函数中是访问不到的。

吞噬参数在和某些[traits and modifiers](http://doc.perl6.org/type/Signature#Parameter_Traits_and_Modifiers)组合使用时会有特殊行为, 像下面描述的那样。

### 类型捕获
---

类型捕获允许把类型约束的说明推迟到函数被调用时。它们允许签名和函数体中的类型都可以引用。

```perl6
sub f(::T $p1, T $p2, ::C) {
    # $p1 和 $p2 的类型都为 T, 但是我们还不知道具体类型是什么
    # C 将会保存一个源于类型对象或值的类型
    my C $closure = $p1 / $p2;
    return sub (T $p1) {
        $closure * $p1;
    }
}

# 第一个参数是 Int 类型, 所以第二个参数也是
# 我们从调用用于 &f 中的操作符导出第三个类型
my &s = f(10,2, Int.new / Int.new);
say s(2);  # 10 / 2 * 2  == 10
```

### Positional vs. Named
---

参数可以是跟位置有关的或者是具名的。所有的参数都是 positional 的, 除了吞噬型散列参数和有前置冒号标记的参数:

```perl6
:($a)   # 位置参数
:(:$a)  # 名字为 a 的具名参数
:(*@a)  # 吞噬型位置参数
:(*%h)  # 吞噬型具名参数
```

在调用者这边, 位置参数的传递顺序和它们声明顺序相同。

```perl6
sub pos($x, $y) { "x = $x y = $y" };
pos(4, 5); #  x = 4 y = 5
```

对于具名实参和具名形参, 只用名字用于将实参映射到形参上。

```perl6
sub named(:$x, :$y) { "x=$x y=$y" }
named( y => 5, x => 4);
```

具名参数也可以和变量的名字不同:

```perl6
sub named(:official($private)) { "公务" if $private }
named :official;
```

别名也是那样做的:

```perl6
sub paint( :color(:colour($c)) ) { } # 'color' 和 'colour' 都可以
sub paint( :color(:$colour) )    { } # same API for the caller
```

带有具名参数的函数可以被动态地调用, 使用 `|`非关联化一个 Pair 来把它转换为一个具名参数。

```perl6
multi f(:$named) { note &?ROUTINE.signature };
multi f(:$also-named) { note &?ROUTINE.signature };

for 'named', 'also-named' -> $n {
    f(|($n => rand))      # «(:$named)␤(:$also-named)␤»
}

my $pair = :named(1);
f |$pair; # «(:$named)␤»
```

同样的语法也可以用于将散列转换为具名参数：

```perl6
my %pairs = also-named => 4;
f |%pairs;        # (:$also-named)
```

### 可选参数和强制参数
---

Positional 参数默认是强制的,  也可以用默认值或结尾的问号使参数成为可选的:

```perl6
:(Str $id)         # 必要参数 required parameter
:($base = 10)      # 可选参数, 默认为 10
:(Int $x?)         # 可选参数, 默认为 Int 类型的对象
```

具名参数默认是可选的, 可以通过在参数末尾加上一个感叹号使它变成强制参数:

```perl6
:(:%config)        # 可选参数
:(:$debug = False) # 可选参数, 默认为 False
:(:$name!)         # 名为 name 的强制具名参数
```

默认值可以依靠之前的参数, 并且每次调用都会被重新计算。

```perl6
:($goal, $accuracy = $goal / 100);
:(:$excludes = ['.', '..']); # a new Array for every call
```

### 解构参数
---

参数后面可以跟着一个由括号括起来的 `sub-signature`, 子签名会解构给定的参数。解构的列表就是它的元素:

```perl6
sub first (@array ($first, *@rest)) { $first }
```

或

```perl6
sub first ([$first, *@]) { $first }
```

而散列的解构是它的键值对儿:

```perl6
sub all-dimensions (% (:length(:$x), :width(:$y), :depth(:$z))) {
    sx andthen $y andthen $z andthen True
}
```

`andthen` 返回第一个未定义的值, 否则返回最后一个元素。短路操作符。`andthen` 左侧的结果被绑定给 `$_` 用于右侧, 或者作为参数传递, 如果右侧是一个 `block` 或 `pointy block` 的话。

一般地, 对象根据它的属性结构。通用的惯用法是在 *for* 循环中解包一个 `Pair`的键和值:

```perl6
for @guest-list.pairs -> (:key($index), :value($guest)) {
    ...
}
```

然而, 这种把对象解包为它们的属性只是默认行为。为了让对象按照不同的方解构, 改变它们的 `Capture`方法。

### 捕获参数
---

在参数前前置一个垂直的 `|`会让参数变为 `Capture`, 并使用完所有剩下的位置参数和具名参数。

这常用在 `proto`定义中( 像 `proto foo (|) {*}` ) 来标示子例程的 `multi`定义可以拥有任何类型约束。

### 参数特性和修饰符
---

默认地, 形式参数被绑定到它们的实参上并且被标记为只读。你可以使用 traits 特性更改参数的只读特性。

`is copy`特性让参数被复制, 并允许在子例程内部修改参数的值。

```perl6
sub count-up ($x is copy) {
    $x = Inf if $x ~~ Whatever;
    .say for 1..$x;
}
```

`is rw`特性让参数只绑定到变量上(或其它可写的容器)。 赋值给参数会改变调用一侧的变量的值。

```perl6
sub swap($x is rw, $y is rw) {
    ($x, $y) = ($y, $x);
}
```

对于吞噬参数, `is rw` 由语言设计者保留做将来之用

## 方法
---

### params 方法
---

```perl6
method params(Signature:D:) returns Positional
```

返回 `Parameter`对象列表以组成签名。

### arity 方法
---

```perl6
method arity(Signature:D:) returns Int:D
```

返回所必须的最小数量的满足签名的位置参数

### count 方法
---

```perl6
method count(Signature:D:) returns Real:D
```

返回能被绑定给签名的最大数量的位置参数。如果有吞噬位置参数则返回 `Inf`。

### returns 方法
---

签名返回的任意约束是:

```perl6
:($a, $b --> Int).returns # Int
```

### ACCEPTS 方法
---

```perl6
multi method ACCEPTS(Signature:D: Capture $topic)
multi method ACCEPTS(Signature:D: @topic)
multi method ACCEPTS(Signature:D: %topic)
multi method ACCEPTS(Signature:D: Signature $topic)
```

前三个方法会看参能否绑定给 capture, 例如, 如果带有那个 Signature 的函数能使用 `$topic`调用:

```perl6
(1,2, :foo) ~~ :($a, $b, :foo($bar)) # true
<a b c d> ~~ :(Int $a)               # False
```

最后一个会为真如果 `$topic`能接收的任何东西也能被 `Signature`接收。

```perl6
:($a, $b) ~~ :($foo, $bar, $baz?)   # True
:(Int $n) ~~ :(Str)                 # False
```
