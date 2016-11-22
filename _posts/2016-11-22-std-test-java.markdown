---
layout: post
title:  "Test CLI UI in Java"
date:   2016-11-22 10:00:28
categories: java test cli
---

Most applications have web or Swing UI or just plain simple REST or SOAP interfaces. Mostly you'll write unit tests but sometimes you'll also need some integrations tests or even simple smoke tests. You can easily create such tests for application using different tools like [Selenium] for web, [FEST] for Swing or [JMeter], [SoapUI], [REST Assured] for REST/SOAP API test scenarios. There are tons of tools and you can use whatever you want. But what to do when you have CLI UI and you only use STD IN and OUT ?

In this situation you can set your own ```System.out``` and ```System.in```. I know that it sounds like bad idea and you're thinking you need to be mad to do so. Actually it's quite simple and brilliant idea that'll allow you to create integration test for CLI applications.

So, how do you set STD OUT ? Obviously you call [System.setOut]. It takes [PrintStream] that can be initialized with some [OutputStream]. Simple enough.

In practice it'll look like this:

{% highlight java %}
ByteArrayOutputStream overridenStdOut = new ByteArrayOutputStream();
System.setOut(new PrintStream(overridenStdOut));
{% endhighlight %}

Then you can make some assertions on it

{% highlight java %}
assertThat(overridenStdOut.toString(), is(expectedOut));
{% endhighlight %}

If you'll make longer scenarios just call ```reset``` on ```ByteArrayOutputStream``` (if you're using it) before another expected print and you can make another assertions.

What about STD IN ? Well it's very similar, but you use [System.setIn] of course. It also directly takes [InputStream]. If you want to imitate some input, e.g. setting username, you can do it like this:

{% highlight java %}
System.setIn(new ByteArrayInputStream("spc".getBytes()));
{% endhighlight %}

It's a little crazy idea, but that's why I like it so much.

[InputStream]: https://docs.oracle.com/javase/7/docs/api/java/io/InputStream.html
[System.setIn]: https://docs.oracle.com/javase/7/docs/api/java/lang/System.html#setIn(java.io.InputStream)
[OutputStream]: https://docs.oracle.com/javase/7/docs/api/java/io/OutputStream.html
[PrintStream]: https://docs.oracle.com/javase/7/docs/api/java/io/PrintStream.html
[System.setOut]: https://docs.oracle.com/javase/7/docs/api/java/lang/System.html#setOut(java.io.PrintStream)
[SoapUI]: https://www.soapui.org/
[REST Assured]: http://rest-assured.io/
[JMeter]: http://jmeter.apache.org/
[Selenium]: http://www.seleniumhq.org/
[FEST]: https://code.google.com/archive/p/fest/