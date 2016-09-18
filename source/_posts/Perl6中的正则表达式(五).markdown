---
title:  Perl 6 中的正则表达式(五)
date: 2015-04-28
tags: 正则表达式
categories: Perl 6
comments: true
---

# 从匹配中返回值

# Match 对象

- 成功的匹配总是返回一个 `Match` 对象, 这个对象通常也被放进 `$/` 中, (具名 `regex`, `token`, 或 `rule` 是一个子例程, 因此会声明它们自己的本地 `$/` 变量, 它通常指 rule 中最近一次的 submatch, 如果有的话)。当前的匹配状态被保存到 regex 的 `$¢` 变量中, 当匹配结束时它最终会被绑定到用户的 `$/`变量中

不成功的匹配会返回 Nil (并把 `$/` 设置为 Nil, 如果匹配已经设置了 `$/`的话)

- 名义上, Match 对象包含一个布尔的成功值, 一个`有序的`子匹配对象(submatch objects)`数组`, 一个`具名的`子匹配对象(submatch objects)`散列`.(它也可选地包含一个用于创建抽象语法树(AST)的**抽象对象**) 为了提供访问这些各种各样值的便捷方法, Match 对象在不同上下文中求值也不同:

- 在布尔上下文中 Match 对象被求值为真或假

```perl6
if /pattern/ {...}
# 或:
/pattern/; if $/ {...}
```

如果模式使用 `:global` 或 `:overlap` 或 `:exhaustive` 修饰符, 会在`第一个匹配处`返回布尔真值.  如果在列表上下文中求值, `Match` 对象会根据需要(lazily)产生剩下的结果.

- 在字符串上下文中, Match 对象会被求值为匹配(match)的字符串化的值,  这通常是整个匹配的字符串.

```perl6
print %hash{ "{$text ~~ /<.ident>/}" };
# 或等价的:
$text ~~ /<.ident>/  &&  print %hash{~$/};
```

但是通常你应该写成 `~$/`, 如果你想要字符串化匹配对象的话.

- 在数字上下文中, Match 对象会被计算成它的匹配的数字值, 这通常是整个匹配的字符串:

```perl6
$sum += /\d+/;
# 或等价的:
/\d+/; $sum = $sum + $/;
```

- 在标量上下文中, Match 对象求值结果为自身.

然而, 有时你想要一个备用标量值伴随着匹配. Match 对象自身描述了一个具体的解析树, 这个额外的值叫做抽象对象;它作为 Match 对象的一个属性伴随着匹配. `.made` 方法默认返回一个未定义值. `$()` 是 `$($/.made // ~$/)` 的简写形式.

因此, `$()` 通常就是整个匹配的字符串, 但是你能在 regex 内部调用 `make` 来重写它:

```perl6
my $moose = $(m[
    <antler> <body>
    { make Moose.new( body => $<body>.attach($<antler>) ) }
    # 匹配成功 -- 忽略该 regex 的剩余部分
]);
```

这把新的抽象节点放进 `$/.made`中. 抽象节点(AST)可以是任何类型. 使用 `make` / `.made` 构造, 创建任意节点类型的抽象语法树就很方便了.

然而, `make` 函数不限于仅仅用作存储 AST 节点并创建抽象语法树。  这就是特殊的 Perl 6 泛函性的内部使用.  `make`函数也不会把任何 item 或 列表上下文强加到它们的参数上, 所以, 你写了某些含糊不清的 listy, 像这样:

```perl6
make ()
make @array
make foo()
```

那么从 `.made` 返回的值会被插值到列表中。 要抑制这, 使用下面这些:

```perl6
make ().item
make []
make $@array
make [@array]
make foo().item
make $(foo())
```

或者在接收终端上使用 `.made.item` 或 `$`变量

`.ast` 方法就是 `.made` 的同义词, 它俩没什么不同. 它的存在一方面是因为历史原因, 另一方面也是为了给阅读你的代码的人标示一个更像 AST 用法的 `made/.make` 

- 你也能使用 ` <(...)>` 构造捕获匹配的一个子集(subset):

```perl6
"foo123bar" ~~ / foo <( \d+ )> bar /
say $();    # says 123
```

这时, 当做字符串匹配时, `$()` 总是一个字符串, 当做列表匹配时, `$()`总是一个或多个元素的列表.这个构造没有设置 `.made` 属性.

- 当用作数组时, Match 对象伪装成一个数组, 数组里是 Match 对象的所有位置捕获.因此,

```perl6
($key, $val) = ms/ (\S+) '=>' (\S+)/;
```

也能被写作:

```perl6
$result = ms/ (\S+) '=>' (\S+)/;
($key, $val) = @$result;
```

要把单个捕获放到字符串中, 使用下标:

```perl6
$mystring = "{ ms/ (\S+) '=>' (\S+)/[0] }";
```

要把所有捕获都放到字符串中, 使用一个禅切:

```perl6
$mystring = "{ ms/ (\S+) '=>' (\S+)/[] }";
```

或把它扔到数组里:

```perl6
$mystring = "@( ms/ (\S+) '=>' (\S+)/ )";
```

注意, 作为一个标量, `$/` 在列表上下文中不会自动展平(flatten). 在列表上下文中使用 `@()`作为 `@($/)` 的简写形式来展平位置捕获. 注意, Match 对象能在列表上下文中按需计算它的匹配.使用 `@()` 来强制进行迫切( eager)匹配.

