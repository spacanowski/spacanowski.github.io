---
layout: post
title:  "GitHub's Scientist for Java"
date:   2016-02-11 20:45:28
categories: java github scientist
---

You're probably reading [GitHub Engineering blog][blog], if not start of GTFO. If you didn't started reading it yet (I know you will) they introduced their tool called [Scientist][scie]. I thought it would be great to have something like that for Java. There wasn't one, so I decided to create my own version. But first thing first.

## Scientist

Idea of Scientist is quite brilliant. Sorry, not quite, it's plainly brilliant. GitHub engineers didn't want to change working code of critical path with new one without knowing how it performs on production and if it really is bug free. So what did they do ? They created Scientist that takes both implementations, registers results and time of executions of both. It executes them in random order. So one time original can be executed first, other the new one. It returns result of original implementations and catches exceptions of new one. That way you can test new implementation on production without any risks and guesses. Didn't I say it was brilliant.

## My solution

I always was a little concerned when replacing existing and working implementation with new one. I didn't even knew I wanted toll like Scientist before I read about it. I took 5 minutes to look for such tool for Java. Or better yet version of Scientist for Java. There was none, so I wrote [my own Scientist for java][my].

Java is sometimes quite difficult language to work with. Thanks to lambdas I was able to work some of the issues but not all of them. For instance testing methods that declare throwing exceptions. To address those issues I created not one Scientist but few. Each with different purpose and for different cases.

As in Java [Metrics][metrics] are now de facto standard for registering performance metrics I use them for that. They're simple with good documentation and have many integrations and provide predefined reporters for getting result in different formats.

### Main scientist

This is the guy that will most probably be used the most and can address most needs. It takes two methods that return values and don't have side effects. For methods with side effects is another one. This one takes two methods, compares their results, catches any exceptions that new implementation could raise and registers execution times and result comparison. Call of it looks something like this:

{% highlight java %}
return Scientist.run("test", () -> criticalPath.original(test, iterations), () -> criticalPath.newOne(test, iterations));
{% endhighlight %}

Where first argument is name of scientist to distinguish his result metrics from others.

### Exceptional Scientist

This one, the ```ExceptionalScientist``` can take methods that declare throwing exceptions. It work exactly like the previous one, but will re-throw any exception that was thrown by original implementation.

{% highlight java %}
    return ExceptionalScientist.run("test", this::original, this::newOne);
}

private String newOne() throws Exception {
    throw new IllegalArgumentException();
}
{% endhighlight %}

### Simple Scientist

```SimpleScientist``` is for cases when tested methods aren't returning any values and we just want to meter their execution times.

### Dangerous Scientist

As name suggest ```DangerousScientist``` is the nasty one. It's for methods with side effects that aren't returning anything. It takes two additional arguments. One is method for comparison, second one for rollback of execution of new implementation. This one unlike others always runs new implementation first and than it's rollback passed as argument. For result comparison and handling of any side effects of new implementation is responsible one using it. There are too many variables to do it automatically, like db connection tools and ORMs. Personally I don't really trust this one and I wouldn't like to test critical paths with side effect, but I know there are situations when it's needed and we replace those. Either way I would prefer to use this one little bastard that to replace such critical path on production blindly.

## Sumarry

That's all I had to say about my implementation of Scientist. If you have some other idea or proposition just comment or make pull request on projects repo [here][my]

[blog]: http://githubengineering.com/
[scie]: http://githubengineering.com/scientist/
[my]: https://github.com/spacanowski/scientist-java
[metrics]: https://dropwizard.github.io/metrics/3.1.0/