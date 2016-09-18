---
title: Using Perl 6(一)
date: 2015-03-14 13:14
categories: Perl 6
tags:
  - Using Perl 6
  - 约束
comments: true
---

## 第一章 概要
---
 Comming soon!
## 第二章 基础
---

假设有一场乒乓球比赛，比赛结果以这种格式记录：
Player1 Player2 | 3:2
这意味着选手1与选手2的比分为3:2, 你需要一个脚本算出每位选手赢了几场比赛并且胜了几局。
输入数据（存储在一个叫做scores的文件中）像下面这样：
```perl6
 Beth Ana Charlie Dave
 Ana Dave        | 3:0
 Charlie Beth    | 3:1
 Ana Beth        | 2:3
 Dave Charlie    | 3:0
 Ana Charlie     | 3:1
 Beth Dave       | 0:3
```

 第一行是选手清单。随后每一行记录着比赛结果。

这里使用Perl6给出一种解决方案：

```perl6
 use v6;

 my $file = open 'scores';
 my @names = $file.get.words ;  #get方法读入一行，每调用一次get，读取一行
 # > @names.perl   #  Array.new("1", "Beth", "Ana", "Charlie", "Dave")
 my %matches;      # 赢得比赛次数
 my %sets;         # 赢得比赛局数

 for $file.lines -> $line {                       # .lines 是惰性的
     my ($pairing, $result) = $line.split(' | '); # 对剩下的每一行调用split操作
     my ($p1, $p2)          = $pairing.words;     # 提取选手1和选手2的名字
     my ($r1, $r2)          = $result.split(':'); # 提取比赛比分

     %sets{$p1} += $r1;  # 选手1赢得的比赛局数
     %sets{$p2} += $r2;  # 选手2赢得的比赛局数

     if $r1 > $r2 { # 如果每场比赛中，选手1赢的局数多于选手2，则选手1赢得的比赛数+1，反之选手2的+1
         %matches{$p1}++;
     } else {
         %matches{$p2}++;
     }
 }

 my @sorted = @names.sort( { %sets{$_} } ).sort({ %matches{$_} } ).reverse;

 for @sorted -> $n {
     say "$n has won %matches{$n} matches and %sets{$n} sets";
 }
```

输出如下：

```perl6
    Ana has won 2 matches and 8 sets
    Dave has won 2 matches and 6 sets
    Charlie has won 1 matches and 4 sets
    Beth has won 1 matches and 4 sets
```

每个 Perl 6程序应该以 use v6;作为开始，它告诉编译器程序期望的是哪个版本的Perl。

在Perl6中，一个变量名以一个魔符打头，这个魔符是一个非字母数字符号，诸如$,@,%或者 &,还有更少见的双冒号 ::
内置函数 open 打开了一个名叫 scores 的文件，并返回一个文件句柄，即一个代表该文件的对象。赋值符号=将句柄赋值给左边的变量，这意味着 $file 现在存储着该文件句柄。


```perl6
 my @names = $file.get.words;
```

 上边这句的右侧对存储在 $file 中的文件句柄调用了 get 方法， get 方法从文件中读取并返回一行，并去掉行的末尾。. words  也是一个方法，用于从get 方法返回的字符串上。.words 方法将它的组件--它操作的字符串，分解成一组单词，这里即意味着不含空格的字符串。它把单个字符串 'Beth Ana Charlie Dave' 转换成一组字符串 'Beth', 'Ana', 'Charlie', 'Dave'.最后，这组字符串存储在数组@names中。

```perl6
 my %matches;
 my %sets;
```

在比分计数程序中，%matches 存储每位选手赢得的比赛数。 %sets 存储每位选手赢得的比赛局数。

```perl6
 for $file.lines -> $line {
 ...
 }
```

for循环中 $file.lines 产生一组从文件 scores 读取的行，从上次 $file.lines 离开的地方开始，一直到文件末尾结束。
在第一次循环中， $line 会包含字符串 `Ana Dave | 3:0;` 在第二次循环中，$line 会包含 `Charlie Beth | 3:1`,以此类推。

```perl6
 my ($pairing, $result) = $line.split(' | ');
```

split此处是一个方法，字符串 '|' 是它的参数。

第一次循环结束：

    Variable       Contents
    $line           'Ana Dave | 3:0'
    $pairing        'Ana Dave'
    $result         '3:0'
    $p1             'Ana'
    $p2             'Dave'
    $r1              '3'
    $r2              '0'



```perl6
 my @sorted = @names.sort({ %sets{$_} }).sort({ %matches{$_} }).reverse;
```

这一句是排序，先按比赛局数多少排序，再按赢得的比赛数排序，然后反转。
打印选手名字的时候以胜负次序排序，代码必须使用选手的分数，而非他们的名字来进行排序。sort 方法的参数是一个代码块，用于将数组元素（选手的名字）转换成用于排序的数据。数组的元素通过变量 $_ 传递到代码块中。

最简单的使用分数排序选手的方法应该是


