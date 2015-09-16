---
layout: post
title:  "Java patterns revisited - Factory"
date:   2015-09-16 06:45:28
categories: java patterns
---

Everybody at least heard about Design Patterns. With the advent of Java 8 some design patterns in Java can be done differently than till now. Actually since there is Dependency Injection some additional patterns can be rewritten to be oh so pretty.

I thought why the hell not. So here it comes the first one, Factory.

Every body nows and used factory pattern even if they didn't know about it. So how did it look till now ?

It looked like this
{% highlight java %}
public Base factory(String name) {
	if ("first".equals(name)) {
		return new FirstImplementation();
	} else if ("second".equals(name)) {
		return new SecondImplementation();
	} else if ("third".equals(name)) {
		return new ThirdImplementation();
	} else if ("fourth".equals(name)) {
		return new FourthImplementation();
	}
	
	throw new IllegalArgumentException();
}
{% endhighlight %}
or this
{% highlight java %}
public Base switchFactory(String name) {
	switch (name) {
	case "first":
		return new FirstImplementation();
	case "second":
		return new SecondImplementation();
	case "third":
		return new ThirdImplementation();
	case "fourth":
		return new FourthImplementation();
	default:
		throw new IllegalArgumentException();
	}
}
{% endhighlight %}

Where Base is some interface and returned objects are it's implementations.

So how does it change with Java 8 ? It becomes just like factory in c++. Well, almost. The idea is the same Map with methods returning proper object. In c++ value of map is reference to function/method, in java it's just [Supplier][supplier] anonymous implementation.
In my opinion it's even prettier than in c++, because you see everything and don't need to look for function that reference is pointing to.

{% highlight java %}
{% raw %}
private Map<String, Supplier<Base>> factory = new HashMap<String, Supplier<Base>>() {{
	put("first", () -> new FirstImplementation());
	put("second", () -> new SecondImplementation());
	put("third", () -> new ThirdImplementation());
	put("fourth", () -> new FourthImplementation());
}};

public Base newFactory(String name) {
	return factory
		.getOrDefault(name, () -> {throw new IllegalArgumentException();})
		.get();
}
{% endraw %}
{% endhighlight %}

That's all folks. Next time will probably be some pattern rewritten with DI.

[supplier]: https://docs.oracle.com/javase/8/docs/api/java/util/function/Supplier.html
