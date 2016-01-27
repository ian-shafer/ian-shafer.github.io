---
layout: post
title: Configuration (Continued)
---

WARNING: This post is a WORK IN PROGRESS.

In my [previous post](/2016/01/25/configuration/) I talked about general configuration best practices. In this post, I'll detail the specifics of how I've configured my latest project.

First, here are some relevant details about the project:

* It's a webapp (https://anthology.co)
* It's also a REST API for our iOS app
* JVM/Scala
* Spring IoC

Configuration is always kicked off from a shell script that lives in my `/bin` directory. All of my scripts in the `/bin` directory start with the following lines:

{% highlight bash linenos %}
#!/bin/bash
source $(dirname $0)/../.include
{% endhighlight %}

Line 1 tells the shell to use `/bin/bash` to execute the script. Line 2 sources `.include`. Here's `.include`:
