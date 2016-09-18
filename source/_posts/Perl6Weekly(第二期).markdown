---
title: Perl6 Weekly(二)
date: 2015-08-22
tags: my scope
categories: Perl 6
comments: true
---


## `:my $foo` 的作用域和用途

在 *regex*、*token* 或 *rule* 中, 定义像下面这样的变量是可能的:

```perl6
token directive {
    :my $foo = "in command";
    <command> <subject> <value>?
}
```

在中提到了一点有关该变量的东西, 我引用过来:

> 任何 grammar regex 实际上是一种`方法`, 并且你可以在这样一个子例程中使用一个冒号跟着任何作用域声明符来声明一个变量, 这些声明符包括 `my`, `our`, `state` 和 `constant` (作为类似的声明符, temp 和 let 也能被识别). 单个语句(直到结尾的分号或行末尾的闭括号为止) 被解析为普通的 Perl 6 代码:

```perl6
token prove-nondeterministic-parsing {
    :my $threshold = rand;
    'maybe' \s+ <it($threshold)>
}
```

有谁能解释下这段代码的应用场景吗？

### what scope does `:my $foo;` have?

`:my $foo` 在它所出现的 rule/token/regex 中拥有词法作用域(lexical scope)。你所得到的作用域要么很大要么很小:

```perl6
grammar g {
    regex r1 {
        { my $foo; ...} # `$foo` 在该 block 的结尾超出作用域。
        ...
        { say $foo;   } # `$foo` 不在作用域中。
    }
}

grammar i {
    my $foo;
    regex r1   { ... } # 在 `r1` 内部, `$foo` 被识别出。
    ...
    regex r999 { ... } # 但是在 r999 中也是。
}
```

### 它的用途?

