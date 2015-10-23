---
layout: post
title:  "Custom javax list of string pattern validator"
date:   2015-10-23 16:26:28
categories: java validation jsr-303
---

Recently I was fixing some validation problems. In this case it was validation during REST requests. We used ```Hibernate validator``` with ```@Valid``` annotation in controller method in Dropwizard to validate if passed data are correct. It's easy, fun and we see all of rules for model in code via annotations on fields. And there it was one tiny problem. Well there always is one tiny problem that usually takes a lot of time to fix. It's a rule and I don't make rules. We needed to validate list of strings. This was list of ids of referenced objects, so it needs to have specific format. Adding additional method in controller or service to validate such list sound kind of lame. Well sound really lame and stupid. So I wasn't going to do that. It's like writing factory method with switch when you have lambdas. There are just certain things it's shame to do. It's just disgrace.

So back to documentation of javax validation to double check if there is ```@Pattern``` for lists. Well surprise, surprise there isn't. So what do you do ? You write this thing. Ok, but how ? Let's first check how ```@Pattern``` is written. There's nothing special, so let's just copy it and add some customizations. There's empty list of constrains. It takes things that implements ```ConstraintValidator```. So let's add one. It just needs two methods to work, ```initialize``` and ```isValid```. By the sound of it I think I know what they should do. So I just implemented class with them (implementing ```ConstraintValidator```) and added to previously empty constraint list.

Usage was expected to be same as original ```@Pattern```, but on list of strings (actually used on anything that implements ```Iterable``` and with string passed to generic). Despite my shock it worked at first try.

So just in few minutes I created string list pattern validation annotation that is it's own validator. Sound awesome. 

It looked like this:

{% highlight java %}
package pl.spc.custom.validation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import java.util.Iterator;

import javax.validation.Constraint;
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import javax.validation.Payload;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = { StringListPattern.Validator.class })
@Documented
public @interface StringListPattern {
    String regexp();
    String message() default "Some items aren't valid";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};

    public class Validator implements ConstraintValidator<StringListPattern, Iterable<String>> {
        String regexp;

        @Override
        public void initialize(StringListPattern c) {
            this.regexp = c.regexp();
        }

        @Override
        public boolean isValid(Iterable<String> items, ConstraintValidatorContext ctx) {
            if (items != null) {
                Iterator<String> iterator = items.iterator();

                while (iterator.hasNext()) {
                    String next = iterator.next();

                    if (!next.matches(this.regexp)) {
                        return false;
                    }
                }
            }

            return true;//Or throw exception when list is null
        }
    }
}
{% endhighlight %}