---
layout: post
title: "七周七语言笔记2: Io"
description: ""
category: "seven languages in seven weeks"
tags: ["programming languages", "Io"]
---
{% include JB/setup %}

- 关键词：Prototype language(like JavaScript), Simplicity, Flexibility, Concurrency

- Deal with only Objects, cloning them as needed, it's all Objects and message passing to objects.

{% highlight Io %}
Io> “Hi ho, Io” print
Hi ho, Io==> Hi ho, Io
{% endhighlight %}

- Create new objects by cloning existing ones.

{% highlight Io %}
Io> Vehicle := Object clone
=> Vehicle_0x1003b61f8:
  type             = "Vehicle"
{% endhighlight %}

- Objects, Prototypes and Inheritance

{% highlight Io %}
Io> Car := Vehicle
==> Car_0x100473938:
type = "Car"

Io> Car slotNames
==> list("type")

Io> Car type
==> Car
{% endhighlight %}

- Methods, `method(<arg name 0>, <arg name 1>, ..., <do message>)`

{% highlight Io %}
Io> method("So, you've come for an argument." println)
==> method(
    "So, you've come for an argument." println
)
Io> method() type
==> Block

Io> Car drive := method("Vroom" println) ==> method(
    "Vroom" println
)
{% endhighlight %}

- Lists and Maps

{% highlight Io %}
Io> toDos := list("find my car", "find Continuum Transfunctioner")
==> list("find my car", "find Continuum Transfunctioner")
Io> toDos size
==> 2
{% endhighlight %}

{% highlight Io %}
Io> list(1, 2, 3, 4)
==> list(1, 2, 3, 4)
Io> list(1, 2, 3, 4) average
==> 2.5

Io> list(1, 2, 3, 4) sum
==> 10
Io> list(1, 2, 3) at(1)
==> 2
Io> list(1, 2, 3) append(4)
==> list(1, 2, 3, 4)
Io> list(1, 2, 3) pop
==> 3
Io> list(1, 2, 3) prepend(0)
==> list(0, 1, 2, 3)
Io> list() isEmpty
==> true
{% endhighlight %}

{% highlight Io %}
Io> elvis := Map clone
==> Map_0x115f580:
Io> elvis atPut("home", "Graceland")
==> Map_0x115f580:
Io> elvis at("home")
==> Graceland
Io> elvis atPut("style", "rock and roll")
==> Map_0x115f580:
Io> elvis asObject
==> Object_0x11c1d90:
  home             = "Graceland"
  style            = "rock and roll"
Io> elvis asList
==> list(list("style", "rock and roll"), list("home", "Graceland"))
Io> elvis keys
==> list("style", "home")
{% endhighlight %}

- Singletons

true, false and nil are singletons

{% highlight Io %}
Io> true clone
==> true
Io> false clone
==> false

Io> nil clone
==> nil
{% endhighlight %}

To create your own singleton:

{% highlight Io %}
Io> Highlander := Object clone
==> Highlander_0x378920:
  type             = "Highlander"
Io> Highlander clone := Highlander
==> Highlander_0x378920:
  clone            = Highlander_0x378920
  type             = "Highlander"
{% endhighlight %}

- Conditionals and Loops

loop, while and for

{% highlight Io %}
Io> loop("getting dizzy..." println)

Io> while(i <= 11, i println; i = i + 1)

Io> for(i, 1, 11, i println)
{% endhighlight %}

Control structure: `if(condition, true code, false code)` and `if(condition) then(true code) else(false code)`.

{% highlight Io %}
Io> if(true, "It is true.", "It is false.")
==> It is true.

Io> if(false) then("It is true") else("It is false")
==> nil
{% endhighlight %}

- Operators

越接近0的operator优先级越高，当然也可以用`()`来改变优先级。


{% highlight Io %}
Io> OperatorTable
==> OperatorTable_0x7fb7d4137d80:
Operators
  0   ? @ @@
  1   **
  2   % * /
  3   + -
  4   << >>
  5   < <= > >=
  6   != ==
  7   &
  8   ^
  9   |
  10  && and
  11  or ||
  12  ..
  13  %= &= *= += -= /= <<= >>= ^= |=
  14  return

Assign Operators
  ::= newSlot
  :=  setSlot
  =   updateSlot

To add a new operator: OperatorTable addOperator("+", 4) and implement the + message.
To add a new assign operator: OperatorTable addAssignOperator("=", "updateSlot") and implement the updateSlot message.
{% endhighlight %}

定义一个自己的`xor`操作符。

{% highlight Io %}
Io> OperatorTable addOperator("xor", 11)
==> OperatorTable_0x7fb7d4137d80:
Operators
  ...
  10  && and
  11  or xor ||
  12  ..
  ...
{% endhighlight %}

然后是在`true`和`false`上实现`xor`方法。

{% highlight Io %}
Io> true xor := method(bool, if(bool, false, true))

Io> false xor := method(bool, if(bool, true, false))
{% endhighlight %}

这样我们就有了一个自己定义的`xor`操作符了。由此可见，Io也和Ruby一样，非常灵活，可以自定义很多东西，而且内部的很多东西也可以被hack，非常好玩。

- Messages

Io中一切都是Message, 除了基本的调用以外，message的反射机制(reflection)也是Io非常重要的能力。通过反射，你可以查找到关于message的任何信息。

一个message包含三个部分：分别是sender, target和arguments。

{% highlight Io %}
Io> postOffice := Object clone
Io> postOffice packageSender := method(call sender)

