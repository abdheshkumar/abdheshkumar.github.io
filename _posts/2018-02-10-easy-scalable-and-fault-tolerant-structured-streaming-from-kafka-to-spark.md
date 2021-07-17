---
id: 140
title: Easy, Scalable and Fault-tolerant Structured Streaming from Kafka to Spark
date: 2018-02-10T15:28:56+00:00
author: abdhesh
layout: post
guid: http://www.learnscala.co/?p=140
permalink: /easy-scalable-and-fault-tolerant-structured-streaming-from-kafka-to-spark/
categories:
  - Kafka
  - Scala
  - Spark
  - Spark Streaming
---
In this blog post, I will explain about spark structured streaming. Let&#8217;s first talk about what is structured streaming and how it works?  
**Structured Streaming** is a scalable and fault-tolerant stream processing engine built on the Spark SQL engine. In short,_ **Structured Streaming provides fast, scalable, fault-tolerant, end-to-end exactly-once stream processing without the user having to reason about streaming**._The Spark SQL engine will take care of running it incrementally and continuously and updating the final result as streaming data continues to arrive.

Let’s say you want to maintain a running program data received from Kafka to console(just an example). Below is the way of express Structured Streaming.

First, create a local SparkSession, the starting point of all functionalities related to Spark.

<pre><code class="scala">val spark = SparkSession.builder
    .master("local[*]")
    .appName("app-name")
    .config("spark.executor.cores", "2")
    .config("spark.executor.memory", "4g")
    .getOrCreate()
</code></pre>

Next, let’s create a streaming DataFrame that represents data received from a server Kafka server.

<pre><code class="scala">/**
Specify one or more locations to read data from
Built in support for Files/Kafka/Socket,pluggable
Can include multiple sources of different types using union()
*/
val upstream = spark.readStream
    .format("kafka")
    .option("kafka.bootstrap.servers", "localhost:9092")
    .option("subscribe", "test-topic")
    .option("startingOffsets", "earliest")
    .load()
</code></pre>

This **upstream** DataFrame represents an unbounded table containing the streaming data. This table contains seven columns data named key, value, topic, partition, offset, timestamp and timestampType. each streaming data becomes a row in the table.  
The **upstream** DataFrame has the following columns:

<table>
  <tr>
    <th>
      key(binary)
    </th>
    
    <th>
      value(binary)
    </th>
    
    <th>
      topic(string)
    </th>
    
    <th>
      partition(long)
    </th>
    
    <th>
      offset(long)
    </th>
    
    <th>
      timestamp(long)
    </th>
    
    <th>
      timestampType(int)
    </th>
  </tr>
  
  <tr>
    <td>
      [binary]
    </td>
    
    <td>
      [binary]
    </td>
    
    <td>
      &#8220;topicA&#8221;
    </td>
    
    <td>
    </td>
    
    <td>
      345
    </td>
    
    <td>
      1486087873
    </td>
    
    <td>
    </td>
  </tr>
  
  <tr>
    <td>
      [binary]
    </td>
    
    <td>
      [binary]
    </td>
    
    <td>
      &#8220;topicB&#8221;
    </td>
    
    <td>
      3
    </td>
    
    <td>
      2890
    </td>
    
    <td>
      1486086721
    </td>
    
    <td>
    </td>
  </tr>
</table>

For more information, you can visit on <a href="http://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html" target="_blank" rel="noopener">Spark-Kafka strutucured streaming options</a>.

<pre><code class="scala">val data = upstream.selectExpr("CAST(value AS STRING)")
val downstream = data
    .writeStream
    .format("console")
    .start()

  downstream.awaitTermination()
</code></pre>

So now you have transformed DataFrame one column named “value” by Casting binary value to string and injected console sink. All data coming from Kafka will print on console.  
Here is an example that will receive data from multiple Kafka topics and will partitioned data by topic name.

<pre><code class="scala">val spark = SparkSession.builder
    .master("local[*]")
    .appName("app-name")
    .config("spark.executor.cores", "2")
    .config("spark.executor.memory", "4g")
    .getOrCreate()
val checkpointLocation = "/tmp/temporary-" + UUID.randomUUID.toString
val upstream = spark.readStream
    .format("kafka")
    .option("kafka.bootstrap.servers", "localhost:9092")
    .option("subscribe", "test,Airport,Airports,Carriers,Planedata")
    .option("startingOffsets", "earliest")
    .load()
    .selectExpr("topic", "CAST(value AS STRING)")// Transform "topic" and "value" columned

  val downstream = upstream
    .writeStream
// Partition by topic. it will create directory by topic name opic=Airport,topic=Carriers,topic=Planedata 
    .partitionBy("topic")
    .format("csv")
    .option("path", "/tmp/data")
    .outputMode("append")
    .trigger(ProcessingTime(3000))
    .option("checkpointLocation", checkpointLocation)
    .start()

  downstream.awaitTermination()</code></pre>

Here is <a href="https://github.com/abdheshkumar/spark-practices/blob/master/src/main/scala/KafkaToHdfsUsingSpark.scala" target="_blank" rel="noopener">complete source code</a>.

**Basic Concepts:**

<img loading="lazy" class="alignnone wp-image-142 size-full" src="http://www.learnscala.co/wp-content/uploads/2018/02/structured-streaming-stream-as-a-table.png" alt="" width="1472" height="792" srcset="http://www.learnscala.co/wp-content/uploads/2018/02/structured-streaming-stream-as-a-table.png 1472w, http://www.learnscala.co/wp-content/uploads/2018/02/structured-streaming-stream-as-a-table-300x161.png 300w, http://www.learnscala.co/wp-content/uploads/2018/02/structured-streaming-stream-as-a-table-768x413.png 768w, http://www.learnscala.co/wp-content/uploads/2018/02/structured-streaming-stream-as-a-table-1024x551.png 1024w" sizes="(max-width: 1472px) 100vw, 1472px" /> 

A query on the input will generate the “Result Table”. Every trigger interval (say, every 1 second), new rows get appended to the Input Table, which eventually updates the Result Table. Whenever the result table gets updated, we would want to write the changed result rows to an external sink.  
Note that Structured Streaming does not materialize the entire table.

**Input Sources**: There are a few built-in sources.  
**File source** &#8211; Reads files written in a directory as a stream of data. Supported file formats are text, csv, json, parquet.  
**Kafka source** &#8211; Reads data from Kafka. It’s compatible with Kafka broker versions 0.10.0 or higher.  
**Socket source (for testing)** &#8211; Reads UTF8 text data from a socket connection. The listening server socket is at the driver. Note that this should be used only for testing as this does not provide end-to-end fault-tolerance guarantees.  
**Rate source (for testing)** &#8211; Generates data at the specified number of rows per second, each output row contains a timestamp and value.

There is a lot to explain about structured streaming so I can not write everything in the single post but hope you get a basic idea how structured stream works with Kafka.

**References:**  
<a href="http://spark.apache.org/docs/latest/structured-streaming-programming-guide.html" target="_blank" rel="noopener">Structured Streaming Programming Guide</a>

<a href="http://spark.apache.org/docs/latest/structured-streaming-kafka-integration.html" target="_blank" rel="noopener">Structured Streaming + Kafka Integration Guide (Kafka broker version 0.10.0 or higher)</a>