---
layout: post
title: Ruby命令行脚本神器:Trollop
description: ""
category:
tags: [Ruby]
---
{% include JB/setup %}

编程的最大作用之一就是把一些让电脑做的重复的事情用程序自动化。基本上这种事情都是用一些简单方便的脚本语言来实现。所以写脚本的能力可以说是合格的程序员的一项基本能力。我个人认为这种能力也许可以算是是对非专业程序员最有价值的技能了。Ruby作为优秀脚本语言的代表，可以说是用来写脚本最好的语言之一了。命令行脚本，顾名思义，就是在命令行运行的脚本。为什么要在命令行运行呢？因为命令行是程序员们最好的朋友嘛。

今天要介绍的库叫[Trollop](https://github.com/wjessop/trollop)。本质上说，其实它就是一个命令行参数的解析器(command line options parser)。其实Ruby自带的库里面就有像[OptionParser](http://ruby-doc.org/stdlib-2.1.2/libdoc/optparse/rdoc/OptionParser.html)这样具有类似功能的库，那我们为什么还要使用Trollop呢？我认为Trollop最大的好处就是简单好用。用时下流行的话来讲就是：Trollop的用户体验太TM的好了。

在Rubygems.org里面是这样介绍Trollop的：

>Trollop is a commandline option parser for Ruby that just gets out of your way. One line of code per option is all you need to write. For that, you get a nice automatically-generated help page, robust option parsing, command subcompletion, and sensible defaults for everything you don't specify.

这个基本上就概括了Trollop的特性。那么我们就用一些实际的例子来看看Trollop到底有多方便吧。要想使用Trollop,需要先运行`gem install trollop`来安装它。

然后现在假设一个场景。我们要写一个脚本，这个脚本需要能够在不同的环境下运行，假如是production/staging。所以我们在运行它的时候就希望可以带不同的环境选项(options)。如下一个程序就可以完成这个任务。

	require "trollop"

	opts = Trollop::options do
  		opt :environment, "the environment to run the script against.", :type => :string
	end

`require "trollop"`是为了包含这个库。下面最重要的一行是在代码块（block）中的:

	opt :environment, "the environment to run the script against.", :type => :string
	
这个代码块就会生成一个名叫opts的Hash，而`:environment`就是其中的key。在程序中要使用这个environment变量该怎么用呢？直接用`opts[:environment]`就可以了。以上这一行中间的String(废话)是对这个选项的描述，最后的`:type`是声明这个值的类型，也就是`opts[:environment]`的类型为String。所以下面这一行：

	puts "Running the script against environment: #{opts[:environment]}."
	
就打印出来了我们从命令行中传入的`environment`值。假如说我保存这个脚本为`test.rb`。那么我们只要就可以运行`ruby test.rb --environment production`。这样就会打印出`Running the script against environment: production.`如何知道都有哪些选项呢？运行一下`ruby test.rb --help`就可以看到如下内容：

	Options:
	 --environment, -e <s>:   the environment to run the script against.
             	--help, -h:   Show this message
             	
这些是Trollop为我们自动生成了，同时我们可以看到，对于我们的environment option, 不经可以用long option:`--environment`，还可以用short option:`-e`。对于help也同样可以用`--help`或者是`-h`。而且这个帮助也是Trollop自动生成的，完全不需要我们做额外的工作。