```perl6
 @names.sort( { %matches{$_} } )
```

这是通过使用赢得比赛的次数来进行排序。然而，Ana和Dave都赢了两场比赛。还需要比较谁赢的的比赛局数多，才能决定比赛的排名。

在双引号括起的字符串中，标量和花括号中的变量能进行变量插值。


```perl6
my $names = 'things';
say 'Do not call me $names';   # Do not call me $names
say "Do not call me $names"; # Do not call me things
```

花括号中的数组进行插值后会变成用空格分隔的条目。花括号中的散列插值后每个散列键值对单独成为一行，每行包含一个健，随后是一个tab符号，然后是键值，最后是一个新行符。


```perl6
TODO: explain <...> quote-words
 say "Math: { 1 + 2 }"     # Math: 3
 my @people = <Luke Matthew Mark>;
 say "The synoptics are: {@people}" # The synoptics are: Luke Matthew Mark

 say "{%sets}"; # From the table tennis tournament

 # Charlie 4
 # Dave 6
 # Ana 8
 # Beth 4
```

当数组和散列变量直接出现在双引号字符串中（并且不在花括号{}里），它们只在它们的名字后跟着一个 postcircumfix ----一对括号，后面跟着语句时才会进行插值。在变量名和后置环缀之间进行方法调用也是可以的（ 例如  @flavours.sort() ）


```perl6
 my @flavours = <vanilla peach>;

 say "we have @flavours";    # we have @flavours ，这里没进行插值
 say "we have @flavours[0]"; # we have vanilla，后置环缀，变量名字后面跟着一对儿括号
 # so-called "Zen slice"
 say "we have @flavours[]";  # we have vanilla peach

 # 以后置环缀结尾的方法调用
 say "we have @flavours.sort()"; # we have peach vanilla

 # 链式方法调用:
 say "we have @flavours.sort.join(', ')";
 # we have peach, vanilla
```

### 练习：
---

例子中的第一行选手的名字是多余的，你可以在参加比赛的选手中找出所有选手的名字！ 如果例子中的第一行被省略了，你如何更改程序？提示：%hash.keys 返回散列 %hash中的所有键。

答案: 移除此行：

```perl6
my @names = $file.get.words;
```

并且将

```perl6
my @sorted = @names.sort({ %sets{$_} }).sort({ %matches{$_} }).reverse;
```

变成:

```perl6
my @sorted = %sets.keys.sort({ %sets{$_} }).sort({ %matches{$_} }).reverse;
```

. 除了移除冗余，你也可以用它来提醒我们，如果一个选手没有在第一行的名字清单中被提到，例如因为输入错误，你该怎样修改你的程序？

答案:引入另外一个散列，合法选手的名字作为键，当读取选手名字的时候查找该散列：


```perl6
 my @names = $file.get.split(' ');
 my %legitimate-players;
 for @names -> $n {      #  -> 两侧要有空格
     %legitimate-players{$n} = 1;
 }
```

 ...

```perl6
 for $file.lines -> $line {
 my ($pairing, $result) = $line.split(' | ');
 my ($p1, $p2)          = $pairing.split(' ');
    for $p1, $p2 -> $p {
        if !%legitimate-players{$p} {
            say "Warning: '$p' is not on our list!";
         }
     }

 ...
 }
```

## 第三章 操作符
---


```perl6
use v6;

my @scores = 'Ana' => 8, 'Dave' => 6, 'Charlie' => 4, 'Beth' => 4;
my $screen-width = 30;

my $label-area-width = 1 + [max] @scores».key».chars;
my $max-score = [max] @scores».value;
my $unit = ($screen-width - $label-area-width) / $max-score;
my $format = '%- ' ~ $label-area-width ~ "s%s\n";

for @scores {
    printf $format, .key, 'X' x ($unit * .value);
}
```

在这个例子中，我们计算一下每位选手在竞标赛中赢得比赛的局数。

```perl6
 my @scores = 'Ana' => 8, 'Dave' => 6, 'Charlie' => 4, 'Beth' => 4;  
```

这一句局包含了三个不同的操作符=和 =>和 ,
以字符串连接操作符~为例， $string ~= "text" 等价于 $string = $string ~"text".

=> 操作符（大键号）创建了一个键值对对象，一个键值对存储着键和值；键在 => 操作符的左侧，值在右侧。这个操作符有一个特殊的特性：编译器会把 =>操作符左侧的任何裸标识符解释为一个字符串。你也可以这样写：


```perl6
my @scores = Ana => 8, Dave => 6, Charlie => 4, Beth => 4;
```

最后逗号操作符 , 构建了一个对象序列，在该情况下，所谓的对象就是键值对。

这三个操作符都是中缀操作符，这意味着它在两个条目之间。

一个项前面可以有0个或多个前缀操作符，所以你可以写比如 4 + -5。+ 号（一个中缀操作符）的后面，编译器期望一个项，为了将 - 号解释为项 5 的一个前缀。


