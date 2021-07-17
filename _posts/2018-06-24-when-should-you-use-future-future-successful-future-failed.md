---
id: 167
title: When should you use Future/Future.successful/Future.failed.
date: 2018-06-24T01:21:48+01:00
author: abdhesh
layout: post
guid: http://www.learnscala.co/?p=167
permalink: /when-should-you-use-future-future-successful-future-failed/
categories:
  - Scala
tags:
  - Futrue
  - Scala
---
In this blog post, I would recommend when should you use Future()/Future.succcessful/Future.failed?

<li style="list-style-type: none;">
  <ol>
    <li>
      <strong>Use Future.apply() or simply Future() (i.e., Future block):</strong> In the situations, where something to be done asynchronously that can complete sometime in future and may deal with some time consuming operations such as network calls, database operations communicate with one or many other services, processing huge data by consuming multiple cores and etc.
    </li>
    <li>
      <strong>Use Future.successful: </strong>When a literal or already computed value to be passed back as a successful future response.
    </li>
    <li>
      <strong>Use Future.failed:</strong> When a known and literal exception to be thrown back without performing any further actions in the future.
    </li>
    <li>
      <strong>Future.fromTry:</strong> When you already computed Try a value
    </li>
  </ol>
</li>

**Future**.successful**, Future.failed, and Future.fromTry** when you need to create an instance of Future and you already have the value.

Reference:

<a href="https://viktorklang.com/blog/Futures-in-Scala-protips-3.html" target="_blank" rel="noopener">https://viktorklang.com/blog/Futures-in-Scala-protips-3.html</a>

<a href="https://functional.works-hub.com/learn/scala-future-blocks-and-futhers-methods-what-to-use-when-4acbd" target="_blank" rel="noopener">https://functional.works-hub.com/learn/scala-future-blocks-and-futhers-methods-what-to-use-when-4acbd</a>

&nbsp;

Thanks.