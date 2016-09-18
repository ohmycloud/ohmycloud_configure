---
title: Perl 6 中的特性(二)
date: 2015-03-16
tags:
  - 特性
  - Examples
categories: Perl 6
comments: true
---


## Set
---

```perl6
my $keywords = set <if for unless while>; # create a set

sub has-keyword(*@words) {
    for @words -> $word {
        return True if $word (elem) $keywords; # 依次检查数组中的元素是否属于集合 $keywords
    }
    False;
}

say has-keyword 'not', 'one', 'here';       # False
say has-keyword 'but', 'here', 'for';       # True

```



## Series Operator
---


```perl6
my @a=<A G C T>;
my $x=@a;
for 1 ... * -> $a {  (( [X~] $x xx $a )).join(',').say;last if $a==4;   };
```



```perl6
# Works with: Rakudo Star version 2010.08
for 10 ... 0 {
    .say;
}

```



```perl6
use v6;
my $file = open 'flip_flop.txt';
for $file.lines -> $line {
say $line if !($line ~~ m/^\;/ ff $line ~~ m/^\"/);
}
```


flip_flop.txt 内容如下：
```text
; next is some lines to skip,include this line
fuck fuck fuck
dam dam dam
mie mie mie
" next is subject
There is more than one way to do it
                                -- Larry Wall

We hope Perl6 is wrote by the hole Socfilia
                                -- Larry Wall
; next is some lines to skip,include this line
fuck fuck fuck
dam dam dam
mie mie mie
" next is subject
programming is hard,Let's go shopping
                               -- Larry Wall
Ruby is Another Perl6
                               -- Larry Wall
```

输出：
```text
There is more than one way to do it
                                -- Larry Wall
We hope Perl6 is wrote by the hole Socfilia
                                -- Larry Wall
programming is hard,Let's go shopping
                               -- Larry Wall
Ruby is Another Perl6
                               -- Larry Wall
```



```perl6
for 1..20 {.say if $_==9 ff $_==16}
say '-' x 10;
for 1..20 {.say if $_==9 ^ff $_==16}
say '-' x 10;
for 1..20 {.say if $_==9 ff^ $_==16}
say '-' x 10;
for 1..20 {.say if $_==9 ^ff^ $_==16}

输出：
9
10
11
12
13
14
15
16
----------
10
11
12
13
14
15
16
----------
9
10
11
12
13
14
15
----------
10
11
12
13
14
15
```



```perl6
# Works with: Rakudo Star version 2010.08
loop {
    say 'SPAM';
}

# In addition, there are various ways of writing lazy, infinite lists in Perl 6:
print "SPAM\n" xx *;      # repetition operator
print "SPAM\n", ~* ... *; # sequence operator
map {say "SPAM"}, ^Inf;   # upto operator

```




## Grammars
---


```perl6
use v6;

BEGIN {
    @*INC.push('/Volumes/WORK/1-Books/3-Perl6/examples/笔记/Grammars');
}
use Add1;

my @experssions = (
    "2 + 3",
    "2 + 4 ",
    "2 + 3 x",
    "2 +",
    "2 3",
    "2 - 3",
);

for @experssions -> $exp {
    print $exp, " ";
    my $result = Add1.parse($exp);
    say $result ?? 'OK' !! 'NOT OK';
    CATCH {
        say "exception received: $!";
    }
}

```



```perl6
use v6;

BEGIN {
    @*INC.push('/Volumes/WORK/1-Books/3-Perl6/examples/笔记/Grammars');
}
use Add2;

my @experssions = (
    "2 + 3",
    "2 + 4 ",
    "2 + 3 x",
    "2 +",
    "2 3",
    "2 - 3",
);

for @experssions -> $exp {
    print $exp, " ";
    my $result = Add2.parse($exp);
    say $result ?? 'OK' !! 'NOT OK';
    CATCH {
        say "exception received: $!";
    }
}

```



```perl6
grammar CardGame {

    rule TOP { ^ <deal> $ }

    rule deal {
        <hand>+ % ';'
    }

    rule hand { [ <card> ]**5 }
    token card {<face><suit>}

    proto token suit {*}
    token suit:sym<♥>  {<sym>}
    token suit:sym<♦>  {<sym>}
    token suit:sym<♣>  {<sym>}
    token suit:sym<♠>  {<sym>}

    token face {:i <[2..9]> | 10 | j | q | k | a }
}

say CardGame.parse("2♥ 5♥ 7♦ 8♣ 9♠");
say CardGame.parse("2♥ a♥ 7♦ 8♣ j♥");

```



```perl6
grammar CardGame {

    rule TOP { ^ <deal> $ }

    rule deal {
       :my %*PLAYED = ();
       <hand>+ % ';'
    }

    rule hand { [ <card> ]**5 }
    token card {<face><suit>}

    proto token suit {*}
    token suit:sym<♥>  {<sym>}
    token suit:sym<♦>  {<sym>}
    token suit:sym<♣>  {<sym>}
    token suit:sym<♠>  {<sym>}

    token face {:i <[2..9]> | 10 | j | q | k | a }
}

class CardGame::Actions {
    method card($/) {
       my $card = $/.lc;
       say "Hey, there's an extra $card"
           if %*PLAYED{$card}++;
   }
}

my $a = CardGame::Actions.new;
say CardGame.parse("a♥ a♥ 7♦ 8♣ j♥", :actions($a));
# "Hey there's an extra a♥"
say CardGame.parse("a♥ 7♥ 7♦ 8♣ j♥; 10♥ j♥ q♥ k♥ a♦",
                   :actions($a));
# "Hey there's an extra j♥"


```



```perl6
﻿use v6;

my %dict;

grammar WordPairs {
    token TOP { <word-pair>* }
    token word-pair { (\S*) ' ' (\S*) "\n" }
}

class WordPairsActions {
    method word-pair($/) { %dict{$0}.push($1) }
}

my $match = WordPairs.parse("{@*ARGS[0]}".IO.slurp, :actions(WordPairsActions));
say ?$match;

say "The pairs count of the key word \"her\" in wordpairs.txt is {%dict{"her"}.elems}";
```



```perl6
﻿use v6;

my $file=open "test.txt", :r;

my %dict;
my $line;

repeat {
    $line=$file.get;
    my ($p1,$p2)=$line.split(' ');
    if ?%dict{$p1} {
        %dict{$p1} = "{%dict{$p1}} {$p2}".words;
    } else {
        %dict{$p1} = $p2;
    }
} while !$file.eof;

## Test
say "The pairs count of the key word \"was\" in wordpairs.txt is {%dict{"was"}.elems}";
```



```perl6
﻿grammar CSV {
    token TOP { [ <line> \n? ]+ }
    token line {
        ^^            # Beginning of a line
        <value>* % \, # Any number of <value>s with commas in `between` them
        $$            # End of a line
    }
    token value {
        [
        | <-[",\n]>     # Anything not a double quote, comma or newline
        | <quoted-text> # Or some quoted text
        ]*              # Any number of times
    }
    token quoted-text {
        \"
        [
        | <-["\\]> # Anything not a " or \
        | '\"'     # Or \", an escaped quotation mark
        ]*         # Any number of times
        \"
    }
}
# method parse($str, :$rule = 'TOP', :$actions) returns Match:D
say "Valid CSV file!" if CSV.parse( q:to/EOCSV/ );
    Year,Make,Model,Length
    1997,Ford,E350,2.34
    2000,Mercury,Cougar,2.38
    EOCSV

say CSV.parse( q:to/EOCSV/, 'line', :$actions );
    Year,Make,Model,Length
    1997,Ford,E350,2.34
    2000,Mercury,Cougar,2.38
    EOCSV
```



```perl6
﻿grammar MyGrammar {
    token chunk {
        { say "chunk: called" }
        ^^
        { say "chunk: found start of line" }
        (\S+)
        { say "chunk: found first identifier: $0" }
		#(\s*)
		#{say "chunk: found space"}
        \=
        { say "chunk: found =" }
		#(\s*)
		#{say "chunk: found space"}
        (\S+) $$
    }
}

say ?MyGrammar.parse("foo = bar", :rule<chunk>);

# output:
#
# chunk: called
# chunk: found start of line
# chunk: found fist identifer: foo
# False

#You can see that the rule matched the start of the line, and foo, but not the equals sign. What's between the two? A space. For which there is no rule to match it. Making chunk a rule instead of a token fixes this problem.
# 因为 (\S+)后面有一个空格, \= 后面也有个空格, rule 能识别空格

# E:\1-技术书籍\Perl6\examples\Grammars>perl6 -e "'aabcd' ~~ /^ (.*){say $0.Str} b/"
#aabcd
#aabc
#aab
#aa
```



```perl6
﻿grammar Perl6VariableNames {

    token variable {
        <sigil> <name>
    }

    #token sigil {
    #    '$' | '@' | '&' | '%' | '::'
    #}

    # 使用 proto
	proto token sigil {*}
    token sigil:sym<$>  { <sym> }
    token sigil:sym<@>  { <sym> }
    token sigil:sym<%>  { <sym> }
    token sigil:sym<&>  { <sym> }
    token sigil:sym<::> { <sym> }

	# [ ... ] are non-capturing groups
	token name {
        <identifier>
        [ '::' <identifier> ] *
    }
	# 标识符以字母开头
    token identifier {
        <alpha> \w+
    }
}

my $match = Perl6VariableNames.parse("@array",:rule('variable'));
say $match;

grammar SigilRichPerl6 is Perl6VariableNames {
    token sigil:sym<ħ> { <sym> } # physicists will love you
}

my $rich = SigilRichPerl6.parse("ħarray",:rule('variable'));
say $rich;

grammar LowBudgetPerl6 is Perl6VariableNames {
    token sigil:sym<$> { '¢' }
}

my $money = LowBudgetPerl6.parse('$array',:rule('variable'));
say $money;
```



