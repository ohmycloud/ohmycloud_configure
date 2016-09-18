---
title: When and Where
date: 2015-03-15 13:15
tags:
  - when
  - where
categories: Perl 6
comments: true
---

## When 可以用在主题化($_)的语句中

Perl 里面有个特殊的变量叫 `$_`, 即主题化变量, the variable in question. 

```perl6
> for ('Swift', 'PHP', 'Python', 'Perl') -> $item  { say $item when $item ~~ /^P/ }    
PHP                                                                                  
Python                                                                               
Perl  
```

```perl6
> for (12, 24, 56, 42) {.say when *>40 }
56
42
```

而 *where* 用于对类型进行约束.

```perl6
> for ('Swift', 'PHP', 'Python', 'Perl', 42) -> $item  where $item ~~ Str  {say $item}   
Swift                                                                                    
PHP                                                                                      
Python                                                                                   
Perl                                                                                     
Constraint type check failed for parameter '$item'       
```

未完待续.
