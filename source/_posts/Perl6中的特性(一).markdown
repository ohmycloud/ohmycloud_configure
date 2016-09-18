---
title: Perl 6 中的特性(一)
date: 2016-04-22
tags:
  - 特性
  - Examples
categories: Perl 6
comments: true
---


## Basics
---
### Array
---


```perl6
use v6;

my @primes = (2,3,5,7,11,13,17,19,23);   # an array gets filled like in Perl 5
# my @primes =  2,3,5,7,11,13,17,19,23 ; # same thing, since unlike P5 round braces just do group
# my @primes = <2 3 5 7 11 13 17 19 23>; # dito, <> is the new qw()

say @primes[];                           # 2 3 5 7 11 13 17 19 23
my $arrayref = [2,3,5,7,11,13,17,19,23]; # in scalar context you get automatically a reference
say @$arrayref;                          # 2 3 5 7 11 13 17 19 23

my $arrayref = item @primes;             # same thing, more explicit
say $arrayref;

my $arrayref = [13,];                    # comma is the new array generator
say $arrayref;

my @primes = 2;                          # array with one element
my @primes = [2,3,5,7,11,13,17,19,23];   # array with one element (arrayref)
say @primes;                             # 2 3 5 7 11 13 17 19 23

my @dev    = {'dan' => 'parrot'};        # array with one element (hashref)
say @dev;                                # ("dan" => "parrot").hash


my @data   = [1..5],[6..10],[11..15];    # Array of Arrays (AoA)
say @data.perl;                          # Array.new([1, 2, 3, 4, 5], [6, 7, 8, 9, 10], [11, 12, 13, 14, 15])

# my @list   = lol @data;                # no change
# say @list.perl;
my @list   = flat @data;                 # returns 1..15
say @list;                               # 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
```



```perl6
my @array = 1,3,5,7,9;

my @arrayplus3 = map( *+3, @array);
say @arrayplus3;

my @arrayplusab = map( *+*+3, @array,11);
say @arrayplusab;

```



```perl6
﻿use v6;
my @rray=17,21,34,47,58,69;

say 'the array has element' if  ? @rray;   # boolean context, Bool::True if array has any value in it, even if its a 0
say 'the array has ',+ @rray ~' element';  # numeric context, number of elements (like in Perl 5 scalar @a)
say ~ @rray;              # string context, you get content of all cells, stringified and joined, same as "@primes[]"

say @rray.elems;          # same as + @rray
say @rray.end;            # number of the last element, equal to @rray.elems-1, # 5
# say @rray.cat;          # same ~ @rray
say @rray.join('-');      # also same result, you can put another string as parameter that gets between all values
say @rray.unshift(7);     # prepend one value to the array, # 7 1 2 3 4 5 6
say @rray.shift;          # remove the first value and return it # 7
say @rray.push(8);        # add one value on the end # 1 2 3 4 5 6 8
say @rray.pop;            # remove one value from the end and return it # 8

my $n=2;
my $pos=1;
my @ind=0..3;
say @rray.splice($pos,$n);# remove on $pos $n values and replace them with values that follow that two parameter
say @rray:delete(@ind);   # delete all cell with indecies of @ind # 1 4 5 6
say @rray:exists(@ind);   # Bool::True if all indecies of @ind have a value (can be 0 or '')

say '-' x 18;
say @rray;
say @rray.pick([$n]);     # return $n (default is 1) randomly selected values, without duplication
say @rray.roll([$n]);     # return $n (default is 1) randomly selected values, duplication possible (就像掷筛子)
say @rray.reverse;        # all elements in reversed order
say @rray.rotate(-$n);    # returns a list where $n times first item is taken to last position if $n is positive, if negative the other way around
# @rray.sort($coderef); # returns a sorted list by a userdefined criteria, default is alphanumerical sorting
say @rray.min;            # numerical smallest value of that array
say @rray.max;            # numerical largest value of that array
my ($a,$b)= @rray.minmax;  # both at once, like in .sort . min or .max a sorting algorith can be provided
say $a,"   $b";
# @rray.map($coderef);  # high oder map function, runs $coderef with every value as $_ and returns the list or results
# @rray.classify($cr);  # kind of map, but creates a hash, where keys are the results of $cr and values are from @rray
# @rray.categorize($cr);# kind of classify, but closure can have no (Nil) or several results, so a key can have a list of values
say @rray.grep({$_>40});   # high order grep, returns only these elements that pass a condition ($cr returns something positive)
# @rray.first($coder);  # kind of grep, return just the first matching value
# say @rray.zip;            # join arrays by picking first element left successively from here and then there

```



```perl6
use v6;
my  @primes=<1 3 5 7 9 11 13 15 17>;
say @primes;                       # all values as list
say @primes.values;                # same thing
say @primes.keys;                  # list of all indices
say "@primes[]";                   # insert all values in a string, uses [] as distinction from mail adresses
my  $prime = @primes[0];           # get the first prime
say $prime;

my $last_prime = @primes[*-1];    # get the last one
say $last_prime;

my @some = @primes[2..5];         # get several
say @some;

my @data   = [1..5],[6..10],[11..15];  # Array of Arrays (AoA)
my $cell = @data[1][2];                # get 8, third value of second value (list)
say $cell;

my  $same_cell = @data[1,2];           # same thing, shorten syntax
say $same_cell;                        # 6 7 8 9 10 11 12 13 14 15

my @numbers = @data[1];                # get a copy of the second subarray (6..10)
say @numbers;
my @copy = @data;                      # copy the whole AoA, no more reference passing, use binding instead
say @copy.perl;

```



```perl6
use v6;
my @original = [1,2],[3,4];
say @original.perl;
my $copy = @original[0]; # $copy points to [1,2]
say $copy;
@original[0][0] = 'fresh stuff'; # $copy[0] holds still 1
say @original;
say $copy;
```



```perl6
my @names = <Patrick Jonathan Larry Moritz Audrey>;
say .key, "\t", ~.values
    for @names.classify( *.chars );
# 根据字符串的字符个数分类
# Output:
# 7	Patrick
# 8	Jonathan
# 5	Larry
# 6	Moritz Audrey

```



```perl6
#.say for slurp("README.txt")\           # whole file into string
#         .words()\                      # split into list of words
#         .classify( *.Str );

my @a = slurp("README.txt").words;
# .say  for @a.classify( *.Str );
# output:
#  可见输出的是一个散列
# that => that that that that that that that that that that that
# the => the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the the
# is => is is is is is is is is is is is is is is is is
my %hash = @a.classify( *.Str );
for %hash.sort({-.value.elems}).hash.kv.[^20] -> $key, $value {
    say $key ,"\t", $value.elems;

}

```



```perl6
.say for slurp("README.txt")\           # whole file into string
         .words()\                      # split into list of words
         .classify( *.Str )\            # group words with multiplicity
         .map({;.key => .value.elems })\
                                        # turn lists into lengths
         .sort( { -.value } )\          # sort descending
         .[ ^10 ];                      # 10 most common words

# Output:
# the => 40
# to => 21
# is => 16
# a => 16
# that => 11
# be => 11
# stack => 10
# implementation => 8
# link => 8
# it => 7

```



```perl6
use v6;
my $range = 'a' .. 'f';
for $range.list -> $elem {
    say $elem;
}

.say for @( $range );
# .say for $list.list;
.say for $range.flat;

```



```perl6
use v6;
my @a=<aa bb cc dd ee ff >;
for @a -> $x,$y,$z {
say $x,$y,$z;
}
say $*PROGRAM_NAME;

aabbcc
ddeeff
three.p6
```

### Hash
---


```perl6
my $a = bag <a a a b b c>;
my $b = bag <a b b b>;

say $a (|) $b;
say $a (&) $b;
say $a (+) $b;
say $a (.) $b;

```



```perl6
use v6;
# my %words := KeyBag.new;
# for slurp.comb(/\w+/).map(*.lc) -> $word {
    # %words{$word}++;
# }
my %words := Bag.new(slurp.comb(/\w+/).map(*.lc));

say "%words{}";
```



```perl6
my %words;
for slurp.comb(/\w+/).map(*.lc) -> $word {
    %words{$word}++;
}

for %words.pairs.sort(-*.value).map({ $_.key, $_.value }) -> $word, $count {
    say "$word: $count";
}

```



```perl6
my %words := Bag.new(slurp.comb(/\w+/).map(*.lc));
# say %words.pairs.list.join("\n");
# .say for %words.pairs.sort(-*.value);  # 按键值从大到小排序，然后打印，- 号是降序
# "and" => 12
# "the" => 11
# "our" => 6
# "be" => 6
# "man" => 6
# "your" => 6
# "us" => 5
# "you" => 5
# "for" => 5
# "o" => 5
# "to" => 5

for %words.pairs.sort(-*.value).map({ $_.key, $_.value }) -> $word, $count {
    say "$word: $count";
}
# .say for %words.pairs.sort(-*.value).fmt("%-15s=>%4d\n");

```



```perl6
my %words := bag slurp.comb(/\w+/).map(*.lc);

for %words.pairs.sort(-*.value).map({ $_.key, $_.value }) -> $word, $count {
    say "$word: $count";
}
```



```perl6
﻿use v6;

my %dev =  'pugs'=>'audreyt', 'pct'=>'pm', "STD"=>'larry';
my %same_dev = :rakudo('jnthn'), :testsuite('moritz');            # adverb (pair) syntax works as well
my %too_dev = ('audreyt', 'pugs', 'pm', 'pct', 'larry', "STD");  # lists get autoconverted in hash context
my %compiler = Parrot => {Rakudo => 'jnthn'}, SMOP => {Mildew => 'ruoso'};       # hash of hashes (HoH)
say %dev.perl;
say %same_dev.perl;
say %too_dev.perl;
say %compiler.perl;

# Hash Slices
my $name='pugs';
my $value = %dev{'pugs'};      # just give me the value related to that key, like in P5
my $value1 = %dev<STD>;         # <> autoquotes like qw() in P5
my $value2 = %dev<<$name>>;     # same thing, just with eval
say $value;
say $value2;

my @values = %dev{'pugs', 'STD'};
my @values2 = %dev<pugs STD>;
my @values3 = %dev<<pugs STD $name>>;
say @values;
say @values2;
say @values3;

say %compiler<Parrot><Rakudo>; # value in a HoH, returns 'jnthn'
say %compiler<SMOP>;           # returns the Pair: Mildew => 'ruoso'

# %dev   {'audrey'};         # error, spaces between varname and braces (postcircumfix operator) are no longer allowed
say %dev\  {'pugs'};        # works, quote the space
# %dev   .<dukeleto>;        # error
say %dev\ .{'pugs'};        # works too, "long dot style", because its its an object in truth
say %dev.{'pugs'};


# Hash Methods
say 'this hash has some pairs' if ? %dev;                    # bool context, true if hash has any pairs
say 'this hash has '~ + %dev ~' pairs';                      # numeric context, returns number of pairs(keys)
say ~ %dev;                    # string context, nicely formatted 2 column table using \t and \n

my $table = %dev;             # same as ~ %dev
say $table;                   # ("pugs" => "audreyt", "pct" => "pm", "STD" => "larry").hash
say %dev.say;                 # stringified, but only $key and $value are separated by \t  #("pugs" => "audreyt", "pct" => "pm", "STD" => "larry").hash
my  @pairs = %dev;             # list of all containing pairs
say @pairs;                    # "pugs" => "audreyt" "pct" => "pm" "STD" => "larry"
say %dev.pairs;                 # same thing in all context  # "pugs" => "audreyt" "pct" => "pm" "STD" => "larry"
say %dev.elems;                 # same as + %dev or + %dev.pairs  # 3
say %dev.keys;                  # returns the list with all keys
say %dev.values;                # list of all values
say %dev.kv;                    # flat list with key1, value1, key 2 ...
say %dev.invert;                # reverse all key => value relations
say %dev.push(@pairs);         # inserts a list of pairs, if a key is already present in %dev, both values gets added to an array
# ("pugs" => ["audreyt", "audreyt"], "pct" => ["pm", "pm"], "STD" => ["larry", "larry"]).hash

my @another_pairs='Rakudo'=>'Perl6';
say %same_dev.push(@another_pairs);
```



```perl6
# You can also destructure hashes (and classes, which you'll learn about later !)
# The syntax is basically `%hash-name (:key($variable-to-store-value-in))`.
# The hash can stay anonymous if you only need the values you extracted.
sub key-of(% (:value($val), :qua($qua))) {
  say "Got val $val, $qua times.";
}

# Then call it with a hash: (you need to keep the brackets for it to be a hash)
key-of({value => 'foo', qua => 1});
my %hash = value => 'Perl6', qua => '2016';
key-of(%hash); # the same (for an equivalent `%hash`);

```



```perl6
my $bag = bag "red" => 2, "blue" => 10;
say $bag.roll(10); # 随机生成 10 组
say $bag.pick(*).join("\n");

$bag = bag "red" => 20000000000000000001, "blue" => 100000000000000000000;
say $bag.roll(10);
say $bag.pick(10).join(" ");
```