- 当作为散列时, Match 对象伪装成一个含有具名捕获的散列. 散列的键不包括任何符号, 所以如果你把变量捕获到变量 `@<foo>`, 它的真实名字为 `$/{'foo'}` 或 `$/<foo>`.然而, 在`$/` 可见的任何地方, 你仍旧能把它作为 `@<foo>` 引用. (但是, 对于两个不同的捕获数据类型,使用同一个名字是错误的.)
[`S05-capture/subrule.t lines 17–119`](https://github.com/perl6/roast/blob/master/S05-capture/subrule.t#L17-L119)

[`S05-match/capturing-contexts.t lines 35–164`](https://github.com/perl6/roast/blob/master/S05-match/capturing-contexts.t#L35-L164)

注意, 作为一个标量, `$/` 在列表上下文中不会自动展平(flatten). 在列表上下文中使用 `%()`作为 ` %($/)` 的简写形式来作为一个散列来展平, 或把它绑定到一个合适类型的变量上. 就像 `@()`, `%()`能在列表上下文中按需产生它的 pair 对儿. 

- 编号过的捕获能被当作命名捕获那样, 所以 `$<0 1 2>`  等价于 `$/[0,1,2]`.  这允许你混写命名捕获和编号捕获.

- `.keys`, `.values` 和 `.kv` 方法对列表和散列都起作用, 列表部分首当其冲.` 

```perl6
'abcd' ~~ /(.)(.)**2 <alpha>/;
say ~$/.keys;           # 0 1 alpha
```

- 在普通代码中, 变量 `$0`,`$1`等等就是 `$/[0]` ,`$/[1]` 的别名, 等等, 因此, 如果最后的匹配失败, 它们都会变为未定义的. (除非它们被显式地绑定到一个闭包中, 而不使用 let 关键字)
[`S32-scalar/undef.t lines 220–280`](https://github.com/perl6/roast/blob/master/S32-scalar/undef.t#L220-L280)

- Match 对象有一些方法提供了关于匹配的额外信息, 例如:

```perl6
if m/ def <ident> <codeblock> / {
   say "Found sub def from index $/.from.bytes ",
       "to index $/.to.bytes";
}
```

当前定义过的方法有

```perl6
$/.from      # 初始匹配位置
$/.to        # 最终匹配位置
$/.chars     # $/.to - $/.from
$/.orig      # 原匹配字符串
$/.Str       # substr($/.orig, $/.from, $/.chars)
$/.made      # 关于该节点(来自于 make)的抽象结果
$/.ast       # 和 $/.made 相同
$/.caps      # 相继的捕获
```

[`S05-capture/caps.t lines 5–94`](https://github.com/perl6/roast/blob/master/S05-capture/caps.t#L5-L94)

```perl6
$/.chunks    # sequential tokenization
$/.prematch  # $/.orig.substr(0, $/.from)
$/.postmatch # $/.orig.substr($/.to)
```

在 regex 内部, 当前匹配状态 `$¢` 也提供了这个:

```perl6
.pos        # 当前匹配位置
```

最后那个值根据匹配是向前处理还是向后处理对应于 `$¢.from` 或 `$¢.to`.( 后一种情况出现在 `<?after ...>` 断言内部 ).

- 就像上面描述的那样, 在列表上下文中, Match 对象返回它的位置捕获. 然而, 有时你更想以它们出现在文本中的顺序, 得到一个展平的 tokens 列表. `.caps` 方法按顺序返回一个所有捕获的列表, 而不管它是如何被绑定命名捕获或带编号捕获上的. (除了顺序, 这儿没有新的信息; 列表中的所有元素都是同样的 Match 对象,并被任意绑定.) 绑定实际上是作为 键/键值对儿返回, 而 键是名字或编号, 而值是 Match 对象自身.

 除了返回那些捕获的 Match 对象外, `.chunks` 方法也在两个捕获之间返回交错的噪音. 就像 `.caps` , 列表元素的顺序跟它们原来在文本中的顺序相同.交错的部分也返回一个 pairs, 而键是 `~`, 值为一个简单的只包含字符串的 Match 对象, 即使未绑定的诸如 `.ws` 的子规则首先遍历文本. 在这样一个 Match对象上调用 `.made` 方法总是返回 一个 `Str`.

如果 `.caps` 或 `.chunks` 发现它们有重叠绑定, 会出现一个警告. 没有这样的重叠, `.chunks` 保证将它匹配到的每一部分字符串映射为它返回的所有匹配的精确的一个元素, 所以, 覆盖范围是完整的.

- 所有与任何 regex, subrule, or subpattern 匹配的尝试, 成功与否, 会返回一个能被求值为布尔值的对象.(这个对象要么是一个 Match, 要么是 Nil.)即:

```perl6
$match_obj = $str ~~ /pattern/;
say "Matched" if $match_obj;
```

- 不管成功与否,这个返回的对象也被自动的绑定到当前环境的词法变量 `$/` 上:

```perl6
$str ~~ /pattern/;
say "Matched" if $/;
```

- 在 regex 里面, 变量 `$¢` 保存着当前 regex 的未完成的 Match 对象, 这就是所谓的匹配状态(类型为 Cursor).通常, 这不应该被被修改, 除非你知道怎么创建并传递匹配状态.所有的 regexes 实际上返回匹配状态即使当你认为它们会返回其它东西时, 因为匹配状态为你追踪模式的成功和失败.

幸运的是, 伴随着默认的具体的 Match 对象, 当你只想返回一个不同的抽象结果时, 你可以使用 `make` 函数把当前匹配状态和返回值关联起来, 这跟 return 有点像, 但是不会 clobber 匹配状态:
[`S05-match/make.t lines 9–24`](https://github.com/perl6/roast/blob/master/S05-match/make.t#L9-L24)

```perl6
$str ~~ / foo                 # Match 'foo'
           { make 'bar' }     # But pretend we matched 'bar'
         /;
say $();                      # says 'bar'
```

通过 `.made` 方法能访问到任何 Match 对象的值(例如一个抽象对象). 因此, 这些抽象对象能被独立的管理.

当前指针对象总是由 Cursor 派生而来, 否则匹配不会起作用. 然而, 在那个约束之下, 当前指针的实际类型定义可当前正在解析的是哪一种语言. 当你进入一个 grammar 的顶部时, 这个指针通常开始于一个对象, 该对象的类型是你所在的 grammar 的名字, 但是当前语言可以通过各种方法修改, 当它们通过返回 blessed 为不同类型的指针对象来修改当前语言, 这可能也或许不是从当前 grammar 中派生出来的.

# 子模式捕获

- regex 中任何闭合在`捕获圆括号`中的那部分就是所谓的 `subpattern`, 例如:

```perl6
  #               subpattern
  #  _________________/\___________________
  # |                                      |
  # |       subpattern  subpattern         |
  # |          __/\__    __/\__            |
  # |         |      |  |      |           |
ms/ (I am the (walrus), ( khoo )**2  kachoo) /;
```

- 如果匹配成功,  regex 中的每个 subpattern 都会产生一个 Match 对象

- 每个 subpattern 要么显式地赋值给一个具名目标,要么隐式地被添加到含有很多匹配的`数组`中去.

对于每一个没有显式地给定名字的 subpattern, 该 subpattern 的 Match 对象被推入到外部的属于周围作用域的 `Match` 对象里面的数组中(即它的父 Match 对象). 周围作用域要么是最内部的周围作用域(如果 subpattern 是嵌套的) 要么是整个 regex 自身.

- 像捕获一样, 这些对数组的赋值是假设的, 如果  subpattern 回溯, 这些赋值会被撤销.

- 举个例子, 如果下面这个模式匹配成功:

```perl6
  #                subpat-A
  #  _________________/\__________________
  # |                                     |
  # |         subpat-B  subpat-C          |
  # |          __/\__    __/\__           |
  # |         |      |  |      |          |
ms/ (I am the (walrus), ( khoo )**2 kachoo) /;
```

则由 *subpat-B*  和 *subpat-C* 产生的 Match 对象会被成功地推入到 *subpat- A*  的 Match 对象里面的数组中.  然后 *subpat-A*  的  `Match` 对象自身会被推入到整个 regex 的 Match 对象里面的数组中.

```perl6
my $str = "I am the walrus, khoo khoo kachoo";
$str ~~ ms/ (I am the (walrus)\, ( khoo )**2 kachoo) /;
say ~$/[0];       # I am the walrus, khoo khoo kachoo
say ~$/[0][0];    # walrus
say ~$/[0][1];    # khoo  khoo
say ~$/[0][1][0]; # khoo
say ~$/[0][1][1]; # khoo
```
可以看出, subpat-A 的 Match 对象是 `$/`数组的一个元素, subpat-A 和 subpat-B 的 Match 对象在同一个数组 `$/[0]` 中。

- 因为这些语义, Perl 6 中的捕获括号是分等级的, 而非线性的. (see "Nested subpattern captures”)

# 访问捕获的子模式

- `Match` 对象的`数组元素`要么使用标准的数组访问记法(例如  `$/[0]`, `$/[1]`, `$/[2]` 等.) 要么通过对应的词法作用域`数字别名`(例如: `$0`, `$1`, `$2`), 所以:

[`S05-match/capturing-contexts.t lines 25–34`](https://github.com/perl6/roast/blob/master/S05-match/capturing-contexts.t#L25-L34)

```perl6
say "$/[1] was found between $/[0] and $/[2]";
```

和下面这个相同:

```perl6
say "$1 was found between $0 and $2";
```

- 注意, 在 Perl 6 中, 数字捕获变量从 `$0`开始, 而非 `$1`, 使用 `$/` 中对应元素的索引中的数字.

- regex 的 Match 对象(例如 $/)中的`数组元素`分别存储着单独的 Match 对象, 这些 Match 对象就是匹配到的`子字符串`, 并被第一个, 第二个,第三个,直到最外面的 subpattern 捕获(非嵌套). 所以这些元素能被看成完全合格的匹配结果. 例如:
[S05-capture/dot.t lines 13–55](https://github.com/perl6/roast/blob/master/S05-capture/dot.t#L13-L55)

```perl6
if m/ (\d\d\d\d)-(\d\d)-(\d\d) (BCE?|AD|CE)?/ {
     ($yr, $mon, $day) = $/[0..2];
     $era = "$3" if $3;                    # stringify/boolify
     @datepos = ( $0.from() .. $2.to() );  # Call Match methods
}
```

# 嵌套的子模式捕获
[`S05-capture/named.t lines 16–25`](https://github.com/perl6/roast/blob/master/S05-capture/named.t#L16-L25)

- 通过嵌套的 subpattern 匹配到的子字符串被赋值给嵌套的 subpattern 的 `父 Match 对象`里面的`数组`中, 而不是 $/ 的数组中.

- 这种行为和 Perl 5 的语义完全不同:

```perl6
    # Perl 5...
    #
    # $1---------------------  $4---------  $5------------------
    # |   $2---------------  | |          | | $6----  $7------  |
    # |   |         $3--   | | |          | | |     | |       | |
    # |   |         |   |  | | |          | | |     | |       | |
   m/ ( A (guy|gal|g(\S+)  ) ) (sees|calls) ( (the|a) (gal|guy) ) /x;
```

- 在 Perl 6中, 嵌套的圆括号产生可能的嵌套的捕获

```perl6
    # Perl 6...
    #
    # $0---------------------  $1---------  $2------------------
    # |   $0[0]------------  | |          | | $2[0]-  $2[1]---  |
    # |   |       $0[0][0] | | |          | | |     | |       | |
    # |   |         |   |  | | |          | | |     | |       | |
   m/ ( A (guy|gal|g(\S+)  ) ) (sees|calls) ( (the|a) (gal|guy) ) /;
```

如上, 在匹配嵌套的 subpattern 时, `$0`, `$1`, `$2` 是平级的, 它们都是父 Match 对象 `$/` 数组中的子元素, 即 `$/[0]`、`$/[1]`、`$/[2]`。而 `$0` 和 `$2` 中有嵌套的 subpattern, 所以 `$0` 和 `$2` 也成为父 subpattern, 依次类推。
 
# 量词化的子模式捕获

- 如果 subpattern 后面直接使用 `?`量词, 它要么产生单个 Match 对象, 要么产生 Nil.(?表示匹配0次或1次。) 如果 subpattern 后直接使用任何其它量词, 它绝不会产生单个 Match 对象. 相反, 它产生一个 Match 对象的列表, 列表中的元素对应于由重复的 subpattern 产生的各自匹配的序列. 如果想区分这两种类别, `?` 是一个 item 量词, 而 `*`, `+` 和 `**` 叫做列表量词.

如果匹配到 0 个值, 则捕获到的值取决于用的是哪个量词. 如果量词是 `?`, 并且匹配次数为 0, 则捕获到 Nil. 如果量词是 `*`, 则是`空列表`, 即 `()`. (如果匹配次数为 0, +量词什么也不会捕获, 因为它会引发回溯, 但是 如果在一个不成功的匹配之后, 又尝试使用它, 则捕获变量会返回 Nil ) . 如果它的最小范围是 0,  `**` 量词会像`*`那样返回 `()`, 否则就会回溯.

注意,  不像 ?,  `** 0..1` 总是被认为是一个列表量词.

把 `?` 看作 item 量词的理由是为了使它符合 `$object.?meth` 定义的方式, 并减少不必要的 `.[0]`下标, 这会使大部分人惊讶.既然 Nil 被认为是未定义的而非`()`的同义词, 使用 `$0 // "default"` 或诸如此类的来安全地解引用捕获就很容易了.

- 因为列表量词化的 subpattern 返回一个 Match 对象的列表, 对应的量词化的捕获数组元素会存储一个(嵌套的)数组而不是单个 Match 对象.例如:

```perl6
if m/ (\w+) \: (\w+ \s+)* / {
   say "Key:    $0";         # Unquantified --> single Match
   say "Values: @($1)";      # Quantified   --> array of Match
}
```

# 间接量词化的子模式捕获

- subpattern 有时会嵌套在一个量词化的非捕获结构中:

```perl6
  #       non-capturing       quantifier
  #  __________/\____________  __/\__
  # |                        ||      |
  # |   $0         $1        ||      |
  # |  _^_      ___^___      ||      |
  # | |   |    |       |     ||      |
 m/ [ (\w+) \: (\w+ \h*)* \n ] ** 2..* /
```

非捕获括号不会创建单独的嵌套词法作用域, 所以那两个 subpattern 实际上仍然在 regex 的顶层作用域中, 因此, 它们的顶层名字是 `$0` 和 `$1`.

- 然而, 因为那两个 subpattern 在量词化结构里面, `$0` 和 `$1` 每个都会包含一个数组.  每次迭代非捕获分组, 数组的元素会是对应 subpattern 返回的 submatch.例如:

```perl6
my $text = "foo:food fool\nbar:bard barb";
```

```perl6
           #   $0--     $1------
           #   |   |    |       |
 $text ~~ m/ [ (\w+) \: (\w+ \h*)* \n? ] ** 2..* /;
```

```perl6
 # 因为它们在一个量词化的非捕获 block 中...
 # say $/[0].perl;
 # $0 包含着下面的等同物:
 #
 #       [ Match.new(str=>'foo'), Match.new(str=>'bar') ]
 #
 # 并且 $1 包含下面的等同物:
 #
 #       [ Match.new(str=>'food '),
 #         Match.new(str=>'fool' ),
 #         Match.new(str=>'bard '),
 #         Match.new(str=>'barb' ),
 #       ]
```

- 与此相反, 如果外部的量词化结构是一个*捕获*结构(例如. 一个 subpattern), 那么它会引入一个嵌套的词法作用域. 外部的量词化结构会返回一个 Match 对象的数组, 代表对每个迭代的内部括号的捕获。即:

```perl6
my $text = "foo:food fool\nbar:bard barb";
```

```perl6
           # $0-----------------------
           # |                        |
           # | $0[0]    $0[1]---      |
           # | |   |    |       |     |
 $text ~~ m/ ( (\w+) \: (\w+ \h*)* \n ) ** 2..* /;
```

```perl6
 # 因为它是一个量词化的捕获 block,
 # $0 包含如下等价物:
 #
 #       [ Match.new( str=>"foo:food fool\n",
 #                    arr=>[ Match.new(str=>'foo'),
 #                           [
 #                               Match.new(str=>'food '),
 #                               Match.new(str=>'fool'),
 #                           ]
 #                         ],
 #                  ),
 #         Match.new( str=>'bar:bard barb',
 #                    arr=>[ Match.new(str=>'bar'),
 #                           [
 #                               Match.new(str=>'bard '),
 #                               Match.new(str=>'barb'),
 #                           ]
 #                         ],
 #                  ),
 #       ]
 #
 # 并且没有 $1
```

- 换句话说, 量词化的非捕获括号把它们的组件聚集到就近展平的列表中, 而量词化的捕获括号把它们的部件聚集到就近的分等级的结构中.

此外,  sublist 彼此间是被同步保存的,作为每个空匹配, 在我们例子中的 `$0[1]`情形下, 如果冒号后面跟着一个换行符, 那么将会在给定的列表中有一个对应的 Nil值。

# 子模式编号

- 给定 subpattern 的索引总是能被静态地决定, 但不是唯一也不是无变化的. subpattern 的编号从每个词法作用域重新开始.( regex, subpattern, 或备选分支中的任意一个)

- 特别地, 在每个 `|` 或 `||` 之后, 捕获括号的索引重新开始.(但是不是在每个 & 或 && 之后)

```perl6
                # $0        $1    $2   $3    $4           $5
   $tune_up = rx/ ("don't") (ray) (me) (for) (solar tea), ("d'oh!")
                # $0      $1      $2    $3        $4
                | (every) (green) (BEM) (devours) (faces)
                /;
```

这意味着, 如果第二个备选分支匹配, 匹配的列表中将会包含 `('every', 'green', 'BEM', 'devours', 'faces')` 而非 Perl 5 的 `(undef, undef, undef, undef, undef, undef, 'every', 'green', 'BEM', 'devours', 'faces')`.

- 注意, 仍旧能模仿无变化的 Perl 5 捕获索引语义.查看下面的 "Numbered scalar aliasing"

# Subrule 捕获
[`S05-capture/named.t lines 36–74`](https://github.com/perl6/roast/blob/master/S05-capture/named.t#L36-L74)

-  在模式中调用任何一个命名的 `<regex>` 被称为 `subrule`, 不管那个正则表达式实际上被定义为一个 `regex`, 或者 `token`, 或者甚至普通的方法或 `multi`.

- 任何别名为具名变量的括号结构也是一个 `subrule`

- 例如, 下面这个正则表达式包含 3 个 subrules:

```perl6
  # subrule       subrule     subrule
  #  __^__    _______^_____    __^__
  # |     |  |             |  |     |
 m/ <ident>  $<spaces>=(\s*)  <digit>+ /
```

- 就像 subpatterns 那样, 在正则表达式中每个成功匹配的 subrule 都产生一个 Match 对象. 但是, 跟 subpatterns 不同的是, 那个 Match 对象没有赋值给它的父 Match 对象里面的数组. 相反, 它被赋值给它的父 Match 对象里面的散列中的一个条目(键值对儿). 例如:

```perl6
        #  .... $/ .....................................
        # :                                             :
        # :              .... $/[0] ..................  :
        # :             :                             : :
        # : $/<ident>   :        $/[0]<ident>         : :
        # :   __^__     :           __^__             : :
        # :  |     |    :          |     |            : :
        ms/  <ident> \: ( known as <ident> previously ) /
```

# 访问捕获的 subrules 

-  Match 对象的散列条目可以使用任何一个标准的散列访问记法(`$/{'foo'}`, `$/`, `$/«baz»`, 等等.) 查阅, 或通过对应的词法作用域别名 (**`$<foo>`**, `$«bar»`, **`$<baz>`**, 等等.)访问. 所以前面的例子也意味着:
[`S05-capture/dot.t lines 62–87`](https://github.com/perl6/roast/blob/master/S05-capture/dot.t#L62-L87)

```perl6
        #    $<ident>             $0<ident>
        #     __^__                 __^__
        #    |     |               |     |
        ms/  <ident> \: ( known as <ident> previously ) /
```

- 注意, subrule 是使用尖括号(*<ident>*) 或者使用内部别名(*<ident=.name>*)还是使用外部别名( **$<ident>=(<.alpha>\w*)** )是没有分别的.

# 同一个 subrule 的重复捕获

- 如果在词法作用域的任何一个分支中出现 2次(或更多) subrules (例如,在同一个 subpattern 和备选中出现2次), 或者, 如果 在给定作用域的任何地方, subrule 是列表量词化的(那就是, 使用除了?之外的任何其它量词), 那么, 它的对应散列条目总是被赋值给 Match 对象的数组中, 而不是赋值给单个 Match 对象.

- 同一个 subrule 的成功匹配( 无论是来自于单独的调用还是来自于单个量词化重复)把单独的 Match 对象追加到这个数组中, 例如:

```perl6
if ms/ mv <file> <file> / {
    $from = $<file>[0];
    $to   = $<file>[1];
}
```

(注意, 为了代码清晰, 我们这里忽略了空白的细微之处 -- 普通的 sigspace rules 只会在字母数字字符之间要求有空白, 这是错误的. 假设我们的 `<file>` subrule 自己处理空白.)

同样地, 使用量词化的 subrule:

```perl6
if ms/ mv <file> ** 2 / {
    $from = $<file>[0];
    $to   = $<file>[1];
}
```

还有使用它们两者的混合:

```perl6
if ms/ mv <file>+ <file> / {
    $to   = pop @($<file>);
    @from = @($<file>);
}
```

- 为了避免名字冲突, 可以使用一个前置的点来抑制原来的名字, 然后使用别名给捕获一个不同的名字:

```perl6
if ms/ mv <file> <dir=.file> / {
    $from = $<file>;  # 只有一个 subrule 叫做 <file>, 所以是标量
    $to   = $<dir>;   # 这个捕获之前叫做 <file>
}
```

同样地, 下面的结构都不会让 `<file>` 产生一个 Match 对象的数组, 因为在同一个词法作用域中, 它们都没有两个或更多的 `<file>` subrules.

```perl6
if ms/ (keep) <file> | (toss) <file> / {
    # 每个 <file> 都是单独的备选分支,
    # 因此 <file> 在任何一个作用域中都没有被重复, 因此, $<file> 不是数组对象.
    $action = $0;
    $target = $<file>;
}
```

```perl6
if ms/ <file> \: (<file>|none) / {
    # 第二个 <file> 嵌套在不同作用域中的 subpattern 中
    $actual  = $/<file>;
    $virtual = $/[0]<file> if $/[0]<file>;
}
```

- 另一方面, 未别名化的方括号没有被授予单独的作用域(因为它们没有关联的 Match 对象).所以:

```perl6
if ms/ <file> \: [<file>|none] / { # 这两个 <file> 在同一个作用域中
    $actual  = $/<file>[0];
    $virtual = $/<file>[1] if $/<file>[1];
}
```

# 别名

别名可以被命名或编号. 它们可以是 *scalar-*, *array-*, 或 *hash-like*. 并且它们能被应用到捕获或非捕获结构中.  下面的章节会突出那些组合语义的特殊功能.

# 让具名标量成为subpatterns的别名
[`S05-capture/named.t lines 26–35`](https://github.com/perl6/roast/blob/master/S05-capture/named.t#L26-L35)

- 如果一个具名标量别名被应用了一组**捕获**圆括号:
[`S05-capture/alias.t lines 17–26`](https://github.com/perl6/roast/blob/master/S05-capture/alias.t#L17-L26)

```perl6
          #         _____/capturing parens\_____
          #        |                            |
          #        |                            |
        ms/ $<key>=( (<[A..E]>) (\d**3..6) (X?) ) /;
```

那么远离外部的圆括号不再像非别名括号那样捕获到 `$/` 数组中。相反, 别名化的圆括号捕获到了 `$/` 散列中; 特别是捕获到键名是别名名字的散列元素中。

- 所以, 在上面的例子中, 一个成功的匹配设置了 `$<key>`(例如 `$/<key>`), 而非 `$0`(例如 不是 `$/[0]`)。

- 更确切地说:

  - `$/` 会包含之间已经放进 `$/[0]` 中的 `Match` 对象。
  - `$/[0]` 会包含 A-E 字母,
  - `$/[1]` 会包含数字,
  - `$/[2]` 会包含可选的 X.

- 了解这种行为的另外一种方法是别名化的括号创建了一种本地作用域的具名 subrule; 圆括号的内容被当作就像它们是单独的 subrule 的一部分, 它的名字就是别名。

# 让具名标量成为非捕获分组的别名

- 如果一个具名标量别名被应用到一组非捕获括号:
  [`S05-capture/alias.t lines 33–68`](https://github.com/perl6/roast/blob/master/S05-capture/alias.t#L33-L68)

```perl6
          #         __/non-capturing brackets\__
          #        |                            |
          #        |                            |
        ms/ $<key>=[ (<[A..E]>) (\d**3..6) (X?) ] /;
```

则对应的 `$/` `Match` 对象只会包含非捕获括号匹配到的字符串.

- 特别地,  `$/` 数组中的条目是空的. 那是因为方括号不会创建嵌套的词法作用域. 所以 subpatterns 是非嵌套的, 并且因此对应于`$0`, `$1`, 和 `$2`, 而不是对应于  `$/[0]`, `$/[1]`, 和 `$/[2]`.

- 换句话说:

  - `$/` 会包含方括号匹配到的整个子字符串  (in a `Match` object, as described above),
  - `$0` 会包含字母 `A-E`,
  - `$1` 会包含数字,
  - `$2 `会包含可选的 X.

# 让具名标量成为 subrules 的别名

- 如果 subrule 被设置了别名, 它会把它的 Match 对象设置为散列的条目, 散列的键是别名的名字, 它和 subrule 原来的名字一样.

```perl6
if m/ ID\: <id=ident> / {
    say "Identified as $/<id> and $/<ident>";    # both names defined
}
```

要抑制原来的名字, 使用带点形式的名字:
  ​
```perl6
if m/ ID\: <id=.ident> / {
    say "Identified as $/<id>";    # $/<ident> is undefined
}
```

- 因此, 给一个带点的 subrule 起别名改变了 subrule 的 Match 对象的目标.在同一个作用域内, 这对于区分对同一个 subrule 的两次或多次调用.例如:

```perl6
if ms/ mv <file>+ <dir=.file> / {
    @from = @($<file>);
    $to   = $<dir>;
}
```

# 给标量别名编号

- 如果使用编号别名而非使用具名别名:

```perl6
m/ $1=(<-[:]>*) \:  $0=<ident> /   # captures $<ident> too
m/ $1=(<-[:]>*) \:  $0=<.ident> /  # doesn't capture $<ident>
```

编号别名的行为就和具名别名的一样(i.e. 上面描述过的各种情况), 除了结果 Match 对象被赋值给对应的合适数组元素, 而非散列元素.

- 如果使用了编号别名, 后续同一作用域中未起别名的 subpatterns 的编号会从那个别名编号开始自动增长(跟枚举数值从最后一个显式值开始增长很像). 即:
[`S05-capture/alias.t lines 27–32`](https://github.com/perl6/roast/blob/master/S05-capture/alias.t#L27-L32)

```perl6
        #  --$1---    -$2-    --$6---    -$7-
        # |       |  |    |  |       |  |    |
       m/ $1=(food)  (bard)  $6=(bazd)  (quxd) /;
```

- 这种后续的行为对于在备选分支中重新建立 Perl5语义中的连续 subpattern 编号特别有用:

```perl6
       $tune_up = rx/ ("don't") (ray) (me) (for) (solar tea), ("d'oh!")
                    | $6 = (every) (green) (BEM) (devours) (faces)
                    #              $7      $8    $9        $10
                    /;
```

- 这也在 Perl 6 中提供了一种简单的方式来重建嵌套的 Perl 5 subpatterns 的非嵌套编号语义:

  ```perl
        # Perl 5...
        #               $1
        #  _____________/\___________
        # |    $2        $3      $4  |
        # |  __/\___   __/\___   /\  |
        # | |       | |       | |  | |
       m/ ( ( [A-E] ) (\d{3,6}) (X?) ) /x;
```

```perl6
        # Perl 6...
        #                $0
        #  ______________/\______________
        # |   $0[0]       $0[1]    $0[2] |
        # |  ___/\___   ____/\____   /\  |
        # | |        | |          | |  | |
       m/ ( (<[A..E]>) (\d ** 3..6) (X?) ) /;
```

```perl6
        # Perl 6 simulating Perl 5...
        #                 $1
        #  _______________/\________________
        # |        $2          $3       $4  |
        # |     ___/\___   ____/\____   /\  |
        # |    |        | |          | |  | |
       m/ $1=[ (<[A..E]>) (\d ** 3..6) (X?) ] /;
```

非捕获括号没有引入作用域, 所以非捕获括号中的 subpatterns 处于 regex 作用域, 并因此在括号顶层开始编号. 给方括号起别名为 `$1`意味着同一级别的下一个 subpattern(例如 `(<[A..E]>)`)的编号继续(i.e. `$2`). 等等.
  ​
# 给量词化结构应用标量别名

- 上面所有的语义可以同等地应用到绑定了量词化结构的别名身上.

- 唯一不同的是, 如果别名化的结构是一个 subrule 或 subpattern, 那么量词化的 subrule 或 subpattern 必然会返回一个 Match 对象的列表. (像 "Quantified subpattern captures") 和 "Repeated captures of the same subrule") 中描述的那样). 所以, 别名所对应的数组元素或散列条目会包含一个数组, 而不是单个 Match 对象.

- 换句话说, 别名和量词化是完全正交的,例如:

```perl6
       if ms/ mv $0=<.file>+ / {
           # <file>+ 返回一个 Match objects 的列表,
           # 所以 $0 包含一个 Match objects 的数组,
           # one for each successful call to <file>
           # $/<file> 不存在 (因为它被点号抑制了)
       }

       if m/ mv \s+ $<from>=(\S+ \s+)* / {
           # 量词化的子模式返回一个 Match objects 的列表,
           # 所以 $/<from> 包含了一个 Match objects 的数组,
           # one for each successful match of the subpattern
           # $0 不存在 ($0 被别名预先清空了)
       }
```

- 注意, 一组量词化的非捕获括号总是返回单个 Match 对象,  该 Match 对象只包含通过全组重复括号匹配到的整个子字符串.(就像 "具名标量别名应用到非捕获括号") 中描述的那样). 例如:

```perl6
"coffee fifo fumble" ~~ m/ $<effs>=[f <-[f]> ** 1..2 \s*]+ /;
say $<effs>;    # 打印 "fee fifo fum"
```

# 数组别名

- 别名也能使用一个数组而非标量作为别名。例如:

[`S05-capture/array-alias.t lines 13–92`](https://github.com/perl6/roast/blob/master/S05-capture/array-alias.t#L13-L92)

```perl6
m/ mv \s+ @<from>=[(\S+) \s+]* <dir> /;

"    a b\tc" ~~ m/@<chars>=( \s+ \S+)+/;
join("|", @<chars>) #     a| b|	c
```

- 使用 `@alias=` 记法而非 `$alias=`  迫使对应散列条目或数组元素总是接收一个 Match 对象的数组, 即使正被起别名的结构通常返回的是单个 Match 对象。 这对于根据结构不同的备选分支创建一致的捕获语义很有用。(通过在所有分支中强制数组捕获):

```perl6
       ms/ Mr?s? @<names>=<ident> W\. @<names>=<ident>
          | Mr?s? @<names>=<ident>
          /;
       # 别名起为 @names 意味着 $/<names> 总是一个 Array 对象, 所以...
       say @($/<names>);
```

- 为了方便和一致性,  `@` 也能用在 regex 外面. 作为`@( $/ )` 的简写形式。 即:

```perl6
       $_ = "Mrs camelia W. rakudo"
       ms/ Mr?s? @<names>=<ident> W\. @<names>=<ident>
          | Mr?s? @<names>=<ident>
          /;
       say @<names>; # [｢camelia｣ ｢rakudo｣]
```

其中 `ms` 中的 **s** 使 regex 中的空格有意义。
  ​

- 如果把数组别名应用到量词化的非捕获括号上, 它会捕获由每次括号的重复匹配到的子字符串, 捕获到对应数组的单独的元素中.即:

```perl6
       ms/ mv $<files>=[ f.. \s* ]* /; # $/<files> assigned a single
                                       # Match object containing the
                                       # complete substring matched by
                                       # the full set of repetitions
                                       # of the non-capturing brackets
```


`$/<files>` 被赋值了单个包含由全套重复的非捕获括号所匹配到的完整子字符串的Match 对象。
  ​
```perl6
       ms/ mv @<files>=[ f.. \s* ]* /; # $/<files> assigned an array,
                                       # each element of which is a
                                       # Match object containing
                                       # the substring matched by Nth
                                       # repetition of the non-
                                       # capturing bracket match
```

`$/<files>` 被赋值给了一个数组, 数组中的每个元素是含有通过第 N 个 重复的非捕获括号匹配所匹配到的子字符串的 Match 对象。

- 如果数组别名应用到捕获的括号（即一个子模式）的量词化对儿上，那么相应的散列或数组元素被分配通过连接由子模式的一个重复返回的每个` Match`对象的数组值构成的列表。也就是说，一个子模式阵列别名变平，并收集混叠子模式中的所有嵌套子模式拍摄。例如：

```perl6
       if ms/ $<pairs>=( (\w+) \: (\N+) )+ / {
           # Scalar alias, so $/<pairs> is assigned an array
           # of Match objects, each of which has its own array
           # of two subcaptures...
           for @($<pairs>) -> $pair {
               say "Key: $pair[0]";
               say "Val: $pair[1]";
           }
       }
```


```perl6
       if ms/ @<pairs>=( (\w+) \: (\N+) )+ / {
           # Array alias, so $/<pairs> is assigned an array
           # of Match objects, each of which is flattened out of
           # the two subcaptures within the subpattern

           for @($<pairs>) -> $key, $val {
               say "Key: $key";
               say "Val: $val";
           }
       }
```

- 同样地, 如果数组别名(array alias)被应用到量词化的 subrule 上, 那么对应于别名的散列元素或数组元素被赋值了一个列表, 这个列表包含了从每次 subrule 重复返回的每个 `Match` 对象的数组值, 它们都被展开到单个数组中: 

```perl6
       rule pair { (\w+) \: (\N+) \n }

       if ms/ $<pairs>=<pair>+ / {
           # 标量别名, 所以 $/<pairs> 包含了一个 Match 对象的数组
           # 数组中的每个值都是 <pair> subrule 调用的结果...

        for @($<pairs>) -> $pair {
               say "Key: $pair[0]";
               say "Val: $pair[1]";
           }
       }
```

```perl6
       rule pair { (\w+) \: (\N+) \n }

       if ms/ mv @<pairs>=<pair>+ / {
           # 数组别名, 所以 $/<pairs> 包含了一个 Match objects 的数组
           # all flattened down from the
           # nested arrays inside the Match objects returned
           # by each match of the <pair> subrule...
           # 从 <pair> subrule 的每次匹配返回的 Match objects 里面嵌套的数组都向下展开

          for @($<pairs>) -> $key, $val {
               say "Key: $key";
               say "Val: $val";
           }
       }
```

- 换句话说, 数组别名在把任何可能出现在量词化 subpattern 或 subrule 中的嵌套的捕获展开到单个数组中时很有用。而标量别名对保存每次重复的顶级数组内部结构很有用。

- 把数字编号变量用作数组别名也是可行的。语义和上面描述的相同。唯一不同的是 `Match` objects 的结果数组被赋值给了该 regex 的匹配数组的合适元素中而不是它的匹配散列的键中。例如:

```perl6
       if m/ mv  \s+  @0=((\w+) \s+)+  $1=((\W+) (\s*))* / {
           #          |                |
           #          |                |
           #          |                 \_ 标量别名, 所以 $1 得到一个数组
           #          |                    数组中的每个元素都是一个 Match 对象
           #          |                    每个 Match 对象都包含了两个嵌套的捕获
           #          |
           #          |
           #           \___ 数组别名, 所以 $0 得到了一个展开的数组
           #                数组中的每个元素都是从每次重复所得到的 (\w+) 捕获

           @from     = @($0);      # 展开的列表
           $to_str   = $1[0][0];   # Nested elems of
           $to_gap   = $1[0][1];   #    unflattened list
       }
```

- 再次注意, 在 regex 外面, `@0` 就是 `@($0)` 的简写形式, 所以上面代码中的第一次赋值也能写作这样:

```perl6
@from = @0;
```

# 散列别名

- 除了使用标量或数组, 使用散列作为别名变量也能指定别名, 例如:
[S05-capture/hash.t lines 13–159](https://github.com/perl6/roast/blob/master/S05-capture/hash.t#L13-L159)

```perl6
m/ mv %<location>=( (<ident>) \: (\N+) )+ /;
```

- 散列别名让当前作用域中 `Match` 对象对应的散列或数组元素被赋值了一个(嵌套的) Hash 对象(而不是一个 `Array` 对象或单个 `Match` 对象)。

- 如果散列别名被应用到 subrule 或 subpattern 上, 那么第一个嵌套的数字捕获变成每个散列条目的键, 剩下的数字捕获成为键值(如果不止一个则在数组中)。

- 就像数组别名那样, 把编号变量用作散列别名也是可行的, 唯一不同的是 `Match` 对象的结果的存储位置:
  ​
```perl6
  rule one_to_many {  (\w+) \: (\S+) (\S+) (\S+) }

     if ms/ %0=<one_to_many>+ / {
       # $/[0] 含有一个散列, 其中每个键由 <one_to_many> 中的第一个 subcapture 提供
       # 每个键值是一个包含了 subrule 的第二个、第三个、第四个等 subcaptures 的数组...

       for %($/[0]) -> $pair {
             say "One:  $pair.key()";
             say "Many: { @($pair.value) }";
         }
     }
```

- 在 regex 外部, `%0` 是  `%($0)`的简写:

```perl6
       for %0 -> $pair {
           say "One:  $pair.key()";
           say "Many: @($pair.value)";
       }
```

# 外部别名

[S05-capture/external-aliasing.t lines 6–38](https://github.com/perl6/roast/blob/master/S05-capture/external-aliasing.t#L6-L38)

- 代替像这样在内部使用别名:

```perl6
m/ mv  @<files>=<ident>+  $<dir>=<ident> /
```

 普通变量名能用作外部别名, 像这样:

```perl6
m/ mv  @OUTER::files=<ident>+  $OUTER::dir=<ident> /
```

- 这时, 每个别名的表现和之前章节所描述的相同。不同的是结果捕获被直接(但仍旧是假设)绑定给指定名字的变量, 而该变量必须已经存在于声明该 regex 的作用域中。

# 从重复匹配中捕获

- 当整个正则表达式使用重复(由 `:x` 或 `:g` 标记指定)或重叠(通过 `:ov` 或 `:ex` 标记指定)匹配成功时, 它会产生一个不同匹配的序列。

- 在任何这些标记之下, 成功的匹配仍旧在 `$/` 中返回单个 `Match` 对象。然而, 该对象可能代表着该 regex 的部分求值。还有, 这个匹配对象的值和通过非重复匹配提供的那些值有些许不同:

  例如:

```perl6
if $text ~~ ms:g/ (\S+:) <rocks> / {
   say "Full match context is: [$/]";
}
```

但是对应于每个单独的匹配的单独匹配对象的列表也是可得到的:
  ​
```perl6
   if $text ~~ ms:g/ (\S+:) <rocks> / {
       say "Matched { +lol() } times";    # Note: forced eager here by +

       for lol() -> $m {
           say "Match between $m.from() and $m.to()";
           say 'Right on, dude!' if $m[0] eq 'Perl';
           say "Rocks like $m<rocks>";
       }
   }
```

  - 这样的匹配之后 `$/` 的布尔值要么是 true, 要么是 false, 取决于模式是否匹配。
  - 字符串值是从第一次匹配开始到最后一次匹配结束的子字符串(包括任何 regex 跳过的发生于其间的部分字符串, 为了找到后来的匹配。)
  - Subcaptures 被作为多维列表返回, 用户可以选择两种方法之一来处理。如果你引用 `@().flat`(或仅仅在展开的列表上下文中使用 `@()`), 那么多维性被忽略而且所有的匹配被展开后(但是仍旧是懒惰的)返回。如果你引用了 `lol()`, 则你可以得到每个单独的 sublist 作为一个 `Parcel` 对象。就像任何多维列表那样, 每个 sublist 各自都可以是懒惰的。

# Grammars

- 你私有的  `ident` rule 不能重写其它人的 `ident` rule. 所以需要某种机制将 rules 限制到一个名称空间中.
- 如果 subs 是 rules 的模型, 那么 `modules/classes` 明显就是用于凝聚它们的模型. 这种 rules 的集合就是所谓的 *grammars*​
- 就像一个类能把具名的 actions 收集在一起,  grammar 也能把一组具名的 `rules` 收集在一起:

```perl6
   class Identity {
       method name { "Name = $!name" }
       method age  { "Age  = $!age"  }
       method addr { "Addr = $!addr" }

       method desc {
           print &.name(), "\n",
                 &.age(),  "\n",
                 &.addr(), "\n";
       }
       # etc.
   }
```

```perl6
   grammar Identity {
       rule name { Name '=' (\N+) }
       rule age  { Age  '=' (\d+) }
       rule addr { Addr '=' (\N+) }
       rule desc {
           <name> \n
           <age>  \n
           <addr> \n
       }

       # etc.
   }
```

- 像类那样, grammars 也能继承:
  [`S05-grammar/inheritance.t lines 6–80`](https://github.com/perl6/roast/blob/master/S05-grammar/inheritance.t#L6-L80)

```perl6
   grammar Letter {
       rule text     { <greet> $<body>=<line>+? <close> }
       rule greet    { [Hi|Hey|Yo] $<to>=\S+? ','       }
       rule close    { Later dude ',' $<from>=.+        }
       token line    { \N* \n                           }
   }

   grammar FormalLetter is Letter {
       rule greet { Dear $<to>=\S+? ','            }
       rule greet { Dear $<to>=\S+? ','            }
       rule close { Yours sincerely ',' $<from>=.+ }
   }
```

- 就像类中的方法,  grammar 中 rule 定义也是继承的(并且是多态的!). 所以没有必要重新指定文本, 行等等.
  [S05-capture/dot.t lines 88–122](https://github.com/perl6/roast/blob/master/S05-capture/dot.t#L88-L122)

- Perl 6 会携带至少一个预定义好的 grammar:

```perl6
   grammar STD {    # Perl 自己的标准 grammar
        rule prog { <statement>* }
        rule statement {
                 | <decl>
                 | <loop>
                 | <label> [<cond>|<sideff>|';']
       }

       rule decl { <sub> | <class> | <use> }
       # etc. etc. etc.
   }
```

- 因此:

```perl6
$parsetree = STD.parse($source_code)
```

- 你可以使用 `:lang` 副词在 regex 的中间切换到不同的 grammar. 例如, 要匹配一个嵌在花括号中来自于 `$funnylang` 的表达式  `<expr>`, 要说:

```perl6
token funnylang { '{' [ :lang($funnylang.unbalanced('}')) <expr> ] '}' }
```

- 通过在 grammar 身上调用  `.parse` 或 `.parsefile` 方法, 字符串就能与 grammar 匹配, 并且可以传递一个可选的 actions 对象给 grammar:

[S05-grammar/action-stubs.t lines 7–36](https://github.com/perl6/roast/blob/master/S05-grammar/action-stubs.t#L7-L36)

```perl6
MyGrammar.parse($string, :actions($action-object))
MyGrammar.parsefile($filename, :actions($action-object))
```

​这创建了一个  `Grammar` 对象,  它的类型指示了当前被解析的语言, 还有派生自哪个用于扩展语言的 grammars. 所有的 grammars 对象派生自 `Cursor`, 所以每个 grammar 对象的值包含了当前匹配的当前状态.  这个新的 grammar 对象然后被作为 `MyGrammar` 的`TOP` 方法(`regex`, `token`, 或 `rule` )的调用者传递. 这个调用的默认 rule 的名字可以使用 `parse` 方法的  `:rule`  具名参数进行重写.  这对于  grammar rules 的单元测试很有用.  作为参数, rules 可以拥有参数, 所以如果必要的话, `:args` 具名参数可以用于传递这样的参数作为 parcel.

Grammar 对象是不可变的, 所以每个匹配返回不同的匹配状态, 并且多个匹配状态可同时存在.  每个这样的匹配状态被认为是 模式怎样会最终匹配的假设. 在模式匹配中, 一个能回溯的选择能在 Perl 6中作为一个匹配状态指针的惰性列表被轻易描绘. 回溯由只抛弃列表前面的值并继续匹配下一个值组成. 因此, 这些匹配指针的管理控制着回溯是怎样工作的, 并且从惰性列表的词形变化表中自然地往下落

`.parse` 和 `.parsefile` 方法锚定到文本的开头和结尾,  并且如果没有到达文本的结尾会失败.(`TOP` rule 能自己检查 `$`, 如果它想产生它自己的错误信息.)

如果你想解析一部分文本, 那么使用 `subparse` 代替. 你可能传递一个 `:pos` 参数从某个不是 0 的位置开始解析. 你可能传递一个 `:rule` 参数来指定你想调用哪个 `subrule`. 通过检查返回的 Match 对象决定最终的位置.

# Action 对象

Action 对象(由 `Grammar.parse` 中的 `:actions` 具名参数提供)的方法对应于 grammar 中的 rules. 当 grammar 中的 rule 匹配时, action 对象中与 grammar 中的同名方法( 如果有的话) 就会用于 grammar 正在构建的 Match 的 AST 中.  

Action 方法只有一个参数(为了方法, `$/`), 它包含了 rule 的 Match 对象. 只要对应的 rule 成功匹配, Action 方法就会被调用, 不管匹配是一个零宽匹配还是一个最终失败的回溯分支, 所以 要通过 AST 来跟踪状态, 并且副作用可能导致意想不到的行为.

Action 方法是在 rule 的调用帧中被调用的, rule 中的动态变量设置被传递给了 action 方法.

# 句法分类

[`S05-syntactic-categories/new-symbols.t lines 7–33`](https://github.com/perl6/roast/blob/master/S05-syntactic-categories/new-symbols.t#L7-L33)

要写你自己的反引号和断言 subrules,  你可以使用下面的句法分类来扩展(你的拷贝) Regex sublanguage:

```perl6
augment slang Regex {
    token backslash:sym<y>   { ... }   # 定义你自己的 \y 和 \Y
    token assertion:sym<>    { ... }   # 定义你自己的 <stuff>
    token metachar:sym<,>    { ... }   # 定义一个新的元字符
    multi method tweak (:$x) { ... }   # 定义你自己的 :x 修饰符
}
```

# 编译指令

各种编译指令能用于控制 regex 编译的各个方面和未提供的用法. 这些被捆绑到特殊声明符 ? 上:

```perl6
 use s :foo;         # control s defaults
 use m :foo;         # control m defaults
 use rx :foo;        # control rx defaults
 use regex :foo;     # control regex defaults
 use token :foo;     # control token defaults
 use rule :foo;      # control rule defaults
```

# 转换

[`S05-transliteration/trans.t lines 11–270`](https://github.com/perl6/roast/blob/master/S05-transliteration/trans.t#L11-L270)

- `tr///` quote-like 操作符现在有一个叫做 `trans()`的方法. 它的参数是一个 pairs 的列表. 你可以使用任何能产生 pair 列表的东西:
```
$str.trans( %mapping.pairs );
```

使用 `.=` 形式做就地转换:

```perl6
$str.=trans( %mapping.pairs );
```

(Perl 6 不支持 `y///` 形式, 这种形式只存在于 sed 中, 因为它们用光了单个字母.)

- pair 的两边可以像 `tr///` 那样解释字符串:

```perl6
$str.=trans( 'A..C' => 'a..c', 'XYZ' => 'xyz' );
```

作为一种退化了的情况, pair 的每一边都可以是单个字符:

```perl6
$str.=trans( 'A'=>'a', 'B'=>'b', 'C'=>'c' );
```

空白字符作为字面字符, 作为转换的来源或目标.  `..` 范围序列是在字符串中唯一能被识别的元语法, 尽管你可以理所当然的在双引号中使用反斜线插值. 如果右侧的字符太短,   最后的字符会被重复直到和左侧字符的长度相等. 如果没有最后的字符, 是因为右侧的字符是一个空字符, 代替的是, 匹配的结果被删除.

- pair 的一边或两边也可以是一个数组对象:

```perl6
$str.=trans( ['A'..'C'] => ['a'..'c'], <X Y Z> => <x y z> );
```

数组版本是基础原始的形式: 字符串形式的语义正等价于这种形式, 首先展开 `..`, 然后再把字符串分割为单个字符, 然后将它们用作数组.

- 数组版本的转换能将一个或多个字符映射为一个或多个字符:

```perl6
 $str.=trans( [' ',      '<',    '>',    '&'    ] =>
              [' ', '<', '>', '&' ]);
```

在多于一个输入字符序列匹配的情况下, 最长的那个匹配胜出.在两个相同序列匹配的情况下, 排在第一的那个匹配胜出。
与字符串形式一样, 缺失的右侧元素重复最后的那个元素,  而一个空的数组会导致删除.

- 字符串和数组形式的识别是基础的. 要实现更强大的功能, 左侧的识别元素可以通过构建字符类, 向前查看等 regex 来指定.

```perl6
$str.=trans( [/ \h /,   '<',    '>',    '&'    ] =>
             [' ', '<', '>', '&' ]);
```

```perl6
$str.=trans( / \s+ / => ' ' );      # 将所有空白挤压为单个空格
$str.=trans( / <-alpha> / => '' );  # 删除所有的非字母字符
```

- 如果箭头右侧是一个闭包, 它会被计算为要替换的值. 如果箭头左侧被一个 regex 匹配, 则在闭包中可以访问到结果匹配对象.

[`S05-transliteration/with-closure.t lines 5–63`](https://github.com/perl6/roast/blob/master/S05-transliteration/with-closure.t#L5-L63)

# 替换
[`S05-substitution/subst.t lines 7–211`](https://github.com/perl6/roast/blob/master/S05-substitution/subst.t#L7-L211)
[`S05-substitution/match.t lines 7–33`](https://github.com/perl6/roast/blob/master/S05-substitution/match.t#L7-L33)

也有 `m//` 和 `s///`形式的方法:

```perl6
$str.match(/pat/);
$str.subst(/pat/, "replacement");
$str.subst(/pat/, {"replacement"});
$str.=subst(/pat/, "replacement");
$str.=subst(/pat/, {"replacement"});
```

`.match` 和 `.subst` 方法支持 `m//` 和 `s///` 的副词作为具名参数, 所以你可以写成:

```perl6
$str.match(/pat/, :g)
```

这等价于

```perl6
$str.comb(/pat/, :match)
```

这儿没有语法糖, 所以为了获得 replacement 延时计算, 你必须把它放到一个闭包中. 只有在 quotelike 形式才提供有语法糖. 首先, 有一个标准的 "triple quote" 形式:

```perl6
s/pattern/replacement/
```

只有非括号字符才能被用于"triple quote"中.  右侧总是被当作在双引号中求值, 不管所选的引号是什么.

就像 Perl 5, 也支持括号形式, 但是不像 Perl 5, Perl 6 只在模式周围使用括号. replacement 被指定为就像普通的 item 赋值一样, 使用普通的引号 rules.  要在右侧选择你自己的引号, 只使用其中的一种 q 形式就好.  上面的替换等价于:
[`S05-substitution/subst.t lines 223–323`](https://github.com/perl6/roast/blob/master/S05-substitution/subst.t#L223-L323)

```perl6
s[pattern] = "replacement"
```

或

```perl6
s[pattern] = qq[replacement]
```

这不是普通的赋值, 因为每次替换一匹配,右侧就会被求值一次. 这因此被称为形式转换. 它会被作为一段创建了动态作用域而非词法作用域的代码被调用. (你也可以把 thunk 看作一个使用当前词法作用域的闭包).实际上, 使用下面这个也没有影响:

```perl6
s[pattern] = { doit }
```

因为那会把闭包替换成字符串.

任何标量赋值操作符都能被使用; 那个替换宏知道怎么转换
[`S05-substitution/subst.t lines 324–480`](https://github.com/perl6/roast/blob/master/S05-substitution/subst.t#L324-L480)

```perl6
$target ~~ s:g[pattern] op= expr
```

为如下这样:

```perl6
$target.=subst(rx[pattern], { $() op expr }, :g)
```

`s///` 的实际实现必须返回一个 Match 对象以使智能匹配能正确工作.  上面的重写只返回了改变了的字符串.

所以, 举个例子, 你可以把每个美元符号的数量乘以 2:

```perl6
s:g[$ <( \d+ )>] *= 2
```

(当然, 优化比实际调用要快)

你会注意到上面一个例子, 由于匹配的结果, 替换只发生在”正式的”字符串上, 即,  `$/.from` 和 `$/.to` 位置之间的那部分字符串.( 这里我们使用   `<(…)>` pair  显式地设置了那些, 否则,我们可能必须使用向前查看来匹配 `$`)

请注意,  `:ii`/`:samecase` 和 `:mm`/`:samemark`  开关实际上是一根绳子上的两个蚂蚱, 当编译器给 quote-like 形式的开关脱去语法糖时, 它会把语义分配给模式和替换部分.  即, 作用于替换上的 `:ii`  隐含了模式上的  `:i`,    `:mm` 隐含了 `:m`.

```perl6
s:ii/foo/bar/
s:mm/boo/far/
```

不是:

```perl6
.subst(/foo/, 'bar', :ii)   # WRONG
.subst(/boo/, 'far', :mm)   # WRONG
```

而是:

```perl6
.subst(rx:i/foo/, 'bar', :ii)   # okay
.subst(rx:m/boo/, 'far', :mm)   # okay
```

它专门不要求实现把正则表达式作为关于大小写和标记的通用实现。追溯重新编译是有害的。如果一个实现确实执行懒惰的一般的大小写和标记语义，它对于依赖于它的程序来说是错误的和不可移植的。 (天了噜, 这究竟怎么翻译?!)

`s///` 和 `.subst` 形式的不同之处在于, `.subst` 返回修改过的字符串(因此不能用作智能匹配器), `s///` 形式要么返回一个  `Match` 对象, 来标示智能匹配成功了, 要么返回一个 `Nil` 值标示没有成功.  

同样地, 对于 `m:g` 匹配和 `s:g` 替换, 可能会找到多个匹配. 这些结构必须在智能匹配时仍旧能继续工作然后返回一个匹配列表. 幸运的是, `List` 是一个知名的类型, 匹配器能返回这个类型来标示匹配成功或失败. 所以这些结只是返回一个成功匹配的列表, 如果没有出现匹配则它会是一个空的列表(因此匹配失败).

# 位置匹配, 固定宽度类型

- 在通常情况下, 要锚定到一个特定的位置你可以使用  `<at($pos)>` 断言, 来说当前位置和你提供的位置对象是相同的. 你可以通过 `:c` 和 `:p` 修饰符设置当前的匹配位置.

然而, 请记住在 Perl 6 中, 字符串位置通常不是整数, 而是指向字符串中特定位置的对象, 不管你使用字节或代码点还是字形来计数. 如果使用的是整数, `at` 断言就会假设你意指当前词法作用域的 Unicode 级别, 假设这个整数是以某种方式在同一个这样的词法作用域中生成的. 如果这在当前字符串允许的 Unicode 抽象级别之外,会抛出异常. 查看 `$02` 获取字符串位置的更多讨论.

-  `Buf` 类型基于固定宽度的单元格, 因此处理整数位置刚刚好, 并把它们当作数组切片. 特别地, `buf8` (也是熟知的 `buf`) 就是老式的字节字符串. 在没有显式修饰符询问数组的值将被看作某种诸如 UTF-32的特殊编码时,  匹配 `Buf` 类型被约束为 ASCII 语义.(这对于那些跟 `Buf` 同构的紧致数组也适用). `Buf` 类型中的位置总是整数, 基本数组的的每个单元格计数 1. 注意 `from` 和 `to` 位置是在元素之间的. 如果匹配一个紧致的数组 `@foo`, 最后的位置 42 标示 `@foo[42]` 是未被包含的首个元素. (*翻译的真辛苦, 还不知所云, 坚持把!*)

# 匹配非字符串
[`S05-nonstrings/basic.t lines 7–46`](https://github.com/perl6/roast/blob/master/S05-nonstrings/basic.t#L7-L46)

- 任何可以绑到字符串上的东西都可以用  regex 匹配. 这个特性对输入流特别有用:

```perl6
my $stream := cat $fh.lines;       # tie scalar to filehandle
# and later...
$stream ~~ m/pattern/;             # match from stream
```

- 任何混合了字符串或对象的非紧致数组能匹配一个 regex, 只要你使用 `Str` 接口把它们呈现为对象, 这不妨碍其它对象含有诸如 `Array` 之类的其它接口. 正常地, 你会使用 `cat` 来生成这样的对象:

```perl6
@array.cat ~~ / foo <,> bar <elem>* /;
```

那个特殊的 `<,>` subrule 匹配元素之间的边界.   `<elem>` 断言匹配任何单独的数组元素.  整个 `<elem>` 元素就是点元字符的等价物.

如果数组元素是字符串, 事实上它们被连接成单个逻辑字符串. 如果数组元素是  tokens 或 其它这样的对象, 那么对象必须为这样的 subrules 提供合适的方法来匹配. 将字符串匹配断言和未提供字符串化查看的对象进行匹配会导致断言失败. 然而, 纯对象列表可以被解析, 只要匹配(包括任何 subrules)把自身约束为这样的断言:

```perl6
<.isa(Dog)>
<.does(Bark)>
<.can('scratch')>
```

把对象和字符混合在数组中也是可以的, 只要它们在不同的元素中. 然而你不能在字符串中嵌入对象. 当然, 任何对象都可以假装它是一个字符串元素, 所以, `Cat` 对象可以用作子字符串, 使用与主字符串中同样的约束.

请注意,匹配数组时,  `.from` 和 `.to` 都会返回不透明对象的警告, 在一个特殊的位置, 这个位置既反映在数组中的位置, 又在数组的字符串中的位置. 不要期望使用这样的值来做匹配,  你也不要期望能跨越元素边界来提取子字符串[猜测:难道不是吗?] :PS  简直无法翻译!

- 要匹配数组中的每一个元素, 使用 hyper 操作符:

```perl6
 @array».match($regex);
```

- 要匹配数组中的任意元素, 使用普通的智能匹配就足够了:

```perl6
@array ~~ $regex;
```

#  `$/` 在什么时候是有效的


为了提供实施自由, `$/` 变量并不能保证被定义, 直到模式到达需要它的序列点.(例如, 完成了匹配, 或者调用了嵌入的闭包, 或者计算一个 Perl 表达式作为它的参数的 submatch.)  在 regex 代码里面,  `$/` 未被正式定义, 引用 `$0` 或其它变量可能被编译产生当前值, 而不用引用 `$/`.  同样地,  引用 `$<foo>` 并不意味着 regex 中就有 `$/<foo>` . 在执行匹配期间, 当前匹配状态实际上存储在词法作用域到匹配部分的 `$¢` 变量中, 但是它不保证和 `$/` 对象的表现一样, 因为 `$/` 是 `Match` 类型, 而匹配状态的类型是从 `Cursor` 派生出来的.

在任何情况下, 这对于用户的简单匹配都是透明的; 在 regex 代码之外(还有 regex 的闭包中) `$/` 变量保证代表那个点的匹配状态. 即,  一般的 Perl 代码总是依靠 `$<foo>` 表示 `$/<foo>`,  依靠 `$0` 表示 `$/[0]` , 不论代码是嵌入在 regex 的闭包中还是在 regex 的外面, 在整匹配之后.

# 作者

```
Damian Conway <damian@conway.org>
Allison Randal <al@shadowed.net>
Patrick Michaud <pmichaud@pobox.com>
Larry Wall <larry@wall.org>
Moritz Lenz <moritz@faui2k3.org>
Tobias Leich <email@froggs.de>
```
