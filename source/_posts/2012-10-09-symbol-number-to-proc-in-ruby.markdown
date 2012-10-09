---
layout: post
title: "Symbol#to_proc in ruby"
date: 2012-10-09 16:45
comments: true
categories: 
---

在ruby中处理列表时,我们会经常这样使用`map`方法

``` ruby
names.map {|name| name.upcase}
```

看起来不错,不过,其实有更优雅的做法

``` ruby
names.map &:upcase
```

漂亮,但是有些费解?其实工作原理很简单:

1. 为`map`方法送入一个block object,其值为`:upcase`
2. 但是`:upcase`不是block object,而是symbol object,于是类型转换
3. 调用`Symbol#to_proc`进行类型转换,于是魔术发生了

那么,`Symbol#to_proc`是如何工作的呢? 在ruby1.9.3中,是用c实现的这个方法,以下是一种用ruby的解决方案

``` ruby
class Symbol
  def to_proc
    proc { |obj, *args| obj.send(self, *args) }
  end
end
```

代码充分利用了闭包的特性,也给我们一些很好的提示,编程时只要你能想到更有表达力的风格,ruby总能做到

---