### Bind
---


```perl6
﻿use v6;

my @a := 1..Inf;
say @a[^10]; # 1 2 3 4 5 6 7 8 9 10

my @primes  := @a.grep(*.is-prime);
say @primes[^10]; # 2 3 5 7 11 13 17 19 23 29

my @nprimes := @primes.map({ "{++state $n}: $_" });
.say for @nprimes[^10];

.say for (1..Inf
    ==> grep(*.is-prime)
    ==> map({ "{++state $n}: $_" })
    )[^10];


```



```perl6
my @a := (1, 2, 3);
say @a.WHAT;
say @a.elems;

my @b = (1, 2, 3);
@b[0] := my $x;
$x = 42;
say @b;

```



```perl6
use v6;

my $original = 5;say $original;
my $mirror;
$original := $mirror;       # normal binding, done on runtime

say $mirror;

$original ::= $mirror;      # same thing, but done during compile time
say $mirror;
$original = 3;
say $mirror;                # prints 3
$original =:= $mirror;       # true, because their bound together
$original === $mirror;       # alsotrue, because content and type are equal

```


## IO
---


```perl6
my $total = 0;
for @files -> $filename {
    $total += lines($filename.IO).grep(
        { $_ !~~ /<&insigline>/ }
).elems;
 CATCH {
        when X::IO {
            note "Couldn't read $filename";
} }
}
say $total;
# As CATCH goes inside of the scope,we can see the scope's lexicals!


```



```perl6
use v6;

# create a file
my $f = open "foo",:w;
$f.print(time);
$f.close;

# copy
my $io = IO::Path.new("foo");
$io.copy("foo2");

# clean up
unlink ("foo2");

# move
$io.rename("foo2");

# clean up
unlink ("foo2");


```



```perl6
use v6;

# create a file
my $f = open "foo",:w;
$f.print(time);
$f.close;

unlink "foo";

```



```perl6
1428936330
```



```perl6
1428936330
```



```perl6
use v6;

my $fn =  $?FILE;

my Instant $i = $fn.IO.accessed;
my $dt = $i.to-posix;

say :$dt.perl;

```



```perl6
# slurp 读入到数组后只是一个元素
my @lines = slurp('3col.txt');
say @lines.elems;

my $fh = open('3col.txt');
for $fh.lines -> $line {
    say $line;
}


```



```perl6
﻿> 15 + 27
42
> <beer vodka whisky>.pick
beer
> <beer vodka whisky>.pick(2)
beer whisky
> (1, 1, *+* ... *)[20]
10946
>  dir>>.path
> dir>>.path ==> grep /\.p6$/
REPL.p6 slides.p6

> type REPL.p6 | perl6 -e "$*IN.slurp-rest.comb(/\w+/) ==> sort *.chars ==> @temp ==> reverse @temp ==> @reverse ==> say @reverse
> type REPL.p6 | perl6 -e "$*IN.slurp-rest.comb(/\w+/) ==> sort *.chars ==> reverse @() ==> join \"\n\" ==> say @()"
> type Hamlet.txt | perl6 -e "say [max] $*IN.slurp-rest.comb(/\d+/)"

# slurp reads a file into a scalar
> dir>>.path ==> grep /\.p6$/ ==> sort { slurp($_).chars }
slurp_feed.p6 REPL.p6 slides.p6

# lines reads the lines of a file into an array
> dir>>.path ==> grep /\.p6$/ ==> sort { +lines($_) }
REPL.p6 slides.p6 slurp_feed.p6
# 求出所有 words的和
> type 3col.txt | perl6 -e "say [+] $*IN.lines>>.words"
> type 3col.txt | perl6 -e "say [+] $*IN.lines>>.words>>.elems # word 的个数
> type 3col.txt | perl6 -e "say [+] $*IN.lines>>.words>>.[2]
```



```perl6
use v6;
my %words;
for slurp.comb(/\w+/).map(*.lc) -> $word {
    %words{$word}++;
}
say %words:kv;
```




## Classes
---

Classes In Perl 6

使用带有 block 的 class 关键字引入一个类：

```perl6
class Puppy {
    ...
}
```

 或使用

```perl6
class Puppy;
...
1;
```

把类相关的东西`单独`写进一个文件.


```perl6
class Enemy {
    method attack-with-arrows   { say "peow peow peow" }
    method attack-with-swords   { say "swish cling clang" }
    method attack-with-fireball { say "sssSSS fwoooof" }
    method attack-with-camelia  { say "flap flap RAWWR!" }
}

# 创建一个方法筛选器， 方法名以 attack-with- 开头
# 对象的 ^methods 方法返回该对象所有的方法，包括自定义的方法
my $selector = { .name ~~ /^ 'attack-with-' / };
given Enemy.new -> $e {
    my $attack-strategy
        = $e.^methods().grep($selector).pick();

    $e.$attack-strategy();           # call a random method
}

```



```perl6
class Parent {
    method frob {
        say "the parent class frobs"
    }
}

class Child is Parent {
    method frob {
        say "the child's somewhat more fancy frob is called"
    }
}
# 对象的实际类型决定了要调用哪个方法
my Parent $test;
$test = Child.new;
$test.frob;          # calls the frob method of Child rather than Parent

```



```perl6
class Dog {
    has $.name is rw;
    has $.color;

    method kugo {
       say "hello ",$.name;
    }
}
my $pet = Dog.new(
    name => 'Spot', color => 'Black'
);
$pet.kugo();
$pet.name = 'Fido'; # OK
$pet.kugo();
$pet.color = 'White'; # Fails

```



```perl6
class Journey {
    has $.origin;
    has $.destination;
    has @!travellers;
    has $.notes is rw;

    method add_traveller($name) {
        if $name ne any(@!travellers) {
            push @!travellers, $name;
        } else {
            warn "$name is already going on the journey!";
        }
    }

    method describe() {
        "From $!origin to $!destination";
    }
    # Private method
    method !do-something-private($x) {
       ($x + 120)*0.88; # 先加价，再打折！
    }

    method price($x) {
        self!do-something-private(2*$x);
    }

}

my $vacation = Journey.new(
    origin      => 'China',
    destination => 'Sweden',
    notes       => 'Pack hiking'
);

say $vacation.origin;
$vacation.notes = 'Pack hiking gear and sunglasses!';
say $vacation.notes;
$vacation.add_traveller('Larry Wall');
say $vacation.price(40);
$vacation.add_traveller('Larry Wall');

```



```perl6
use MethodModifiers;
class C {
method m() is before { say "before"; } method m() { say "in the method"; } method m() is after { say "after"; }
} C.m;

```



```perl6
class Journey {
    has $.origin;
    has $.destination;
    has @!travellers;
    has $.notes is rw;

    method add_traveller($name) {
        if $name ne any(@!travellers) {
            push @!travellers, $name;
        }
        else {
            warn "$name is already going on the journey!";
        }
    }

    method describe() {
        "From $!origin to $!destination"
    }
}



```



```perl6
class Point {
    has $.x;
    has $.y = 2 * $!x;
}

my $p = Point.new( x => 1, y => 2);
say "x: ", $p.x;
say "y: ", $p.y;

my $p2 = Point.new( x => 5 );
# the given value for x is used to calculate the right
# value for y.
say "x: ", $p2.x;
say "y: ", $p2.y;

```



```perl6
class Journey {
    has $.origin;
    has $.destination;
    has @!travellers;
    has $.notes;  # 没有添加 is rw 限制时, 属性默认是只读的!
}

my $j = Journey.new(
    origin      => 'Sweden',
    destination => 'China',
    notes       => 'Be careful your money!'
);

say $j.origin;
say $j.destination;
say $j.notes;

# now, try to change notes
$j.notes = 'gun nima dan'; # Cannot modify an immutable Str
say $j.notes;

```



```perl6
class Journey {
    has $.origin;
    has $.destination;
    has @!travellers;
    has $.notes is rw;
}

# Create a new instance of the class.
my $vacation = Journey.new(
    origin      => 'Sweden',
    destination => 'Switzerland',
    notes       => 'Pack hiking gear!'
);

# 使用取值器, 这输出 Sweden.
say $vacation.origin;
# 使用 rw 存取器修改属性的值
$vacation.notes = 'Pack hiking gear and sunglasses!';
say $vacation.notes;

```



```perl6
class Paper   { }
class Scissor { }
class Stone   { }

multi win(Paper   $a, Stone   $b) { 1 }
multi win(Scissor $a, Paper   $b) { 1 }
multi win(Stone   $a, Scissor $b) { 1 }
multi win(Any     $a, Any     $b) { 0 }

say win(Paper.new, Scissor.new); # 0
say win(Stone.new, Stone.new); #0
say win(Paper.new, Stone.new); #1

```



```perl6
class Point2D {
    has $.x;
    has $.y;

    submethod BUILD(:$!x, :$!y) {
        say "Initalizing Point2D";
    }
}

class InvertiblePoint2D is Point2D {
    submethod BUILD() {
        say "Initilizing InvertiblePoint2D";
    }
    method invert {
        self.new(x => - $.x, y => - $.y);
    }
}

say InvertiblePoint2D.new( x => 1, y => 2);


```


## Junctions
---


```perl6
my @bad_ext = ('plx', 'pm', 'pl', 'p6');
my $file_ext = 'p6';
if lc($file_ext) eq any(@bad_ext) {
    say "$file_ext files is  allowed, You are a Perler";
}


```



