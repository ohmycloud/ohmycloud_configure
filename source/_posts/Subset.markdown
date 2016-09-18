---
title: Subset
date: 2015-03-19
tags: 
  - Subset
  - where
  - given when
  - proceed
categories: Perl 6
comments: true
---

简书对插入代码支持的不给力啊。 Perl 6 语法肯定识别不出， 但是空格对齐也不给力啊。 

---

Subset 可用于声明某一类型的子集。

输入一个密码， 要求密码必须满足如下条件：

1、 至少 8 位
2、 必须包含大写字母、小写字母 和 数字

```perl6
use v6; 

 subset Length8      of Str where *.chars < 8;
 subset UpCase       of Str where none('A'..'Z')  ∈ *.comb.Set;
 subset LowerCase    of Str where none('a'..'z')  ∈ *.comb.Set;
 subset IntNumber    of Str where none('0'..'9')  ∈ *.comb.Set;

 my $guess = prompt('Enter your password:');
    
 given $guess {
     when Length8     { say '密码长度必须为 8 位 以上'; proceed }
     when  UpCase     { say '密码必须包括大写字母';     proceed } 
     when LowerCase   { say '密码必须包含小写字母';     proceed }
     when IntNumber   { say '密码必须包含数字';                }   
 }
```

该程序具有可扩展性， 要增加一种密码验证， 只有添加一个 subset 就好了，然后在 given/When 里面增加一个处理。

proceed ：vi. 前进; 继续下去。
`proceed` 相当于 `continue`， 不像 C 里面的 falling through， Perl 6 里面的 proceed 在继续执行下一个  `when`  语句时会计算 when 后面的条件。 所以， 只要有 proceed ， 则 proceed 后面的那个条件就会被执行。
