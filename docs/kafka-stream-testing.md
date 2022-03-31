In this blog post, I will explain how to test Kafka stream topologies.

Kafka Stream topologies can be quite complex and it is important for developers to test their code. There is a new artifact <a href="https://mvnrepository.com/artifact/org.apache.kafka/kafka-streams-test-utils" target="_blank" rel="noopener"><strong>kafka-streams-test-utils</strong></a> providing a **TopologyTestDriver**, **ConsumerRecordFactory**, and **OutputVerifier** class. You can include the new artifact as a regular dependency to your unit tests and use the test driver to test your business logic of your Kafka Streams application.

Add below dependency in **build.sbt**

<pre><code class="scala">libraryDependencies += "org.apache.kafka" % "kafka-streams-test-utils" % "1.1.0" % Test</code></pre>

Below code example is well-known word count application.

<pre><code class="scala">import java.lang.Long

import org.apache.kafka.common.serialization._
import org.apache.kafka.common.utils.Bytes
import org.apache.kafka.streams.kstream.{KStream, KTable, Materialized, Produced}
import org.apache.kafka.streams.state.KeyValueStore
import org.apache.kafka.streams.{StreamsBuilder, Topology}

import scala.collection.JavaConverters._
class WordCountApplication {
  
  def countNumberOfWords(inputTopic: String,
                         outputTopic: String, storeName: String): Topology = {
    val builder: StreamsBuilder = new StreamsBuilder()
    val textLines: KStream[String, String] = builder.stream(inputTopic)
    val wordCounts: KTable[String, Long] = textLines
      .flatMapValues(textLine =&gt; textLine.toLowerCase.split("\\W+").toIterable.asJava)
      .groupBy((_, word) =&gt; word)
      .count(Materialized.as(storeName).asInstanceOf[Materialized[String, Long, KeyValueStore[Bytes, Array[Byte]]]])
    wordCounts.toStream().to(outputTopic, Produced.`with`(Serdes.String(), Serdes.Long()))
    builder.build()
  }
}</code></pre>

Unit test for above topology.

<pre><code class="scala">
import java.util.Properties
import org.apache.kafka.common.serialization._
import org.apache.kafka.streams.StreamsConfig
trait TestSpec {
protected val stringDeserializer = new StringDeserializer()
protected val longDeserializer = new LongDeserializer()
val config = new Properties()
config.setProperty(StreamsConfig.APPLICATION_ID_CONFIG, "word-count-application")
config.setProperty(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "dummy:1234")
config.setProperty(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String.getClass.getName)
config.setProperty(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass.getName)
}
</code></pre>

<pre><code class="scala">import org.apache.kafka.common.serialization.StringSerializer
import org.apache.kafka.streams.TopologyTestDriver
import org.apache.kafka.streams.state.KeyValueStore
import org.apache.kafka.streams.test.ConsumerRecordFactory
import org.scalatest.{FlatSpec, Matchers}

class WordCountSpec extends FlatSpec with Matchers with TestSpec {

  it should "count number of words" in {
    val wordCountApplication = new WordCountApplication()
    val driver = new TopologyTestDriver(wordCountApplication.countNumberOfWords("input-topic", "output-topic", "counts-store"), config)
    val recordFactory = new ConsumerRecordFactory("input-topic", new StringSerializer(), new StringSerializer())
    val words = "Hello Kafka Streams, All streams lead to Kafka"
    driver.pipeInput(recordFactory.create(words))
    val store: KeyValueStore[String, java.lang.Long] = driver.getKeyValueStore("counts-store")
    store.get("hello") shouldBe 1
    store.get("kafka") shouldBe 2
    store.get("streams") shouldBe 2
    store.get("lead") shouldBe 1
    store.get("to") shouldBe 1
    driver.close()
  }
}
</code></pre>

Let me explain classes used in testing the topology.

**TopologyTestDriver: **  
This class makes it easier to write tests to verify the behaviour of topologies created with Topology or StreamsBuilder. You can test simple topologies that have a single processor, or very complex topologies that have multiple sources, processors, sinks, or sub-topologies.

**The best thing about TopologyTestDriver is, it works without a real Kafka broker, so the tests execute very quickly with very little overhead.**

**Using the TopologyTestDriver in tests is easy: ** simply instantiate the driver and provide a Topology and StreamsBuilder#build() and Properties configs, use the driver to supply an input message to the topology, and then use the driver to read and verify any messages output by the topology.

**ConsumerRecordFactory:**  
Although the driver doesn&#8217;t use a real Kafka broker, it does simulate Kafka Consumer and Producer that read and write raw (byte[]) messages.  
You can either deal with messages that have keys(byte[]) and values.

**Driver Set-up:**  
In order to create a **TopologyTestDriver** instance, you need a Topology and a Properties.  
The configuration needs to be representative of what you&#8217;d supply to the real topology, so that means including several key properties (StreamsConfig).  
For example, the following code fragment creates a configuration that specifies a local Kafka broker list (which is needed but not used), a timestamp extractor, and default serializers and deserializers for string keys and values:

<pre><code class="scala">
val props = new Properties()
props.setProperty(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9091")
props.setProperty(StreamsConfig.DEFAULT_TIMESTAMP_EXTRACTOR_CLASS_CONFIG, CustomTimestampExtractor.class.getName())
props.setProperty(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName())
props.setProperty(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName())
Topology topology = ...
TopologyTestDriver driver = new TopologyTestDriver(topology, props)
</code></pre>

**Processing messages:**  
Here&#8217;s an example of an input message on the topic named input-topic.

<pre><code class="scala"> val factory = new ConsumerRecordFactory(strSerializer, strSerializer)
 driver.pipeInput(factory.create("input-topic","key1", "value1"))</code></pre>

When TopologyTestDriver#pipeInput()(Send an input message with the given key, value, and timestamp on the specified topic to the topology and then commit the messages) is called, the driver passes the input message through to the appropriate source that consumes the named topic, and will invoke the processor(s) downstream of the source.

If your topology&#8217;s processors forward messages to sinks, your test can then consume these output messages to verify they match the expected outcome.  
For example, if our topology should have generated 2 messages on output-topic-1 and 1 message on output-topic-2, then our test can obtain these messages using the TopologyTestDriver#readOutput(String, Deserializer, Deserializer)} method(Read the next record from the given topic):

<pre><code class="scala"> val record1: ProducerRecord[String, String] = driver.readOutput("output-topic-1", strDeserializer, strDeserializer);
 val record2: ProducerRecord[String, String]= driver.readOutput("output-topic-1", strDeserializer, strDeserializer);
val record3: ProducerRecord[String, String] = driver.readOutput("output-topic-2", strDeserializer, strDeserializer);</code></pre>

**Processor state:**  
Some processors use Kafka state storage(StateStore), so this driver class provides the generic  
**getStateStore(store-name)** as well as store-type specific methods so that your tests can check the underlying state store(s) used by your topology&#8217;s processors.  
In our previous example, after we supplied a single input message and checked the three output messages, our test could also check the key-value store to verify the processor correctly added, removed, or updated internal state.  
Our test might have pre-populated some state before submitting the input message and verified afterwards that the processor(s) correctly updated the state.

Here is <a href="https://github.com/abdheshkumar/kafka-stream-testing" target="_blank" rel="noopener">Kafka-Streaming-Test github code</a> that I used in the blog post. In next blog post, I will write how to test complex topologies like joins/KTables.

References:

<a href="https://kafka.apache.org/11/documentation/streams/developer-guide/testing.html" target="_blank" rel="noopener">Kafka Streaming testing</a>

&nbsp;