```perl6
use v6;

my $filename = "temp.txt";
my $fh = open $filename, :w;

for dir(test => all(/p6$/, /^<-[._]>/ )) -> $file {
    $fh.say(“```perl66”);
    my $string = slurp $file;
    $fh.say($string);
    $fh.say(“```”);
    $fh.say();
}

$fh.close;

```

## Lazy List
---


```perl6
use v6;
my @integers = 1..*;
    for @integers -> $i {
        say $i;
        last if $i % 17 == 0;
    }
```



```perl6
# A list comprehension is a special syntax in some programming languages to describe lists. It is similar to the way mathematicians describe sets, with a set comprehension, hence the name.

# Some attributes of a list comprehension are that:
# 1. They should be distinct from (nested) for loops within the syntax of the language.
# 2. They should return either a list or an iterator (something that returns successive members of a collection, in order).
# 3. The syntax has parts corresponding to that of set-builder notation.

# Write a list comprehension that builds the list of all Pythagorean triples with elements between 1 and n. If the language has multiple ways for expressing such a construct (for example, direct list comprehensions and generators), write one example for each.

use v6;

my $n = 20;
my @list := gather for 1..$n -> $x {
         for $x..$n -> $y {
           for $y..$n -> $z {
             take $x,$y,$z if $x*$x + $y*$y == $z*$z;
           }
         }
       }
.say for  @list;

# Note that gather/take is the primitive in Perl 6 corresponding to generators or coroutines in other languages. It is not, however, tied to function call syntax in Perl 6. We can get away with that because lists are lazy, and the demand for more of the list is implicit; it does not need to be driven by function calls.
```



## Loops
---


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



## Main
---


```perl6
 #!/usr/bin/env perl6

    constant @months = <January February March April May June July
                        August September October November December>;
    constant @days = <Su Mo Tu We Th Fr Sa>;


    sub center(Str $text, Int $width) {
        my $prefix = ' ' x ($width - $text.chars) div 2;
        my $suffix = ' ' x $width - $text.chars - $prefix.chars;
        return $prefix ~ $text ~ $suffix;
    }

    sub MAIN(:$year = Date.today.year, :$month = Date.today.month) {
        my $dt = Date.new(:year($year), :month($month), :day(1) );
        my $ss = $dt.day-of-week % 7;
        my @slots = ''.fmt("%2s") xx $ss;

        my $days-in-month = $dt.days-in-month;
        for $ss ..^ $ss + $days-in-month {
            @slots[$_] = $dt.day.fmt("%2d");
            $dt++
        }

        my $weekdays = @days.fmt("%2s").join: " ";
        say center(@months[$month-1] ~ " " ~ $year, $weekdays.chars);
        say $weekdays;
        for @slots.kv -> $k, $v {
            print "$v ";
            print "\n" if ($k+1) %% 7 or $v == $days-in-month;
        }
    }

     # April 2014
# Su Mo Tu We Th Fr Sa
       # 1  2  3  4  5
 # 6  7  8  9 10 11 12
# 13 14 15 16 17 18 19
# 20 21 22 23 24 25 26
# 27 28 29 30
```



```perl6
﻿# Perl 6 supports writing a MAIN subroutine that is invoked at startup.
# Automatically maps arguments to parameters and generates usage instructions.


sub MAIN($number, Bool :$upto) {
    my @fib = 1, 1, *+* ... Inf;
    if $upto {
    say join ',', @fib[0 ..^ $number];
    }
    else {
        say @fib[$number - 1];
    }
}

#`(
> perl6 MAIN.p6 10
55

> perl6 MAIN.p6 10 --upto
Usage:
  MAIN.p6 [--upto] <number>

# 可选参数写在必选参数前面
> perl6 MAIN.p6 --upto 10
1,1,2,3,5,8,13,21,34,55
)

```



```perl6
use v6;
sub MAIN($file1, $file2) {
    my $words1 = bag slurp($file1).comb(/\w+/).map(*.lc);
    my $words2 = set slurp($file2).comb(/\w+/).map(*.lc);
    my $unique = ($words1 (-) $words2);
    for $unique.list.sort({ -$words1{$_} })[^10] -> $word {
        say "$word: { $words1{$word} }";
    }
}
```




## Types
---

## Sort
---


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

```



```perl6
use v6;

my %grade = "example.txt".IO.lines.map: {
    m:s/^(\w+) (<[A..F]><[+-]>?)$/
        or die "Can't parse line '$_'";

    ~$0 => ~$1;
}

say %grade;

say "Zsófia's grade: %grade<Zsófia>";
say "List of students with a failing grade:";

say " " ~%grade.grep(*.value ge 'F')».key.join(" ");

say "Distribution of grades by letter:";
say "  {.key}: {+.value} student{"s" if .value != 1}"
    for %grade.classify(*.value.comb[0]).sort(*.key);

```



```perl6
# Sort an array of composite structures by a key. For example, if you define a composite structure that presents a name-value pair (in pseudocode):
# Define structure pair such that:
   # name as a string
   # value as a string


# and an array of such pairs:
# x: array of pairs


# then define a sort routine that sorts the array x by the key name.

# This task can always be accomplished with Sorting Using a Custom Comparator. If your language is not listed here, please see the other article.

# Works with: Rakudo Star version 2010.08

my class Employee {
   has Str $.name;
   has Rat $.wage;
}

my $boss     = Employee.new( name => "Frank Myers"     , wage => 6755.85 );
my $driver   = Employee.new( name => "Aaron Fast"      , wage => 2530.40 );
my $worker   = Employee.new( name => "John Dude"       , wage => 2200.00 );
my $salesman = Employee.new( name => "Frank Mileeater" , wage => 4590.12 );

my @team = $boss, $driver, $worker, $salesman;

my @orderedByName = @team.sort( *.name )>>.name;
my @orderedByWage = @team.sort( *.wage )>>.name;

say "Team ordered by name (ascending order):";
say @orderedByName.join('  ');
say "Team ordered by wage (ascending order):";
say @orderedByWage.join('  ');

# this produces the following output:
# Team ordered by name (ascending order):
# Aaron Fast   Frank Mileeater   Frank Myers   John Dude
# Team ordered by wage (ascending order):
# John Dude   Aaron Fast   Frank Mileeater   Frank Myers


# Note that when the sort receives a unary function, it automatically generates an appropriate comparison function based on the type of the data.

```



```perl6
class Student {
    has Str $.name;
    has Int $.grade is rw;
}

my $stu1 = Student.new(name => "zhangwuji", grade => 124);
my $stu2 = Student.new(name => "yangguo",   grade => 128);
my $stu3 = Student.new(name => "zhaomin",   grade => 145);
my $stu4 = Student.new(name => "sunyizhe",  grade => 145);
my $stu5 = Student.new(name => "zhouziruo", grade => 128);
my $stu6 = Student.new(name => "qiaofeng",  grade => 124);

my @students = $stu1, $stu2,$stu3,$stu4,$stu5,$stu6;

# classify
# classify 返回一个散列
for @students.classify( *.grade ).sort -> $group {
    say "These students got grade $group.key():";
    say .name for $group.value.list;
}


# sort
my @c = @students.sort: -> $a, $b {$a.grade <=> $b.grade};
say @c.perl;
```



```perl6
use v6;

 my $file = open 'scores';
 my @names = $file.get.words;

 my %matches;
 my %sets;

 for $file.lines -> $line {
 my ($pairing, $result) = $line.split(' | ');
 my ($p1, $p2) = $pairing.words;
 my ($r1, $r2) = $result.split(':');

 %sets{$p1} += $r1;
 %sets{$p2} += $r2;

 if $r1 > $r2 {
 %matches{$p1}++;
 } else {
 %matches{$p2}++;
 }
 }

 my @sorted = @names.sort({ %sets{$_} }).sort({ %matches{$_} }).reverse;

 for @sorted -> $n {
 say "$n has won %matches{$n} matches and %sets{$n} sets";
 }
```


## Subset
---

- 限制字符串长度

```perl6
   subset NonEmptyString
       of Str
       where *.chars > 0; # 可以把约束条件写到多行

   sub firstName(NonEmptyString $name) {
       say "your name is $name";
   }

   firstName('Larry');
   firstName('');
```

输出：
```text
your name is Larry
Constraint type check failed for parameter '$name'
  in sub firstName at subset.p6:5
  in block <unit> at subset.p6:10
```

- 限制值域

```perl6
subset PointLimit of Int where -10 <= * <= 10;
sub test(PointLimit $number) {
    say $number;
}
test(-5); # -5

subset SmallInt of Int where -10 .. 10;
sub small(SmallInt $number) {
    say $number;
}
small(8);


```


- 检测密码是否合法

```perl6
use v6;
# 安全的密码
# 至少 8 位
# 包含大写字母、小写字母
# subset 不能使用 set(*.comb)  形式？

subset Password of Str where *.chars >=8 && any('A'..'Z','a'..'z') ∈ *.comb.Set;

sub passwordCheck(Password $password) {
    say "Password is Valid";
}

passwordCheck("abcdABCD");

```


- 检测密码是否有效并提醒

```perl6
use v6;

subset Length8    of Str where *.chars < 8;
subset UpCase     of Str where none('A'..'Z') ∈ *.comb.Set;
subset LowerCase  of Str where none('a'..'z') ∈ *.comb.Set;
subset IntNumber  of Str where none('0'..'9') ∈ *.comb.Set;

my $guess = prompt('Enter your password:');

given $guess {
    when Length8   { say '密码长度必须为 8 位 以上'; proceed }
    when  UpCase   { say '密码必须包括大写字母';     proceed }
    when LowerCase { say '密码必须包含小写字母';     proceed }
    when IntNumber { say '密码必须包含数字';                 }
}
```


该程序具有可扩展性， 要增加一种密码验证， 只有添加一个 subset 就好了，然后在 given/when 里面增加一个处理。
proceed 相当于 continue， 不像 C 里面的 falling through， Perl 6 里面的 proceed 在继续执行 when 语句时会计算 when 后面的条件

when 可以作为语句修饰符单独使用

```perl6
doit() when 42
```

等价于

```perl6
doit() when $_ ~~ 42
```

这在列表解析里面很有用

```perl6
my @lucky = ($_ when /7/ for 1..100);
7 17 27 37 47 57 67 70 71 72 73 74 75 76 77 78 79 87 97

```



```perl6
subset NonEmptyString
    of Str
    where *.chars > 0;

sub firstName(NonEmptyString $name) {
    say "your name is $name";
}

firstName('Larry');
firstName('');


```


## Traits
---


```perl6
sub add (Int $inputA, Int $inputB --> Int)
{
    my $result = $inputA+$inputB;

    say $result;         # Oops, this is the last statement, so its return value is used for the subroutine
}

# my $sum = add(5,6);
# Type check failed for return value; expected 'Int' but got 'Bool'

sub add2 (Int $inputA, Int $inputB --> Int)
{
    my $result = $inputA+$inputB;

    return $result;
}

my $sum2 = add2(5,6);
say $sum2;

sub add3 (Int $inputA, Int $inputB) returns Int
{
    my $result = $inputA+$inputB;

    return $result;
}

my $sum3 = add3(5,6);
say $sum3;

```



```perl6
sub fib(Int $nth where * >= 0) {
  given $nth {
    when 0 { 0 }
    when 1 { 1 }
    default { fib($nth-1) + fib($nth-2) }
 }
}
say fib(3);
#say fib(-3);

subset FirstName
    of Str
  where 0 < *.chars && *.chars < 256;

sub first_name(FirstName $name){
    say "$name";
}

first_name("Wall");

subset PointLimit of Int where -10 <= * <= 10;
sub test(PointLimit $number) {
    say $number;
}
test(-5);

subset SmallInt of Int where -10 .. 10;
sub small(SmallInt $number) {
    say $number;
}

small(8);

```



```perl6
multi sub trait_mod:<is>(Routine $r, :$doubles!) {
    $r.wrap({
        2 * callsame;
    });
}

sub square($x) is doubles {
    $x * $x;
}

say square 3;

```



```perl6
my @list of Int = 1..10000;
say @list[99].WHAT;

```



```perl6
sub divide(Int $a,
           Int $b where { $^n != 0 }) {
    return $a / $b;
}
say divide(120, 3); # 42
# say divide(100, 0); # Type check failure

# Here is an example of using subtypes to distinguish between two candidates

multi say_short(Str $x) {
    say $x;
}
multi say_short(Str $x
                  where { $_.chars >= 12 }) {
   say substr($x, 0, 10) ~ '...';
}
say_short("Beer!");         # Beer!
say_short("BeerBeerBeer!"); # BeerBeerBe...

```



```perl6
# Typed Parameters Can restrict a parameter to only accept arguments of a certain type.
sub show_dist(Str $from, Str $to, Int $kms) {
    say "From $from to $to is $kms km.";
}
show_dist('Copenhagen', 'Beijing', 7305);
show_dist(7305, 'Copenhagen', 'Beijing');

```


## Twigils
---


```perl6
 class Point {
        has $.x;
        has $.y;

        method Str() {
           return ($.x, $.y); # 注意我们这次使用 . 而不是 !
        }
    }

my $p = Point.new(x=>10,y=>20);
my ($height,$wide) = $p.Str();
say "高度:$height";
say "宽度:$wide";


 class SaySomething {
        method a() { say "a"; }
        method b() { say $.a; }
    }

    SaySomething.a; # prints "a"
    SaySomething.b;

```



```perl6
my @fave_foods = <hanbao pingguo TV>;
for @fave_foods -> $food {
    say "Jonathan likes to eat $food";
}
# The bit between the curly braces is done for each thing in the array
# -> $name means “declare $name and put the current thing into it”

# $^identifier 变量用于块中:

my @str = <a very long but shorthand really>;
my @sorted = sort { $^a.chars <=> $^b.chars}, @str;
say @sorted;

# sort 可以更简洁
my @s = sort { .chars }, @str;
say @s;

my $block = {
    $^a + $^b;
};
say $block(1,99);

```



```perl6
use v6;
# Twigils影响变量的作用域。请记住, twigils 对基本的魔符插值没有影响，那就是:
# 如果  $a 内插， $^a, $*a, $=a, $?a, $.a, 等等也会内插. 它仅仅取决于 $.

    my $lexical   = 1;
    my $*dynamic1 = 10;
    my $*dynamic2 = 100;

    sub say-all() {
        say "$lexical, $*dynamic1, $*dynamic2";
    }

    # prints 1, 10, 100
    say-all();

    {
        my $lexical   = 2;
        my $*dynamic1 = 11;
        $*dynamic2    = 101;

        # prints 1, 11, 101 ,why 2, 11 ,101?
        # $lexical isn't looked up in the caller's scope but in the scope &say-all was defined in.
        # The two dynamic variables are looked up in the callers scope and therefore have the values 11 and 101.
        # 翻译过来就是, $lexical 不是在调用者的作用域内被查找, 而是在 &say-all 被定义的作用域那儿
        # 也就是第一行的 $lexical = 1 了. 另外两个动态作用域变量在调用者的作用域内被查找, 所以值为 11 和 101
        say-all();
    }

    # prints 1, 10, 101
    say-all();

```



```perl6
use v6;

# ? twigil 编译常量
say "$?FILE: $?LINE"; # wenhao.p6: 4

```



```perl6
# ^ twigil 为block 块 或 子例程 申明了一个形式位置参数.
# 形如 $^variable 的变量是一种占位变量. 它们可用在裸代码块中来申明代码块的形式参数. 看下面代码中的块:

for ^4 {
    say "$^seconds follows $^first";
}

# 1 follows 0
# 3 follows 2

```


## Whatever
---

 当作为一个项使用时， 我们把 * 叫做 "whatever"
 当不是实际值时，它用作占位符
 例如, 1, 2, 3 ... *
 意思是没有终结点的自然数序列

 Whatever 闭包
 Whatever 最强大的用处是 Whatever 闭包

 对于 whatever 没有特殊意义的普通操作符：
     把 whatever 当作参数传递时就创建了一个闭包！
 所以，举个例子，  `* + 1` 等价于 `-> $a { $a + 1 }`
 `* + *` 等价于 `-> $a, $b { $a + $b }`

```perl6
@list.grep(* > 10) # 返回 @list 数组中所有大于 10 的元素
@list.map(* + *)   # 返回 @list 中每对儿的和
```

如果给 @a[ ] 的方括号里面传递一个闭包， 它会把 @a 数组的元素个数作为参数传递并计算！

- 数组的最后一个元素

```perl6
my @a =  1,22,33,11;
say @a[*-1];
say @a[->$a {$a-1}]; # $a  即为数组@a 的元素个数
```

- 数组的倒数第二个元素

```perl6
say @a[*-2];
say @a[->$a {$a-2}];
```

所以 `@a[*/2]` 是 @a 数组的中间元素
`@a[1..*-2]`  是@a 中不包含首尾元素的其它元素
`1, 1, * + * ... *`  是一个无限列表
* + * 是后来值的生成规则， 最后一个 * 表示没有终结测试。


```perl6
# 把闭包存储在标量中
my $a = -> $b { $b + 1 }
$a(3) # 4
```



Perl 6 的列表求值是惰性的
只要你不要求最后一个元素， 无限列表是没问题的。
使用绑定 (:=) 操作符把列表赋值给变量：

```perl6
    my @fib := 1, 1, * + * ... *
```


如果我稍后要 @fib[40] 的值， 会生成足够多的元素以获取数组的第 41 个元素
那些生成的元素会被记忆。
尽管未来， 如果列表未绑定给变量， 之前的值会被忘记
大部分 Perl 6 列表函数能作用并生成惰性列表
@a.map 和 @a.grep 都生成惰性列表， 即使 @a 不是惰性的。
@fib.grep(* %% 2) 是一个偶数惰性列表，例如
@fib Z @a 生成一个惰性列表： @fib[0], @a[0], @fib[1], @a[1] ...
给 for 循环传递一个无限列表是没问题的， 它会循环直到停止。


## When
---


```perl6
# given 和 when：
# given 接收一个参数和一个 block
# given 把它接收的参数设置为 $_, 然后调用后面的 block
# when 也接收一个参数和一个 block
# when 将 $_ 和 when 接收的参数进行智能匹配
# 如果结果是 True， 则执行代码块
# 然后控制就跳出 when 的包围圈

my $ticks = 0;
given $ticks {
    when 1   {say "";      }
    when 1/2 {say "/";     }
    when Int {say $_.Str;  }
    when Rat {say $_.perl; }
    die "Duration must be Int or Rat, but it's { $_.WHAT }";
}

# given 和 when 可以单独使用
# 设置了 $_,  进行一系列操作时，given 比较方便
# 当 $_ 被设置后 ， when 可以用在任何地方
# when 在 for 循环中很方便

my $boring-lines = 0;
for $*IN.lines {
    when /"Lunasa" | "Altan"/ { say "Found band!";       }
    when /"fiddle" | "flute"/ { say "Found instrument!"; }
    $boring-lines++;
}


```




## Smart-Matching
---


```perl6
    constant A = 100;
    constant B = 100;

    my (%powers, %count);

    # find bases which are powers of a preceeding root base
    # store decomposition into base and exponent relative to root
    for 2..Int(sqrt A) -> \a {
        next if a ~~ %powers;
        %powers{a, a**2, a**3 ...^ * > A} = a X=> 1..*;
    }

    # count duplicates
    for %powers.values -> \p {
        for 2..B -> \e {
            # raise to power \e
            # classify by root and relative exponent
            ++%count{p.key => p.value * e}
        }
    }

    # add +%count as one of the duplicates needs to be kept
    say (A - 1) * (B - 1) + %count - [+] %count.values;
```



```perl6
use v6;
my %lilei     ='Math'=>98,'Chinese'=>'72','English'=>'128';
my %hanmeimei ='Math'=>98,'Chinese'=>'72','English'=>'128';
say "they have the same course" if %lilei.keys ~~ %hanmeimei.keys;
say 'true' if %lilei{%hanmeimei.keys} ~~ %hanmeimei.values;

my $a = 2
say so $a ~~ 1..3
say so $a ~~ Int
say so $a ~~ 23
say so $a ~~ {$_.Str ne $_.perl}
say so (1..3).ACCEPTS($a)

```




## Regexes
---


```perl6
 use v6;
 print 'ok' if '1,2,3' ~~ / \d+ % ',' /; # ok
```



```perl6
# caps 方法返回所有的捕获，命名的和位置的，按照它们匹配的文本在原始字符串中出现的顺序返回。返回的值是一个 Pair 对象列表。键是捕获的名字或数量，键值是对应的 Match 对象。

if 'abc' ~~ m/(.) <alpha> (.) / {
            say $/.caps.WHAT; # (Parcel)
            my @a = $/.caps;  
            say @a;           # 0 => ｢a｣ alpha => ｢b｣ 1 => ｢c｣ ( 0 => ｢a｣ 是一个 Pair 对象
            say " -> 这次匹配有  @a.elems() 个 Pair";
               for $/.caps {
                    say .key, ' => ', .value.Str; # 键值是 对应的Match 对象, 需要调用 Str 方法, 得到字符串.

             }
 }

 # Output:
 #          0 => a
 #      alpha => b
 #          1 => c


#  复习下 Parcel
# Parcel 由 () <>  逗号分割的列表, 或其它引用结构
# ()
# 1,2,3
# <a b c>
# <<a b c>>
# qw/a b c/

```



```perl6
my $s = 'the quick brown fox jumped over the the lazy dog';
if $s ~~ m/ << (\w+) \W+ $0 >> / { # if 不再需要圆括号
    say "Found '$0' twice in a row";
    say "Found '$/[0]' twice in a line" # $/[0]  可以简写为 $0
}

```



```perl6
 my $ingredients = 'milk, flour, eggs and sugar';
 # prints "milk, flour, eggs", 如果 say $/[0] 只会打印 || , 因为[] 是非捕获组!
 $ingredients ~~ m/ [\w+]+ % [\,\s*] / && say "|$/|";
# |milk, flour, eggs|
# 这里 \w+ 匹配一个单词，并且 [\w+]+ % [\,\s*]  匹配至少一个单词，并且单词之间用逗号和任意数量的空白分隔。
 '1,2,3' ~~ / \d+ % ',' / && say "|$/|";
# |1,2,3|
# %必须要跟在量词后面，否则报错。
# 在 [\w+] 里面 [ ] 是非捕获组

```



```perl6
# 如果你在捕获后面加上量词，匹配对象中的对应的项是一列其它对象：

use v6;
my $ingredients = 'eggs, milk, sugar and flour';

if $ingredients ~~ m/(\w+)+ % [\,\s*] \s* 'and' \s* (\w+)/ {
    say 'list: ', $/[0].join(' | ');
    say 'end: ', "$/[1]";
    say $/.elems; # 数组 $/ 中含有 2 个元素
    say $/[0].WHAT;  # ARRAY, 第一个捕获 $/[0] 其实是一个数组
    say $/[0].elems; # 3, 第一个 (\w+) 匹配了 3 次
}

# 这打印:
# list: eggs | milk | sugar
# end: flour

#  第一个捕获(\w+)被量词化了，所以$/[0]包含一列单词。代码调用 .join方法将它转换为字符串。
#  不管第一个捕获匹配了多少次（并且有$/[0]中有多少元素），第二个捕获$/[1]始终可以访问。

```



```perl6
use v6;
my $str = 'Germany was reunited on 1990-10-03, peacefully';

if $str ~~ m/ (\d**4) \- (\d\d) \- (\d\d) / {
    say $/.WHAT;  # Match
    say $/.elems; # 3
    say 'Year: ',"$/[0]";
    say 'Month: ',"$/[1]";
    say 'Day: ',"$/[2]";

    # usage as an array:
    say $/.join('-'); # prints 1990-10-03
}

# Year: 1990
# Month: 10
# Day: 03
# 1990-10-03
```



```perl6
my $s = 'the quick brown fox jumped over the the lazy dog';
my regex word { \w+ [ \' \w+ ]?              }
my regex dup  { « <danci=&word> \W+ $<danci> » } # 要使用 &name 调用正则, 就像调用子例程一样 &sub , 调用后的结果起名为 danci, 就像给子例程起名字一样
if $s ~~ m/ <dupword=&dup> / {
    say "Found '{$<dupword><danci>}' twice in a row";
    # say $/.keys(); # dupword, 获取散列的键
    say $/;
}

# 这段代码引入了一个名为 word 的正则表达式，它至少匹配一个单词字符，后面跟着一个可选的单引号和更多的单词字符。
# 另外一个名为 dup （duplcate的缩写，副本的意思）的正则包含一个单词边界锚点。

# 在正则里面，语法 <&word> 在当前词法作用域内查找名为word的正则并匹配这个正则表达式。
# <name=&regex> 语法创建了一个叫做 name 的捕获，它记录了 &regex 匹配的内容。

# 在这个例子中，dup 调用了名为 word 正则，随后匹配至少一个非单词字符，之后再匹配相同的字符串（ 前面word 正则匹配过的）一次，它以另外一个字符边界结束。这种向后引用的语法记为美元符号 $  后面跟着用尖括号包裹着捕获的名字。

# 在 if 代码块里， $<dupword> 是  $/{'dupword'} 的快捷写法。因为 $/ 是一个特殊的散列, 所以可以通过键 {'dupword'} 访问到散列的值. 它访问正则 dup 产生的匹配对象。
# dup 也有一个叫 danci 的子规则。从那个调用产生的匹配对象用 $<dupword><danci>来访问。

# 直接打印 $/ 的结果, $/ 这里又变成了一个特殊的散列, fuck, 上次它不是一个特殊的数组吗? 百变星君啊,擦!
#

# ｢the the｣
#  dupword => ｢the the｣
#   danci => ｢the｣

```



```perl6
 use v6;
 my $ingredients = 'eggs, milk, sugar and flour';

 if $ingredients ~~ m/(\w+)+ % [\,\s*] \s* 'and' \s* (\w+)/ {
 say 'list: ', $/[0].join(' | ');
 say 'end: ', "$1";
 }
```



```perl6
 use v6;
 my $str = 'Germany was reunited on 1990-10-03, peacefully';

 if $str ~~ m/ (\d**4) \- (\d\d) \- (\d\d) / {
 say 'Year: ',"$/[0]";
 say 'Month: ',"$/[1]";
 say 'Day: ',"$/[2]";
 # usage as an array:
 say $/.join('-'); # prints 1990-10-03
 }

```



```perl6
my regex insigline {
^ \s* [ <?> | '#' .* | '{' | '}' ] \s* $
}
  sub MAIN(*@files) {
      my $total = 0;
      for @files -> $filename {
           try {
           $total += lines($filename.IO).grep(
              { $_ !~~ /<&insigline>/ } ).elems;
      }
     note "can't read $filename " if $!;
   }
say $total;
}

```



```perl6
use v6;
 my $ingredients = 'eggs, milk, sugar and flour';

 if $ingredients ~~ m/:s ( \w+ )+ % \,'and' (\w+)/ {
 say 'list: ', $/[0].join(' | ');
 say 'end: ', "$/[1]";
 }
```



## Subroutine
---
`-->` 在 Perl 6 中是什么意思？
`-->` 就是指定返回值的类型


```perl6
sub add (Int $inputA, Int $inputB --> Int)
{
    my $result = $inputA+$inputB;

    say $result; # this is the last statement, so its return value is used for the subroutine
}

my $sum = add(5,6);
# Type check failed for return value; expected 'Int' but got 'Bool'


sub add2 (Int $inputA, Int $inputB --> Int)
{
    my $result = $inputA+$inputB;

    return $result;
}

my $sum2 = add2(5,6);
say $sum2;
```


使用  `return` 约束更清晰：


```perl6
sub add3 (Int $inputA, Int $inputB) returns Int
{
    my $result = $inputA+$inputB;

    return $result;
}

my $sum3 = add3(5,6);
say $sum3;
```


## Signatures
---

签名的几种形式：

```perl6
sub optional               {...}
sub basic($foo)            {...}
sub default($foo = 3)      {...}
sub named(:$name)          {...}
sub typed(Bool $is_foo)    {...}
sub constraint(Str $name
    where *.chars > 0)     {...}
```



```perl6
﻿use v6;

my @a=1,2,3;
my $s='Escape Plan';
my %h='Rakudo'=>'Star','STD'=>'Larry';

# 捕获就是一系列实参的签名
my $capture = \(@a,$s,%h);      # creating a capture, "\" was free since there are no references anymore
say(|$capture).perl;            # flatten into argument list (hash like context)
# ||$cap;                       # flatten into semicolon list (array like context)

```



```perl6
﻿use v6;

# A set of parameters form a signature. 一组形参组成签名
# A set of arguments form a capture.    一组实参组成捕获

sub greet($name, :$greeting = 'Hi') {
    say "$greeting, $name!";
}
greet('Лена', greeting => 'Привет');
```



```perl6
sub rectangle(:$width!, :$height!, :$char = 'X') {
    say $char x $width for ^$height;
}

rectangle char => 'o', width => 8, height => 4;
rectangle :width(20), :height<5>;

```



```perl6
﻿use v6;

# Sometimes, you need to do some more powerful validation on arguments.

sub discount($price, $percent
             where (1 <= $percent <= 100)) {
    say "You get $percent% off! Pay EUR " ~ $price - ($price * $percent / 100);
}
discount(100, 20);
discount(100, 200);
```



```perl6
﻿use v6;

# Be careful about using type constraints on arrays and hashes. The type constraints the elements.
# 在对数组和散列使用类型限制时要小心. 类型限制的是元素!

sub total(Array @distances) { # 限制数组 @distances 中的每个元素为数组.
    # WRONG! Takes an Array of Arrays!
}

sub total(Int @distances) {
    # Correct, takes an array of Ints.
}
```



```perl6
﻿use v6;

# Dispatch By Arity(he number of arguments that a function can take)
# Example (from Test.pm): dispatch by different number of parameters

multi sub todo($reason, $count) is export {
    $todo_upto_test_num = $num_of_tests_run + $count;
    $todo_reason = '# TODO ' ~ $reason;
}

multi sub todo($reason) is export {
    $todo_upto_test_num = $num_of_tests_run + 1;
    $todo_reason = '# TODO ' ~ $reason;
}
```



```perl6
﻿use v6;

# Can use multiple dispatch with constraints to do a lot of "write what you know" style solutions

# Factorial:
# fact(0) = 1
# fact(n) = n * fact(n - 1)

multi fact(0)  { 1 };
multi fact($n) { $n * fact($n - 1) };

say fact(10);


# Fibonacci Sequence:
# fib(0) = 0
# fib(1) = 1
# fib(n) = fib(n - 1) + fib(n - 2)

# mutil 声明的子例程语句结尾不需要跟分号;
multi fib(0)  { 0 }
multi fib(1)  { 1 }
multi fib($n) { fib($n - 1) + fib($n - 2) }

say fib(10);
```



```perl6
﻿use v6;

# Example: part of a JSON emitter

multi to-json(Array $a) {
    return '[ ' ~
        $a.values.map({ to-json($_) }).join(', ') ~
        ' ]';
}

multi to-json(Hash $h) {
    return '{ ' ~
        $h.pairs.map({
            to-json(.key) ~ ': ' ~ to-json(.value)
        }).join(', ') ~
        ' }';
}
```



```perl6
sub fst(*@ [$fst]){
    say $fst;
}

fst(1);
fst(1,2);

```



```perl6
sub is-in(@array, $elem) {
  # this will `return` out of the `is-in` sub
  # once the condition evaluated to True, the loop won't be run anymore
  map({ return True if $_ ==  $elem }, @array);
}

my @array = 1,2,3,4,5;
is-in(@array,3);

```



```perl6
sub escape ($str) {
    $_ = $str;
    # Puts a slash before non-alphanumeric characters
    s:g[<-alpha-digit>] = "\\$/";
}

say escape "foobar";

{
    sub escape ($str) {
        $_ = $str;
        # Writes each non-alphanumeric character in its hexadecimal escape
        s:g[<-alpha-digit>] = "\\x[{ $/.base(16) }]";
    }

    say escape "foo#bar?"; # foo\x[23]bar\x[3F]
}

# Back to original escape function
say escape "foo#bar?";
```



```perl6
sub greet($name, $greeting = 'Ahoj') {
    say "$greeting, $name!";
}
greet('Anna'); # Ahoj Anna
greet('Лена', 'Привет '); # Привет, Лена"

```



```perl6
use v6;

# In Perl 6, passing an array or hash works like passing a reference
# 在 Perl 6中, 传递数组或散列就像传递引用那样

sub example(@array, %hash) {
    say @array.elems;
    say %hash.keys.join(', ');
}

my @numbers = 1,2,3,4;
my %ages = Jnthn => 25, Noah => 120;
example(@numbers, %ages);
```



```perl6
use v6;

# Empty list sorts to the empty list
multi quicksort([]) { () }

# Otherwise, extract first item as pivot...
multi quicksort([$pivot, *@rest]) {

    # Partition.
    my @before = @rest.grep(* < $pivot);
    my @after  = @rest.grep(* >= $pivot);
    # Sort the partitions.
    (quicksort(@before), $pivot, quicksort(@after))
}

my @unsorted = <13 1 9 12 4 2015>;
say quicksort(@unsorted); # 1  4  9  12  13  2015
```



```perl6
use v6;

sub convert_currency($amount, $rate) {
    $amount = $amount * $rate;
    return $amount;
}

sub convert_currency_copy($amount is copy, $rate) {
    $amount = $amount * $rate;
    return $amount;
}

sub convert_currency_rw($amount is rw, $rate) {
    $amount = $amount * $rate;
    return $amount;
}



my $price = 99;
$price = convert_currency($price, 11.1);
$price_copy = convert_currency_copy($price, 11.1);
$price_rw = convert_currency_rw($price, 11.1);

say $price;
say $price_copy;
say $price_rw;

```



```perl6
use v6;

# In Perl 6, every value knows its type.

say 42.WHAT;
say "camel".WHAT;
say [1, 2, 3].WHAT;
say (sub ($n) { $n * 2 }).WHAT;

# (Int)
# (Str)
# (Array)
# (Sub)

# A type name in Perl 6 represents all possible values of that type.
```



```perl6
﻿use v6;

# Sometimes, you want to accept any type, but then transform it into another type before binding to the parameter
# 强制类型转换

sub show_dist($from, $to, $kms as Int) {
   say "From $from to $to is $kms km.";
}
show_dist('Kiev', 'Lviv', '469');
show_dist('Kiev', 'Lviv', 469.35);
```



```perl6
use v6;

# Can restrict a parameter to only accept arguments of a certain type.


sub show_dist(Str $from, Str $to, Int $kms) {
    say "From $from to $to is $kms km.";
}
show_dist('Kiev', 'Lviv', 469);
show_dist(469, 'Kiev', 'Lviv'); #  Error!
```



```perl6
sub  foo(@array [$fst, $snd]) {
  say "My first is $fst, my second is $snd ! All in all, I'm @array[].";
  # (^ remember the `[]` to interpolate the array)
}
my @tail = 1,2;
foo(@tail);

 #=> My first is 2, my second is 3 ! All in all, I'm 2 3

```



```perl6
use v6;

# Can extract values by attribute (only those that are declared with accessors)

sub nd($r as Rat (:$numerator, :$denominator)) {
    say "$r = $numerator/$denominator";
}
nd(4.2);
nd(3/9);
```



```perl6
sub slurp-in-array(@ [$fst, *@rest]) { # You could keep `*@rest` anonymous
  say $fst + @rest.elems;   # `.elems` returns a list's length.
                            # Here, `@rest` is `(3,)`, since `$fst` holds the `2`.
}
my @array = <2 3 4 5>;
slurp-in-array(@array);

```




## Built-in types and functions
---
### .classify
---

```perl6
.say for slurp("README.txt")\    # whole file into string
         .words()\               # split into list of words
         .classify( *.Str );

my @a = slurp("README.txt").words;
.say  for @a.classify( *.Str );

# 输出的是一个散列
# that => that that
# the  => the the the the the the the the the
# is   => is is is
# ......

my %hash = @a.classify( *.Str );
# 输出前 10 个最常见的单词
for %hash.sort({-.value.elems}).hash.kv.[^20] -> $key, $value {
    say $key ,"\t", $value.elems;

}
```

---


分类后，生成一个散列， 键是  分类依据（可以根据字符数，字符等），下面这例是根据字符数分类， 它会把字符数相同的元素归为一类， 键值就是数组里的元素。


```perl6
my @names = <Patrick Jonathan Larry Moritz Audrey>;
say .key, "\t", ~.values
     for @names.classify( *.chars );

# Output:
# 5       Larry
# 6       Moritz Audrey
# 7       Patrick
# 8       Jonathan

.say for slurp("README")\            # whole file into string
         .words()\                   # split into list of words
         .classify( *.Str )\         # group words w/ multiplicity
         .map({; .key => .value.elems })\   # 分号的作用是什么？
                                     # turn lists into lengths
         .sort( { -.value } )\       # sort descending
         .[ ^10 ];                   # 10 most common words
`#(
*.Str
I => I I I I, On => On On, a => a a a, black => black black black black black, day => day day day day, love => love, read => read, summer => summer summer, sunny => sunny sunny, to => to

 .map({; .key => .value.elems })\
On => 2 a => 3 sunny => 2 summer => 2 day => 4 I => 4 black => 5 love => 1 to => 1 read => 1  

 .sort( { -.value } )\
black => 5 day => 4 I => 4 a => 3 On => 2 sunny => 2 summer => 2 love => 1 to => 1 read => 1  
)
```


```perl6
class Student {
    has Str $.name;
    has Int $.grade is rw;
}
my $stu1 = Student.new(name => "zhangwuji", grade => 124);
my $stu2 = Student.new(name => "yangguo",   grade => 128);
my $stu3 = Student.new(name => "zhaomin",   grade => 145);
my $stu4 = Student.new(name => "sunyizhe",  grade => 134);

