---
layout: post
title:  "Variables and dependencies version management in gradle microservices"
date:   2016-07-17 13:55:28
categories: java gradle microservice
---

In work I develop project in microservice architecture. It's really great. I'm responsible for only few services and some common libs. As always there's small, tiny, micro 'but'. Here's what happened. One week we moved our nexus to different address and I've had to change all links in gradle build files. As we work with standard GitHub dev flow it took some time to finish (all PRs, reviews, etc.). Next week we decided to upgrade our libs and frameworks to newest versions. Again it took some time to finish because most services use Dropwizard and same libs. Another week passed and I made changes to our common lib and needed to release new version of it. Guess what ? Yep, I needed to change version in all gradle build files. I was quite pissed and said enough is enough and came up with idea to solve this goddamnedexhaustingtakingtoolongshitty problem. Here's how I did it.

## Idea - or my eureka moment

It was one of my greatest ideas and I don't care what others think as it solves this problem splendidly. I thought why have same definitions in all gradle files when I can replace them with variables and store values in one place instead of having to copy them in multiple places. So I created [variable-management-plugin][variable-management-plugin]. My very own gradle plugin for managing variables.

We all have some nexus or other dependency management tool, so I thought why not serve/host wee json file with all of variables in one place. So I would have to change one file instead of 10. And instead of creating 10 pull requests I would need to trigger some Jenkins jobs. We have so much tests that I can make changes with quite high probability that all functionalities still work as expected. I obviously select one microservice per framework or library and check locally if creators didn't change API (there are some crazy bastards that change API without warning).

## How to use it

Configure address to your versions json file in ```gradle.settings``` or in ```build.gradle``` by assiging address to ```variablesManagementUrl``` variable. If you want to store your variables json in secure way set ```variablesManagementUser``` and ```variablesManagementPsswd``` with username and password respectively. Credentials will be send during get request of json file.

build.gradle example:
{% highlight groovy %}
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "com.github.spacanowski:variable-managment-plugin:1.0"
    }
}
apply plugin: "pl.spc.variable.management"

project.ext {
    variablesManagementUrl = 'http://<address>/<path>/<versions_json_file>'
}
{% endhighlight %}

If you use configuration by build.gradle plugin dependency definition must be done before applying said plugin.

Remember that plugin must be applied before use of first variable that will be imported from json.

### Example

versions json example
{% highlight json %}
{
    "groovyVersion": "2.4.7",
    "junitVersion": "4.+",
    "guavaVersion": "18.0",
    "secretNexus": "https://nexus.secretnexus.com/content/repositories/snapshots/"
}
{% endhighlight %}

gradle script variables usage
{% highlight groovy %}
repositories {
    maven { url "${secretNexus}" }
    mavenCentral()
}

dependencies {
    compile group: 'org.codehaus.groovy', name: 'groovy-all', version: "${groovyVersion}"
    compile group: 'com.google.guava', name: 'guava', version: "${guavaVersion}"
    testCompile group: 'junit', name: 'junit', version: "${junitVersion}"
}
{% endhighlight %}

## Summary

Now I can maintain my sanity and don't waste so much time on simple tasks. Hope you'll like it too.

[variable-management-plugin]: https://github.com/spacanowski/variable-management-plugin