```perl6
my $label-area-width = 1 + [max] @scores».key».chars;
```

» 是一个特殊的符号，打印不出来可以用两个大于号>>代替。
中缀操作符 max 返回两个值中的较大者，所以 2 max 3 返回 3。方括号包裹着一个中缀操作符让Perl将该中缀操作符应用到列表中的元素之间。[max] 1,5,3,7 和 1 max 5 max 3 max 7 一样，结果都为7.

同样地，[+]用来计算列表元素的和，[*]用来计算列表元素的积，[<=]用来检查一个列表的值是否按递增排序。

```perl6
@scores».key».chars
my @scores = Ana => 8, Dave => 6, Charlie => 4, Beth => 4;
```


```perl6
Ana     8 Dave  6 Charlie       4 Beth  4
```


```perl6
@scores.key
```


```perl6
Method 'key' not found for invocant of class 'Array'
```


```perl6
 @scores>>.key
```


```perl6
Ana Dave Charlie Beth
```

就像@variable.method 在@variable上调用一个方法一样，@array».method 对@array中的每一项调用method方法，并且返回一个返回值的列表。即@scores>>.key返回一个列表。

```perl6
 @scores>>.key>>.chars  #每个名字含有几个字符
```


```perl6
4 7 4
```

表达式 [max] @scores».key».chars 给出(3,4,7,4)中的最大值。它与下面的表达式相同：


```perl6
 @scores[0].key.chars
 max @scores[1].key.chars
 max @scores[2].key.chars
 max ...
```


```perl6
 @scores[0]
```


```perl6
"Ana" => 8
```


```perl6
 @scores[0].key
```


```perl6
Ana
```


```perl6
 my $format = '%- ' ~ $label-area-width ~ "s%s\n";
 for @scores {
     printf $format, .key, 'X' x ($unit * .value);
 }
```

定义一个格式，%-表示左对齐，~是字符串连接操作符.for循环中，@scores中的每一项被绑定给特殊变量$_, .key是每项的键，即名字， .value是每项的键值，即得分。小 x 是字符串重复操作符。

### 关于优先级的的一句话
---


```perl6
 my @scores = 'Ana' => 8, 'Dave' => 6, 'Charlie' => 4, 'Beth' => 4;
```

等号右侧产生一个列表（因为逗号，操作符），这个列表由对儿组成（因为 =>）,并且结果赋值给数组变量。
在Perl5中会这样解释:

```perl6
 (my @scores = 'Ana') => 8, 'Dave' => 6, 'Charlie' => 4, 'Beth' => 4;
```

以至于数组@scores中只有一个项，表达式的其余部分被计算后丢弃。

优先级规则控制着编译器如何解释这一行。Perl 6的优先级规则申明 中缀操作符 => 比 , 中缀操作符对于参数的绑定更紧，而逗号操作符比 等号赋值操作符绑定的更紧。

实际上有两种不同优先级的赋值操作符。当赋值操作符右侧是一个标量时，使用较紧优先级的项赋值操作符，否则使用较松优先级的列表赋值操作符。（如同螺丝的松紧）
比较 $a = 1, $b = 2 和@a = 1, 2，前者是在一个列表中赋值给两个变量，后者是将含有两个项的一个列表赋值给一个变量。


```perl6
 say 5 - 7 / 2; # 5 - 3.5 = 1.5
 say (5 - 7) / 2; # (-2) / 2 = -1
```

Perl 6 中的优先级可以用圆括号改变，但是如果圆括号直接跟在标识符的后面而不加空格的话，则会被解释为参数列表。例如：


```perl6
 say(5 - 7) / 2; # -2
```

只打印出了 5-7 的值。

              优先级表
    Example             Name
    (), 42.5			(tightest precedence)
    42.rand			    term
    $x++			    method calls and postcircumĕxes
    $x**2			    autoincrement and autodecrement
    ?$x, !$x			exponentiation operator
    +$x, ~$x			boolean preĕx
    2*3, 7/5			preĕx context operators
    1+2, 7-5			multiplicative inĕx operators
    $a x 3			    additive inĕx operators
    $x ~".nn"			replication operators
    1&2			        string concatenation
    1|2			        junctive AND
    abs $x			    junctive OR
    $x cmp 3			named unary preĕx
    $x == 3			    non-chaining binary operators
    $x && $y			chaining binary operators
    $x || $y			tight AND inĕx
    $x > 0 ?? 1 !! -1	tight OR inĕx
    $x = 1			    conditional operator
    not $x			    item assignment
    1, 2			    loose unary preĕx
    1, 2 Z @a			comma
    @a = 1, 2			list inĕx
    $x and say "Yes"	list preĕx, list assignment
    $x or die "No"		loose AND inĕx
    ;			        loose OR inĕx
    			        statement terminator
    			        (loosest precedence)




### 比较和智能匹配
---