my @students = $stu1, $tu2,$stu3,$stu4;
my Student @students = get-students();

for @students.classify( *.grade ).sort -> $group {
    say "These students got grade $group.key():";
    say .name for $group.value.list;
}
```

### .pick and .roll
---

```perl6

say @deck.pick();                   # pick a card, any card...

say @deck.pick(5);                  # poker hand

my @shuffled = @deck.pick(*);       # here, '*' means 'keep going'

my @urn = <black white white>;      # beads, 1/3 black, 2/3 white
.say for @urn.roll(50);             # like .pick, but new each time

for @urn.roll(*) {
    .say;                           # infinite bead selector
}

say [+] (1..6).roll(4);             # 4d6

class Enemy {
    method attack-with-arrows   { say "peow peow peow"    }
    method attack-with-swords   { say "swish cling clang" }
    method attack-with-fireball { say "sssSSS fwoooof"    }
    method attack-with-camelia  { say "flap flap RAWWR!"  }
}

my $selector = { .name ~~ /^ 'attack-with-' / };
given Enemy.new -> $e {
    my $attack-strategy
        = $e.^methods().grep($selector).pick();

    $e.$attack-strategy();           # call a random method
}
```

### .sort
---


```perl6
# 1 if $a is higher, -1 if $b is higher, 0 if equal
$a <=> $b;

