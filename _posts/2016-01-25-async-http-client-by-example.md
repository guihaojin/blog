---
layout: post
title: "Async HTTP Client实例解释"
description: ""
category:
tags: ["async", "HTTP", "eventmachine"]
---
{% include JB/setup %}

一直听人说什么同步(sync)，异步(async)，阻塞(blocking)，非阻塞(non-blocking)，感觉自己似懂非懂的样子，于是决定自己动手实验一下。

我的基本思路很简单：就是先起一个简单的Server，然后写个用sync HTTP client的例子，再写一个用async HTTP client的例子，它们分别call server 若干次，最后比较结果。

- 一个简单的Web Server

[Sinatra ](http://www.sinatrarb.com/)是一个Ruby的micro-framework。为了简单起见，我们就用它来实现我们的Server。代码如下：

{% highlight Ruby %}
require "sinatra"

get "/slow" do
  sleep 2
  "slow"
end
{% endhighlight %}

这里只有一个`/slow` route，它先sleep 2秒，然后返回”slow”给client。这里的sleep就是模拟一个长时间执行的任务。假设代码保存在server.rb里，在Terminal里运行`ruby server.rb`，我们的测试server就准备好了。可以用浏览器打开`localhost:4567/slow`(4567是Sinatra默认端口)，等上大约2秒，就可以看到server返回的“slow”。

- 一个用sync HTTP client的例子

先上代码：

{% highlight Ruby %}
require "rest-client"

# a slow response route
url = "http://localhost:4567/slow"

n = 10

start_time = Time.now
p "Start time: #{start_time}"

n.times do |i|
  response = RestClient.get url
  p "Time now: #{Time.now}, response #{i}: #{response.to_str}"
end

p "Time elapsed: #{Time.now - start_time} s.”
{% endhighlight %}

这里做的事情很简单，就是连续call我们之前准备的server的10次，同时打印出时间戳，并计算总共用时多少。将代码保存在sync_client.rb文件中，然后运行`ruby sync_client.rb`就有类似于以下的结果：

{% highlight Ruby %}
"Start time: 2016-01-24 23:53:41 -0800"
"Time now: 2016-01-24 23:53:43 -0800, response 0: slow"
"Time now: 2016-01-24 23:53:45 -0800, response 1: slow"
"Time now: 2016-01-24 23:53:47 -0800, response 2: slow"
"Time now: 2016-01-24 23:53:49 -0800, response 3: slow"
"Time now: 2016-01-24 23:53:51 -0800, response 4: slow"
"Time now: 2016-01-24 23:53:53 -0800, response 5: slow"
"Time now: 2016-01-24 23:53:55 -0800, response 6: slow"
"Time now: 2016-01-24 23:53:57 -0800, response 7: slow"
"Time now: 2016-01-24 23:53:59 -0800, response 8: slow"
"Time now: 2016-01-24 23:54:01 -0800, response 9: slow"
"Time elapsed: 20.087692 s.”
{% endhighlight %}

可以看到，每次call之间的间隔2秒，10次call总共花了大约20秒。

- 一个用async HTTP client的例子

还是先上代码：

{% highlight Ruby %}
require "eventmachine"
require "em-http-request"

# a slow response route
url = "http://localhost:4567/slow"

n = 10

start_time = Time.now
p "Start time: #{start_time}"

EventMachine.run do
  n.times do |i|
    EventMachine::HttpRequest.new(url).get.callback do |http|
      p "Time now: #{Time.now}, response #{i}: #{http.response}"
    end
  end

  p "After HttpRequest."
end
{% endhighlight %}

这里用到了Ruby的一个叫[eventmachine](https://github.com/eventmachine/eventmachine)的library和[em-http-request](https://github.com/igrigorik/em-http-request)这样一个Asynchronous HTTP client。稍微解释一下代码，这里最难理解的可能就是EventMachine部分，`EventMachine.run do`后面一直到`end`之前就是我们主要的逻辑(Ruby里的do…end就相当于{…})。这里面同样是做了10次HTTP call到http://localhost:4567/slow，在这之后还输出一句`After HttpRequest.`。运行这段代码我们可以看到类似于如下的结果：

{% highlight Ruby %}
"Start time: 2016-01-25 20:14:33 -0800"
"After HttpRequest."
"Time now: 2016-01-25 20:14:35 -0800, response 0: slow"
"Time now: 2016-01-25 20:14:35 -0800, response 2: slow"
"Time now: 2016-01-25 20:14:35 -0800, response 3: slow"
"Time now: 2016-01-25 20:14:35 -0800, response 7: slow"
"Time now: 2016-01-25 20:14:35 -0800, response 4: slow"
"Time now: 2016-01-25 20:14:35 -0800, response 8: slow"
"Time now: 2016-01-25 20:14:35 -0800, response 6: slow"
"Time now: 2016-01-25 20:14:35 -0800, response 5: slow"
"Time now: 2016-01-25 20:14:35 -0800, response 1: slow"
"Time now: 2016-01-25 20:14:35 -0800, response 9: slow"
{% endhighlight %}

这里面有三点需要注意：

1. 我们首先看到`After HttpRequest`要比HTTP response先打印出来，但是在代码里我们是在HTTP请求之后才打印它的；
2. 所有的HTTP response的时间戳都是`2016-01-25 20:14:35`，说明他们差不多是在同一时间返回的；
3. 从HTTP response的id可以看到，responses的顺序不是从小到大的(再跑一遍还可能看到不一样的顺序)；

这是因为，EventMachine是event-driven的库，里面的em-http-request也是异步非阻塞的HTTP client，所以在等待HTTP response的时候就先执行了’p "After HttpRequest.”`。然后在几乎同一时间所有的HTTP request都有了response，打印出了结果。这些responses也是先来先打印，没有固定的顺序的。

* 结果

从这个例子我们可以很直观的看到async HTTP client跟sync HTTP client有什么不同，会造成什么样的结果。总的来说，异步非阻塞的模式节省了时间，同时提高了资源利用率。有人说也可以用multi-thread来做，这个我同意，但是multi-thread最终在每一个thread里，还是会block，资源利用率还是不够高。

最后，文章代码在[这里](https://github.com/guihaojin/async-http-client-example)可以找到。
