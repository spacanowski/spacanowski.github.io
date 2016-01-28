---
layout: post
title:  "Java 7+ time units full of awesomeness"
date:   2016-01-28 22:45:28
categories: java silent-features
---

In Java 7 few waited for features were introduced. G1 GC, type interference for generic instance creation (code name 'diamonds'), multi catch, try-with-resource to name few.

Some of them went unnoticed. One of my favorites is [TimeUnits][tu] class from concurrent utils. It's just enum but with cool features. It has some really, really cool features. So let us explore some of them.

The easiest one and something you'll use the most is time units conversion. Who didn't wrote multiplications to get how many milliseconds are in day or hour ? Something like:

{% highlight java %}
private static final long DAY = 24*60*60*1000;
private static final long HOUR = 60*60*1000;
{% endhighlight %}

Or something even more crazier. If you said 'I didn't', stop lying to yourself and others. Well now you don't have to. Now you can write something readable. Something that no one needs to check or split in many variables. Now you can write this little beauty.

{% highlight java %}
TimeUnit.DAYS.toMillis(1);
{% endhighlight %}

Every standard time unit (days, hours, minutes, seconds, milliseconds, nanoseconds and microseconds) is supported and can be converted to others. Method ```toSomeTimeUnit()``` (e.g. ```toMillis(...)```, ```toDays(...)```, etc) takes as argument how many of base units you want to change to units from method name. In example above I converted 1 day to milliseconds. Well, you get the idea. You can use the same with ```convert(...)``` method, which will be easier to read and understand by devs who didn't know this awesome enum existed.

{% highlight java %}
TimeUnit.MILLISECONDS.convert(1, TimeUnit.DAYS);
{% endhighlight %}

It's the same as example above.

By now you're probably wandering why it's in concurrent utils package. The answer is simple. It has 3 methods that will make your life easier when writing multi-threaded apps. Those methods are ```sleep(...)```, ```timedJoin(...)``` and ```timedWait(...)```. Those are just opaqued methods ```Thread.sleep(long millis)```, ```Thread.join(long millis)```, ```Object.wait(long timeout)```. The difference is that you don't have to pass milliseconds but time units of your own choosing. Which one do you think is more readable in this code ?

{% highlight java %}
try {
    TimeUnit.MINUTES.sleep(1);
    //Same as
    Thread.sleep(60000);
} catch (InterruptedException e) {
    // cry me a river
}
{% endhighlight %}

Yep, thought so. All of this is even prettier and more readable when you use static imports.

So I think thats all you need to know about ```TimeUnit```. Now you can go and refactor your code and buy me a beer if we met by any chance.

[tu]: https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/TimeUnit.html