```perl6
grammar StationDataParser {
    token TOP          { ^ <keyval>+ <observations> $             }
    token keyval       { $<key>=[<-[=]>+] '=' \h* $<val>=[\N+] \n }
    token observations { 'Obs:' \h* \n <observation>+             }
    token observation  { $<year>=[\d+] \h* <temp>+ %% [\h*] \n    }
    token temp         { '-'? \d+ \. \d+                          }
}

class StationData {
    has $.name;
    has $.country;
    has @.data;

    submethod BUILD(:%info (:Name($!name), :Country($!country), *%), :@!data) {
    }
}

class StationDataActions {
    method TOP($/) {
        make StationData.new(
            info => $<keyval>.map(*.ast).hash,
            data => $<observations>.ast
        );
    }

    method keyval($/) {
        make ~$<key> => ~$<val>;
    }
    method observations($/) {
        make $<observation>.map(*.ast).grep(*.value.none <= -99);
    }
    method observation($/) {
        make +$<year> => $<temp>.map(*.Num);
    }
}

say StationDataParser.parse( q:to/EOCSV/, :actions(StationDataActions)).ast
Name= Jan Mayen
Country= NORWAY
Lat=   70.9
Long=    8.7
Height= 10
Start year= 1921
End year= 2009
Obs:
1921 -4.4 -7.1 -6.8 -4.3 -0.8  2.2  4.7  5.8  2.7 -2.0 -2.1 -4.0  
1922 -0.9 -1.7 -6.2 -3.7 -1.6  2.9  4.8  6.3  2.7 -0.2 -3.8 -2.6  
2008 -2.8 -2.7 -4.6 -1.8  1.1  3.3  6.1  6.9  5.8  1.2 -3.5 -0.8  
2009 -2.3 -5.3 -3.2 -1.6  2.0  2.9  6.7  7.2  3.8  0.6 -0.3 -1.3
EOCSV
```



```perl6
# file lib/JSON/Tiny/Grammar.pm

 grammar JSON::Tiny::Grammar {
     rule TOP      { ^[ <object> | <array> ]$ }
     rule object   { '{' ~ '}' <pairlist>     }
     rule pairlist { <pair>* % [ \, ]         }
     rule pair     { <string> ':' <value>     }
     rule array    { '[' ~ ']' [ <value>* % [ \, ] ] }

 proto token value { <...> };

 token value:sym<number> {
     '-'?
     [ 0 | <[1..9]> <[0..9]>* ]
     [ \. <[0..9]>+ ]?
     [ <[eE]> [\+|\-]? <[0..9]>+ ]?
 }

 token value:sym<true>   { <sym>    };
 token value:sym<false>  { <sym>    };
 token value:sym<null>   { <sym>    };
 token value:sym<object> { <object> };
 token value:sym<array>  { <array>  };
 token value:sym<string> { <string> }

 token string {
     \" ~ \" [ <str> | \\ <str_escape> ]*
 }

 token str {
     [
         <!before \t>
         <!before \n>
         <!before \\>
         <!before \">
         .
     ]+
     # <-["\\\t\n]>+
 }

 token str_escape {
     <["\\/bfnrt]> | u <xdigit>**4
 }

 }


 # test it:
 my $tester = '{
     "country": "Austria",
     "cities": [ "Wien", "Salzburg", "Innsbruck" ],
     "population": 8353243
 }';

 if JSON::Tiny::Grammar.parse($tester) {
     say "It's valid JSON";
 } else {
     # TODO: error reporting
     say "Not quite...";
 }
```



```perl6
grammar JSON::Tiny::Grammar {
     rule TOP      { ^[ <object> | <array> ]$ }
     rule object   { '{' ~ '}' <pairlist>     }
     rule pairlist { <pair>* % [ \, ]         }
     rule pair     { <string> ':' <value>     }
     rule array    { '[' ~ ']' [ <value>* % [ \, ] ] }

 proto token value { <...> };

 token value:sym<number> {
     '-'?
     [ 0 | <[1..9]> <[0..9]>* ]
     [ \. <[0..9]>+ ]?
     [ <[eE]> [\+|\-]? <[0..9]>+ ]?
 }

 token value:sym<true>   { <sym>    };
 token value:sym<false>  { <sym>    };
 token value:sym<null>   { <sym>    };
 token value:sym<object> { <object> };
 token value:sym<array>  { <array>  };
 token value:sym<string> { <string> }

 token string {
     \" ~ \" [ <str> | \\ <str_escape> ]*
 }

 token str {
     [
         <!before \t>
         <!before \n>
         <!before \\>
         <!before \">
         .
     ]+
     # <-["\\\t\n]>+
 }

 token str_escape {
     <["\\/bfnrt]> | u <xdigit>**4
 }

 }


 # test it:
 my $tester = '{
     "country": "Austria",
     "cities": [ "Wien", "Salzburg", "Innsbruck" ],
     "population": 8353243
 }';

 if JSON::Tiny::Grammar.parse($tester) {
     say "It's valid JSON";
 } else {
     # TODO: error reporting
     say "Not quite...";
 }




 class JSON::Tiny::Actions {
    method TOP($/)      { make $/.values.[0].ast              }
    method object($/)   { make $<pairlist>.ast.hash           }
    method pairlist($/) { make $<pair>>>.ast                   }
    method pair($/)     { make $<string>.ast => $<value>.ast  }
    method array($/)    { make [$<value>>>.ast]                }
    method string($/)   { make join '', $/.caps>>.value>>.ast }

 # TODO: make that
 # make +$/
 # once prefix:<+> is sufficiently polymorphic
method value:sym<number>($/) { make try $/       }
method value:sym<string>($/) { make $<string>.ast }
method value:sym<true>  ($/) { make Bool::True    }
method value:sym<false> ($/) { make Bool::False   }
method value:sym<null>  ($/) { make Any           }
method value:sym<object>($/) { make $<object>.ast }
method value:sym<array> ($/) { make $<array>.ast  }

method str($/)               { make ~$/           }

method str_escape($/) {
    if $<xdigit> {
        make chr(:16($<xdigit>.join));
    } else {
        my %h = '\\' => "\\",
        'n' => "\n",
        't' => "\t",
        'f' => "\f",
        'r' => "\r";
        make %h{$/};
    }
  }
}

my $actions = JSON::Tiny::Actions.new();
my $data_structure = JSON::Tiny::Grammar.parse($tester, 'TOP', :$actions);
say $data_structure;
```



```perl6
﻿use v6;

grammar KeyValuePairs {
    token TOP {
	    [ <pair> \n+ ]*
	}

	token ws { \h* }

	rule pair {
	    <key=.identifier> '=' <value=.identifier>
	}

	token identifier { \w+ }
}

class KeyValuePairsActions {
    method identifier($/)  { make ~$/                   }
	method pair      ($/)  { make ~$<key> => ~$<value>  }
	method TOP       ($/)  { make $<pair>>>.made        }
}

my $string = q:to/EOI/;
second=b
hits=42
perl=6
EOI

my $actions = KeyValuePairsActions.new;
my $match = KeyValuePairs.parse($string, :$actions).made;

for @$match -> $p {
    say "key: $p.key()\tValue: $p.value()";
}

```



```perl6
#use Module::Name::Actions;
grammar Legal-Module-Name {
  token TOP {

	# identifier followed by zero or more separator identifier pairs
    ^ <identifier> [<separator><identifier>] ** 0..* $
  }

  token identifier  {
    # leading alpha or _ only
    <[A..Za..z_]>
    <[A..Za..z0..9]> ** 0..*
  }

  token separator  {
    '::' # colon pairs
  }
}

class Module::Name::Actions {
  method TOP($/)
  { make $/.values.ast ~ '-----';
    if $<identifier>.elems > 5
    {
      warn 'Module name has a lot of identifiers, consider simplifying the name';
    }
  }
}

my $proposed_module_name = 'Superoooo::Newoooo::Moduleooooooooooo';

my $actions = Module::Name::Actions.new();
my $match_obj = Legal-Module-Name.parse($proposed_module_name, :actions($actions));

say $match_obj.Str;
```



```perl6
use Grammar::Debugger;
use Grammar::Tracer;

# 第一个 Grammar, 修改了很多次, 借助于 Grammar::Debugger 和 Grammar::Tracer 方便看出 Grammar 在哪里失败.
# TOP 里面添加了 ^ 和 $ 限制时, 其后面的 token 和 rule 不能在添加 ^ 和 $, 否则匹配失败

use v6;
grammar Markdown::Toc {
    # token TOP   is breakpoint    {^ \s* <section>* $}
	token TOP    {^ \s* <section>* $}
	token section   {
	    <sname>  <ws> \n
		<lines>*
	    <subsection>+
	}

    token subsection  {
    	<subsname> <ws> \n
        <lines>*
        <s2section> *
	}

    token s2section {
	    <s2name> <ws> \n
		<lines>*
    }

	token sname          { <sigil2>   <ws> <snumber=.hnumber>  <ws> <shead=.hline>     }
	token subsname       { <sigil3>   <ws> <subnumber=.number> <ws> <shline=.hline>    }
	token s2name         { <sigil4>   <ws> <s2number=.number>  <ws> <s2hline=.hline>   }
	token lines          {
	    [
		    <!after '#' ** 2..* >
            \N
        ]+
	    \n
	}
	token number {
	    \d+ % \.
	}

	token hline {
	    \N+
	}
	token hnumber { \w+      }
	token ws      { \h*      }
	token sigil2  { '#' ** 2 }
	token sigil3  { '#' ** 3 }
	token sigil4  { '#' ** 4 }
}

my $str = q:to/EOF/;
## 第四章 子例程和签名


一个子例程就是一段执行特殊任务的代码片段。它可以对提供的数据（`实参`）操作，并产生结果（返回值）。子例程的签名是它`所含的参数`和它产生的`返回值`的描述。从某一意义上来说，第三章描述的操作符也是Perl 6用特殊方式解释的子例程。

### 4.1.0 申明子例程

 一个子例程申明由几部分组成。首先， `sub `表明你在申明一个子例程，然后是可选的子例程的名称和`可选的签名`。子例程的主体是一个用花括号扩起来的代码块。
默认的，子例程是本地作用域的，就像任何使用 `my` 申明的变量一样。这意味着，一个子例程只能在它被申明的作用域内被调用。使用 `our` 来申明子例程可以使其在`当前包`中可见。
EOF


```perl6
class Markdown::Toc::Actions {
	method s2section($/)        {
	    my $first = ~$<s2name><s2number>;
	    my $second = ~$<s2name><s2hline>;
		my $remove_dot = $first.subst(rx/\./,'',:g);
		my $remove_space = $second.subst(rx/\s+/,'-',:g);
		make '    - '~'['~$first~' '~$second~']' ~ '(#'~$remove_dot~$remove_space ~ ')' => $<s2section>>>.made;
	}

