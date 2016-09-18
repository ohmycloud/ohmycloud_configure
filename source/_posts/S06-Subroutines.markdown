---
title: S06-Subroutines
date: 2016-06-27
tags:
  - S06
  - Submethods
  - Subroutines
  - 子例程
categories: Perl 6
comments: true
---



原文[Subroutines](http://design.perl6.org/S06.html)

## 标题
---

概要 6: 子例程

## 版本
---

创建于: 2003 年 5 月 21

上一次更新: 2015 年 10 月 16

版本: 169

该文档总结了概要 6, 其中涵盖了子例程和新的类型系统。

## 子例程和其它 code 对象
---

**Routine** 是所有以关键字声明的 code blocks 的父类型。所有的 routines(程序)生来就带有未定义的 `$_`、`$!`和`$/` 值, 除非该 routine 显式地声明了它们。一个编译单元, 例如模块文件或 **EVAL** 字符串, 也被看作是 routine, 要不然你就不能引用它们里面的 `$!` 或 `$/`。

使用 `->` 声明或使用裸花括号的非-routine 的 code Blocks, 生来只有 `$_`, 它被别名到它的 `OUTER::<$_>` 上, 除非被绑定为参数。 block 通常使用由最里面的闭合 routine 所定义的 `$!` 和 `$/`, 除非在 block 中显式地声明了 `$!` 或 `$/`。

形实转换程序是一段不会立即执行的代码, 例如因为它是条件操作符的一部分, 或者它是某个属性的默认初始化函数。它没有自己的作用域, 所以在形实转换程序中定义的任何新的变量, 都会漏到它们所在的作用域中。然而, 要知道任意和所有的 lazy(惰性)结构, 不管是基于 block 的还是基于形实转换程序的, 例如 gather 或者 start 或者 `==>` 应该声明它们自己的 `$/` 和 `$!` , 以至于那些变量的用户值不会发生异步碰撞(clobbered asynchronously)。

**Subroutines** (关键字: sub) 是带有参数列表的非继承程序。

**Methods** (关键字: method) 是有关联对象(即它们的调用者)的可继承程序并且属于特殊的类别或类。

**Submethods** (关键字: submethod) 是非继承方法, 或者是伪装成方法的子例程。它们拥有一个调用者并属于特殊的类别或类。

**Regexes** (关键字: regex)是执行模式匹配的方法(grammar 的)。它们关联的 block 拥有特殊的语法(查看 Synopsis 5)。(我们也使用术语 `regex` 用于传统形式的匿名模式。)

**Tokens** (关键字: token) 是执行低级别(low-level) 非回溯(默认的)模式匹配的 regexes。

**Rules** (关键字: rule) 是执行非回溯(默认的)模式匹配(而且 rules 中空白是开启的)的 regexes。

**Macros** (关键字: macro 或 slang) 是被安装的程序或方法, 它们作为编译程序的一部分被调用, 因此能临时取得随后编译的控制权以像编译器作弊那样作弊。

## Routine 修饰符
---

> [`S06-multi/value-based.t` lines `7–104`](https://github.com/perl6/roast/blob/master/S06-multi/value-based.t#L7-L104)
> [`S06-multi/type-based.t` lines `50–57`](https://github.com/perl6/roast/blob/master/S06-multi/type-based.t#L50-L57)
> [`S06-multi/type-based.t` lines `58–283`](https://github.com/perl6/roast/blob/master/S06-multi/type-based.t#L58-L283)
> [`S06-multi/syntax.t` lines `7–183`](https://github.com/perl6/roast/blob/master/S06-multi/syntax.t#L7-L183)

**Multis** (关键字: multi) 是可以拥有多个共享同一个名字的变体的 routines, 它们通过元数、类型或其它约束被选择。

**Prototypes** (关键字: proto) 指定了在 *proto* 声明的作用域中, 那个名字的所有 muitis 所共享的共性(例如参数名, 不变性和结合性)。理论上讲, *proto* 是包裹在 *multis* 分发周围的通用包装器(wrapper)。对于每个需要一个不同候选者列表的作用域, 每个 *proto* 被具体化到一个实际的分发器(dispatcher)。

**Only** (关键字: only) 不和其它 routines 共享它们的短名字的 routines。这对所有的 routines 是默认的修饰符, 除非作用域中已经有了一个同名的 `proto`。(对于 subs, 执政的那个 *proto* 必须已经在同一个文件中声明过, 所以 从设置中或其它模块中声明的 `proto` 没有这种效果, 除非被显式地导入。)

在具名 routine 中, 修饰符关键字可以出现在 routine 关键字前面:

```perl6
only     sub foo  {...}
proto    sub foo  {...}
dispatch sub foo  {...} # 内部的
multi    sub foo  {...}

only     method bar  {...}
proto    method bar  {...}
dispatch method bar  {...} # 内部的
multi    method bar  {...}
```

如果 routine 关键字被省略了, 那么就默认为 `sub`。

修饰符关键字不能应用到匿名 routines 上。

`proto` 是一个通用的分发器, 其中带有唯一候选者列表的任意给定作用域会被实例化到某个分发子例程中。因此 `proto`从来不会被直接调用, 这和 *role* 不能用作实例化对象很像。



当你调用任何可能拥有多个候选者的子例程(或者方法, 或者 rule)时, 基础分发器真正调用的是 **only** sub 或方法 — 但是如果有多个候选者, 将被找到的 "only" 就是一个分发器。这个实例化的分发器总是被首先调用(至少在理论上 — 这通常会被优化掉)。本质上, *dispatch* 总是像 `only` sub 那样, 但是 *dispatch* 自身可以代理给任何它所管理的候选者。

首先为所有的候选者审查参数是分发器(dispatch)的责任。任何不能成功约束为分发器签名的调用都会立即失败。(它的签名是属于该 proto 所实例化的候选者签名的拷贝。) 然而, 该分发器(dispatch)没有必要把原来的捕获(capture)发送给它的候选者。在分发器签名中约束为位置的具名参数对于所有随后的它所管理的 multis 的调用都会变成位置的。

然后, 该分发器(dispatch)从调用者或对象的角度考虑它所管理的候选者列表, 把它们按照某种顺序排序, 然后根据每个不同的调度器(dispatchers)定义的多重分派的规则来分派它们。在 multi subs 的情况下, 候选者列表在编译时是被知晓的。在 multi methods 的情况下, 在运行时生成(或重新生成)候选者列表可能是必要的, 根据关于继承树时所知道的。

这种默认的分发器行为在原来的 *proto* 中通过一个含有单个 `*` (即 "whatever")的 block 使用符号给表现了。因此, 标准的 `proto` 会仅仅有一个 `{*}` 主体。

```perl6
proto method bar {*}
```

(我们不使用 … 代替它是因为它会在运行时失败, 而该 proto 实例化的 dispatch blocks 不是 stubs, 但是想要被执行。)

 其它语句可以插入在 `{*}` 语句之前和之后以在 multi dispatch 之前或之后捕获控制:

```perl6
proto foo ($a, $b) { say "Called with $a, $b"; {*}; say "Returning"; }
```

(那个 `proto` 只对带有副作用并不返回值的 multis 很有用, 因为它返回了 say 的结果, 这可能不是你想要的。查看下面的来修复它 。)