# 根据分数排序 students
@students.sort: -> $a, $b { $a.grade <=> $b.grade };

# same thing
@students.sort: { $^a.grade <=> $^b.grade };

# same thing
@students.sort: { .grade };

# same thing
@students.sort: *.grade;

# leg gives -1, 0 or 1 according to lexicographic ordering
# 'leg' is for Str, 'cmp' is now for type-agnostic sort
$a leg $b;

# sort students by name (Unicode order)
@students.sort: { $^a.name leg $^b.name };

# same thing
@students.sort: *.name;

# don't worry, things are properly cached; no re-evaluations
@items.sort: *.expensive-calculation();

# ...which means this works (and is a fair shuffle)
@deck.sort: { rand }

# ...but this is cuter :)
@deck.pick(*);
```


### 操作符重载
---

```perl6
sub infix:<±>($number, $fuzz) {
    $number - $fuzz + rand * 2 * $fuzz;
}

say 15 ± 5;                          # somewhere between 10 and 20

sub postfix:<!>($n) { [*] 1..$n }
say 5!;                              # 120

class Physical::Unit {
    has Int $.kg = 0;                # these attrs denote powers of units
    has Int $.m  = 0;                # eg $.kg == 2 means that this object
    has Int $.s  = 0;                # has a kg**2 unit

    has Numeric $.payload;

    method multiply(Physical::Unit $other) {
        Physical::Unit.new(
            :kg( $.kg + $other.kg ),
            :m( $.m + $other.m ),
            :s( $.s + $other.s ),
            :payload( $.payload * $other.payload )
        )
    }

    method invert() {
        Physical::Unit.new(
            :kg( -$.kg ), :m( -$.m ), :s( -$.s ),
            :payload( 1 / $.payload )
        )
    }

    method Str {
        $.payload
        ~ ($.kg ?? ($.kg == 1 ?? " kg" !! "kg**$.kg") !! "")
        ~ ($.m  ?? ($.m  == 1 ?? " m"  !! "m**$.m")   !! "")
        ~ ($.s  ?? ($.s  == 1 ?? " s"  !! "s**$.s")   !! "")
    }
}

sub postfix:<kg>(Numeric $payload) { Physical::Unit.new( :kg(1), :$payload ) }
sub postfix:<m>(Numeric $payload) { Physical::Unit.new( :m(1), :$payload ) }
sub postfix:<s>(Numeric $payload) { Physical::Unit.new( :s(1), :$payload ) }

# Note how we now use 'multi sub', so as not to shadow the original infix:<*>
multi sub infix:<*>(Physical::Unit $a, $b) {
    $a.clone( :payload($a.payload * $b) );
}

multi sub infix:<*>($a, Physical::Unit $b) {
    $b.clone( :payload($a * $b.payload) );
}

multi sub infix:<*>(Physical::Unit $a, Physical::Unit $b) {
    $a.multiply($b);
}