	method subsection($/) {
	    my $first = ~$<subsname><subnumber>;
	    my $second = ~$<subsname><shline>;
		my $remove_dot = $first.subst(rx/\./,'',:g);
		my $remove_space = $second.subst(rx/\s+/,'-',:g);
		make '  - '~'['~$first~' '~$second~']' ~ '(#'~$remove_dot~$remove_space ~ ')'	=> $<s2section>>>.made;  
	}

    method section($/)    {
	    my $first = ~$<sname><snumber>;
		my $second = ~$<sname><shead>;
		#my $remove_dot = $first.subst(rx/\./,'',:g);
		my $remove_space = $second.subst(rx/\s+/,'',:g);
	    make '- '~'['~$first~' '~$second~']' ~ '(#'~$first~ '-'~$remove_space ~ ')'  => $<subsection>>>.made;
	}

    method TOP($/)    { make $<section>>>.made;                                                            }

}

my $actions = Markdown::Toc::Actions.new;
my $match  = Markdown::Toc.parse($str, :$actions).made;
#say $match.Str;

for @$match -> $p {
    say $p.key();
    for $p.value() -> $v {
        for $v -> $n {
		   .say for $n.hash().keys();
		   for $n.hash().values() -> $three {
		       .say for $three.hash().keys();
			}
		}

    }
}
```



```perl6
#use Grammar::Debugger;
#use Grammar::Tracer;

# 第一个 Grammar, 修改了很多次, 借助于 Grammar::Debugger 和 Grammar::Tracer 方便看出 Grammar 在哪里失败.
# TOP 里面添加了 ^ 和 $ 限制时, 其后面的 token 和 rule 不能在添加 ^ 和 $, 否则匹配失败

use v6;
grammar Markdown::Toc {
    # token TOP   is breakpoint    {^ \s* <section>* $}
	token TOP    {^ \s* <section>* $}
	token section   {
	    <sname> <ws>
		<lines>*
	    <subsection>+
	}

    token subsection  {
    	<subsname> <ws>
        <lines>*
        <s2section> *
	}

    token s2section {
	    <s2name> <ws>
		<lines>*
    }

	token sname          { <sigil2>   <ws> <snumber=.hnumber> <ws> <shead=.hline>  }
	token subsname       { <sigil3>   <ws> <subnumber=.number>    <shline=.hline>      }
	token s2name         { <sigil4>   <ws> <s2number=.number> <s2hline=.hline>   }
	token lines          {
	    [
		    <!after '#' ** 2..* >
            \N
        ]+
	    \n
	}

	token number {
	    \d+ % \.
	}

	token hline {
	    \N+
	}
	token hnumber { \w+}
	token ws     { \s*      }
	token sigil2 { '#' ** 2 }
	token sigil3 { '#' ** 3 }
	token sigil4 { '#' ** 4 }
}

my $str = q:to/EOF/;
## 第三章 操作符
blabla
#blabla
blabla
blabla
###  3.1 关于优先级的的一句话
blabla
###  3.2 比较和智能匹配
blabla
####   3.2.1 数字比较
blabla
####    3.2.2 字符串比较
blabla
####    3.2.3 智能匹配
blabla
### 3.3 测试
## 第四章 子例程和签名
blabla
###  4.1 申明子例程
blabla
###  4.2 添加签名
blabla
####   4.2.1 基础
blabla
####    4.2.2 传递数组、散列和代码
blabla
####    4.2.3 插值、数组和散列
blabla
EOF

class Markdown::Toc::Actions {
	method s2section($/)        {
	    my $first = ~$<s2name><s2number>;
	    my $second = ~$<s2name><s2hline>;
		my $remove_dot = $first.subst(rx/\./,'',:g);
		my $remove_space = $second.subst(rx/\s+/,'-',:g);
		make '    - '~'['~$first~' '~$second~']' ~ '(#'~$remove_dot~$remove_space ~ ')' => $<s2section>>>.made;
	}

	method subsection($/) {
	    my $first = ~$<subsname><subnumber>;
	    my $second = ~$<subsname><shline>;
		my $remove_dot = $first.subst(rx/\./,'',:g);
		my $remove_space = $second.subst(rx/\s+/,'-',:g);
		make '  - '~'['~$first~' '~$second~']' ~ '(#'~$remove_dot~$remove_space ~ ')'	=> $<s2section>>>.made;  
	}

    method section($/)    {
	    my $first = ~$<sname><snumber>;
		my $second = ~$<sname><shead>;
		#my $remove_dot = $first.subst(rx/\./,'',:g);
		my $remove_space = $second.subst(rx/\s+/,'',:g);
	    make '- '~'['~$first~' '~$second~']' ~ '(#'~$first~ '-'~$remove_space ~ ')'  => $<subsection>>>.made;
	}

    method TOP($/)    { make $<section>>>.made;                                                            }

}

my $actions = Markdown::Toc::Actions.new;
my $match  = Markdown::Toc.parse($str, :$actions).made;
#say $match.Str;

for @$match -> $p {
    say $p.key();
    for $p.value() -> $v {
        for $v -> $n {
		   .say for $n.hash().keys();
		   for $n.hash().values() -> $three {
		       .say for $three.hash().keys();
			}
		}

    }
}
```



```perl6
use v6;

grammar KeyValuePairs {
    token TOP {
        [<pair> \n+]*
    }
    token ws { \h* }

    rule pair {
        <key=.identifier> '=' <value=.identifier2>
    }
    token identifier {
        \w+
    }
     token identifier2 {
        \w+
    }
}

class KeyValuePairsActions {
    method identifier($/) { $/.make: '[' ~$/ ~ ']'                }
    method identifier2($/) { $/.make: '{' ~$/ ~ '}'                }
    method pair      ($/) { $/.make: $<key>.made => $<value>.made }
    method TOP       ($/) { $/.make: $<pair>».made                }
}

my  $res = KeyValuePairs.parse(q:to/EOI/, :actions(KeyValuePairsActions)).made;
    second=b
    hits=42
    perl=6
    EOI
say $res;
for @$res -> $p {
    say "Key: $p.key()\tValue: $p.value()";
}

```



```perl6
﻿grammar VariableNames {

    token variable {
        <sigil> <name>
    }

    token sigil {
        '$' | '@' | '&' | '%' | '::'
    }

	# [ ... ] are non-capturing groups
	token name {
        <identifier>
        [ '::' <identifier> ] *
    }
	# 标识符以字母开头
    token identifier {
        <alpha> \w+
    }
}

my $match = VariableNames.parse("@array",:rule('variable'));
say $match;

# we inherit from the original grammar...
grammar VARIABLENAMES is VariableNames {

    # ... and override that parsing rule that we want to change
    token identifier {
        # char classes are <[ ... ]> in Perl 6
        <[A..Z]> <[A..Z0..9_]>*
    }
}
my $test = VARIABLENAMES.parse("%A_HASH_TABLE",:rule('variable'));
say $test;

grammar LackMoney is VariableNames {
    token sigil {
        '¢' | '@' | '&' | '%' | '::'
    }
}

# 继承以后, 带¢的变量能够解析, 带$的变量解析不了了
my $money = LackMoney.parse('$i_m_not_dollor',:rule('variable'));
say so $money; # false

```



```perl6
use v6;
grammar URL {
        token TOP {
            <schema> '://'
            [<ip> | <hostname> ]
            [ ':' <port>]?
            '/' <path>?
        }
        token byte {
            (\d**1..3) <?{ $0 < 256 }>
        }
        token ip {
            <byte> [\. <byte> ] ** 3
        }
        token schema {
            \w+
        }
        token hostname {
            (\w+) ( \. \w+ )*
        }
        token port {
            \d+
        }
        token path {
            <[ a..z A..Z 0..9 \-_.!~*'():@&=+$,/ ]>+
        }
    }

my  $match = URL.parse('http://perl6.org/documentation/');
say $match.WHAT();
say $match<path>;       # perl6.org
say "hello ";
```



```perl6
#use Grammar::Debugger;
#use Grammar::Tracer;

grammar SalesExport::Grammar {
    token TOP { ^ <country>+ $ }
    token country {
        <cname=.name> \n
        <destination>+
    }

    token destination {
        \s+ <dname=.name> \s+ ':' \s+
        <lat=.num> ',' <long=.num> \s+ ':' \s+
        <sales=.integer> \n
    }

    token name    { \w+          }
    token num     { \d+ [\.\d+]? }
    token integer { \d+          }
}

my $string = q:to/THE END/;
Norway
    Oslo : 59.914289,10.738739 : 2
    Bergen : 60.388533,5.331856 : 4
Ukraine
    Kiev : 50.456001,30.50384 : 3
Switzerland
    Wengen : 46.608265,7.922065 : 3
THE END

class SalesExport::Grammar::Actions {
	method destination($/) { make ~$<dname> => $<sales>          }
    method country($/)     { make ~$<cname> => $<destination>    }
    method TOP($/)         { make $<country>>>.made              }
}

my $actions = SalesExport::Grammar::Actions.new;
my $grammar_action = SalesExport::Grammar.parse($string, :actions($actions)).made;

# 获取所有国家的名字
for @$grammar_action -> $p {
    say "$p.key()";
}

say  "-" x 45;
for @$grammar_action -> $p {
    for $p.value() -> $d {
	   for @$d -> $n {
	      say ~$n<dname>;
	   }
	  }
}

say  "-" x 45;

# 计算每个国家卖了多少票
for @$grammar_action -> $c {
    for $c.value() -> $d {
	   my $sales_count=0;
	   for @$d -> $n {
	      $sales_count += ~$n<sales>;
	   }
	   say $sales_count;
	  }
}


 #`(
# say $string;
my $grammar_object = SalesExport::Grammar.parse($string);
if $grammar_object {
     say "It's works";
 } else {
     # TODO: error reporting
     say "Not quite works...";
 }


# say $grammar_object;
#  say $grammar_object<country>.Str;
say "_" x 45;
# say $grammar_object<country>[0];
# say $grammar_object<country>[1].Str;

 say "_" x 45;
# say $grammar_object<country>[].Str;
# say $grammar_object<country>.values;

# 获取国家的名字
say $grammar_object<country>[0]<name>.Str;
say $grammar_object<country>[1]<name>.Str;
say $grammar_object<country>[2]<name>.Str;

 say "_" x 45;
# 获取目的地
say $grammar_object<country>[0]<destination>[0]<name>.Str;
say $grammar_object<country>[0]<destination>[1]<name>.Str;

 say "_" x 45;
# 获取经度
say $grammar_object<country>[0]<destination>[0]<lat>.Str;
say $grammar_object<country>[0]<destination>[1]<lat>.Str;

 say "_" x 45;
# 获取纬度
say $grammar_object<country>[0]<destination>[0]<long>.Str;
say $grammar_object<country>[0]<destination>[1]<long>.Str;

 say "_" x 45;
# 获取sales
say $grammar_object<country>[0]<destination>[0]<sales>.Str;
say $grammar_object<country>[0]<destination>[1]<sales>.Str;

 say "_" x 45;
 # 获取所有国家
say $grammar_object<country>»<name>.Str;

 say "_" x 45;
 # 获取第一个国家的所有目的地
 say $grammar_object<country>[0]<destination>»<name>.Str;

 say "_" x 45;
 # 获取第一个国家的所有的 sales
 say $grammar_object<country>[0]<destination>»<sales>.Str;
)
```

 只能在叶子节点上(最后一个正则名字的前面)使用超运算符 »
  S/匹配对象中, 键就是正则的名字, 键值就是匹配到的部分内容.

```perl6
﻿#use Grammar::Debugger;
#use Grammar::Tracer;

