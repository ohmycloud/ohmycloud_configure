---
title: MAIN 中的冒号
date: 2015-07-05
tags:
  - MAIN
  - 冒号
categories: Perl 6
comments: true
---



假如我有多个文本文件, 我要写一段脚本来进行替换操作。在命令行中提供一些列选项供配置。文本格式如下:

```
Perl 6 很灵活
Perl 6 很强大
Perl 6 很复杂

I 123 Love Perl Six
尽管它456还很稚嫩789
对了, 它的 logo 是一个可爱的456花蝴蝶。
```

我想把每行中第一次出现的 3 位数字替换为 "在木星"。

@蘑菇 的脚本:

```perl6
use v6;
use MONKEY-SEE-NO-EVAL;

sub MAIN(Str :$regex, Str :$substr, Str :$ext = '.out', Int :$ignore-line = 0, *@files) {
    for @files -> $file {
        my $out = open $file ~ ".out", :w; # 写入文件

        for $file.IO.lines.kv -> $index, $line is copy {
            next if $index <= $ignore-line; # 忽略前 $ignore-line几行
            $line ~~ EVAL "s/" ~ $regex ~ "/" ~ $substr ~ "/"; # 根据正则表达式进行替换
            say $/;
            $out.say($line);
        }
        $out.close;
    }
}
```

查看本脚本的用法

> perl6 colon_in_signature.p6 --help

```
Usage:
colon_in_signature.p6 [-r|--regex=<Str>] [-s|--substr=<Str>] [-e|--ext=<Str>] [-i|--ignore-line=<Int>] [<files> ...]
```

可以看出命令行选项有**短名称**(如 -r)和**长名称**(如 --regex)。我们使用的时候可以使用短名称:

```
> perl6 colon_in_signature.p6  -r="\d ** 3" -s="在木星" -i=2 1.txt 2.txt 3.txt
```

也可以使用长名称:

```
> perl6 colon_in_signature.p6  --regex="\d ** 3" --substr="在木星" --ignore-line=2 1.txt 2.txt 3.txt
```

 也可以只使用长名称, 但是我们需要修改下 **MAIN** 函数:

```perl6
sub MAIN(Str :$regex, Str :$substr, Str :$ext = '.out', Int :$ignore-line = 0, *@files)
```

再次查看帮助:

>  perl6 colon_in_signature.p6  --help

```
Usage:
colon_in_signature.p6 [--regex=<Str>] [--substr=<Str>] [--ext=<Str>] [--ignore-line=<Int>] [<files> ...]
```

方括号表示该选项是可选的。但是现在只有长名称:

```perl6
> perl6 colon_in_signature.p6 --regex="\d ** 3" --substr="在木星" 1.txt 2.txt 3.txt
```

也能既有长名又有短名:

```perl6
sub MAIN(Str :r(:$regex), Str :$substr, Str :$ext = '.out', Int :$ignore-line = 0, *@files) 
```

这其中的原理是什么呢? **MAIN** 在处理命令行参数时, 是把选项当作散列来用的:

- 带短名称

```
> my $regex = '\d ** 3' #->  \d ** 3
> :r(:$regex)           #->  r => regex => \d ** 3
```

- 不带短名称

```perl6
> (:$regex)            #->   regex => \d ** 3
```

`:r(:$regex)` 拥有一长一短两个键, `(:$regex)` 只拥有一个长键。
