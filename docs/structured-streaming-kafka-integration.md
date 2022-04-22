In this post, I will show you how to create an end-to-end structured streaming pipeline. Let&#8217;s say, we have a
requirement like:  
**JSON data being received in Kafka, Parse nested JSON, flatten it and store in structured Parquet table and get
end-to-end failure guarantees.**

```scala
//Step-1 Creating a Kafka Source for Streaming Queries
val rawData = spark.readStream
  .format("kafka")
  .option("kafka.boostrap.servers", "")
  .option("subscribe", "topic")
  .load()

//Step-2
val parsedData = rawData
  .selectExpr("cast (value as string) as json")
  .select(from_json("json", schema).as("data"))
  .select("data.*")

//Step-3 Writing Data to parquet
val query = parsedData.writeStream
  .option("checkpointLocation", "/checkpoint")
  .partitionBy("date")
  .format("parquet")
  .start("/parquetTable")
```

**Step-1: Reading Data from Kafka**  
Specify kafka options to configure  
**How to configure kafka server?**  
kafka.boostrap.servers => broker1,broker2 .load()  
**What to subscribe?**  
subscribe => topic1,topic2,topic3 // fixed list of topics  
subscribePattern => topic* // dynamic list of topics  
assign => {&#8220;topicA&#8221;:[0,1] } // specific partitions  
**Where to read?**  
startingOffsets => latest (default) / earliest / {&#8220;topicA&#8221;:{&#8220;0&#8243;:23,&#8221;1&#8221;:345} }

**Step-2: Transforming Data**  
Each row in the source(rawData) has the following schema:

| Column        | Type      |
| --------------|-----------|
| `key`         | binary    |  
| `value`       | binary    |
| `topic`       | string    |
| `partition`   | int       |  
|`offset`       | long      | 
|`timestamp`    | long      | 
|`timestampType`| int       |  

Cast binary value to string Name it column json  
**//selectExpr(&#8220;cast (value as string) as json&#8221;)**  
Parse json string and expand into nested columns, name it data  
**//select(from_json(&#8220;json&#8221;, schema).as(&#8220;data&#8221;)))**

**Step-3: Writing to parquet.**  
Save parsed data as Parquet table in the given path  
Partition files by date so that future queries on time slices of data is fast  
Checkpointing  
Enable checkpointing by setting the checkpoint location to save offset logs  
**//.option(&#8220;checkpointLocation&#8221;, &#8230;)**  
start actually starts a continuous running StreamingQuery in the Spark cluster  
**//.start(&#8220;/parquetTable/&#8221;)**

Stay tuned for next post. 🙂

**Reference**: https://spark.apache.org/docs/2.2.0/structured-streaming-kafka-integration.html