grammar SalesExport::Grammar {
    token TOP { ^ <country>+ $ }
    token country {
        <cname=.name> \n
        <destination>+
    }

    token destination {
        \s+ <dname=.name> \s+ ':' \s+
        <lat=.num> ',' <long=.num> \s+ ':' \s+
        <sales=.integer> \n
    }

    token name    { \w+          }
    token num     { \d+ [\.\d+]? }
    token integer { \d+          }
}

my $string = q:to/THE END/;
Norway
    Oslo : 59.914289,10.738739 : 2
    Bergen : 60.388533,5.331856 : 4
Ukraine
    Kiev : 50.456001,30.50384 : 3
Switzerland
    Wengen : 46.608265,7.922065 : 3
THE END

class SalesExport::Grammar::Actions {
	method destination($/) { make ~$<dname> => [$<sales>.map(*.Num+10),$<lat>.map(*.Num+90) ]         }
    method country($/)     { make ~$<cname> => $<destination>>>.made            }
    method TOP($/)         { make $<country>>>.made                             }
}

my $actions = SalesExport::Grammar::Actions.new;
my $grammar_action = SalesExport::Grammar.parse($string, :actions($actions)).made;
#say $grammar_action.Str;
# 获取所有国家的名字
for @$grammar_action -> $p {
    say "$p.key()";
}
say '-' x 45;
# 获取所有目的地
for @$grammar_action -> $p {
    for $p.value() -> $d {
	    for @$d -> $n{
		    say $n.key();
		}
	}
}
say '-' x 45;
# 获取出售的票数
for @$grammar_action -> $p {
    print "$p.key()\t";
    for $p.value() -> $d {
	    my $count;
	    for @$d -> $n{
		    $count += $n.value()[0];
		}
	say $count;
	}
}

say '-' x 45;
# 获取经度 lat
for @$grammar_action -> $p {
    for $p.value() -> $d {
	    for @$d -> $n{
		    say $n.value()[1];
		}
	}
}
```


这将打印:
```text
Norway
Ukraine
Switzerland
---------------------------------------------
Oslo
Bergen
Kiev
Wengen
---------------------------------------------
Norway  26
Ukraine 13
Switzerland     13
---------------------------------------------
149.914289
150.388533
140.456001
136.608265
```


```perl6
grammar TestGrammar {
    token TOP { ^ \d+ $ }
}

class TestActions {
    method TOP($/) {
        $/.make(2 + ~$/);
    }
}

my $actions = TestActions.new;
my $match = TestGrammar.parse('40', :$actions);
say $match;         # ｢40｣
say $match.made;    # 42

```



```perl6
﻿grammar MyGrammar {
    token TOP {
        ^ [ <comment> | <chunk> ]* $
    }

    token comment {
        '#' \N* \n
    }
    token chunk {
      ^^  (\S+) '=' (\S+) $$
    }
}

# 如何调试 Grammars
# try to parse the whole thing
say ?MyGrammar.parse("#a comment\nfoo = bar");            # False, 整体调试
# and now one by one
say so MyGrammar.parse("#a comment\n", :rule<comment>);   # True, 只单独调试 comment
say so MyGrammar.parse("foo = bar", :rule<chunk>);        # False, 只单独调试 chunk, 失败, 说明 chunk 不能匹配! 原因是空白符没有匹配
```



```perl6
﻿use v6;

grammar TestGrammar {
    token TOP   { ^ <digit> $ }
	token digit { \d+         }
}

class TestActions {
    method TOP($/) {
	   # $/.make( 2 + ~$/);
	   make +$<digit> + 2 ;
	}
}

my $actions = TestActions.new;
my $match   = TestGrammar.parse('40', :$actions);
say $match;
say $match.made;
```



```perl6
#use Grammar::Debugger;
#use Grammar::Tracer;

grammar SalesExport {
    token TOP { ^ <country>+ $ }
    token country {
        <name> \n
        <destination>+
    }
    token destination {
        \s+ <name> \s+ ':' \s+
        <lat=.num> ',' <long=.num> \s+ ':' \s+
        <sales=.integer> \n
    }
    token name    { \w+ [ \s \w+ ]*   }
    token num     { '-'? \d+ [\.\d+]? }
    token integer { '-'? \d+          }
}


# Now we can turn any file in this format into a data structure.
#  tripes.txt 最后一行要有一个空行
my $parsed = SalesExport.parsefile('tripes.txt');

if $parsed {
    my @countries = @($parsed<country>);
	#for @countries -> $country { say ~$country<name>};
	for @countries { say [+] .<destination>»<sales>;}
}

if $parsed {
    my @countries = @($parsed<country>);
    my $top1 = @countries.max({
       [+] .<destination>»<sales>
       });
    say "Most popular today: $top1<name> ", [+] $top1<destination>>><sales>;
}
else {
    die "Parse error!";
}

```


tripes.txt

```perl6
Russia
    Vladivostok : 43.131621,131.923828 : 4
    Ulan Ude : 51.841624,107.608101 : 2
    Saint Petersburg : 59.939977,30.315785 : 10
Norway
    Oslo : 59.914289,10.738739 : 2
    Bergen : 60.388533,5.331856 : 4
Ukraine
    Kiev : 50.456001,30.50384 : 3
Switzerland
    Wengen : 46.608265,7.922065 : 3
    Bern : 46.949076,7.448151 : 1

```




```perl6
﻿use v6;

my $file=open "wordpairs.txt", :r;

my %dict;
my $line;

repeat {
    $line=$file.get;
    my ($p1,$p2)=$line.split(' ');
    if ?%dict{$p1} {
        %dict{$p1} = "{%dict{$p1}} {$p2}".words;
    } else {
        %dict{$p1} = $p2;
    }
} while !$file.eof;
```


wordpairs.txt

```perl6
it was
was the
the best
best of
of times
times it
it was
was the
the worst
worst of
of times
times it
it was
was the
the age
age of
of wisdom
wisdom it
```


## Great List Refactor
---


```perl6
﻿> map {$^x + 2}, ( (1,2),3, (4,5))
3 4 5 6 7
> map {$_ + 2}, ( (1,2),3, (4,5))
3 4 5 6 7
> (10,(11,12,13),(14,15)).[2]
14 15
```


## Perl 6 Examples
---

- 1、生成8位随机密码


```perl6
my  @char_set = (0..9, 'a'..'z', 'A'..'Z','~','!','@','#','$','%','^','&','*');
say @char_set.pick(8).join("") # 不重复的8位密码

say @char_set.roll(8).join("")  # 可以重复
```

- 2、打印前5个数字

```perl6
.say for 1..10[^5]
.say for 1,2,3,4 ... [^10]  # 这个会无限循环
```


- 3、排序

- 3.1 按数值排序


```perl6
> my %hash='Perl'=>100,'Python'=>100,'Go'=>100,'CMD'=>20,"Php"=>80,"Java"=>85;
> %hash.values
100 100 100 20 80 85
> %hash.values.sort
20 80 85 100 100 100
> %hash.values.sort(-*)
100 100 100 85 80 20
```



- 3.2 按分数排序散列：


```perl6
use v6;
my %hash = 'Perl'=>80,
         'Python'=>100,
             'Go'=>95,
            'CMD'=>20,
            "Php"=>80,
           "Java"=>85;

for %hash.sort({-.value}).hash.keys -> $key {
    say $key, "\t", %hash{"$key"}
}

# Python	100
# Go	95
# Java	85
# Perl	80
# Php	80
# CMD	20
```



```perl6
> ('xx'..'zz').classify(*.substr(1))<z>
xz yz zz

加密：
sub rot13 { $^s.trans('a..z' => 'n..za..m', 'A..Z' => 'N..ZA..M') }

# 执行外部命令
shell( "ssh www.myopps.com uptime" )
shell( "ls" )
shell( "ls -a" )
# shell 将命令的执行结果直接发送到屏幕

my $list = QX("ls")
# 可以将命令的结果保存到变量中。

#  批量创建文件夹
for 'A'.. 'Z' -> $i { shell("mkdir $i") }
```

- 4、求 1! + 2! + 3! + 4! +5! + 6! +7! +8! +9! +10!


```perl6
     > multi sub postfix:<!>(Int $x){ [*] 1..$x }
     > say [+] 1!,2!,3!,4!,5!,6!,7!,8!,9!,10! # 4037913