```perl6
 my @a = 1, 2, 3;
 my @b = 1, 2, 3;
 say @a === @a; # Bool::True
 say @a === @b; # Bool::False
 # these use identity for value
 say 3 === 3 # Bool::True
 say 'a' === 'a'; # Bool::True
 my $a = 'a';
 say $a === 'a'; # Bool::True
```

    @b===@a

False
    @a eqv @b

True
    '2' eqv 2

False

只有当两个对象有相同的类型和相同的结构时， eqv 操作符才返回 True。在前面定义的例子中，@a  eqv  @b 结果为 True， 因为 @a 和 @b 各自包含相同的值，另一方面， '2' eqv 2 返回 'False' ,因为一个参数是字符串，另一个是整数，类型不相同。

#### 数字比较
---

使用 == 中缀操作符查看两个对象是否有相同的数字值。如果某个对象不是数字，Perl 会在比较之前尽力使其数字化。如果没有更好的方式将对象转换为数字，Perl 会使用默认的数字 0 。

```perl6
 say 1 == 1.0;  # Bool::True
 say 1 == '1';  # Bool::True
 say 1 == '2';  # Bool::False
 say 3 == '3b'; # fails
```

跟数字比较相关的还有 <,<=,>,>= 。如果两个对象的数字值不同，使用 != 会返回 True 。

如果你将数组或列表作为数字，它会计算列表中项的个数。

```perl6
my @colors = <red blue green>;
 if @colors == 3 {
     say "It's true, @colors contains 3 items";
 }
```

#### 字符串比较
---

Perl 6 中使用 eq 比较字符串，必要时会将其参数转换为字符串。


```perl6
 if $greeting eq 'hello' {
    say 'welcome';
 }
```

Table 3.2: Operators and Comparisons

```perl6
 数字比较	字符串比较	意思
 ------------------------------------------
 ==	      eq	        等于
 !=	      ne	        不等于
 !==	  !eq	        不等于
 <	      lt	        小于
 <=	      le	        小于或等于
 >	      gt	        大于
 >=	      ge	        大于或等于
```

例如，'a' lt 'b' 为 true，'a' lt 'aa' 也为 true。 != 是 !==的便捷形式，它实际是 ! 元操作符加在 中缀操作符 ==之前。同样地， ne 和 !eq 是一样的。

三路操作符

三路操作符有两个操作数，如果左侧较小，返回 Order::Increase ，两侧相等则返回 Order::Same，如果右侧较小则返回 Order::Decrease。对于数字使用 三路操作符 <=> ,对于字符串，使用三路操作符 leg （取自 lesser，equal，greater）。中缀操作符 cmp 是一个对类型敏感的三路操作符，它像 <=> 一样比较数字，像 leg 一样比较字符串，（举例来说）并且比较键值对儿时，先比较键，如果键相同再比较键值：

```perl6
 say 10 <=> 5;     # +1
 say 10 leg 5;     # because '1' lt '5'
 say 'ab' leg 'a'; # +1, lexicographic comparison
```

三路操作符的典型用处就是用在排序中。列表中的.sort 方法能使用一个含有两个值的块或一个函数，比较它们，并返回一个小于，等于或大于 0 的值。 sort  方法根据该返回值进行排序：

```perl6
 say ~<abstract Concrete>.sort;
 # output: Concrete abstract

 say ~<abstract Concrete>.sort:  -> $a, $b { uc($a) leg uc($b) };
 # output: abstract Concrete
```

默认的，比较是大小写敏感的，通过比较它们的大写变形，而不是比较它们的值，这个例子使用了大小写敏感排序。

#### 智能匹配
---

使用 ~~ 做正确的事情。


```perl6
 if $pints-drunk ~~ 8 {
    say "Go home, you've had enough!";
 }

 if $country ~~ 'Sweden' {
     say "Meatballs with lingonberries and potato moose, please."
 }

 unless $group-size ~~ 2..4 {
     say "You must have between 2 and 4 people to book this tour.";
 }
```

智能匹配总是根据 ~~右侧值的类型来决定使用哪种比较。上个例子中，比较的是数字、字符串和范围。
智能匹配的工作方式 $answer ~~ 42 等价于 42.ACCPETS( $answer ).对 ~~ 操作符右侧的操作数调用 ACCEPTS 方法，并将左操作数作为参数传入。



## 第四章 子例程和签名
---


一个子例程就是一段执行特殊任务的代码片段。它可以对提供的数据（`实参`）操作，并产生结果（返回值）。子例程的签名是它`所含的参数`和它产生的`返回值`的描述。从某一意义上来说，第三章描述的操作符也是Perl 6用特殊方式解释的子例程。

### 申明子例程
---

 一个子例程申明由几部分组成。首先， `sub `表明你在申明一个子例程，然后是可选的子例程的名称和`可选的签名`。子例程的主体是一个用花括号扩起来的代码块。
默认的，子例程是本地作用域的，就像任何使用 `my` 申明的变量一样。这意味着，一个子例程只能在它被申明的作用域内被调用。使用 `our` 来申明子例程可以使其在`当前包`中可见。