multi sub infix:</>(Physical::Unit $a, $b) {
    $a.clone( :payload($a.payload / $b) );
}

multi sub infix:</>($a, Physical::Unit $b) {
    $b.invert.clone( :payload($a / $b.payload) );
}

multi sub infix:</>(Physical::Unit $a, Physical::Unit $b) {
    $a.multiply($b.invert);
}

say 5m / 2s;                         # 2.5 m s**-1
say 100kg * 2m / 5s;                 # 40 kg m s**-1
```


### Z
---

```perl6
# Z (the 'zip operator') means "mix these lists together"
my @tastes = <spicy sweet bland>;
my @foods = <soup potatoes tofu>;
@tastes Z @foods; # <spicy soup sweet potatoes bland tofu>

class Student {
    has Str $.name;
    has Int $.grade is rw;
}

my $stu1 = Student.new(name => "zhangwuji", grade => 124);
my $stu2 = Student.new(name => "yangguo",   grade => 128);
my $stu3 = Student.new(name => "zhaomin",   grade => 145);
my $stu4 = Student.new(name => "sunyizhe",  grade => 145);
my $stu5 = Student.new(name => "zhouziruo", grade => 128);
my $stu6 = Student.new(name => "qiaofeng",  grade => 124);

my @students = $stu1, $stu2,$stu3,$stu4,$stu5,$stu6;
# » 为每个元素调用方法
.say for @students».grade;   # all the grades

for @students».name Z @students».grade -> $name, $grade {
    say "$name got a $grade this year";
}

# Note that the latter list is infinite -- it works anyway
for @students».name Z (1..6).roll(*) -> $name, $roll {
    say "$name rolls a $roll";
}

# you can also Z together two lists with an infix op
my @total-scores = @first-scores Z+ @second-scores;

# strings as keys, the appropriate number of 1s as values
my %hash = @names Z=> 1 xx *;    # xx is list repeat

# line people up with increasing numbers
my %people2numbers = @people Z=> 1..*;

# don't have a good op? roll your own!
sub infix:<likes>($liker, $likee) { "$liker is fond of $likee" }
# note how the op infix:<Zlikes> is automatically available
my @relations = @likers Zlikes @likees;

```



```perl6
> sub infix:<likes>($liker, $likee) { "$liker is fond of $likee" }
sub infix:<likes> (Any $liker, Any $likee) { #`(Sub+{Precedence}|140676897934560) ... }
> "aaa" likes "bbb"
aaa is fond of bbb
> my @a = <a b c>;
a b c
> my @b = <1 2 3>;
1 2 3
> @a Zlikes @b
a is fond of 1 b is fond of 2 c is fond of 3
> .say for @a Zlikes @b
a is fond of 1
b is fond of 2
c is fond of 3
```


### infix:<...>
---

```perl6
1 ... $n                                    # integers 1 to $n
$n ... 1                                    # and backwards

1, 3 ... $n                                 # odd numbers to $n
1, 3, ... *                                 # odd numbers
1, 2, 4 ... *                               # powers of two
map { $_ * $_ }, (1 ... *)                  # squares

1, 1, -> $a, $b { $a + $b } ... *           # fibonacci
1, 1, { $^a + $^b } ... *                   # ditto
1, 1, *+* ... *                             # ditto

'Camelia', *.chop ... '';                   # all prefixes of 'Camelia'
# Camelia Cameli Camel Came Cam Ca C
# See http://blog.plover.com/CS/parentheses.html
# for the principle behind this
sub next-balanced-paren-string($s) {
    $s ~~ /^ ( '('+ ) ( ')'+ ) '(' /;
    [~] $s.substr(0, $/.from),
        "()" x ($1.chars - 1),
        "(" x ($0.chars - $1.chars + 2),
        ")",
        $s.substr($/.to);
}

my $N = 3;

my $start = "()" x $N;
my &step = &next-balanced-paren-string;
my $end = "(" x $N ~ ")" x $N;

for $start, &step ... $end -> $string {
    say $string;
}

# Output:
# ()()()
# (())()
# ()(())
# (()())
# ((()))

```



## Chained Comparisons
---
## Muti dispatch
---

```perl6
multi foo(Int $x) { 1 }
multi foo(Int $x) is default { 2 }
say foo(1); # 2
```



```perl6
# Operator overloading in Perl 6 will be done by multi-dispatch routines
# (In fact, all of the built-in operators are invoked by a multi-dispatch.

# Dispatch By Arity
# 􏰀 Arity = number of arguments that a routine takes
#􏰀  Could do the previous example as:

multi sub greet($name) {
    say "Ahoj, $name!";
}
multi sub greet($name, $greeting) {
    say "$greeting, $name!";
}
greet('Anna'); # Ahoj Anna
greet('Лена', 'Привет '); # Привет, Лена"

# Type-Based Dispatch
#  􏰀 We can write types in a signature
#􏰀   They are used to help decide which candidate to call

multi sub double(Int $x) {
    return 2 * $x;
}
multi sub double(Str $x) {
    return "$x $x";
}
say double(21);      # 42
say double("he");   # he he

```



```perl6
multi sub MAIN('send', $filename) {
 ...
}
multi sub MAIN('fetch', $filename) {
 ...
}
multi sub MAIN('compare', $file1, $file2) {
 ...
}

#`(
... 是 yadayadayada 占位符
> perl6 "multiple_MAIN.p6"
Usage:
  multiple _MAIN.p6 send <filename>
  multiple _MAIN.p6 fetch <filename>
  multiple _MAIN.p6 compare <file1> <file2>
)
```

## Module management
---
## Native Library Calls
---
## Operators
---

```perl6
my @suits  = <♣ ♢ ♡ ♠>;
my @ranks  = 2..10, <J Q K A>;

# concatenate each rank with each suit (2♣ 2♢ 2♡ ... A♠)
my @deck = @ranks X~ @suits;

# build a hash of card names to point values
# 52 张牌， 4 种花色， A 的值为 11 ， J、Q、K 为 10
my %points = @deck Z @( (2..10, 10, 10, 10, 11) Xxx 4 );

# 把牌打乱
@deck .= pick(*); # shuffle the deck
my @hand = @deck.splice(0, 5); # 抓取前 5 张牌
say ~@hand; #  显示抓取的是哪 5 张
say [+] %points{@hand}; # 这 5 张牌面的值加起来是多少

```



```perl6
sub postfix:<!>($n where $n > 0) {
   [*] 2..$n
}
say 5!;

constant fact = 1, [\*] 1..*;
say fact[5];

```



```perl6
use v6;
my $file = open 'flip_flop.txt';
for $file.lines -> $line {
say $line if !($line ~~ m/^\;/ ff $line ~~ m/^\"/);
}

#`(
flip_flop.txt 内容如下：
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
输出：
There is more than one way to do it
                                -- Larry Wall
We hope Perl6 is wrote by the hole Socfilia
                                -- Larry Wall
programming is hard,Let's go shopping
                               -- Larry Wall
Ruby is Another Perl6
                               -- Larry Wall
圣诞节中的例子：

)
for 1..20 {.say if $_==9 ff $_==16}
say '-' x 10;
for 1..20 {.say if $_==9 ^ff $_==16}
say '-' x 10;
for 1..20 {.say if $_==9 ff^ $_==16}
say '-' x 10;
for 1..20 {.say if $_==9 ^ff^ $_==16}


```



## Awesome Operators
---

In Perl 6 we have a few new operators...

## Junctions
---

```perl6
if $status eq 'error' || $status eq 'warning' {
...
}
```

在 Perl 6 里面:

```perl6
if $status eq 'error' | 'warning' {
...
}
```


Perl 5 里面:

```perl6
while $value < $limit1 && $value < $limit2 {
...
}
```

Perl 6 里面

```perl6
while $value < $limit1 & $limit2 {
...
}
```


- Sequences

```perl6
say 1, 2, 4 ... 1024;
1 2 4 8 16 32 64 128 256 512 1024

my @fib = 1, 1, *+* ... *;
say @fib[0..9]
1 1 2 3 5 8 13 21 34 55 89
```

-  ^ (zero up to...)

```perl6
my @fib = 1, 1, *+* ... *;
say @fib[^10]
1 1 2 3 5 8 13 21 34 55 89
```


- Awesome Meta-operators
- Higher order operators
- Operators that **operate on operators**

- Reduction
 Puts an operator between every element in a list

```perl6
say [*] 1..10
3628800

my @sorted = 1,4,7,9,11;
my @unsorted = 3, 1, 9, 25;

say [<] @sorted;
say [<] @unsorted;

Bool::True
Bool::False
```

- Zip
Take elements from two or more lists and combine them with some operator


```perl6
say 1 .. 6 Z~ 'A'..'F’
1A 2B 3C 4D 5E 6F
```


- Cross
- All permutations of two or more lists, combined with some operator


```perl6
say 1 .. 3 X~ 'A'..'F‘
1A 1B 1C 1D 1E 1F 2A 2B 2C 2D 2E 2F 3A 3B 3C 3D 3E 3F
```


- Your Awesome Operators

 What if Perl 6 built in operators are not enough?

 You can add your own!

- Factorial
 Add a ! operator to do factorial


```perl6
sub postfix:<!>($n) { [*] 1..$n }
say 10!
3628800
```


    # And you have all of unicode!
    # Insert In Middle
    ## Operator to add an element to the middle of an array

-                      中


```perl6
sub infix:<中>(@array, $ins) {
    @array.splice(+@array / 2, 0, $ins);
    return @array;
}

my @a = 1,2,4,5;
say @a 中 3;

1 2 3 4 5
```


## Feeds
---


```perl6
use v6;
use Test;

{

    my @data = <1 2 4 5 7 8>;
    my @odds = <1 5 7>;

    eval_dies_ok('@data <== grep {$_ % 2} <== @data', 'a chain of feeds may not begin and end with the same array');

    @data = <1 2 4 5 7 8>;
    @data <== grep {$_ % 2} <== eager @data;
    # rakudo 2 todo 'feeds + eager'
    is(~@data, ~@odds, '@arr <== grep <== eager @arr works');

    @data = <1 2 4 5 7 8>;
    @data <== eager grep {$_ % 2} <== @data;
    is(~@data, ~@odds, '@arr <== eager grep <== @arr works');
}

```



```perl6
use v6;

use Test;

# L<S06/"Feed operators">
# L<S03/"Feed operators">

=begin pod

Tests for the feed operators

    ==> and <==

=end pod

plan 24;

{
    my @a = (1, 2);
    my (@b, @c);

    @a ==> @b;
    @c <== @a;

    is(~@b, ~@a, "ltr feed as simple assignment");
    is(~@c, ~@a, "rtl feed as simple assignment");
}

{
    my @a = (1 .. 5);
    my @e = (2, 4);

    my (@b, @c);
    @a ==> grep { ($_ % 2) == 0 } ==> @b;
    @c <== grep { ($_ % 2) == 0 } <== @a;
    my @f = do {@a ==> grep {($_ % 2) == 0}};
    my @g = (@a ==> grep {($_ % 2) == 0});

    is(~@b, ~@e, "array ==> grep ==> result");
    is(~@c, ~@e, "result <== grep <== array");
    is(~@f, ~@e, 'result = do {array ==> grep}');
    is(~@g, ~@e, 'result = (array ==> grep)');
}

{
    my ($got_x, $got_y, @got_z);
    sub foo ($x, $y?, *@z) {
        $got_x = $x;
        $got_y = $y;
        @got_z = @z;
    }

    my @a = (1 .. 5);

    @a ==> foo "x";

    is($got_x, "x", "x was passed as explicit param");
    #?rakudo 2 todo 'feeds + signatures'
    ok(!defined($got_y), "optional param y was not bound to fed list");
    is(~@got_z, ~@a, '...slurpy array *@z got it');
}

{
    my @data = <1 2 4 5 7 8>;
    my @odds = <1 5 7>;

    eval_dies_ok('@data <== grep {$_ % 2} <== @data', 'a chain of feeds may not begin and end with the same array');

    @data = <1 2 4 5 7 8>;
    @data <== grep {$_ % 2} <== eager @data;
    #?rakudo 2 todo 'feeds + eager'
    is(~@data, ~@odds, '@arr <== grep <== eager @arr works');

    @data = <1 2 4 5 7 8>;
    @data <== eager grep {$_ % 2} <== @data;
    is(~@data, ~@odds, '@arr <== eager grep <== @arr works');
}

# checking the contents of a feed: installing a tap
{
    my @data = <0 1 2 3 4 5 6 7 8 9>;
    my @tap;

    @data <== map {$_ + 1} <== @tap <== grep {$_ % 2} <== eager @data;
    is(@tap, <1 3 5 7 9>, '@tap contained what was expected at the time');
    #?rakudo todo 'feeds + eager'
    is(@data, <2 4 6 8 10>, 'final result was unaffected by the tap variable');
}

