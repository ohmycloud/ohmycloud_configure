---
title: HOW 方法
date: 2016-07-22
tags: HOW
categories: Perl 6
comments: true
---



[Why can't I call the methods method on a Perl 6's ClassHOW object?](http://stackoverflow.com/questions/35819026/why-cant-i-call-the-methods-method-on-a-perl-6s-classhow-object)

我可以在某个对象身上调用 `^methods` 方法, 以列出我能调用的方法的名字列表:

```perl6
my $object = 'Camelia'
my @object_methods = $object.^methods;
@object_methods.map( {.gist} ).sort.join("\n").say;
```

`^methods` 返回一个列表,  通过对列表中的每个元素调用 `.gist` 方法, 得到一个人类友好的形式。 

但是 `^methods` 中的 `^` 暗含了 `.HOW` 方法:

我想下面这段代码本该有效的, 但是没有:

```perl6
my $object = 'Camelia';
my @object_objects = $object.HOW.methods;
```

但是我得到了一个错误:

```
Too few positionals passed; expected 2 arguments but got 1
  in any methods at gen/moar/m-Metamodel.nqp line 490
  in block <unit> at...
```

通过 `.` 调用的常规方法把点号左边的调用者作为第一个参数隐式地传递给该方法。通过 `.^` 调用的元方法传递了两个参数:作为调用者的元对象(meta-object)和作为第一个位置参数的实例:

举个例子:

```perl6
$obj.^can('sqrt')
```

就是下面这段代码的语法糖:

```perl6
$obj.HOW.can($obj, 'sqrt')
```

在你的例子中, 你应该写作:

```perl6
my @object_methods = $object.HOW.methods($object);
```



## :D 和 :D: 之间的区别
---

在一段定义 `shift` 子例程的Perl 6 代码里:

```perl6
multi sub    shift(Array:D)
multi method shift(Array:D:)
```

我知道 `:D` 意味着 `Array`是 `defined` (有定义的)而非 `Any` 或 `nil`, 但是 `:D:` 是什么意思呢? 文档中真的很难找到。

`D`后面的冒号表示方法的调用者作为第一个参数被隐式地传入进来。如果你想在签名中使用显式的参数(例如给它添加一个诸如 :D 的类型笑脸或者给它一个更具描述性的名字), 你需要使用 `:` 而非 `,` 以把它和剩余的参数列表分割开。这甚至在空列表时也是必要的, 所以这能消除含有常规位置参数的签名中的歧义。

更多信息, 请查看[Invocant_parameters]( http://design.perl6.org/S06.html#Invocant_parameters)。


