```


- 5、列出对象所有可用的方法
使用元对象协议， 即`对象名.^methods`

```perl6
> "SB".^methods
```


> BUILD Int Num chomp chop substr pred succ match ords lines samecase samespace tr
im-leading trim-trailing trim words encode wordcase trans indent codes path WHIC
H Bool Str Stringy DUMP ACCEPTS Numeric gist perl comb subst split

- 6、 匿名子例程

```perl6
my $x = sub($a){ $a+2 };say $x($_) for 1..4
my $x = -> $a { $a+2 };say $x($_) for 1..4
my $x = * + 2;say $x($_) for 1..4
```

以后是不是不会写这种 `=*+2` 的都不好意思说自己会写Perl6

- 7、字符串翻转与分割


```perl6
> 1223.flip
3221
> 'abcd'.flip
dcba
> 1234.comb
1 2 3 4
> 1234.comb(/./)
1 2 3 4
> 'abcd'.comb
a b c d
```

- 8、有这么一个四位数A，其个位数相加得到B，将B 乘以 B的反转数后得到 A，请求出这个>数字。

举例， 1458 就符合这个条件，1+4+5+8 ＝ 18， 18 ＊ 81 ＝1458

请找出另一个符合上面条件的四位数。

```perl6
> (^37).map: { my $r = $_ * .flip; 1000 < $r and $_ == [+] $r.comb and say $r }
```

解释下：
(^37) 产生一个范围  0 .. ^37 , 就是 0到36之前的数，在表达式中代表 B

来个正常思维的：


```perl6
> my $b;
> for 1000..^10000 -> $i {$b=[+] $i.comb;say $i if $b*$b.flip == $i;}
```

1458
1729

- 9、 大小写转换


```perl6
> my $word= "I Love Perl 6"
I Love Perl 6
> $word.wordcase()
I Love Perl 6
> my $lowercase = "i love perl 6"
i love perl 6
> $lowercase.wordcase()
I Love Perl 6
> $word.samecase('A')
I LOVE PERL 6
> $word.samecase('a')
i love perl 6
> $word.samecase('a').wordcase()
I Love Perl 6
```


- 10、 多行文本


```perl6
my $string = q:to/THE END/;
Norway
    Oslo : 59.914289,10.738739 : 2
    Bergen : 60.388533,5.331856 : 4
Ukraine
    Kiev : 50.456001,30.50384 : 3
Switzerland
    Wengen : 46.608265,7.922065 : 3
THE END

say $string;
```

- 11、 超运算符与子例程


```perl6
use v6;

my @a = <1 2 3 4>;
sub by2($n){
    return 2*$n;
}

sub power2($n) {
    return $n ** 2;
}
my @b = @a>>.&by2>>.&power2;
say @b; # 4 16 36 64
```

为什么是 &function 呢：
the name of the by2 function is &by2, just as the name of the foo scalar is $foo and the name of the foo array is @foo

- 12、 如何在Perl 6 中执行外部命令并捕获输出


```perl6
> my $res = qqx{mkdir 123456}

# 或使用 qx{ }
> my $res = qx{mkdir 112233}
```


- 13、   Does Perl6 support something equivalent to Perl5's __DATA__ and __END__ sections?


```perl6
use v6;
=foo This is a Pod block. A single line one. This Pod block's name is 'foo'.

=begin qux
This is another syntax for defining a Pod block.
It allows for multi line content.
This block's name is 'qux'.
=end qux

=data A data block -- a Pod block with the name 'data'.

# Data blocks are P6's version of P5's __DATA__.
# But you can have multiple data blocks:

=begin data
Another data block.
This time a multi line one.
=end data

$=pod.grep(*.name eq 'data').map(*.contents[0].contents.say);

say '-' x 45;
for @$=pod {
  if .name eq 'data' {
    say .contents[0].contents
  }
};
```


- 14、生成含有26个英文字母和下划线的 junction


```perl6
> any('A'..'Z','a'..'z','_');
any(A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z, a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z, _)
```


- 15、判断一个字符是否在某个集合中


```perl6
>  so any('A'..'Z','a'..'z') ∈ set("12a34".comb)
```

"12a34".comb 会把字符串分割为单个字符，返回一个字符数组

- 16、生成 IP 地址范围

```perl6
.say for "192.168.10." <<~>> (0..255).list
```

- 17、 生成 OC 中的测试数组


```perl6
 .say for "@" <<~>> '"Perl' <<~>>  (1..30).list <<~>> '",'
```


    @"Perl"1",
    @"Perl"2",
    @"Perl"3",
    @"Perl"4",
    @"Perl"5",
    …

- 18、我想以AGCT4种字母为基础生成字符串。

比如希望长度为1，输出A,G,C,T。
如果长度为2，输出AA,AG,AC,AT,GA,GG,GC,GT,CA,CG,CC,CT,TA,TG,TC,TT。这样的结果。

@a X~ ""   # 长度为1
(@a X~ @a) # 长度为2
(@a X~ @a) X~ @a     # 长度为3
@a X~ @a X~ @a X~ @a # 长度为4


```perl6
> my @a=<A G C T>
A G C T
> my $x=@a
A G C T
> $x xx 2
A G C T A G C T
> $x xx 3
A G C T A G C T A G C T
> ($x xx 3).WHAT
(List)
> $x.WHAT
(Array)

> ([X~] $x xx 2).join(',')
AA,AG,AC,AT,GA,GG,GC,GT,CA,CG,CC,CT,TA,TG,TC,TT
```

惰性操作符：


```perl6
my @a=<A G C T>;
my $x=@a;  # 或者使用 $x =@('A','G','C','T')
for 1 ...^ * -> $a {(([X~] $x xx $a)).join(',').say;last if $a==4;};
```


## Best Of Perl 6
---

- Command Line 命令行


```perl6

               Perl 5                                     Perl 6
 print "bananas are good\n";                     say "bananas are good";
 print "and I said: $quotes{\"me\"}\n";          say "and I said: %quotes{"me"}.";
 print "and I said: $quotes{\"me\"}\n";          say "and I said: %quotes<me>.";
 print "What is ... ";                           $result = prompt "What is ... ";
 chomp($result = <>);
```


- File IO


```perl6

               Perl 5                                     Perl 6
 $content = do { local $/;                       $content = slurp "poetry.txt";
    open my $FH, "poetry.txt"; <$FH>
 };

chomp(@content = do {                            @content = lines "poetry.txt";
    open my $FH, "poetry.txt"; <$FH>
});
```


- Automatic multithreading

Applying operations to junctions and arrays is now syntactically compact and readable. Perl 6 will create threads where appropriate to use multiple processors, cores or hyperthreading for high level language SIMD concurrent processing.



```perl6
               Perl 5                                     Perl 6
 my $sum;                                        my $sum = [+] @numbers;
 $sum += $_ for @numbers;
 for (0 .. $#factor1) {                          @product = @factor1 >>*<< @factor2;
   $product[$] = $factor1[$] * $factor2[$_];
 }
```


The Perl 5 code is a simplification, of course Perl6 "does the right thing" when the arrays have different lengths.

- Comparison 比较

Here are junctions, then chained comparison operators.


```perl6
               Perl 5                                     Perl 6
 if ($a == 3 or $a == 4 or $a == 7) {...}        if $a = 3 | 4 | 7 {...}
 if (4 < $a and $a < 12) {...}                   if 4 < $a < 12    {...}
 if (4 < $a and $a <= 12) {...}                  if $a ~~ 4^..12   {...}
 $a = defined $b ? $b : $c;                      $a = $b // $c;
```

The defined-OR operator eases lot of cases where Perl 5 newbies could fall into traps.

- Case 结构

```perl6
               Perl 5                                      Perl 6
                                                     given $a {
 if ($a == 2 or $a == 5) {...} }}                      when 2 | 5  {...}
 elsif ($a == 6 or $a == 7 or $a == 8 or $a == 9) {}   when 6 .. 9 {...}
 elsif ($a =~ /g/) {...}                               when 'g'    {...}
 else {...}                                            default     {...}
                                                     }
