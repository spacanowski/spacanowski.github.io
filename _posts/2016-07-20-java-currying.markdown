---
layout: post
title:  "Dude, where is my currying or why java cannot make it right ?"
date:   2016-07-20 20:32:28
categories: java scala functional
---

Currying is great. If you want to make oh so pretty functions with clear calls it's the way to go. So why Big O when adding Java lambdas couldn't make call to functional interfaces seamless but called explicitly ? Let me give you example of what I mean.

In Scala we can do 

{% highlight scala %}
def add(x: Int): Int => Int = y => x + y

add(1)(2)
{% endhighlight %}

Pretty as hell.

In javascript we do

{% highlight javascript %}
function add(x) {
    return function(y) {
        return x + y;
    };
}

add(1)(2)
{% endhighlight %}

Still ok. Maybe even easier to read.

But Java ? Oh, that's just another pair of shoes.

{% highlight java %}
Function<Integer, Function<Integer, Integer>> add = (x) -> (y) -> x + y;

add.apply(1).apply(2)
{% endhighlight %}

Declaration is still very pretty, despite awful and long as month type. But call ? What the fuck ? What happened ? Where did you took turn from good to worst ?

My proposition: for Java 10 don't care about new features, just fix functional ones.