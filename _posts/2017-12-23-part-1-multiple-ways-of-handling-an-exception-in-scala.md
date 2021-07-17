---
id: 114
title: 'Part-1: Multiple ways of handling an exception in scala'
date: 2017-12-23T14:17:51+00:00
author: abdhesh
layout: post
guid: http://www.learnscala.co/?p=114
permalink: /part-1-multiple-ways-of-handling-an-exception-in-scala/
categories:
  - Scala
tags:
  - Exception
  - Scala
  - Try
  - try/catch/finally
---
There are multiple ways of handling an exception in Scala. In this blog, I will explain one by one.

**1- Using try/catch/finally**

<pre><code class="scala">
 val tryCatch = try {
    //Code here that might raise an exception
    throw new Exception
  } catch {
    case ex: Exception =&gt;
      //Code here for handle an exception
  }

  val tryMultipleCatch = try {
    //Code here that might raise an exception
    throw new Exception
  } catch {
    case ae: ArithmeticException =&gt;
    //Code here for handle an exception
    case ex: Exception =&gt;
    //Code here for handle an exception
  }

  val tryMultipleCatchFinally = try {
    //Code here that might raise an exception
    throw new Exception
  } catch {
    case ae: ArithmeticException =&gt;
    //Code here for handle an ArithmeticException
    case ex: Exception =&gt;
    //Code here for handle an Exception
  } finally {
    println(":::::")
    //Code here, will always be execute whether an exception is thrown or not
  }

  val tryCatchWithValue: Int = try {
    //Code here that might raise an exception
    "NonNumericValue".toInt
  } catch {
    case ne: NumberFormatException =&gt; 0
  } 
</code></pre>

**2. Using scala.util.Try**  
The **Try** type represents a computation that may either result in an exception, or return a successfully computed value.  
Instances of **Try[T]**, are either an instance of **scala.util.Success[T]** or **scala.util.Failure[T]**

The **Try** has an ability to _pipeline_, or chain, operations, catching exceptions along the way like **flatMap** and **map** combinators.

<pre><code class="scala">
import scala.util.{Failure, Success, Try}

  val withTry = Try("1".toInt) // Success(1)
  withTry match {
    case Success(value) =&gt; println(value)
    case Failure(ex) =&gt;
      //Code here for handle an exception
      println(ex)
  }

  val tryWithRecover = Try("Non-Numeric-Value".toInt) match {
    case Success(value) =&gt; println(value)
    case Failure(ex) =&gt; println(ex)
  }



  //Try's map,flatMap,fold etc
  def inc(n: Int): Int = n + 1

  val try1 = Try("abc".toInt)
  val tResult = try1.map(f =&gt; inc(f))// The function `inc` will execute when `Try("abc".toInt)` doesn't raise an exception
</code></pre>

**Try&#8217;s recover and recoverWith:** Applies the given function f if this is a Failure, otherwise returns this if this is a Success.

<pre><code class="scala">
//Recover with value
val tryWithRecoverF = Try("Non-Numeric-Value".toInt).recover {
    //Here you pattern match on type of an exception
    case ne: NumberFormatException =&gt; 0
    case ex: Exception =&gt; 0
  }

//Recover with an another Try
  def recoverWith(first: String, second: String): Try[Int] = {
    //The code of recoverWith function will execute when `Try(first.toInt)` raise an exception
    Try(first.toInt).recoverWith {
      case ne: NumberFormatException =&gt; Try(second.toInt)
    }
  }
</code></pre>

Note: all **Try** combinators like **map,flatMap, filter, fold, recover, recoverWith, transform, collect** will catch exceptions

<pre><code class="scala">
def compute(number: Int, divideBY: Int): Int = number / divideBY

  val t1 = Try("123".toInt).map(n =&gt; compute(n, 2)) //Success(61)
  val t2 = Try("123".toInt).map(n =&gt; compute(n, 0)) //Failure(java.lang.ArithmeticException: / by zero)
def computeWithTry(value: String): Try[Int] = Try(value.toInt)

  val r1: Try[Int] = computeWithTry("123")
  r1.fold(
    ex =&gt; println(ex),
    value =&gt; println(compute(value, 2))
  )

  computeWithTry("123").fold(
    ex =&gt; println(s"Exception--${ex}"),
    value =&gt; println(compute(value, 0))
  ) // Exception--java.lang.ArithmeticException: / by zero

  computeWithTry("abc").fold(
    ex =&gt; println(ex),
    value =&gt; println(compute(value, 2))
  )

  computeWithTry("123").map(n =&gt; compute(n, 2)) //Success(61)
  computeWithTry("123").map(n =&gt; compute(n, 0)) //Failure(java.lang.ArithmeticException: / by zero)
  computeWithTry("abc").map(n =&gt; compute(n, 2)) //Failure(java.lang.NumberFormatException: For input string: "abc")
</code></pre>

**Note**: only non-fatal exceptions are caught by the combinators on Try ([see scala.util.control.NonFatal](http://www.scala-lang.org/api/2.12.4/scala/util/control/NonFatal$.html)). Serious system errors, on the other hand, will be thrown.

Here you can find <a href="https://gist.github.com/abdheshkumar/7965a9e5df7982878ac61ce09fe92da6" target="_blank" rel="noopener">complete code</a>

Stay tuned for next part 🙂

**References: <a href="http://www.scala-lang.org/api/2.12.4/scala/util/Try.html" target="_blank" rel="noopener">scala.util.Try</a>**