```

That new construct (backported to 5.10) is clear to read, very versatile and when used in combination with junctions, becomes even clearer.

- 强大的循环

List iteration via for is now much more versatile.


```perl6
               Perl 5                                     Perl 6
 for my $i (0..15) {...}                         for ^16 -> $i        {...}
 for (my $i=15; $i>1; $i-2) {...}                for 15,*-2...1 -> $i {...}   # 15 13 11 9 7 5 3 1
 for my $key (keys %hash) {                      for %hash.kv -> $key, $value {
   print "$key => $hash{$key}\n"; ...              say "$key => $value"; ...
 for my $i (0..$#a) {                            for zip(@a; @b; @c) -> $a, $b, $c {...}
   my $a = @a[$i];
   my $b = @b[$i];
   my $c = @c[$i]; ...
```


- 子例程中的具名参数

```perl6
               Perl 5                                     Perl 6
 sub routine {                                   sub routine ($a, $b, *@rest) {...}
   my $a = shift;
   my $b = shift;
   my @rest = @_;
 }
```


- Objects with auto generated new and getters and setters

Simple Object creation is now as easy as it gets.

```perl6
               Perl 5                                     Perl 6
 package Heart::Gold;                            class Heart::Gold {
                                                   has $.speed;
 sub new {                                         method stop { $.speed = 0 }
   bless {speed => 0 }, shift;                   }  
 }
                                                 my Heart::Gold $hg1 .= new;
 sub speed {                                     $hg1.speed = 100;
   my $self = shift;                             my $hg2 = $hg1.clone;
   my $speed = shift;
   if (defined $speed) { $self->{speed} = $speed }
   else { $self->{speed} }
 }

 sub stop {
   my $self = shift;
   $self->{speed} = 0;
 }
```


## Perl 6 Variable
---

- Variable Types

Perl 6 (as Perl 5) knows 3 basic types of variables: Scalars (single values), Arrays (ordered and indexed lists of several values) and Hashes (2 column table, with ID and associated value pairs). They can be easily distinguished, because in front of their name is a special character called sigil (latin for sign). It's the $ (similar to S) for Scalars, @ (like an a) for Arrays and a % (kv pair icon) for a Hash. They are now invariant (not changing), which means for instance, an array vaiable starts always with an @, even if you just want a slice of the content.


```perl6
$scalar
@array
@array[1]              # $array[1]   in Perl 5
@array[1,2]            # @array[1,2] in Perl 5
%hash
%hash{'ba'}            # $hash{'ba'} in Perl 5
%hash{'ba','da','bim'} # @hash{'ba','da','bim'} in Perl 5
```


The sigils also mark distinct namespaces, meaning: in one lexical scope you can have 3 different variables named $stuff, @stuff and %stuff. These sigils can also be used as an operator to enforce a context in which the following data will be seen.

The fourth namespace is for subroutines and similar, even if you don't usually think of them as variables. It's sigil & is used to refer to subroutines without calling them.

All special namespaces from Perl 5 (often marked with special syntax), like tokens (__PACKAGE__), formats, file or dir handles, or builtins are now regular variables or routines.

Because all variables contain objects, they have methods. In fact, all operators, including square or curly bracket subscripts, are just methods of an object with a fancy name.

The primary sigil can be followed by a secondary sigil, called a twigil, which indicates a special scope for that variable.

Scalar

This type stores one value, usually a reference to something: a value of a data type, a code object, an object or a compound of values like a pair, junction, array, hash or capture. The scalar context is now called item context, hence the scalar instruction from Perl 5 was renamed to item.


```perl6
$CHAPTER = 3;              # first comment!
$bin = 0b11;               # same value in binary format
$pi = 3.14159_26535_89793; # the underscores just ease reading
$float = 6.02e-23;         # floating number in scientific notation
$text = 'Welcome all!';    # single quoted string

# double quoted string, does eval $pi to it's content
$text = " What is $pi?";
$text = q:to'EOT';         # heredoc string

    handy for multiline text
    like HTML templates or email

EOT
$handle = open $file_name; # file handle
# an object from a class with a nested namespace
$object = Class::Name.new();
$condition = 3|5|7;                # a junction, a logical conjunction of values
$arrayref = [0,1,1,2,3,5,8,13,21]; # an array stored as a single item

# a hash stored as a single item
$hashref = {'audreyt' => 'pugs',
            'pm'      => 'pct',
            'damian'  => 'larrys evil henchman'};
# pointing to a callable
$coderef = sub { do_something_completely_diffenent(@_) };
```

(For info on some of those terms: comment, binary format, the underscores ease reading, scientific notation, single-quoted string, double-quoted string, heredoc string, file handle, class, junction, list of values, hash, callable.)

Unlike Perl 5, references are automatically dereferenced to a fitting context. So you could use these $arrayrefs and $hashrefs similarly to an array or hash, making $ the universal variable prefix, pretty much like in PHP. The primary difference is that $ prefixed lists are not flattened in lists.


```perl6
my $a = (1, 2, 3);
my @a = 1, 2, 3;
for $a { }          # just one iteration
for @a { }          # three iterations
```


Scalar Methods


```perl6
my $chapter = 3;
undefine $chapter;
defined $a; # false, returns 0
```

- Array

An array is an ordered and indexed list of scalars. If not specified otherwise, they can be changed, expanded and shortened anytime and used as a list, stack, queue and much more. As in Haskell, lists are processed lazily, which means: the compiler looks only at the part it currently needs. This way Perl 6 can handle infinite lists or do computation on lists that have not been computed yet. The lazy command enforces this and the eager command forces all values to be computed.

The list context is forced with a @() operator or list() command. That's not autoflattening like in Perl 5 (automatically convert a List of Lists into one List). If you still want that, say flat(). Or say lol() to explicitly prevent autoflattening.


```perl6
@primes = (2,3,5,7,11,13,17,19,23); # an array gets filled like in Perl 5
@primes =  2,3,5,7,11,13,17,19,23 ; # same thing, since unlike P5 round braces just do group
@primes = <2 3 5 7 11 13 17 19 23>; # ditto, <> is the new qw()
$primes = (2,3,5,7,11,13,17,19,23); # same array object just sits in $primes, $primes[0] is 2
$primes = item @primes;             # same thing, more explicit
$primes = 2,;                       # just 2, first element of the Parcel
@primes = 2;                        # array with one element
@primes = [2,3,5,7,11,13,17,19,23]; # array with one element (List of Lists - LoL)
@dev    = {'dan' => 'parrot'};      # array with one element (a Hash)
@data   = [1..5],[6..10],[11..15];  # Array of Arrays (LoL)
@list   = lol @data;                # no change
@list   = flat @data;               # returns 1..15
```


- Array Slices


```perl6
@primes                       # all values as list
@primes.values                # same thing
@primes.keys                  # list of all indices
"@primes[]"                   # insert all values in a string, uses [] to distinguish from mail adresses
$prime = @primes[0];          # get the first prime
$prime = @primes[*-1];        # get the last one
@some = @primes[2..5];        # get several
$cell = @data[1][2];          # get 8, third value of second value (list)
$cell = @data[1;2];           # same thing, shorten syntax
@numbers = @data[1];          # get a copy of the second subarray (6..10)
@copy = @data;                # shallow copy of the array
```

- Array Methods

Some of the more important things you can do with lists. All the methods can also used like ops in "elems @array;"


```perl6
? @array;              # boolean context, Bool::True if array has any value in it, even if it's a 0
+ @array;              # numeric context, number of elements (like in Perl 5 scalar @a)
~ @array;              # string context, you get content of all cells, stringified and joined, same as "@primes[]"

@array.elems;          # same as + @array
@array.end;            # number of the last element, equal to @array.elems-1
@array.cat;            # same ~ @array
@array.join('');       # also same result, you can put another string as parameter that gets between all values
@array.unshift;        # prepend one value to the array
@array.shift;          # remove the first value and return it
@array.push;           # add one value on the end
@array.pop;            # remove one value from the end and return it
@array.splice($pos,$n);# starting at $pos remove $n values and replace them with values that follow those two
```


- parameters


```perl6
@array.delete(@ind);   # delete all cells with indices in @ind
@array.exists(@ind);   # Bool::True if all indices of @ind have a value (can be 0 or '')
@array.pick([$n]);     # return $n (default is 1) randomly selected values, without duplication
@array.roll([$n]);     # return $n (default is 1) randomly selected values, duplication possible (like roll dice)
@array.reverse;        # all elements in reversed order
# returns a list where $n times first item is taken to last
# position if $n is positive, if negative the other way around
@array.rotate($n);

@array.sort($coderef); # returns a list sorted by a user-defined criteria, default is alphanumerical sorting
@array.min;            # numerical smallest value of that array
@array.max;            # numerical largest value of that array
$a,$b= @array.minmax;  # both at once, like in .sort,  .min, or .max, a sorting algorithm can be provided

@array.map($coderef);  # high oder map function, runs $coderef with every value as $_ and returns the list or results
@array.classify($cr);  # kind of map, but creates a hash, where keys are the results of $cr and values are from @array
@array.categorize($cr);# kind of classify, but closure can have no (Nil) or several results, so a key can have a list of values
@array.grep({$_>1});   # high order grep, returns only these elements that pass a condition ($cr returns something positive)
@array.first($coder);  # kind of grep, return just the first matching value
@array.zip;            # join arrays by picking first element left successively from here and then there
There is even a whole class of metaoperators that work upon lists.
```


- Hash

In Perl 6 a Hash is an unordered list of Pairs. A Pair is a single key => value association and appears in many places of the language syntax. A hash allows lookup of values by key using {} or <> syntax.


```perl6
%dev =  'pugs'=>'audreyt', 'pct'=>'pm', "STD"=>'larry';
%dev = :rakudo('jnthn'), :testsuite('moritz');            # adverb (pair) syntax works as well
%dev = ('audreyt', 'pugs', 'pm', 'pct', 'larry', "STD");  # lists get autoconverted in hash context
%compiler = Parrot => {Rakudo => 'jnthn'}, SMOP => {Mildew => 'ruoso'};       # hash of hashes (HoH)
```


- Hash Slices


```perl6
$value = %dev{'key'};      # just give me the value related to that key, like in P5
$value = %dev<pm>;         # <> autoquotes like qw() in P5
$value = %dev<<$name>>;    # same thing, just with eval
@values = %dev{'key1', 'key2'};
@values = %dev<key1 key2>;
@values = %dev<<key1 key2 $key3>>;
%compiler<Parrot><Rakudo>; # value in a HoH, returns 'jnthn'
%compiler<SMOP>;           # returns the Pair: Mildew => 'ruoso'

%dev   {'audrey'};         # error, spaces between varname and braces (postcircumfix operator) are no longer allowed
%dev\  {'allison'};        # works, quote the space
%dev   .<dukeleto>;        # error
%dev\ .{'patrick'};        # works too, "long dot style", because it's an object in truth
```


- Hash Methods


```perl6
? %dev                     # bool context, true if hash has any pairs
+ %dev                     # numeric context, returns number of pairs(keys)
~ %dev                     # string context, nicely formatted 2 column table using \t and \n

$table = %dev;             # same as ~ %dev
%dev.say;                  # stringified, but only $key and $value are separated by \t
@pairs = %dev;             # list of all containing pairs
%dev.pairs                 # same thing in all context
%dev.elems                 # same as + %dev or + %dev.pairs
%dev.keys                  # returns a list of all keys
%dev.values                # list of all values
%dev.kv                    # flat list with key1, value1, key 2 ...
%dev.invert                # reverse all key => value relations
%dev.push (@pairs)         # inserts a list of pairs, if a key is already present in %dev, both values gets added to an array
```


- Callable

Internally subroutines, methods and alike are variables with the sigil & and stored in a fourth namespace. Unlike Perl 5, all subroutines can be overwritten or augmented with user defined routines. Of course scalars can also contain routines.


```perl6
&function = sub { ... };         # store subroutine in callable namespace
function();                      # call/run it

$coderef = sub { ... };          # store it in a scalar
$coderef($several, $parameter);  # run that code
```


- Data Types

In contrast to variable types (container types) every value has a type too. These are organized internally as classes or roles and can be categorized into 3 piles: the undefined, immutable, and the mutable types.

You can assign one of these types to scalar, array, or hash variables, which enforces the contents to be that type.


```perl6
my Int $a;
my Int @a;  # array of Int
```


- Pair

Pairs are new and their syntax is used nearly everywhere in the language where there is an association between a name and a value.


```perl6
$pair = 'jakub' => 'helena';  # "=>" is the pair constructor
$pair = :jakub('helena');     # same in adverbial notation
$pair = :jakub<helena>;       # same using <>, the new qw()
$pair.key                     # returns 'jakub'
$pair.value                   # returns 'helena'
$pair.isa(Pair)               # Bool::True
```


- Enumeration

enum

- Capture

Captures are also a new type, which holds the parameters a routine gets. Because Perl now knows both positional and named parameters, it is a mixture of a list and array.


```perl6
$cap = \(@a,$s,%h,'a'=>3);    # creating a capture, "\" was free since there are no references anymore
|$cap                         # flatten into argument list (without |, it will pass it as a single value)
||$cap                        # flatten into semicolon list (meant for variadic functions that take list of lists)
```


One important difference between a capture and a compound structure of lists and hashes: While assignments with = will copy the complete content of the named variables, this is not so in the case of a capture. When I change sinthelastexample, thecontentofcap changes too, because when parameters to a routine are variables, they are also interpolated in the moment the routine is called, not when it's defined.

- Properties and Traits

Properties

xxx

Traits

xxx

Scoping

scope declarator, scopes


```perl6
my $var;
state
temp
let
our $var;
$*var;
Twigils
```

xxx

- Assignment and Binding

Assignment

As rightfully expected, assignments are done with the equal sign. But unlike Perl 5 you always get a copy of the right side data assigned to the left, no matter how nested the data structure was (lists of lists eg). You never get in Perl 6 a reference with =. The only exception may be seen captures.


```perl6
my @original = [1,2],[3,4];
my $copy = @original[0]; # $copy points to [1,2]
@original[0][0] = 'fresh stuff'; # $copy[0] holds still 1
```


- Binding

Since every variable in Perl 6 is a reference, programmers can use binding to get 2 variables that point to the same memory location.


```perl6
$original = 5;
$original := $mirror;       # normal binding, done on runtime
$original ::= $mirror;      # same thing, but done during compile time
$original = 3;
say $mirror;                # prints 3
$original =:= $mirror       # true, because they're bound together
$original === $mirror       # also true, because content and type are equal
```



——-
    问题描述：

给定 2 个整数， A 和 B。 求这两个数的和

    输入数据：
    A 和 B 来自输入流， 以空格分割

    输出数据：
    两个数据的和

    Example：

    Input    Output
    2   2      4
    3   2      5


```perl6
say [+] .words for lines
```



——-

    任务：
    以任意的顺序生成 含有 N 个开括号"["  和 N 个闭括号"]" 的字符串

    检查生成的字符串是否平衡
    Example：

   (empty)   OK

   []        OK   ][        NOT OK

   [][]      OK   ][][      NOT OK

   [[][]]    OK   []][[]    NOT OK


- Depth counter


```perl6
sub balanced($s) {
    my $l = 0;
    for $s.comb {
        when "]" {
            --$l;
            return False if $l < 0;
        }
        when "[" {
            ++$l;
        }
    }
    return $l == 0;
}

my $n = prompt "Number of brackets";
my $s = (<[ ]> xx $n).pick(*).join;
say "$s {balanced($s) ?? "is" !! "is not"} well-balanced"
```


- FP oriented



```perl6
sub balanced($s) {
    .none < 0 and .[*-1] == 0
        given [\+] '\\' «leg« $s.comb;
}

my $n = prompt "Number of bracket pairs: ";
my $s = <[ ]>.roll($n*2).join;
say "$s { balanced($s) ?? "is" !! "is not" } well-balanced"
```


- String munging



```perl6
sub balanced($_ is copy) {
    () while s:g/'[]'//;
    $_ eq '';
}

my $n = prompt "Number of bracket pairs: ";
my $s = <[ ]>.roll($n*2).join;
say "$s is", ' not' xx not balanced($s)), " well-balanced";
```


- Parsing with a grammar


```perl6
grammar BalBrack { token TOP { '[' <TOP>* ']' } }

my $n = prompt "Number of bracket pairs: ";
my $s = ('[' xx $n, ']' xx $n).pick(*).join;
say "$s { BalBrack.parse($s) ?? "is" !! "is not" } well-balanced";
```



——-

 -   凯撒加密

实现一个凯撒加密， 编码和解码都要有

 key 是一个 1 到 25 之间的整数


```perl6
my @alpha = 'A' .. 'Z';
sub encrypt ( $key where 1..25, $plaintext ) {
    $plaintext.trans( @alpha Z=> @alpha.rotate($key) );
}
sub decrypt ( $key where 1..25, $cyphertext ) {
    $cyphertext.trans( @alpha.rotate($key) Z=> @alpha );
}

my $original = 'THE FIVE BOXING WIZARDS JUMP QUICKLY';
my $en = encrypt( 13, $original );
my $de = decrypt( 13, $en );

.say for $original, $en, $de;

say 'OK' if $original eq all( map { .&decrypt(.&encrypt($original)) }, 1..25 );
```


    Output:
    THE FIVE BOXING WIZARDS JUMP QUICKLY
    GUR SVIR OBKVAT JVMNEQF WHZC DHVPXYL
    THE FIVE BOXING WIZARDS JUMP QUICKLY
    OK




——-

- 日期格式化

使用 "2007-11-10" 和 " Sunday, November 10, 2007" 日期格式显式当前日期


```perl6
use DateTime::Utils;

my $dt = DateTime.now;

say strftime('%Y-%m-%d', $dt);
say strftime('%A, %B %d, %Y', $dt);
```



——-

- 阶乘

 n 的阶乘定义为 `n*(n-1)*(n-2)…*1`, 零的阶乘为1.

 定义一个函数返回一个数字的阶乘。

- 使用自定义后缀操作符


```perl6

sub postfix:<!>($n where $n > 0) {
    [*] 2..$n
}
say 5!

```


- [\*]


```perl6

my @a = 1, [\*] 1..*;
say @a[5];

```





——-

- 动画


```perl6

my $row-count = 6;

constant $peg = "*";
constant @coin-icons = "\c[UPPER HALF BLOCK]", "\c[LOWER HALF BLOCK]";

sub display-board(@positions, @stats is copy, $halfstep) {
    my $coin = @coin-icons[$halfstep.Int];

    state @board-tmpl = {
        # precompute a board
        my @tmpl;
        sub out(*@stuff) {
            @tmpl.push: @stuff>>.ords.item;
        }
        # three lines of space above
        for (1..3) {
            out "  ", " " x (2 * $row-count);
        }
        # $row-count lines of pegs
        for ($row-count...1) Z (1...$row-count) -> $spaces, $pegs {
            out "  ", " " x $spaces, ($peg xx $pegs).join(" "), " " x $spaces;
        }
        # four lines of space below
        for (1..4) {
            out "  ", " " x (2 * $row-count);
        }
        @tmpl
    }();

    my $midpos = $row-count + 2;

    my @output;
    {
        # collect all the output and output it all at once at the end
        sub say(Str $foo) {
            @output.push: $foo, "\n";
        }
        sub print(Str $foo) {
            @output.push: $foo;
        }

        # make some space above the picture
        say "" for ^10;

        my @output-lines = map { [map *.clone, @$_].item }, @board-tmpl;
        # place the coins
        for @positions.kv -> $line, $pos {
            next unless $pos.defined;
            @output-lines[$line][$pos + $midpos] = $coin.ord;
        }
        # output the board with its coins
        for @output-lines -> @line {
            say @line>>.chr.join("");
        }

        # show the statistics
        my $padding = 0;
        while any(@stats) > 0 {
            $padding++;
            print "  ";
            @stats = do for @stats -> $stat {
                given $stat {
                    when 1 {
                        print "\c[UPPER HALF BLOCK]";
                        $stat - 1;
                    }
                    when * <= 0 {
                        print " ";
                        0
                    }
                    default {
                        print "\c[FULL BLOCK]";
                        $stat - 2;
                    }
                }
            }
            say "";
        }
        say "" for $padding...^10;
    }
    say @output.join("");
}

sub simulate($coins is copy) {
    my $alive = True;

    sub hits-peg($x, $y) {
        if 3 <= $y < 3 + $row-count and -($y - 2) <= $x <= $y - 2 {
            return not ($x - $y) %% 2;
        }
        return False;
    }

    my @coins = Int xx (3 + $row-count + 4);
    my @stats = 0 xx ($row-count * 2);
    # this line will dispense coins until turned off.
    @coins[0] = 0;
    while $alive {
        $alive = False;
        # if a coin falls through the bottom, count it
        given @coins[*-1] {
            when *.defined {
                @stats[$_ + $row-count]++;
            }
        }

        # move every coin down one row
        for ( 3 + $row-count + 3 )...1 -> $line {
            my $coinpos = @coins[$line - 1];

            @coins[$line] = do if not $coinpos.defined {
                Nil
            } elsif hits-peg($coinpos, $line) {
                # when a coin from above hits a peg, it will bounce to either side.
                $alive = True;
                ($coinpos - 1, $coinpos + 1).pick;
            } else {
                # if there was a coin above, it will fall to this position.
                $alive = True;
                $coinpos;
            }
        }
        # let the coin dispenser blink and turn it off if we run out of coins
        if @coins[0].defined {
            @coins[0] = Nil
        } elsif --$coins > 0 {
            @coins[0] = 0
        }

        # smooth out the two halfsteps of the animation
        my $start-time;
        ENTER { $start-time = now }
        my $wait-time = now - $start-time;

        sleep 0.1 - $wait-time if $wait-time < 0.1;
        for @coin-icons.keys {
            sleep $wait-time max 0.1;
            display-board(@coins, @stats, $_);
        }
    }
}

sub MAIN($coins = 20, $peg-lines = 6) {
    $row-count = $peg-lines;
    simulate($coins);
}

```


调用方式： perl6 Galton_box_animation.p6 50 8


——-

- 列表和迭代

Perl 6 中的列表扩展为惰性列表、无限列表、元素可变列表、元素不可变列表、类型列表、展开行为等等。


对于程序员来说，列表潜在是懒惰并含有无限元素的序列。列表是可变的，你可以通过诸如 push、pop、shift、unshift、splice等操作符来操作序列。列表中的元素可以是可变的或者不可变的。


列表对象是基于位置的，意味着它们能被绑定到数组变量上，并且支持 `.[]` 后缀操作符。


列表也是懒惰的，因为列表中的元素可以来自于能按需产生元素的生成函数（叫做迭代）。


数组就是一个所有元素都存储在标量容器的列表。


逗号操作符 `infix:<,>` 创建 Parcel 对象。这些不应该改和列表混淆； Parcel 是一种未经加工的元素序列。Parcel 是不可变的，尽管 Parcel中的元素可以是不可变的，也可是不可变的。

Parcel 来自于短语  "parenthesis cell". 因为很多 Parcel 对象出现在圆括号里面。然而，除了空的 parcel，是逗号操作符创建了 Parcel 对象。

    ()       # empty Parcel
    (1)      # 一个整数
    (1,2)    # a Parcel with two Ints
    (1,)     # a Parcel with one Int


```perl6
> (1).WHAT()
(Int)
> (1,).WHAT()
(Parcel)
```


Parcel 也是位置的，并且对于诸如  `.[]` 和 `.elems` 列表操作会使用 展开上下文。查看下面的  "Flattening contexts"。访问没有展开的原始参数，你可以使用 `.arg($n)` 代替 `.[$n]`, 和 `.args` 代替 `.elems`


```perl6
> (1,2,3).elems
3
> (1,2,3).[2]
3
> (1,2,3).[1]
2

> my $a =(1,2,3,(4,5,6),7).[3]
4 5 6
> $a.WHAT.say
(Parcel)
> $a.[2]
6
> $a.[1]
5

> [+] $a.list
15

```


列表和Parcel 对象都把其它容器对象作为元素。在一些上下文中，我们想把容器对象的值插入到列表或 parcel的周围，而在其它上下文中，我们想保留所有的子容器。这样的插值叫做 展开。

列表和Parcel都是可迭代的，可迭代表明它支持 `.iterator` 方法

标量容器中存储的对象不会在 flattening 上下文中插值，即使那个对象是可迭代的。




    my @a = 3,4,5;
    for 1,2,@a  { .say }        # 5次迭代
1
2
3
4
5

    my $s = @a;
    for 1,2,$s { ... }         # 3次迭代
1
2
3 4 5

这里，`$s` 和 `@a` 指向同一个数组对象，但是标量容器的出现阻止 `$s` 被展开到 for 循环中。

.list 和 .flat 方法能被用于还原展开行为：


```perl6
    for 1,2,$s.list { .say }    # 5次遍历
    for 1,2,@($s)   { .say  }   # 5次遍历，@()会强制为列表上下文
```

1
2
3
4
5


相反，`.item` 方法和 `$()` 能用于防止插值：


```perl6
    my @b = 1,2,@a;           # @b 有5个元素
    my @c = 1,2,@a.item;      # @c 有3个元素
    my @c = 1,2,$(@a);        # 同上

> say +@c
3
```


迭代器


`.reify($n)` 方法要求迭代器返回一个含有至少`$n`个具体元素的 Parcel，后面跟着序列中剩余元素的附加的迭代器，例如：


```perl6
   my $r = 10..50;
   say $r.iterator.reify(5).perl;  # (10, 11, 12, 13, 14, 15..50)
```



> say $r.iterator.reify(*).perl
(10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 37, 48, 49, 50)

- Feed operators

feed操作符是完全懒惰的，意味着在使用者要求任何元素之前不会执行任何操作。这就是

  my @a <== grep { ... } <== map { ... } <== grep { ... } <== 1, 2, 3

是完全懒惰的。


——
- Grammars 文法

    Named Regexes
    Creating Grammars
    Methods
    method parse
    method subparse
    method parsefile
    Action Classes

文法是一种强大的工具, 用于拆解文本,并通常返回数据结构
例如, Perl 6 是使用 Perl 6 风格的文法解析和执行的.
对普通 Perl 6 用户来说,一个更实用的例子就是 JSON::Simple 模块, 这个模块能反序列化任何有效的 JSON 文件, 反序列化代码还写了不到 100 行, 简单,可扩展.

词法允许你组织正则, 就像类允许你组织普通代码的方法一样.
## 命名正则 Named Regexes
---

命名正则有特殊的语法, 与**子例程**的定义类似:


```perl6
my regex number { \d+ [ \. \d+ ]? }
```

这个例子中, 我们必须使用 ** my ** 关键词指定这个正则是词法作用域的, 因为 **命名正则** 通常用在 词法中.
给正则命名后有利于在其他地方`复用`正则:


```perl6
say "32.51"    ~~ &number;
say "15 + 4.5" ~~ / <number> \s* '+' \s* <number> /
```

首先说下, 使用 `regex/token/rule` 定义了一个正则表达式后怎么去调用它:
就像调用一个子例程那样, 使用 `&` 符号:
& 后面跟正则表达式的名字,  即 &regex_name
regex 不是命名正则仅有的标识符 -- 实际上, 它用的不多. 大多数时候, 用的最多的是 `token` 和 `rule` 标识符. 它们都是`不能回溯`的, 这意味着正则引擎在匹配失败时不会备份和重试. 这通常是你想要的, 但不是对所有场合都合适:


```perl6
my regex works-but-slow { .+ q }
my token fails-but-fast { .+ q }
my $s = 'Tokens won\'t backtrack, which makes them fail quicker!'; # Tokens 不会沿原路返回, 这让它们更快地失败!
say so $s ~~ &works-but-slow; # True
say so $s ~~ &fails-but-fast; # False, the entire string get taken by the .+
```


`token` 和 `rule` 标识符的不同之处在于 `rule` 标识符让 `Regex` 的 `:sigspace` 起作用了:


```perl6
my token non-space-y { once upon a time }
my rule space-y      { once upon a time }
say 'onceuponatime'    ~~ &non-space-y;
say 'once upon a time' ~~ &space-y;
```

## 创建文法 Creating Grammars
---


```perl6
class Grammar is Cursor { }
```


使用 grammar 关键字而非 `class` 关键字声明文法. Grammars 应该只用于`解析文本`; 如果你想`提取`复杂的数据, 建议将 `action` 类 和 `grammar` 结合使用.



```perl6
grammar CSV {
    token TOP { [ <line> \n? ]+ }
    token line {
        ^^            # Beginning of a line
        <value>* % \, # Any number of <value>s with commas in `between` them
        $$            # End of a line
    }
    token value {
        [
        | <-[",\n]>     # Anything not a double quote, comma or newline
        | <quoted-text> # Or some quoted text
        ]*              # Any number of times
    }
    token quoted-text {
        \"
        [
        | <-["\\]> # Anything not a " or \
        | '\"'     # Or \", an escaped quotation mark
        ]*         # Any number of times
        \"
    }
}

say "Valid CSV file!" if CSV.parse( q:to/EOCSV/ );
    Year,Make,Model,Length
    1997,Ford,E350,2.34
    2000,Mercury,Cougar,2.38
    EOCSV
```


- 方法	Methods

- 方法解析


```perl6
method parse($str, :$rule = 'TOP', :$actions) returns Match:D
```

让 grammar 与 $str 匹配,使用 $rule 作为起始 rule, 选择性地将 $action 作为 action 对象应用.

如果 grammar 不能解析全部文本就会失败. 如果只想解析部分字符串, 使用 subparse
返回结果匹配对象, 并设置调用者的 `$/` 变量为结果匹配对象.


```perl6
say CSV.parse( q:to/EOCSV/ );
    Year,Make,Model,Length
    1997,Ford,E350,2.34
    2000,Mercury,Cougar,2.38
    EOCSV
```



```perl6
	This outputs:

｢Year,Make,Model,Length
1997,Ford,E350,2.34
2000,Mercury,Cougar,2.38
｣
 line => ｢Year,Make,Model,Length｣
  value => ｢Year｣
  value => ｢Make｣
  value => ｢Model｣
  value => ｢Length｣
 line => ｢1997,Ford,E350,2.34｣
  value => ｢1997｣
  value => ｢Ford｣
  value => ｢E350｣
  value => ｢2.34｣
 line => ｢2000,Mercury,Cougar,2.38 ｣
  value => ｢2000｣
  value => ｢Mercury｣
  value => ｢Cougar｣
  value => ｢2.38 ｣
```


##  method subparse
---


```perl6
 method subparse($str, :$rule = 'TOP', :$actions) returns Match:D
```


将 `$str` 与 grammar 匹配, 使用 `$rule` 作为`起始 rule`, 选择性将 `$action` 作为 `action` 对象应用.
不像 `parse` , `subparse` 允许 `grammar` 只匹配所提供的字符串的一部分.

## method parsefile
---


```perl6
    method parsefile(Cool $filename as Str, *%opts) 返回 Match:D
```

	使用 parse 方法解析 文件 $filename 的内容, 传递任何命名选项到 %opts

## Action Classes
---


In fact, named regexes can even take extra arguments, using the same syntax as subroutine parameter lists
实际上, 命名正则甚至能接受额外的参数, 它使用的语法跟子例程参数列表的语法一样.

​写一个程序打印从 1  到 100 的整数，但是对 3 的倍数打印 "Fizz", 对 5 的倍数打印 "Buzz", 对于即是 3 的倍数，又是 5 的倍数的打印 "FizzBuzz".



```perl6
for 1 .. 100 {
    when $_ %% (3 & 5) { say 'FizzBuzz'; }
    when $_ %% 3       { say 'Fizz';     }
    when $_ %% 5       { say 'Buzz';     }
    default            { .say;           }
}
Or abusing multi subs:


```perl6
multi sub fizzbuzz(Int $ where * %% 15) { 'FizzBuzz' }
multi sub fizzbuzz(Int $ where * %% 5)  { 'Buzz'     }
multi sub fizzbuzz(Int $ where * %% 3)  { 'Fizz'     }
multi sub fizzbuzz(Int $number )        { $number    }
(1 .. 100)».&fizzbuzz.join("\n").say;
```


Most concisely:


```perl6
say 'Fizz' x $_ %% 3 ~ 'Buzz' x $_ %% 5 || $_ for 1 .. 100;
```

And here's an implementation that never checks for divisibility:


```perl6
.say for
    (('' xx 2, 'Fizz') xx * Z~
    ('' xx 4, 'Buzz') xx *) Z||1 .. 100;
```