```perl6
 {
 our sub eat() {
     say "om nom nom";
 }

 sub drink() {
     say "glug glug";
 }
 }
 our &eat; # makes the package-scoped sub eat available in this lexical scope

 eat(); # om nom nom
 drink(); # 失败, can't drink outside of the block
```

our 也能让子例程从包或模块的外部是可见的：

```perl6
 module EatAndDrink {
     our sub eat() {
     say "om nom nom";
 }

 sub drink() {
     say "glug glug";
 }
 }
 EatAndDrink::eat(); # om nom nom
 EatAndDrink::drink(); # fails, not declared with "our"
 ```

你也可以`导出`一个子例程，让它在另外的作用域内可见。

```perl6
 # in file Math/Trivial.pm
 # TODO: find a better example
 # TODO: explain modules, search paths
 module Math::Trivial {
     sub double($x) is export {
     return 2 * $x;
 }
```

然后在其它程序或模块中你可以这样写:

```perl6
 use Math::Trivial; # imports sub double
 say double(21); # 21 is only half the truth
```

Perl 6的子例程都是对象。你可以将它们随意传递并存储在数据结构中。编程语言设计者常常将它们称之为first-class 子例程；它们就像数组和散列一样作为语言的基础。

First-class 子例程能帮助你解决复杂的问题。例如，为了做出一个微型的ASCII艺术舞蹈图，你可能要建立一个散列，键是舞蹈动作的名称，键值是匿名散列。假使使用者能键入一系列舞蹈动作（可能是站在舞蹈平台上或其它外部输入设备）。 你怎么保持一个变量清单中都是合法的行为，允许使用者输入，并限制输入是一系列安全的行为呢？

```perl6
 my %moves =
 hands-over-head       => sub { say '/o\ '  },
 bird-arms             => sub { say '|/o\| '},
 left                  => sub { say '>o '   },
 right                 => sub { say 'o< '   },
 arms-up               => sub { say '\o/ '  };

 my @awesome-dance = <arms-up bird-arms right hands-over-head>;

 for @awesome-dance -> $move {
     %moves{$move}.();  # 在散列上调用方法
 }
```

    outputs:
     \o/
    |/o\|
      o<
     /o\.

### Adding Signatures
---
**添加签名**

子例程的签名执行两个任务。首先，它申明哪个调用者可能或必须将参数传递给子例程。第二，它申明子例程中的变量被绑定到哪些参数上。这些变量叫做参数。Perl 6的签名更深入，它们允许你`限制参数的类型`，值和参数的定义，并准确匹配复杂数据结构的某一部分。此外，它们也允许你显式地指定子例程返回值的类型。

#### 基础
---

签名最简单的形式是，绑定到输入参数上的用逗号分隔的一列变量的名字。

```perl6
 sub order-beer($type, $pints) {
    say ($pints == 1 ?? 'A pint' !! "$pints pints") ~ " of $type, please."
 }

 order-beer('Hobgoblin', 1);    # A pint of Hobgoblin, please.
 order-beer('Zlatý Bažant', 3);  # 3 pints of Zlatý Bažant, please.
```

这里使用的关系绑定而非赋值就是签名。默认地，在Perl6 中，子例程中引用到传入参数的签名的变量是`只读`的。这意味着你`不能`从子例程内部`修改`它们。
如果只读绑定太受限制了，你可以将 `is rw` (rw是read/write的缩写) 特性应用到参数上以降低这种限制。这个特性说明参数是可读可写的，这允许你从子例程内部修改参数。使用的时候必须小心，因为它会`修改`传入的原始对象。如果你试图传入一个字面值，一个常量，或其它类型的不可变对象到一个有 `is rw`  特性的参数中，绑定会在调用时失败并抛出异常:
 ```perl6
 sub make-it-more-so($it is rw) {
     $it ~= substr($it, $it.chars - 1) x 5;
 }

 my $happy = "yay!";
 make-it-more-so($happy);
 say $happy; # yay!!!!!!   # 原始传入对象被修改了
 make-it-more-so("uh-oh"); # 失败，不能修改一个常量
 ```

如果你想将参数的本地副本用在子例程内部而不改变调用者的变量，----使用 `is copy` 特性：
 ```perl6
 sub say-it-one-higher($it is copy) {
     $it++;
     say $it;
 }

 my $unanswer = 41;
 say-it-one-higher($unanswer); # 42
 say-it-one-higher(41); # 42
 $unanswer;  #41
 ```

在诸如C/C++ and Scheme等其它类型的编程语言中,这种广为人知的求值策略就是“按值传递”。当使用 `is copy `特性时，只有本地副本被赋值。其它任何传递给子例程的参数在调用者的作用域内保持不变。（一个不可变对象是当这个对象被创建后，它的状态不会改变，作为比较，一个可变对象的状态在创建后是会被改变的）

#### 传递数组、散列和代码
---

