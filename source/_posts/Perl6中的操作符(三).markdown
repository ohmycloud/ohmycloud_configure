---
title: Perl 6 中的操作符(三)
date: 2016-03-23
tags:
  - 操作符
  - 优先级
categories: Perl 6
comments: true
---

## 范围和范围迭代器语法

`..` 范围操作符有各种在两端带有 `^`符号的变体以表明把那个端点排除在范围之外。 它总会产生一个 Range 对象。 Range 对象是不可变的， 主要用于匹配间隔。

1..2 是从1到2包含端点的间隔，  而 1^..^2 不包含端点但是匹配任何在它俩之间的实数。

对于不同类型的数字参数， 范围会被强制为更宽的类型，所以：

``` perl
1 .. 1.5
```

被看作为：

``` perl
1.0 .. 1.5
```

这些强制由 multi 签名定义。（其它类型可能有不同的强制策略。）特别要说明的是， 使用 Range 作为末端是非法的：

``` perl
0 ..^ 10  # 0 .. 9
0 .. ^10  # ERROR
```

如果范围右侧是非数字类型， 那么右侧的参数被强转为数字， 然后按上面那样使用。

因此，第二个参数中的 Array 类型会被假定用作数字， 如果左侧的参数是数字的话：

``` perl
0 ..^ @x    # okay
0 ..^ +@x   # same thing
```

 对于字符串也类似：

``` perl
0 .. '1.5'  # okay
0 .. +'1.5' # same thing
```

Whatever 类型也支持代表 -Inf/+Inf。 如果端点之一是一个 WhateverCode, 那么范围会被引导为另一个 WhateverCode。

Range 对象支持代表它们的左侧和右侧参数的 .min 和 .max 方法。 .bounds 方法返回一个含有那两个值的列表以代表间隔。 Ranges 不会自动反转：

2..1 总是一个 null 范围。（然而， 序列操作符 .. 能够自动反转，看下面。）

在 Range 的每个端点处， Range 对象支持代表排除（有^）或包含（没有^）的 `.excludes_min` and `.excludes_max` 方法。

```
Range      | .min | .max | .excludes_min | .excludes_max
-----------+------+------+---------------+------------
1..10      | 1    | 10   | Bool::False   | Bool::False
2.7..^9.3  | 2.7  | 9.3  | Bool::False   | Bool::True
'a'^..'z'  | 'a'  | 'z'  | Bool::True    | Bool::False
1^..^10    | 1    | 10   | Bool::True    | Bool::True
```

如果用在列表上下文中， Range 对象返回一个迭代器， 它产生一个以最小值开头，以最大值结尾的序列。

任一端点可以使用 ^ 排除掉。因此 1..2 产生 (1,2) 但是 `1^..^2` 等价于 2..1 并不产生值， 就像 () 做的那样。要指定一个倒数的序列， 使用反转：

``` perl
reverse 1..2
reverse 'a'..'z'
```

作为选择， 对于数字序列， 你能使用序列操作符代替范围操作符：

``` perl
100,99,98 ... 0
100, *-1 ... 0      # same thing
```

换句话说，任何用作列表的 Range 会假定 .succ 语义， 绝对不会是 `.pred` 语义。 没有其它的增量被允许；如果你想通过某个不是 1 的增量数字来增加一个数字序列，

你必须使用 ... 序列操作符。（Range 操作符的 `:by` 副词因此被废弃了。）

```
0, *+0.1 ... 100    # 0, 0.1, 0.2, 0.3 ... 100
```

只有正念叨的类型支持 .succ 方法的 Range 才能被迭代。如果它不支持， 任何迭代尝试都会失败。


## [一元区间](https://desgin.perl6.org/S03.html#Unary_ranges)

一元操作符 ^ 生成一个从 0 直到 其参数（不包括该参数）的区间。所以 ^4 等价于  0..^4.