使用 `:my $foo;` 形式的变量声明以在 rule/token/regex 中声明本地作用域的变量, 如果没有进一步的声明, 那么这些变量能在 rule/token/regex 中的任何地方通过所声明的名字来引用。举个例子, 你可以看看 Rakudo 的 Grammar.nqp 源代码中的 [`token babble`](http://github.com/rakudo/rakudo/blob/nom/src/Perl6/Grammar.nqp#L108) 中声明的 `@extra_tweaks` 变量的用法。

使用 `:my $*foo;` 形式的变量声明来声明动态的词法变量。动态变量能够, 在没有进一步声明的情况下, 在闭合词法作用域和闭合动态作用域中通过它们声明的名字来引用。作为说明, 请查看 [the declaration of `@*nibbles` in Rakudo's Grammar module](http://github.com/rakudo/rakudo/blob/nom/src/Perl6/Grammar.nqp#L4861) 和 [its use in Rakudo's Actions module](http://github.com/rakudo/rakudo/blob/nom/src/Perl6/Actions.nqp#L8943) 。

### 一般的使用场景

在 [regular expressions](https://en.wikipedia.org/wiki/Regular_expression) 中一般不使用 `:…` 风格的声明。`:...;` 结构通常用在特别复杂和庞大的 grammars 中。对于这些使用场景, 依靠 Perl 6 的正则表达式和闭包的一致性是合适的。正是这使得 rule/token/regex 级别的 `:...;` 变量声明变得正当。

### Regexes 和 closures 的一致性

很多 grammars 都是[上下文有关的](http://trevorjim.com/python-is-not-context-free/).

Perl 6 使 regexes 和 closures 统一了:

```perl6
say Regex.^mro; (Regex) (Method) (Routine) (Block) (Code) ...
```

mro 是方法解析顺序, 这足以说明 regex 实际上是一种特殊类型的方法(就像方法是一种特殊类型的子例程一样)。



## [Perl6: is there a phaser that runs only when you fall out of a loop?](http://stackoverflow.com/questions/36117329/perl6-is-there-a-phaser-that-runs-only-when-you-fall-out-of-a-loop)

```perl6
#!/usr/bin/env perl6

use v6.c;

ROLL:
for 1..10 -> $r {
    given (1..6).roll {
        when 6 {
            say "Roll $r: you win!";
            last ROLL;
        }
        default {
            say "Roll $r: sorry...";
        }
    }
    LAST {
        say "You either won or lost - this runs either way";
    }
}

```

更优雅的写法:

```perl6
constant N = 5;
for flat (1..6).roll xx * Z 1..N -> $_, $n {
    print "roll $n: $_ ";

    when 6 {
        put "(won)";
        last;
    }

    default {
        put "(lost)";
    }

    LAST {
        print "result: ";
        when 6 { put "winner :)" }
        default { put "loser :(" }
    }
}
```



## [怎么从命令行传递一个复数给 sub MAIN?](http://stackoverflow.com/questions/35872082/how-do-i-pass-a-complex-number-from-the-command-line-to-sub-main)

```perl6
#!/usr/bin/env perl6

use v6.c;

sub MAIN($x)
{
    say "$x squared is { $x*$x }";
}
```

我要在命令行中传递一个复数给 MAIN:

> ```
> % ./square i
> ```

```
Cannot convert string to number: base-10 number must begin with valid digits or '.' in '⏏i' (indicated by ⏏)
  in sub MAIN at ./square line 7
  in block <unit> at ./square line 5

Actually thrown at:
  in sub MAIN at ./square line 7
  in block <unit> at ./square line 5
```

当我把脚本变为:

```perl6
#!/usr/bin/env perl6

use v6.c;

sub MAIN(Complex $x)
{
    say "$x squared is { $x*$x }";
}
```

它竟然彻底罢工了:

```perl
% ./square i
Usage:
  square <x>

% ./square 1
Usage:
  square <x>
```

一种方法是使用 [Coercive type declaration](http://design.perl6.org/S02.html#Coercive_type_declarations) (强制类型声明), 从 Str 到 Complex:

```perl6
sub MAIN(Complex(Str) $x) {
    say "$x 的平方为 { $x * $x }";
}
```

那么:

```
% ./squared.pl 1
1+0i 的平方为 1+0i
% ./squared.pl 1+2i
1+2i 的平方为 -3+4i
```

但是:

```
$ ./test.pl6 2
Usage:
  ./test.p6 <x> 
```

所以你真正需要的是把其它 Numeric 类型强转为 Complex 类型:

```perl6
#!/usr/bin/env perl6

use v6.c;

sub MAIN ( Complex(Real) $x ) {
    say "$x squared is { $x*$x }";
}
```

我使用 Real 而非 Numeric, 因为 Complex 已经涵盖了其它的了。



## [Blessing a Hash into an object](https://www.reddit.com/r/perl6/comments/4aoi82/blessing_a_hash_into_an_object/)

为什么我写的这段代码不对呢？

```perl6
class WordCount {
  has %!words; # Tried with both . and !
  method new($string) {
    my %words;
    my @sentence = split(/\s+/, $string);
    for @sentence -> $word {
      %words{$word}++;
    }
    return self.bless(:%words);
  }

  method sayCounts() {
    my @keys = keys(%!words);
    for @keys -> $key {
      say $key ~ " " ~ %!words{$key};
    }
  }
}

sub MAIN {
  my $sentence = "the boy jumped over the dog";
  my $wordCount = WordCount.new($sentence);
  $wordCount.sayCounts();
}
```

Perl6-ify:

```perl6
use v6;

class WordCount {
  has Int %.words is default(0);

  method new($string) {
    my Int %words;
    for $string.split(/\s+/) -> $word {
      %words{$word}++;
    }

    self.bless(:%words)
  }

  method gist {
    %.words.map({.value ~ " " ~ .key}).join("\n")
  }
}

my $word-count = WordCount.new('the boy jumped over the dog');
say $word-count;
```

散列中的每一项都是一个 **Pair**:

```perl6
my %w = a => 1;
%w.map({ say $_.^name }) # OUTPUT«Pair␤»
```

所以:

```perl6
%.words.map({.value ~ " " ~ .key}).join("\n")
```

等价于:

```perl6
%.words.kv.map( -> $word, $count { "$word $count" } ).join("\n")
```

你还可以使用 sub-signature(子签名)来解构 `.map` 提供的 `Pair`：

```perl6
%.words.map( -> (:key($word), :value($count)) { "$word $count" } ).join("\n")
```