一个变量的魔符表明它的本意用途。在签名中，变量的魔符也起着限制传入的参数类型的作用。例如， @ 符号检查传入的对象行使位置角色（一个角色包含像数组和列表的类型）。如果传递的东西不能匹配这样的限制，会引起调用失败：

```perl6
 sub shout-them(@words) {
     for @words -> $w {
         print uc("$w ");
     }
 }

 my @last_words = <do not want>;

 shout-them(@last_words); # DO NOT WANT
 shout-them('help'); # Fails; a string is not Positional  字符串不是位置参数
 ```

类似地， % 符号表明调用者必须传递一个行使关系角色的对象；即允许通过<...>或{...} 进行索引的东西。 & 符号要求调用者传递一个诸如匿名散列之类的行使能调用的角色的对象。在那种情况下，你也可以不用 & 符号调用可调用的参数：

```perl6
 sub do-it-lots(&it, $how-many-times) {
     for 1..$how-many-times {
         it();
     }
 }

 do-it-lots(sub { say "Eating a stroopwafel" }, 10);   #此处是一个匿名子例程
 ```

标量使用 $ 符号，并表明没有限制。什么都可以绑定在它上面，即使它使用另外的符号绑定到一个对象上。

#### 插值、数组和散列
---

有时你想从数组中填充占位参数。你可以通过在数组前添加一个垂直竖条或管道字符 ( `|` ): eat(|@food)    而不是写作eat(@food[0],@food[1], @food[2], ...) 等将它们吸进参数列表( | 像不像一个吸管, ^_^)。

同样地，你可以将散列插值进具名参数:

```perl6
 sub order-shrimps($count, :$from) {
     say "I'd like $count pieces of shrimp from the $from, please";
 }

 my %user-preferences = from => 'Northern Sea';

 order-shrimps(3, |%user-preferences);
 # I'd like 3 pieces of shrimp from the Northern Sea, please
```

#### 可选参数
---

为使参数可选，要么给签名的参数赋值为默认值：

```perl6
 sub order-steak($how = 'medium') {
     say "I'd like a steak, $how";
 }

 order-steak();
 order-steak('well done');
```

或者在参数名字的后面添加一个问号(?):

```perl6
 sub order-burger($type, $side?) {
     say "I'd like a $type burger" ~
     ( defined($side) ?? " with a side of $side" !! "" );
 }

 order-burger("triple bacon", "deep fried onion rings");
```

如果没有参数被传递，参数会被绑定成一个未定义的值。 defined(...) 函数用来检查是否有值。

#### 强制参数
---

默认地，位置参数是必不可少的。然而，你可以通过在参数后面追加一个感叹号来显式地指定该参数是必须的：

```perl6
 sub order-drink($size, $flavor!) {
     say "$size $flavor, coming right up!";
 }

 order-drink('Large', 'Mountain Dew'); # OK
 order-drink('Small'); # Error
```

#### 具名实参和形参
---

* arguments  实参
* parameters 形参

当一个子例程有很多参数时，调用者很难记清传递参数的顺序。这种情况下，通过`名字`传递参数往往更容易。这样，参数出现的顺序就无关紧要了:

```perl6
 sub order-beer($type, $pints) {
     say ($pints == 1 ?? 'A pint' !! "$pints pints") ~ " of $type, please."
 }

 order-beer(type => 'Hobgoblin', pints => 1);
 # A pint of Hobgoblin, please.

 order-beer(pints => 3, type => 'Zlatý Bažant');
 # 3 pints of Zlatý Bažant, please.
```

你也可以指定参数只能按名字被传递（这意味着它不允许按位置传递）。这样的话，在参数名字前加一个`冒号`：

```perl6
 sub order-shrimps($count, :$from = 'Northern Sea') {
     say "I'd like $count pieces of shrimp from the $from, please";
 }

 order-shrimps(6);    # takes 'Northern Sea'
 order-shrimps(4, from => 'Atlantic Ocean');
 order-shrimps(22, 'Mediterranean Sea');   # 不允许, :$from is named only
```

不像位置参数，命名参数默认是可选的。在命名参数后面追加一个 ! 号使命名参数强制性存在。

```perl6
 sub design-ice-cream-mixture($base = 'Vanilla', :$name!) {
     say "Creating a new recipe named $name!"
 }

 design-ice-cream-mixture(name => 'Plain');
 design-ice-cream-mixture(base => 'Strawberry chip'); # 错误,没有指定 $name
```

**重命名参数**

因为按名字传递实参给形参是合理的, 形参的名字应该应该作为子例程公共 API 的一部分被考虑在内. 小心地挑选它们吧! 有时候, 使用一个名字暴露形参而使用另外一个名字绑定到变量会很方便:


```perl6
 sub announce-time(:dinner($supper) = '8pm') {
     say "We eat dinner at $supper";
 }

 announce-time(dinner => '9pm'); # We eat dinner at 9pm
```

参数可以有多个名字，如果你的用户有些是英国人，有些是美国人，你可能这样写：