# <<== and ==>> pretending to be unshift and push, respectively
# rakudo skip 'double-ended feeds'
{
    my @odds = <1 3 5 7 9>;
    my @even = <0 2 4 6 8>;

    my @numbers = do {@odds ==>> @even};
    is(~@numbers, ~(@even, @odds), 'basic ==>> test');

    @numbers = do {@odds <<== @even};
    is(~@numbers, ~(@odds, @even), 'basic <<== test');
}

# feeding to whatever using ==> and ==>>

# rakudo skip 'double-ended feeds'
{
    my @data = 'a' .. 'e';

    @data ==> *;
    is(@(*), @data, 'basic feed to whatever');

    <a b c d> ==>  *;
    0 .. 3    ==>> *;
    is(@(*), <a b c d 0 1 2 3>, 'two feeds to whatever as array');
}

# feed and Inf
# nieza skip "unhandled exception
{
  lives_ok { my @a <== 0..Inf }
}

# nieza skip "Unhandled exception"
{
  my $call-count = 0;
  my @a <== gather for 1..10 -> $i { $call-count++; take $i };
  @a[0];
  #?rakudo todo "isn't lazy"
  is $call-count, 1;
}

# no need for temp variables in feeds: $(*), @(*), %(*)
# rakudo skip '* feeds'
# DOES 4
{
    my @data = 'a' .. 'z';
    my @out  = <a e i o u y>;

    @data ==> grep {/<[aeiouy]>/} ==> is($(*), $(@out), 'basic test for $(*)');
    @data ==> grep {/<[aeiouy]>/} ==> is(@(*), @(@out), 'basic test for @(*)');
    @data ==> grep {/<[aeiouy]>/} ==> is(%(*), %(@out), 'basic test for %(*)');

    # XXX: currently the same as the @(*) test above. Needs to be improved
    @data ==> grep {/<[aeiouy]>/} ==> is(@(*).slice, @(@out).slice, 'basic test for @(*).slice');
}


done;

# vim: ft=perl6

```



```perl6
use v6;
use Test;
{
    my ($got_x, $got_y, @got_z);
    sub foo ($x, $y?, *@z) {
        $got_x = $x;
        $got_y = $y;
        @got_z = @z;
    }

    my @a = (1 .. 5);

    @a ==> foo "x";

    is($got_x, "x", "x was passed as explicit param");
    # rakudo 2 todo 'feeds + signatures'
    ok(!defined($got_y), "optional param y was not bound to fed list");
    is(~@got_z, ~@a, '...slurpy array *@z got it');
}


```



```perl6
﻿my @a =slurp('Hamlet.txt').comb(/\w+/);
my @result = (@a ==> sort *.chars ==>  reverse @() ==> join "\n");
say  @result; # @() 不使用临时数组存储中间变量
```


## Hyper Operator
---

```perl6
use v6;

sub add($x) {
    sleep 3;
    $x ** 2 + 1;
}
# hyper  运算符现在还未实现并行， 该程序 sleep 了 9 秒
my @array = 1, 3, 5;
.say for @array».&add;
```


超运算符能运用在自定义的子例程上面：

为什么会使用 `.&` 语法呢？
因为 $obj.method 确实是一个方法调用， 而 `$obj.$coderef` 不是
add 函数的名字是 `&add`,  就像 foo 标量的名字是 `$foo`, 数组 foo 的名字叫做 @foo 一样
在 Perl 6 中， 如果你提到一个子例程却不带 & 符号， 那就是调用了它。在 add 前面加上 &  符号才能引用一个函数。


```perl6
> sub add($x) {$x * 2 + 1}
sub add (Any $x) { #`(Sub|140286460109400) ... }
> 2.add
Method 'add' not found for invocant of class 'Int'
> 2.&add
5
> add(2)
5
> my $function = sub add($x) {$x * 2 + 1}
sub add (Any $x) { #`(Sub|140286460109552) ... }
> 2.$function
5
> $function(2)
5
> my @a = 1, 2, 3
1 2 3
> @a>>.$function
3 5 7
> say $function.WHAT
(Sub)


$obj.$function 等价于 $function($obj)
```



## Meta Operators
---


```perl6
﻿use v6;

my @lines = slurp('3col.txt');
for @lines -> $line {
   my @b = $line.comb(/\d+/);
   say "@b[]";
   say "-" x 45;
}

# 没有打印出满意的结果, 因为 slurp 是把所有文本作为一个字符串吸入的.

#`(
my $fh = open('3col.txt');
my $num;
for $fh.lines -> $line {
    $num += $line.words.[2];
}
say $num;
)

my $fh = open('3col.txt');
# say [+] ($fh.lines>>.words).[2];
my @l = $fh.lines>>.comb(/\d+/);
say @l.elems;
```



```perl6
﻿use v6;

# 该源文件必须保存为 UTF8 格式 才不会报 UTF-8 错误, 即使是中文注释

sub infix:<中>(@array, $ins) {
    @array.splice(+@array / 2, 0, $ins);
    return @array;
}

my @a = 1,2,4,5;
say @a 中 3;

```



```perl6
# Loop over multiple arrays (or lists or tuples or whatever they're called in your language) and print the ith element of each. Use your language's "for each" loop if it has one, otherwise iterate through the collection in order with some other loop.

# For this example, loop over the arrays (a,b,c), (A,B,C) and (1,2,3) to produce the output
# aA1
# bB2
# cC3

for <a b c> Z <A B C> Z 1, 2, 3 -> $x, $y, $z {
   say $x, $y, $z;
}

# The Z operator stops emitting items as soon as the shortest input list is exhausted. However, short lists are easily extended by replicating all or part of the list, or by appending any kind of lazy list generator to supply default values as necessary.

# Note that we can also factor out the concatenation by making the Z metaoperator apply the ~ concatenation operator across each triple:
for <a b c> Z~ <A B C> Z~ 1, 2, 3 -> $line {
   say $line;
}

# We could also use the zip-to-string with the reduction metaoperator:
.say for [Z~] [<a b c>], [<A B C>], [1,2,3]

```



```perl6
﻿use v6;

class PowerBy2 {
    has $.number;

	method power_by2() {
	    return  $.number ** 2;
	}
}

my  $test = PowerBy2.new(number=>10);
say $test.power_by2;

my @a = <1 2 3 4>;
my @b = @a>>.power_by2;
say @b;
```



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



```perl6
use v6;
use List::Utils;

my @array = <1 3 4 6 7>;
my @b = sliding-window(@array,2);
.say for @b;
say '-' x 15;
my @c = combinations(@array);
.say for @c;
say '-' x 15;

my @d = combinations(@array,3);
.say for @d;

say '-' x 15;
my @e = combinations(@array,4);
.say for @e;

say '-' x 15;
my @take = take-while((1...*), * <= 10);
.say for @take;

say '-' x 15;
my @takerange = take-while((1...*), * + 4 < 10);
.say for @takerange;

say '-' x 15;
my @aa = uniq-by(<A B C a b c d D e>, *.uc);
my @bb = uniq-by((1..* Z -2..*), *.abs)[^10];
my @cc = uniq-by((1..* Z -2..*), *+1)[^10];
.say for @aa;
say '-' x 15;
.say for @bb;
say '-' x 15;
.say for @cc;
```



```perl6
use Test;
use List::Utils;

is take-while((1...*), * <= 10), ~(1...10), "take-while works on a basic infinite loop";
is take-while((1...*), * <= -1), "", "take-while works if condition is initially false";

done;

```



```perl6
my @a = 0..9;
my @b = 'a'..'z';
my @guess = @a X~ @b;
say @guess;

```



```perl6
﻿use v6;
my @a=1,2,3;
my @b=4,5,6;
my @c=7,8,9;
for zip(@a; @b; @c) -> $a, $b, $c {say $a,$b,$c;}

# 多行注释
my $things = #`( i wonder how many of these
 I will need, hm maybe 3, or 4, better 5 ) 5; # same as $things = 5;
say $things;

say Q/hello/;
say Q{world};
say Q|1234|;
say Q,comma,;
say Q[maohao];
say Q*askiles*;
say Q:a/@a[0]/;  # 1
say @a[1];


```



```perl6
use v6;

my @a = <王 孙 刘>;
my @b = <伟 亦 百>;
my @c = <霆 哲 爽>;
for zip(@a;@b;@c) -> $a,$b,$c {
    say "$a$b$c";
}

```



```perl6
use v6;

my @a=<aa bb cc dd ee ff >;
for @a -> $x,$y,$z {
    say $x,$y,$z;
}
say $*PROGRAM_NAME;
say $*CWD;
# aabbcc
# ddeeff
# three.p6
```



```perl6
One place infinite lazy lists do not work are the hyper meta operators
The idea is that conceptually they work on the entire list at once
Indeed, they are allowed to work on its elements in any order, and in parallel
(In practice, none of the Perl 6 compilers handle parallel processing yet)
Our first example of a meta operator: an operator built from a simpler operator
@a »+« @b produces an array which is the sum of the other two arrays
@a »%%» 2 produces an array of Bools telling which elements of @a are divisible by 2
~«@a is effectively the same as @a».Str -- it returns an array of strings

当无限惰性列表遇到 hyper 元操作符时, 就不起作用了。从概念上来讲, 它们实际上立即作用于整个列表, 它们被允许以任意顺序作用它们的元素
还能并行。（事实上, 目前没有一个 Perl 6 编译器能处理并行）
我们第一个关于元操作符的例子：一个构建自简单操作符的操作符 `@a »+« @b` 生成了一个数组, 这个数组是另外两个数组对应元素的和。
@a »%%» 2 生成一个布尔值的数组, @a 中的哪个元素能被 2 整除。
~«@a 和 @a».Str 一样 -- 返回一个字符串的数组。

如果你需要对两个无限惰性列表的元素进行求和, 你可以使用 `zip` 元操作符, 使用 `@a Z+ @b` 代替 `@a »+« @b`。
`Z+` 对列表进行惰性求值并按需返回它们的值。
等价于  `(@a Z @b).map(* + *)`。
<a b c> Z~ <1 2 3> 返回 a1 b2 c3
同样地 , 交叉操作符 X 有一个等价的 元操作符
<a b c> X~ <1 2 3> 返回 a1 a2 a3 b1 b2 b3 c1 c2 c3

If you do need to sum the elements of two infinite lazy lists, you can use the zip meta operator
Instead of @a »+« @b, you can do @a Z+ @b
Z+ evaluates the lists lazily and returns its values lazily
It's effectively doing (@a Z @b).map(* + *)
<a b c> Z~ <1 2 3> returns a1 b2 c3
Likewise, the cross operator X has a meta operator equivalent
<a b c> X~ <1 2 3> returns a1 a2 a3 b1 b2 b3 c1 c2 c3

另外一个作用于数组/列表的元操作符是 reduce:
`[+] @a` 对 @a 的所有元素进行求和并返回和, 这在功能上等价于 `@a[0] + @a[1] + ... + @a[*-1]`
任何中缀操作符都能用在 `+` 那个位置上, 显然这对无限惰性列表没有作用。
但是有另外一种形式的 reduce 元操作符, 它能返回一个惰性列表。
`[\*] 1..*` 返回一个惰性列表  `1, 1*2, 1*2*3, 1*2*3*4 ...`
那就是说, it returns each internal step of the evaluation of [*]

Another meta operator which works on arrays/lists is reduce:
[+] @a sums all of the elements of @a and returns the sum
It's functionally equivalent to @a[0] + @a[1] + ... + @a[*-1]
Any infix operator can be used in place of + there
Obviously this will not work for infinite lazy lists
But there is another form of the reduce meta operator which returns a lazy list
[\*] 1...* returns the lazy list 1, 1*2, 1*2*3, 1*2*3*4 ...
That is to say, it returns each internal step of the evaluation of [*]

其它元操作符：
赋值： 传统 op= (eg +=)
取反： 中缀关系操作符能用作 !op, 即 ` $a !eq $b ` 等价于 `!($a eq $b)`。
反转： Rop 反转了 op 的参数, 所以 $a R- $b 和 $b - $a 相同。

Other meta operators:
Assignment: The traditional op= (eg +=)
Negation: Infix relational operators can be used as !op
That is, $a !eq $b is equivalent to !($a eq $b)
Reversing: Rop reverses the arguments to op
So $a R- $b is the same as $b - $a

"自然地", 元操作符能嵌套起来
"Naturally," meta operators can be nested
<a b c> X~ <1 2 3>   is a1 a2 a3 b1 b2 b3 c1 c2 c3
<a b c> RX~ <1 2 3>  is 1a 1b 1c 2a 2b 2c 3a 3b 3c
<a b c> RXR~ <1 2 3> is a1 b1 c1 a2 b2 c2 a3 b3 c3
That's one of the few useful applications of this I know of :)

New operators can be defined just like any other sub
multi sub infix:<+>(MyInt $a, MyInt $b) overloads addition for MyInt
sub postfix:<!>(Int $a) { [*] 1..$a; } creates a factorial operator:
5! yields 120, just as you would expect
sub prefix:<$$$>($a) { "$a billion dollars" }
$$$10 yields the string 10 billion dollars
In theory, these new operators can be used with meta operators too