Io> mailer := Object clone
Io> mailer deliver := method(postOffice packageSender)

Io> mailer deliver
==>  Object_0x7fb7d278cc90:
  deliver          = method(...)
{% endhighlight %}

这里mailer是message sender。下面的例子是调用`call target`。

{% highlight Io %}
Io> postOffice messageTarget := method(call target)
Io> postOffice messageTarget
==>  Object_0x7fb7d272c570:
  messageTarget    = method(...)
  packageSender    = method(...)
{% endhighlight %}

下面是获取message name和arguments:

{% highlight Io %}
Io> postOffice messageArgs := method(call message arguments)

Io> postOffice messageName := method(call message name)

Io> postOffice messageArgs("one", 2, :three)
==> list("one", 2, : three)

Io> postOffice messageName
==> messageName
{% endhighlight %}

很多语言是以用值来传递参数，比如说Java先计算每个参数的值，把值放在栈中。Io并不是这样的，它只传递message和上下文，然后接收者(receiver)来evaluate message。你甚至可以用message来实现控制结构。假如你想实现unless方法：

{% highlight Io %}
Io> unless := method(
... (call sender doMessage(call message argAt(0))) ifFalse(
... call sender doMessage(call message argAt(1))) ifTrue(
... call sender doMessage(call message argAt(2)))
... )

Io> unless(1 == 2, write("One is not two\n"), write("one is two\n"))
One is not two
==> false
{% endhighlight %}

doMessage有点类似Ruby的eval，但level更低。Ruby的eval是把string当做代码执行，而doMessage是执行任意message。Io的解释器是delayed binding and execution,不像很多其他语言是先计算所有的参数，并返回值放到stack上。

- Reflection

没太看懂。。。[例子](http://media.pragprog.com/titles/btlang/code/io/animals.io)

- DSL

用以下形式表示联系人电话：

{% highlight Io %}
{
    "Bob Smith": "5195551212",
    "Mary Walsh": "4162223434"
}
{% endhighlight %}

这个问题有很多方法可以解决。一种常见的方式是parse,但是这里我们看看另一种方法，就是用DSL将它解析为Io的hash。

{% highlight Io %}
# START:range
OperatorTable addAssignOperator(":", "atPutNumber")
# END:range
# START:curlyBrackets
curlyBrackets := method(
  r := Map clone
  call message arguments foreach(arg,
       r doMessage(arg)
       )
  r
)
# END:curlyBrackets
# START:atPutNumber
Map atPutNumber := method(
  self atPut(
       call evalArgAt(0) asMutable removePrefix("\"") removeSuffix("\""),
       call evalArgAt(1))
)
# END:atPutNumber
# START:use
s := File with("phonebook.txt") openForReading contents
phoneNumbers := doString(s)
phoneNumbers keys   println
phoneNumbers values println
# END:use
{% endhighlight %}

- Concurrency

Io拥有非常优异的并发库。主要包括：coroutines, actors, and futures.

Io中并发的基础是协程(coroutine),协程是一种让进程可以自愿悬挂或者继续的方式。

{% highlight Io %}
vizzini := Object clone
vizzini talk := method(
            "Fezzik, are there rocks ahead?" println
            yield
            "No more rhymes now, I mean it." println
             yield)

fezzik := Object clone

fezzik rhyme := method(
            yield
            "If there are, we'll all be dead." println
            yield
            "Anybody want a peanut?" println)

vizzini @@talk; fezzik @@rhyme

Coroutine currentCoroutine pause
{% endhighlight %}

fezzik和vizzni是两个独立的包含coroutine的Object实例。这里异步触发talk和rhyme方法，让他们同时运行。

Coroutine是更高级的并发，比如Actors，的基础。Actors是一种通用的并发单元，可以发送message，处理message，包括创建其他的actors。在Io中，每个actor的messages都放在一个队列中，然后处理。

- Actors

> Here’s the beauty of Io. Sending an asynchronous message to any object makes it an actor. End of story.

{% highlight Io %}
Io> slower := Object clone
==>  Object_0x7fd7f34aeb60:

Io> faster := Object clone
==>  Object_0x7fd7f34b7de0:

Io> slower start := method(wait(2); writeln("slowly"))
==> method(
    wait(2); writeln("slowly")
)
Io> faster start := method(wait(1); writeln("quickly"))
==> method(
    wait(1); writeln("quickly")
)
Io> slower start; faster start
slowly
quickly
==> nil

Io> slower @@start; faster @@start; wait(3)
quickly
slowly
==> nil
{% endhighlight %}

没有@@的时候两个start按顺序执行，第一个结束后执行第二个。用了@@之后每个对象都立即在自己的线程中执行，并且返回nil，并且两个对象都是actors了。

- Future

> A future is a result object that is immediately returned from an asyn- chronous message call.

{% highlight Io %}
futureResult := URL with("http://google.com/") @fetch
writeln("Do something immediately while fetch goes on in background...")
writeln("This will block until the result is available.")
writeln("fetched ", futureResult size, " bytes")
{% endhighlight %}

第一行代码会立即返回一个future object，而在最后一行需要futrueResult的时候，在获得future的值之前，它都会阻塞进程。

### 总结

- Strengths

    1. Small footprint(所以常用在嵌入式系统中)
    2. Simplicity(very small language core)
    3. Flexibility
    4. Concurrency(coroutine, actors)

- Weaknesses

    1. Syntax(too simple to be productive)
    2. Community(太小众)
    3. Performance(actually hard to evaluate without context)