```perl6
 sub paint-rectangle(
     :$x = 0,
     :$y = 0,
     :$width = 100,
     :$height = 50,
     :color(:colour($c))) {

     # print a piece of SVG that represents a rectangle
     say qq[<rect x="$x" y="$y" width="$width" height="$height" >]
 }

 # both calls work the same
 paint-rectangle :color<Blue>;
 paint-rectangle :colour<Blue>;

 # of course you can still fill the other options
 paint-rectangle :width(30), :height(10), :colour<Blue>;
```

**可选的命名参数语法**

命名变量通常是成对的（键值对）。写一个 `Pairs` 有多种方式。各种方法的不同之处就是清晰性，因为每种选择提供不同的引述机制。下面的三种调用是一样的意思：

```perl6
 announce-time(dinner => '9pm');
 announce-time(:dinner('9pm'));
 announce-time(:dinner<9pm>);
```

如果传递的是布尔值，你可以省略键值对的键值：

```perl6
 toggle-blender( :enabled); # enables the blender 开启果汁机
 toggle-blender(:!enabled); # disables the blender 关闭果汁机
```

形如 `:name` 但不带值的命名参数有一个隐式的布尔真值 `Bool::True`. 它的对立形式是 `:!name` , 其值是隐式的布尔假值 `Bool::false`. 如果你使用变量创建了一个 `pair`, 你可以将变量名作为 `pair` 的键复用.


```perl6
 my $dinner = '9pm';
 announce-dinner :$dinner; # same as dinner => $dinner;
```

                            Pair forms and their meanings.

        Shorthand      Long form                          Description
    :allowed           allowed => Bool::True               Boolean flag
    :!allowed          allowed => Bool::False              Boolean flag
    :bev<tea coffee>   bev => ('tea', 'coffee')            List
    :times[1, 3]       times => [1, 3]                     Array
    :opts{ a => 2 }    opts => { a => 2 }                  Hash
    :$var              var => $var Scalar                  variable
    :@var              var => @var Array                   variable
    :%var              var => %var Hash                    variable
    :&var              vaf => &var Callable/ Subroutine    variable


你可以使用在任何可以使用 Pair 对象的上下文使用表中的任意一种形式. 例如, 生成散列:


```perl6
 # TODO: better example
 my $black = 12;
 my %color-popularities = :$black, :blue(8), red => 18, :white<0>;

 # 与此相同：
 # my %color-popularities =
 # black => 12,
 # blue => 8,
 # red => 18,
 # white => 0;
```

最后, 通过位置而非名字传递一个已存在的 Pair 对象到子例程中, 要么把它放在圆括号中 ( 就像 (:$thing) ), 或者使用 => 操作符引起左侧的字符串: "thing" => $thing.

**参数的顺序**

当位置参数和命名参数都出现在签名中时，所有的位置参数都要出现在命名参数之前：


```perl6
 sub mix(@ingredients, :$name)    { ... } # OK
 sub notmix(:$name, @ingredients) { ... } # Error
```

必须的位置参数要在可选的位置参数之前。然而，命名参数没有这种限制。

```perl6
 sub copy-machine($amount, $size = 'A4', :$color!, :$quality) { ... } # OK
 sub fax-machine($amount = 1, $number) { ... } # Error
```

#### Slurpy 参数
---

有时候，你会希望让子例程接受任何数量的参数，并且将所有这些参数收集到一个数组中。为了达到这个目的，给签名添加一个数组参数，就是在数组前添加一个 * 号前缀：


```perl6
 sub shout-them(*@words) {
     for @words -> $w {
         print uc("$w ");
     }
 }

  #现在你可以传递项
 shout-them('go'); # GO
 shout-them('go', 'home'); # GO HOME
```

除了集合所有的值之外，slurpy 参数会展平任何它接收到的数组，最后你只会得到一个展平的列表，因此：

```perl6
 my @words = ('go', 'home');
 shout-them(@words);
```

会导致 `*@words` 参数有两个字符串元素，而非只有单个数组元素。

你可以选择将某些参数捕获到位置参数中，并让其它参数被吸进数组参数里。这种情况下， `slupy` 应该放到最后。相似地， `*%hash` slurps 所有剩下的未绑定的命名参数到散列 %hash中。`Slurpy` 数组和散列允许你传递所有的位置参数和命名参数到另一个子例程中。


```perl6

 sub debug-wrapper(&code, *@positional, *%named) {
     warn "Calling '&code.name()' with arguments "
     ~ "@positional.perl(), %named.perl()\n";
     code(|@positional, |%named);
     warn "... back from '&code.name()'\n";
 }

 debug-wrapper(&order-shrimps, 4, from => 'Atlantic Ocean');

```

### 返回结果
---

子例程也能返回值。之前本章中的 ASCII 艺术舞蹈例子会更简单当每个子例程返回一个新字符串：