You may have the impression that Perl 6 is operator crazy
If so, you are right
Part of the Perl 6 philosophy is to have a very rich set of operators
It's up to programmers not to abuse this
eg As an infix, + (by itself) should always refer to mathematical addition
But that's a convention, not a hard technical rule of the language

prefix:<+> is just sugar for calling an object's Numeric conversion method
Operators which start with + are numeric operators
For instance, +& converts both its arguments to Int and does bitwise AND on them
prefix:<?> is sugar for .Bool, and ?| converts its arguments to Bool and ORs them
prefix:<~> is sugar for .Str, conversion to a string



```



## Syntax flexibility
---
## Concurrency
---
## Phasers
---
## Misc
---
## Meta-Object Programming
---
## Lexing and Parsing
---
## Roles
---

所以到底什么是 role 呢？ 零个或多个方法和属性的集合。

role 不像class，它不能被实例化（如果你尝试了，会生成一个 class）。Perl 6 中 Classes 是可变的，而 roles 是不可变的。

声明 Roles 就像声明 Class 一样：


```perl6
role DebugLog {
    has @.log_lines;
}

has $.log_size is rw = 100;

method log_message($message) {
@!log_lines.shift if
    @!log_lines.elems >= $!log_size;
    @!log_lines.push($message);
}
```

使用 `does` trait 将 role 组合到 Class 中：


```perl6
class WebCrawler does DebugLog {
    ...
}
```

这会把方法和属性添加到 class WebCrawler 里面去。结果就像它们起初被写到 class 中一样。


```perl6
role Bull-Like {
    has Bool $.castrated = False;
    method steer {
        # Turn your bull into a steer
        $!castrated = True;
        return self;
    }
}
role Steerable {
    has Real $.direction;
    method steer (Real $d = 0) {
        $!direction += $d;
    }
}
class Taurus does Bull-Like does Steerable {
    method steer ($!direction?) {
        self.Steerable::steer($!direction?);
    }
 }

```



```perl6
role Hammering {
    method hammer($stuff) {
        say "You hammer on $stuff. BAM BAM BAM!";
    }
}

class Hammer does Hammering {}
class Gavel  does Hammering {}
class Mallet does Hammering {}

my $hammer = Hammer.new;    # create a new hammer object
say $hammer ~~ Hammer;      # "Bool::True" -- yes, this we know
say $hammer ~~ Hammering;   # "Bool::True" -- ooh!

my $unkown_object = Gavel.new;
if $unkown_object ~~ Hammering {
    $unkown_object.hammer("that nail over there");     # will always work
}

```



```perl6
role Serializable {
    method serialize() {
        self.perl; # very primitive serialization
    }
    method deserialize($buf) {
        EVAL $buf; # reverse operation to .perl
    }
}

class Point does Serializable {
    has $.x;
    has $.y;
}
my $p = Point.new(:x(1), :y(2));
my $serialized = $p.serialize;      # method provided by the role
say $serialized;
my $clone-of-p = Point.deserialize($serialized);
say $clone-of-p.x;      # 1

```



```perl6
﻿role Observable {
    has @!observers;

    method subscribe($observer) {
        push @!observers, $observer;
        $observer
    }

    method unsubscribe($observer) {
        @!observers .= grep({ $^o !=== $observer });
    }

    method publish($obj) {
        @!observers>>.handle($obj)
    }
}

class ReadLineSource does Observable {
    has $.fh;
    method enterloop() {
        loop {
            self.publish($.fh.get());
        }
    }
}

multi grep($matcher, Observable $ob) {
    my class GrepSubscriber does Observable {
        has $.matcher;
        method handle($obj) {
            if $obj ~~ $.matcher {
                self.publish($obj);
            }
        }
    }
    $ob.subscribe(GrepSubscriber.new(:$matcher))
}

my $src = ReadLineSource.new(fh => $*IN);
$src
    ==> grep(/^\d+$/)
    ==> into my $nums;

$nums
    ==> grep(*.Int.is-prime)
    ==> call(-> $p { say "That's prime!" });

$nums
    ==> map(-> $n {
            state $total += $n;
            $total >= 100 ?? 'More than 100' !! ()
        })
    ==> first()
    ==> call(-> $msg { say $msg });
```



```perl6
role Point {
    has $.x;
    has $.y;
    method abs { sqrt($.x * $.x + $.y * $.y) }
}
say Point.new(x => 6, y => 8).abs;

```



```perl6
role Sleeping {
    method lie {
        say "Reclining horizontally...";
    }
}

role Lying {
    method lie {
        say "Telling an untruth...";
    }
}

# 如果解决方法同名的冲突呢？
# 在 class 中定义一个同名的方法即可
class SleepingLiar does Sleeping does Lying {
    method lie {
        say "Lying in my sleep....";
    }
}

my $sleep = SleepingLiar.new;
$sleep.lie; # Lying in my sleep....

# 调用其中之一的 roles 的 lie 方法
class SleepingSheep does Sleeping does Lying {
    method lie {
        self.Sleeping::lie;
    }
}

my $sleepSheep = SleepingSheep.new;
$sleepSheep.lie; # Reclining horizontally...

```



```perl6
role Paintable {
    has $.colour is rw;
    method paint { ... }
}
class Shape {
    method area { ... }
}

class Rectangle is Shape does Paintable {
    has $.width;
    has $.height;
    method area {
        $!width * $!height;
    }
    method paint() {
        for 1..$.height {
            say 'x' x $.width;
        }
    }
}

Rectangle.new(width => 8, height => 3).paint;
# 这打印下面 3 行
xxxxxxxx
xxxxxxxx
xxxxxxxx

```



```perl6
role Serializable {
    method serialize() {
        self.perl; # very primitive serialization
    }
    method deserialization-code($buf) {
        EVAL $buf; # reverse operation to .perl
    }
}

class Point does Serializable {
    has $.x;
    has $.y;
}
my $p = Point.new(:x(1), :y(2));
my $serialized = $p.serialize;      # method provided by the role
my $clone-of-p = Point.deserialization-code($serialized);
say $clone-of-p.x;      # 1

```



```perl6
role Sleeping {
    method lie {
        say "Reclining horizontally...";
    }
}

role Lying {
    method lie {
        say "Telling an untruth...";
    }
}

class SleepingLiar does Sleeping does Lying {}    # CONFLICT!

# Method 'lie' must be resolved by class SleepingLiar because it exists in multiple roles (Lying, Sleeping)

```



## rosettacode
---


```perl6
# 问题： 你有 100 扇关着的门排成一排， 然后你穿过这些门 100 次。第一次穿过的时候，穿越每一扇门， 如果门是开着的就关闭它， 如果门是关着的就打开它。第二次穿越的时候，每两扇门穿越一下，（第 2、4、6扇门）；第三次穿越的时候， 每 3 扇门（第 3、6、9），等等， 直到你穿过第 100 扇门为止。
# 问： 最后一次穿过门之后， 每扇门的状态是开是关？
# 提示： 剩下开着的门就是那些能开方的整数the only doors that remain open are whose numbers are perfect squares of integers
#`(
my @doors = False xx 101;

($_ = !$_ for @doors[0, * + $_ ...^ * > 100]) for 1..100;

say "Door $_ is ", <closed open>[ @doors[$_] ] for 1..100;
)

say "Door $_ is open" for map {$^n ** 2}, 1..10;

say "Door $_ is open" for 1..10 X** 2;

say "Door $_ is ", <closed open>[.sqrt == .sqrt.floor] for 1..100;

# « U+00AB  , » U+00BB  Vim => Ctrl+V => u => 00AB


```



```perl6
# Works with: Rakudo Star version 2010.08
for 10 ... 0 {
    .say;
}

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



```perl6
# A list comprehension is a special syntax in some programming languages to describe lists. It is similar to the way mathematicians describe sets, with a set comprehension, hence the name.

# Some attributes of a list comprehension are that:
# 1. They should be distinct from (nested) for loops within the syntax of the language.
# 2. They should return either a list or an iterator (something that returns successive members of a collection, in order).
# 3. The syntax has parts corresponding to that of set-builder notation.

# Write a list comprehension that builds the list of all Pythagorean triples with elements between 1 and n. If the language has multiple ways for expressing such a construct (for example, direct list comprehensions and generators), write one example for each.

use v6;

my $n = 20;
my @list := gather for 1..$n -> $x {
         for $x..$n -> $y {
           for $y..$n -> $z {
             take $x,$y,$z if $x*$x + $y*$y == $z*$z;
           }
         }
       }
.say for  @list;

# Note that gather/take is the primitive in Perl 6 corresponding to generators or coroutines in other languages. It is not, however, tied to function call syntax in Perl 6. We can get away with that because lists are lazy, and the demand for more of the list is implicit; it does not need to be driven by function calls.
```



```perl6
# Loop over multiple arrays (or lists or tuples or whatever they're called in your language) and print the ith element of each. Use your language's "for each" loop if it has one, otherwise iterate through the collection in order with some other loop.

# For this example, loop over the arrays (a,b,c), (A,B,C) and (1,2,3) to produce the output
# aA1
# bB2
# cC3

for <a b c> Z <A B C> Z 1, 2, 3 -> $x, $y, $z {
   say $x, $y, $z;
}

# The Z operator stops emitting items as soon as the shortest input list is exhausted. However, short lists are easily extended by replicating all or part of the list, or by appending any kind of lazy list generator to supply default values as necessary.

# Note that we can also factor out the concatenation by making the Z metaoperator apply the ~ concatenation operator across each triple:
for <a b c> Z~ <A B C> Z~ 1, 2, 3 -> $line {
   say $line;
}

# We could also use the zip-to-string with the reduction metaoperator:
.say for [Z~] [<a b c>], [<A B C>], [1,2,3]

```



```perl6
# 给定一组排好序的数， 如果数字是连续的，就用 - 符号连接 头和尾
# 例如 -6, -3, -2, -1, 0, 1, 3, 4, 5, 7, 8, 9, 10, 11, 14, 15, 17, 18, 19, 20
# 处理后变为： -6,-3-1,3-5,7-11,14,15,17-20
# Task:编写一个函数， 将范围连接起来
#  0,  1,  2,  4,  6,  7,  8, 11, 12, 14,
#  15, 16, 17, 18, 19, 20, 21, 22, 23, 24,
#  25, 27, 28, 29, 30, 31, 32, 33, 35, 36,
#  37, 38, 39

sub range-extraction (*@ints) {
    my $prev = NaN;
    my @ranges;

    for @ints -> $int {
        if $int == $prev + 1 {
            @ranges[*-1].push: $int;
        }
        else {
            @ranges.push: [$int];
        }
        $prev = $int;
    }
    join ',', @ranges.map: -> @r { @r > 2 ?? "@r[0]-@r[*-1]" !! @r }
}

say range-extraction
    -6, -3, -2, -1, 0, 1, 3, 4, 5, 7, 8, 9, 10, 11, 14, 15, 17, 18, 19, 20;

say range-extraction
    0,  1,  2,  4,  6,  7,  8, 11, 12, 14,
    15, 16, 17, 18, 19, 20, 21, 22, 23, 24,
    25, 27, 28, 29, 30, 31, 32, 33, 35, 36,
    37, 38, 39;

```



```perl6
# The sleep function argument is in units of seconds, but these may be fractional (to the limits of your system's clock).
my $sec = prompt("Sleep for how many microfortnights? ") * 1.2096;
say "Sleeping...";
sleep $sec;
say "Awake!";

```



```perl6
# Sort an array of composite structures by a key. For example, if you define a composite structure that presents a name-value pair (in pseudocode):
# Define structure pair such that:
   # name as a string
   # value as a string


# and an array of such pairs:
# x: array of pairs


# then define a sort routine that sorts the array x by the key name.

# This task can always be accomplished with Sorting Using a Custom Comparator. If your language is not listed here, please see the other article.

# Works with: Rakudo Star version 2010.08

my class Employee {
   has Str $.name;
   has Rat $.wage;
}

my $boss     = Employee.new( name => "Frank Myers"     , wage => 6755.85 );
my $driver   = Employee.new( name => "Aaron Fast"      , wage => 2530.40 );
my $worker   = Employee.new( name => "John Dude"       , wage => 2200.00 );
my $salesman = Employee.new( name => "Frank Mileeater" , wage => 4590.12 );

my @team = $boss, $driver, $worker, $salesman;

my @orderedByName = @team.sort( *.name )>>.name;
my @orderedByWage = @team.sort( *.wage )>>.name;

say "Team ordered by name (ascending order):";
say @orderedByName.join('  ');
say "Team ordered by wage (ascending order):";
say @orderedByWage.join('  ');

# this produces the following output:
# Team ordered by name (ascending order):
# Aaron Fast   Frank Mileeater   Frank Myers   John Dude
# Team ordered by wage (ascending order):
# John Dude   Aaron Fast   Frank Mileeater   Frank Myers


# Note that when the sort receives a unary function, it automatically generates an appropriate comparison function based on the type of the data.

```
