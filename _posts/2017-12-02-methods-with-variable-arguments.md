---
id: 73
title: Methods with variable arguments
date: 2017-12-02T21:02:17+00:00
author: abdhesh
layout: post
guid: http://www.learnscala.co/?p=73
permalink: /methods-with-variable-arguments/
categories:
  - Scala
tags:
  - Scala
  - variable arguments
---

- Scala methods can have **variable arguments** (_vararg_).
- A method can be specified to have a variable number of arguments by adding a **_\*_** after the type of the parameter.
- As an example, let&#8217;s define a method that takes a variable number of arguments of type String and that returns their concatenation as String:
- For obvious reasons, a method can only have one parameter that has variable arguments and it should be the **last parameter.**

{% highlight ruby %}
scala> def concatStrings(s: String*): String = s.mkString
concatStrings: (s: String*)String

scala> concatStrings("a", "b", "c")
res0: String = abc

scala> def concatStringsSep(separator: String, s: String\*): String =
s.mkString(separator)

scala> concatStringsSep("/", "a", "b", "c")
res1: String = a/b/c
{% endhighlight %}
you can pass sequence as variable length arguments to a function.

{% highlight ruby %}
scala> val listOfStrings = List("first","second","third")
listOfStrings: List[String] = List(first, second, third)

scala> concatStrings(listOfStrings:\_\*)
res5: String = firstsecondthird

scala> concatStringsSep(",",listOfStrings:\_\*)
res6: String = first,second,third
{% endhighlight %}