```perl6

 my %moves =
 hands-over-head => sub { return '/o\ '   },
 bird-arms       => sub { return '|/o\| ' },
 left            => sub { return '>o '    },
 right           => sub { return 'o< '    },
 arms-up         => sub { return '\o/ '   };

 my @awesome-dance = <arms-up bird-arms right hands-over-head>;

 for @awesome-dance -> $move {
     print %moves{$move}.();
 }

 print "\n";

 ```

子例程也能返回多个值（译者注：那不就是返回一个列表嘛）：


```perl6

 sub menu {
     if rand < 0.5 {
         return ('fish', 'white wine')
     } else {
         return ('steak', 'red wine');
     }
 }

 my ($food, $beverage) = menu();

```

 如果你把 return 语句排除在外，则在子例程内部运行的最后一个语句产生的值被返回。这意味着前一个例子可以简化为：


```perl6

 sub menu {
     if rand < 0.5 {
         'fish', 'white wine'
     } else {
         'steak', 'red wine';
     }
 }

 my ($food, $beverage) = menu();

```

记得：当子例程中的控制流极其复杂时，添加一个显式的 return 会让代码更清晰，所以 return 还是加上的好。
return 另外的副作用就是执行后立即退出子例程：


```perl6

 sub create-world(*%characteristics) {
     my $world = World.new(%characteristics);
     return $world if %characteristics<temporary>;

     save-world($world);
 }

```

...并且你最好别放错你的新单词 $word 如果它是临时的。因为这是你要获取的仅有的一个。


### 返回值的类型
---

像其它现代语言一样，Perl 6 允许你显式地指定子例程返回值的类型。这允许你限制从子例程中返回的值的类型。使用 returns 特性可以做到这样：


```perl6
 sub double-up($i) returns Int {
     return $i * 2;
 }

 my Int $ultimate-answer = double-up(21);  # 42
```

 当然，使用这个 `returns` 特性是可选的

### Working With Types
---

很多子例程不能完整意义上使用任意参数工作，但是要求参数支持确定的方法或有其它属性。这种情况下，限制参数类型就有意义了，诸如传递不正确值作为参数，当调用子例程时，这会引起Perl 发出错误，或者甚至在编译时，如果编译器足够聪明来捕捉错误。

#### 基本类型
---
最简单的限制子例程接收可能的值的方法是在参数前写上类型名。例如，一个子例程对其参数执行数值计算，这要求它的参数类型是 Numeric：


```perl6
 sub mean(Numeric $a, Numeric $b) {
    return ($a + $b) / 2;
 }

 say mean 2.5, 1.5;
 say mean 'some', 'strings';
```

 产生输出：

```perl6
Nominal type check failed for parameter '$a';
expected Numeric but got Str instead
```

nominal 类型是一个人实际类型的名字，这里是 Numeric。
如果多个参数有类型限制，每个参数必须填充它绑定的参数限制的类型


#### 添加限制
---

有时，类型的名字不足以描述参数的要求。这种情况下，你可能使用 where 代码块添加一个额外的限制：

```perl6
 sub circle-radius-from-area(Real $area where { $area >= 0 }) {
     ($area / pi).sqrt
 }

 say circle-radius-from-area(3); # OK
 say circle-radius-from-area(-3); # Error
```

 因为这种计算只对非负面积值有意义，该子例程的参数包含了一个限制，对于非负值它会返回真。如果这个限制返回一个假的值，类型检查会失败，当有些东西调用该子例程时。

where 之后的代码块是可选的。Perl 通过通过智能匹配 where 后面的参数来执行检查。
例如，它可能接受在某一确定范围中的参数：

```perl6
 sub set-volume(Numeric $volume where 0..11) {
     say "Turning it up to $volume";
 }
```

或者你可以将参数限制为散列的键：

```perl6
 my %in-stock = 'Staropramen' => 8, 'Mori' => 5, 'La Trappe' => 9;

 sub order-beer(Str $name where %in-stock) {
       say "Here's your $name";
       %in-stock{$name}--;
       if %in-stock{$name} == 0 {
           say "OH NO! That was the last $name, folks! :'(";
           %in-stock.delete($name);
      }
 }
```

### 抽象参数和具体参数
---

下面检测变量是否定义。
例如，下面是Perl5 代码:

```perl6
 sub foo {
     my $arg = shift;
     die "Argument is undefined" unless defined $arg;

     # Do something
 }
 ```

在Perl 6 中这样写:

```perl  
 sub foo(Int:D $arg) {
     # Do something
 }
```

留意附加在参数类型后面的 :D 笑脸。这个动词表明给定的参数必须被绑定到一个具体的对象上。如果不是的话，会抛出一个运行时异常。这就是为什么它那么高兴！作为对比， 动词 :U 用于表明该参数需要一个未定义的或抽象的对象。此外， 动词:_ 允许定义或未定义的值。实际上，使用 :_ 有点多余。

最后，动词 :T 能用于表明参数只能是类型对象，例如

```perl6
 sub say-foobar(Int:T $arg) {
     say 'FOOBAR!';
 }

 say-foobar(Int);
 # FOOBAR!
```
