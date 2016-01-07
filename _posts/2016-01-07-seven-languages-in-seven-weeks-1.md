---
layout: post
title: "七周七语言笔记1: Ruby"
description: ""
category:
tags: ["programming languages", "seven languages in seven weeks", "Ruby"]
---
{% include JB/setup %}

- 关键词：纯OOP, 动态类型，脚本语言，高效(对程序员友好)，灵活&强大

{% highlight bash %}
>> 4.class
=> Fixnum

>> 4.methods
=> ["inspect", "%", "<<", "singleton_method_added", "numerator", ...
 "*", "+", "to_i", "methods", ...
 ]
 {% endhighlight %}

- 程序块和yield关键词(可参考我的另一篇[文章](http://guihaojin.github.io/2014/04/28/why-do-i-love-ruby/))

- OO: 所有的类，包括Class，Module都是继承自Object/BasicObject类

![Inheritance](http://skilldrick.co.uk/wp-content/uploads/2011/08/Ruby.png)

- Mixin: 一种支持多继承的机制,用module实现(Ruby只支持单继承)

{% highlight bash %}
module ToFile
  def filename
    "object_#{self.object_id}.txt"
  end
  def to_f
    File.open(filename, 'w') {|f| f.write(to_s)}
  end
end

class Person
  include ToFile
  attr_accessor :name

  def initialize(name)
    @name = name
  end
  def to_s
    name
  end
end

Person.new('matz').to_f
{% endhighlight %}

这里module ToFile就是Mixin，Person类中`include ToFile`就加入了ToFile中的功能。同理，别的类也可以添加ToFile的功能。[Enumarable](http://ruby-doc.org/core-2.2.3/Enumerable.html)是Ruby中最典型的Mixin。

- metaprogramming(还是可以参考我之前的[文章](http://guihaojin.github.io/2014/04/28/why-do-i-love-ruby/))

这里有一个method missing的例子可以看看：

{% highlight bash %}
class Roman
  def self.method_missing name, *args
    roman = name.to_s
    roman.gsub!("IV", "IIII")
    roman.gsub!("IX", "VIIII")
    roman.gsub!("XL", "XXXX")
    roman.gsub!("XC", "LXXXX")

    (roman.count("I") +
     roman.count("V") * 5 +
     roman.count("X") * 10 +
     roman.count("L") * 50 +
     roman.count("C") * 100)
  end
end

puts Roman.X
puts Roman.XC
puts Roman.XII
puts Roman.X
{% endhighlight %}

method_missing这个方法是Ruby找不到函数定义的时候会调用的，这里我们override了这个方法，就相当于定义了一堆Roman的方法。非常简洁优雅。

- 核心优势

Ruby是一门理想的脚本语言。当规模不是很大的时候它也是一门优秀的web开发语言。因为它是纯粹的面向对象语言，这就保证了它在处理对象时候的一致性，都是Object#method(对象调用方法)的形式。同时，duck typing也能让语言最大限度的实现多态性。而Ruby的module和开放类让程序员可以很容易地对类/对象添加原本不存在的行为。这些特性集合在一起，使得Ruby成为一门极为高效的语言。

Note: 因为我自己已经有了数年的Ruby使用经验，所以有些地方就一笔带过了。