```
for ^4 { say $_ } # 0, 1, 2, 3
```

## [范围的自动填充](https://desgin.perl6.org/S03.html#Auto-priming_of_ranges)

[这一节是推断的，并且可能会在 6.0 中被忽略.]
因为在 item 上下文中 Range 对象通常是无意义的，用作标量操作符的 Range 对象一般会尝试把操作分配给终点并返回另外一个适当的修改过的 Range 代替。
很像两个项的连结(junction), 但是只使用合适的间隔语义。 (值得注意的这种自动线程化的例外包括 `infix:<~~>`, 它是做智能匹配的, 还有 `prefix:<+>`, 它返回范围的长度。) 因此如果你想使用长度而不是终点来做切片， 你可以这样说：

```perl6
@foo[ start() + ^$len ]
```

它是下面这种形式的简写:

```perl6
@foo[ start() + (0..^$len) ]
```

它有点等价于:

```perl6
@foo[ list do { my $tmp = start(); $tmp ..^ $tmp+$len } ]
```

换句话说，数值化的操作符和其它有顺序的类型通常被重载以在 Range 身上完成某些有意义的事情。

# [链式比较](https://desgin.perl6.org/S03.html#Chained_comparisons)

[S03-operators/relational.t lines 102–238](https://github.com/perl6/roast/blob/master/S03-operators/relational.t#L102-L238)  

Perl 6 支持比较操作符的自然扩展, 它允许多个操作数:

``` perl
 if 1 < $a < 100                        { say "Good, you picked a number *between* 1 and 100." }
 if 3 < $roll <= 6                      { print "High roll"  }
 if 1 <= $roll1 == $roll2 <= 6          { print "Doubles!"   }
```

如果第一个比较失败了则产生比较短路链:

[S03-operators/short-circuit.t lines 236–297](https://github.com/perl6/roast/blob/master/S03-operators/short-circuit.t#L236-L297)

```perl6
1 > 2 > die("this is never reached");
```

链子中得每个参数至多会被求值一次:


[S03-operators/short-circuit.t lines 226–235](https://github.com/perl6/roast/blob/master/S03-operators/short-circuit.t#L226-L235)

```perl6
1 > $x++ > 2    # $x 只精确地增长一次
```

注意: 任何以  *<* 开头的操作符必须在前面拥有空格，否则它会被解释为散列的下标。


# [调用者标记](https://desgin.perl6.org/S03.html#Invocant_marker)

当使用 Perl 6 方法调用的「间接对象」语法时追加的 `:` 标记了调用者(invocant)。下面的两个语句是等价的:

```perl6
$hacker.feed('Pizza and cola');
feed $hacker: 'Pizza and cola';
```

冒号也可以用在普通方法调用中以标示它应该被解析为列表操作符:

```perl6
$hacker.feed: 'Pizza and cola';
```
这个冒号是一个单独的令牌(token)。副词前面的冒号不是单独的令牌(token)。因此，在最长令牌(longest-token)规则下,

```perl6
$hacker.feed:xxx('Pizza and cola');
```

被标记为应用到方法上作为它的「最高层级的在前面的操作符」("toplevel preceding operator")的副词:


```perl6
$hacker.feed :xxx('Pizza and cola');
```

不是作为 `.feed` 参数列表中的 xxx sub:

```perl6
$hacker.feed: xxx('Pizza and cola');  # wrong
```

如果你两种意义的冒号你都想要，为了既提供副词又提供某种位置参数, 你必须放置两次冒号:

```perl6
$hacker.feed: :xxx('Pizza and cola'), 1,2,3;
```

(因为类型的原因它需要把空格放在标签的冒号之后。)

要特别注意因为副词的优先级:

```perl6
1 + $hacker.feed :xxx('Pizza and cola');
```

会把 `:xxx` 副词应用到 `+` 操作符上, 而不是应用到方法调用上。这是不可能成功的。


[S03-operators/adverbial-modifiers.t lines 7–201](https://github.com/perl6/roast/blob/master/S03-operators/adverbial-modifiers.t#L7-L201)

# [流操作符](https://desgin.perl6.org/S03.html#Feed_operators)

S03-feeds/basic.t lines 6–163  

新的操作符 `==>` 和 `<==` 就像Unix里的管道一样，但是它作用于函数或语句，接受并返回列表.因为这些列表由不相关联的对象组成并不流动， 我们把它们叫做喂食（feed）操作符而非管道。例如：

```perl6
     @result = map { floor($^x / 2) },
              grep { /^ \d+ $/      },
              @data;
```

也能写成向右偏的流操作符：

```perl6
    @data ==> grep { /^ \d+ $/       }
          ==> map  { floor($^x / 2)  }
          ==> @result;
```

或者使用左方向的流操作符：

```perl6
    @result <== map { floor($^x / 2) }
            <== grep { /^ \d+ $/     }
            <== @data;
```

每一种形式更清晰地表明了数据的流动。查看 [S06](http://design.perl6.org/S06.html)  了解更多关于这两个操作符的信息。

# [元操作符](https://desgin.perl6.org/S03.html#Meta_operators)

Perl 6 的操作符被极大地规范化了，例如，通过分别在数字、字符串和布尔操作符前前置 `+`、`~`、`?` 来表明按位操作是作用在数字、字符串还是单个位身上。但是那只是一种命名约定，并且如果你想添加一个新的按位 `¬` 操作符， 你必须自己添加  `+¬`, `~¬`, 和 `?¬` 操作符。 类似地，范围中排除末端的脱字符(`^`)在那里只是约定而已。


和它相比， Perl 6 有 8 个标准的元操作符用于把已存在的给定操作符转换为相关的更强大的操作符（或者至少不是一般的强大）。换句话说，这些元操作符正是高阶函数（以其它函数作为参数的函数）的便捷形式。

包含元操作符的结构被认为是 "metatokens"， 这意味着它们不受普通匹配规则的制约， 尽管它们的部件受制约。 然而，像普通的 tokens 那样， metatokens 不允许在它们的子部件之间有空格。

## 赋值操作符

S03-operators/autovivification.t lines 4–111  

C 和 Perl 程序员对于赋值操作符已经司空见惯了。（尽管 .= 操作符现在意味着在左边对象的身上调用一个可变方法， ~= 是字符串连结。）

大部分非关系中缀操作符能通过后缀 = 被转换为对应的赋值操作符。

```perl6
A op= B;
```

```perl6
A = A op B;
```

## 否定关系操作符

任何能返回 Bool 值的中缀关系操作符都可以通过前置一个 `!` 将它转换为否定的关系操作符。有几个关系操作符还有传统的便捷写法：

```perl6
    Full form   Shortcut
    ---------   --------
    !==         !=
    !eq         ne
```

但是大部分关系操作符没有传统的便捷写法:

```perl6
    !~~
    !<
    !>=
    !ge
    !===
    !eqv
    !=:=
```

为了避免 `!!` 操作符迷惑视线， 你不可以修改任何已经以`!` 开头的操作符。

否定操作符的优先级和基（base）操作符的优先级相同。

你只可以否定那些返回 Bool 值的操作符。 注意诸如 `||` 和 `^^` 的逻辑操作符不返回 Bool 值， 而是返回其中之一的操作数。



## 翻转操作符

在任意中缀操作符上前置一个 R，会翻转它的两个参数。例如，反向比较：

- Rcmp
- Rleg
- R<=>

任何翻转操作符的优先级和根操作符的优先级是一样的。结合性没有被翻转。

```perl
    [R-] 1,2,3   # produces 2 from 3 - (2 - 1)
```

要得到另外一种效果，可以先翻转列表：

``` perl
    [-] reverse 1,2,3  # produces 0
```

## 超运算符

Unicode 字符  `»` (\x[BB]) 和 `«` (\x[AB]) 和它们的 ASCII连字 `>>` 和 `<<` 用于表示`列表操作`, 它作用于列表中的每个元素, 然后返回单个列表(或数组)作为结果. 换句话说,  超运算符在 item 上下文中计算它的参数, 但是随后将操作符分派到每个参数身上，结果作为列表返回。

当书写超运算符时， 里面不允许出现空格， 即， 两个 "hyper" 标记之间不能有空格， 并且该操作符是能修改参数的。 在外面空格策略和它的 base 操作符相同。 同样地， 超运算符的优先级和它的 base 操作符相同。 这意味着对于大部分操作符，你必须使用圆括号括起你使用逗号分割的列表。。

例如：

``` perl
     -« (1,2,3);                   # (-1, -2, -3)
     (1,1,2,3,5) »+« (1,2,3,5,8);  # (2,3,5,8,13)，尖括号都朝内时，两边列表元素的个数必须相同
```

（如果你发现你自己这样做了， 问问你自己是否真的在使用对象或列表； 在后一种情况中， 可能其它的诸如 Z 或 X 的元操作符更合适， 并且不需要括号）

一元超运算符（要么前缀，要么后缀）只有一个 hyper 标记， 位于它的参数那边， 而中缀操作符通常在参数的每一边以表明有两个参数。

### 一元超运算符

一元超运算符的意思取决于操作符是否是结构非关联的操作符。 大部分操作符不是结构的。

对于中缀操作符，如果两者其中之一的一个参数的长度不够，那么 Perl 会「提高」它，但是只有你把 hyper 标记「尖」的那一端指向它时，Perl 才会提升长度不够的那一端。

```Perl6
 (3,8,2,9,3,8) >>->> 1;          # (2,7,1,8,2,7)
 @array »+=» 42;                 # add 42 to each element
```

实际上，对于一个无序的诸如 Bag 的类型来说，一个提升过的标量是唯一能工作于该类型中的东西:

```Perl6
 Bag(3,8,2,9,3,8) >>->> 1;       # Bag(2,7,1,8,2,7) === Bag(1,2,2,7,7,8)
 # Cannot modify an immutable Bag
```

```Perl6
>  Bag(3,8,2,9,3,8)  # Bag 的用法以改变
bag(9, 8(2), 3(2), 2)
```

换句话说，把小于号那端指向一个参数告诉超运算符在那边做我想要做的(dwim, do what i means)。如果你不知道一边或是另一边会是 underdimensioned，那么你可以在两边都做我想做的:

```Perl6
$left «*» $right
```

注意：如果你担心 Perl 会被像这样的东西所迷惑：Note: if you are worried about Perl getting confused by something like this:

```Perl6
func «*»
```

那么你无需担心，因为不想之前的版本， Perl 6 从来不会猜测下一个东西是一个项(term)还是一个操作符。在这种情况下，它总是期望一个项除非 func 被预先定义为一个类型或值的名字。
升级绝对不会发生在 hyper 的「钝」的那一端上。如果你这样写：

```Perl6
$bigger «*« $smaller
$smaller »*» $bigger
```

那么会抛出异常，而如果你这样写：

```Perl6
$foo »*« $bar
```

那么你要求形状的两边是一样长的，否则会抛出异常。

对于所有 hyper dwimminess，如果运算符的另一边期望列表的地方出现的是一个标量，那么那个标量会被当做一个重复了 `*` 次的那个元素的列表。

一旦我们有两个列表要处理，那么我们不得不决定使两边的元素长度相一致。如果两边都是 dwimmy，那么较短的那个列表会重复尽可能多的次数以使元素的个数合适。

如果只有一边是 dwimmy，那么那一端的列表只会被增长或截断以适应另一边的 non-dwimmy 的那个列表。

不管从数组的形状的 dwim 是强制的还是自然发生的，一旦选择了 dwim 的那一端，在 dwimmy 端的 dwim 语义总是：

下面是一些例子:

``` perl
    (1,2,3,4) »+« (1,2)    # always error，尖括号都朝内时，两边元素必须个数相同
    (1,2,3,4) «+» (1,2)    # 2,4,4,6     rhs dwims to 1,2,1,2
    (1,2,3)   «+» (1,2)    # 2,4,4       rhs dwims to 1,2,1
    (1,2,3,4) «+« (1,2)    # 2,4         lhs dwims to 1,2
    (1,2,3,4) »+» (1,2)    # 2,4,4,6     rhs dwims to 1,2,1,2
    (1,2,3)   »+» (1,2)    # 2,4,4       rhs dwims to 1,2,1
    (1,2,3)   »+» 1        # 2,3,4       rhs dwims to 1,1,1
```

当使用`一元`操作符时, 你总是把钝的那端对准单个运算对象, 因为没有出现重复的东西:

``` perl
     @negatives = -« @positives;
     @positions»++;            # Increment all positions
     @positions.»++;           # Same thing, dot form
     @positions».++;           # Same thing, dot form 报错
     @positions.».++;          # Same thing, dot form
     @positions\  .»\  .++;    # Same thing, unspace form
     @objects.».run();
     ("f","oo","bar").».chars; # (1,2,3)
```

注意方法调用实际上是后缀操作符, 而非中缀操作符, 所以,  你不能在点号后面放上一个 ` «`

超运算符在嵌套数组中是被递归地定义的， 所以：

``` perl
    -« [[1, 2], 3]              #    [-«[1, 2], -«3] 得到 -1 -2 -3
                                # == [[-1, -2], -3]
```

```
[[1, 2], 3] «+» [4, [5, 6]]  #    [[1,2] «+» 4, 3 «+» [5, 6]]，得到 5 6 8 9
                             # == [[5, 6], [8, 9]]
```

超运算符也能作用于散列，就像作用于数组一样。

```
%foo «+» %bar;
```

得到两个键的交集（对应的键值相加）

``` perl
> my %foo = "Tom" => 98, "Larry" => 100, "Bob" => "49";
("Tom" => 98, "Larry" => 100, "Bob" => "49").hash
```

``` perl
> my %bar = "Tom" => 98, "Larry" => 100, "Vivo" => 86
("Tom" => 98, "Larry" => 100, "Vivo" => 86).hash
```

``` perl
> %foo «+» %bar
("Tom" => 196, "Larry" => 200).hash
```

而：

``` perl
>  %foo »+« %bar;
("Tom" => 196, "Larry" => 200, "Bob" => 49, "Vivo" => 86).hash
```

得到两个键的并集（键值相加）

不对称的 hypers 也有用; 例如， 如果你说：

``` perl
    %outer »+» %inner;
```

只有在 %outer 中已经存在的 %inner 键才会出现在结果中.

``` perl
> my %inner = "a" => 11;
> my %outer = "a" => 9, "b" => 12;
> %outer »+» %inner # a => 20, b => 12
```

然而，

``` perl
%outer »+=« %inner;
```

假设你想让 %outer 拥有键的并集，累加键值

``` perl
> my %inner = "a" => 11;
> my %outer = "a" => 9, "b" => 12;
> %outer »+=« %inner;  # a => 20, b => 12
> say %outer           # a => 20, b => 12
> say %inner           # a => 11
```

注意， hypers 允诺你不必关心处理以怎样的顺序发生，只保证结果的结构和输入的形式保持一致。从系统角度也不能保证操作是并行化的。

高效的并行化要求某种程度的不带更多额外工作的工作分割，系统被允许平衡并行处理的惰性需求。

例如， 一个算法想把一个列表分成2个等长的子列表是不会起作用的， 如果你不得不提前计算好列表长度， 因为你不是总能计算出长度。可以采取各种方法：

按需切换要并行处理的群组， 或交错循环地使用一组 N 个核心的处理器，或任何东西。在该限制下， 一个简单、非并行、逐项的惰性实现就在 sepc 之中了，但是不太可能高效的使用多核。‘

不考虑性能要求，如果算法依赖于这些采用的方法， 那也是错误的。

## Reduction 操作符

任何中缀操作符（除了 non-associating 操作符）都可以在 term 位置处被方括号围住， 以创建使用使用该操作符进行换算的列表操作符：

``` perl
    [+] 1, 2, 3;      # 1 + 2 + 3 = 6
    my @a = (5,6);
    [*] @a;           # 5 * 6 = 30
```

对于所有的元操作符来说,  在 `metatoken` 里面是不允许有空格的.

换算操作符和列表前缀的优先级相同。 实际上， 换算操作符就是一个列表前缀，被当作一个操作符来调用。因此， 你可以以两种方式的任何一种来实现换算操作符。要么你也能写一个显式的列表操作符：

``` perl
    multi prefix:<[+]> (*@args) is default {
        my $accum = 0;
        while (@args) {
            $accum += @args.shift();
        }
        return $accum;
    }
```

或者你能让系统根据对应的中缀操作符为你自动生成一个：

``` perl
    &prefix:<[*]>  ::= &reduce.assuming(&infix:<*>, 1);
    &prefix:<[**]> ::= &reducerev.assuming(&infix:<**>);
```

如果换算操作符和中缀操作符的定义是独立的， 那换算操作符和该操作符的结合性要相同：

``` perl
    [-] 4, 3, 2;      # 4-3-2 = (4-3)-2 = -1
    [**] 4, 3, 2;     # 4**3**2 = 4**(3**2) = 262144
```

对于  list-associative 操作符（优先级表中的 X），实现必须把参数的 listiness 考虑在内； 即，如果重复地应用一个二元版本的操作符会产生错误的结果，那么它就不会被那样实现。 例如：

``` perl
    [^^] $a, $b, $c;  # means ($a ^^ $b ^^ $c), NOT (($a ^^ $b) ^^ $c)
```

对于 chain-associative 操作符（像 <）， 所有的参数被一块儿接收， 就像你显式地写出：

```
[<] 1, 3, 5;      # 1 < 3 < 5
```



对于列表中缀操作符， 输入列表不会被展平， 以至于多个 parcels 可以以逗号分割形式的参数传递进来：  

``` perl
  [X~] (1,2), <a b>;  # 1,2 X~ <a b>
```

如果给定的参数少于 2 个， 仍然会用给定的参数尝试分派， 并根据那个分派的接受者来处理少于 2 个参数的情况。 注意，默认的列表操作符签名是最通用的， 所以， 你被允许根据类型去定义不同的方式处理单个参数的情况：

``` perl
    multi prefix:<[foo]> (Int $x) { 42 }
    multi prefix:<[foo]> (Str $x) { fail "Can't foo a single Str" }
```

然而， 0 参数的情况不能使用这种方式定义， 因为没有类型信息用于分派。操作符要想指定一个同一值应该通过指定一个接收 0 个参数的 multi 变体来实现这：

``` perl
    multi prefix:<[foo]> () { 0 }
```

在内建操作符中，举个例子，  `[+]()` 返回 0 ， `[*]()` 返回 1 。


默认地， 如果有一个参数， 内建的换算操作符就返回那个参数。 然而， 这种默认对于像 `<` 那样返回类型和接收参数不同的操作符没有效果，所以这种类型的操作符重载了单个参数的情况来返回更有意义的东西。为了和链式语义保持一致， 所有的比较操作符都对于 1 个或 0 个参数返回 Bool::True。

你也可以搞一个逗号操作符的换算操作符。 这正是 `circumfix:<[ ]>` 匿名数组构建器的列表操作符形式：

``` perl
    [1,2,3]     # make new Array: 1,2,3
    [,] 1,2,3   #  与上相同
```

内置换算操作符返回下面的同一值：


``` perl
    [**]()      # 1     (arguably nonsensical)
    [*]()       # 1
    [/]()       # fail  (换算没有意义)
    [%]()       # fail  (换算没有意义)
    [x]()       # fail  (换算没有意义)
    [xx]()      # fail  (换算没有意义)
    [+&]()      # -1    (from +^0, the 2's complement in arbitrary precision)
    [+<]()      # fail  (换算没有意义)
    [+>]()      # fail  (换算没有意义)
    [~&]()      # fail  (sensical but 1's length indeterminate)
    [~<]()      # fail  (换算没有意义)
    [~>]()      # fail  (换算没有意义)
    [+]()       # 0
    [-]()       # 0
    [~]()       # ''
    [+|]()      # 0
    [+^]()      # 0
    [~|]()      # ''    (length indeterminate but 0's default)
    [~^]()      # ''    (length indeterminate but 0's default)
    [&]()       # all()
    [|]()       # any()
    [^]()       # one()
    [!==]()     # Bool::True    (also for 1 arg)
    [==]()      # Bool::True    (also for 1 arg)
    [before]()  # Bool::True    (also for 1 arg)
    [after]()   # Bool::True    (also for 1 arg)
    [<]()       # Bool::True    (also for 1 arg)
    [<=]()      # Bool::True    (also for 1 arg)
    [>]()       # Bool::True    (also for 1 arg)
    [>=]()      # Bool::True    (also for 1 arg)
    [~~]()      # Bool::True    (also for 1 arg)
    [!~~]()     # Bool::True    (also for 1 arg)
    [eq]()      # Bool::True    (also for 1 arg)
    [!eq]()     # Bool::True    (also for 1 arg)
    [lt]()      # Bool::True    (also for 1 arg)
    [le]()      # Bool::True    (also for 1 arg)
    [gt]()      # Bool::True    (also for 1 arg)
    [ge]()      # Bool::True    (also for 1 arg)
    [=:=]()     # Bool::True    (also for 1 arg)
    [!=:=]()    # Bool::True    (also for 1 arg)
    [===]()     # Bool::True    (also for 1 arg)
    [!===]()    # Bool::True    (also for 1 arg)
    [eqv]()     # Bool::True    (also for 1 arg)
    [!eqv]()    # Bool::True    (also for 1 arg)
    [&&]()      # Bool::True
    [||]()      # Bool::False
    [^^]()      # Bool::False
    [//]()      # Any
    [min]()     # +Inf
    [max]()     # -Inf
    [=]()       # Nil    (same for all assignment operators)
    [,]()       # []
    [Z]()       # []

```

```
[=] $x, @y, $z, 0
[+=] $x, @y, $z, 1
```

等价于：

```
$x = @y[0] = @y[1] = @y[2] ... @y[*-1] = $z = 0
$x += @y[0] += @y[1] += @y[2] ... @y[*-1] += $z += 1
```

而不是：

```
$x = @y = $z = 0;
$x += @y += $z += 1;
```

```
my :($b, $c);               # okay
sub foo :($a,$b) {...}      # okay
```

 `->` "pointy block" token 也引入签名, 但是这种情况你必须省略冒号和括号. 例如, 如果你在定义 loop block 的 "循环变量":

``` perl
    for @dogpound -> Dog $fido { ... }
```

```
$foo.bar.baz.bletch.whatever.attr[] = 1,2,3;
```

空的 [] 和 .[] 后缀操作符被解释为 0 维下标, 这返回整个数组, 不是作为一个一维的空切片, 返回空元素.  这同样适用于散列上的  {} 和 .{} , 还有  `<>`, `.<>`,` «»`, 和 `.«»`。
