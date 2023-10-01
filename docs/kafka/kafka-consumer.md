### Kafka consumers and consumer group

#### Consumer
1. A Kafka consumer is an application or process that subscribes to one or more Kafka topics and reads messages from those topics.
2. Consumers can be written in various programming languages, and Kafka provides client libraries for popular languages like Scala, Kotlin, Java, Python, and more.
3. Each consumer maintains its offset, which is the position of the last message it has successfully consumed in each partition it's subscribed to.

#### Consumer Group
1. A consumer group is a logical grouping of Kafka consumers that work together to consume messages from one or more topics.
2. Consumer groups allow multiple consumers to distribute the workload and process messages in parallel.
3. Kafka ensures that only one consumer consumes each partition of a topic within the same consumer group. This load balancing is automatic.
4. Consumer groups enable parallel processing of messages. Each consumer in the group handles a subset of the partitions, allowing for high-throughput data processing.
5. For example, if you have a topic with 10 partitions and a consumer group with three consumers, each consumer will handle some partitions (e.g., three partitions per consumer).
6. To scale the processing capacity of your application, you can add more consumers to a consumer group.
7. Consumer lag refers to the delay between the production of a message and its consumption by a consumer.
8. Monitoring consumer group lag helps ensure that the system is processing data in a timely manner.
9. When working with consumer groups, you can choose between different offset commit strategies, such as at-most-once, at-least-once, and exactly-once semantics, depending on your application's requirements and message processing guarantees.
10. When consumers join or leave a consumer group or when partitions are added or removed, Kafka performs a consumer group rebalance to ensure that partitions are evenly distributed among consumers.

#### Consumer configs

#### bootstrap.servers:
This is one of the most critical configurations, specifying the list of Kafka brokers that the consumer should connect to for initial discovery of the Kafka cluster. It should be set to a comma-separated list of broker addresses.

#### group.id:
- **group.id** identifies the consumer group to which the consumer belongs.
- Consumers with the same group ID will work together to consume messages from topics.
- It's essential to set a unique **group.id** for each consumer group in your application.

#### auto.offset.reset:
This configuration determines what happens when a consumer starts reading from a partition that doesn't have a committed offset. 

Options include:

- **earliest**: Start from the earliest available offset.
- **latest**: Start from the latest offset.
- none: Throw an exception if no offset is found.

Choose the appropriate setting based on your application's requirements.

#### enable.auto.commit:
- If set to true, consumer offsets are automatically committed at regular intervals. If set to false, you must manually commit offsets using **commitSync()** or **commitAsync()**.
- The choice between auto-commit and manual commit depends on your application's processing semantics.

#### auto.commit.interval.ms:
If auto-commit is enabled, this configuration determines how frequently offsets are automatically committed (in milliseconds).

#### max.poll.records:
This setting limits the maximum number of records that the poll() method can return in a single call. It can be used to control the batch size of messages fetched by the consumer.

#### max.poll.interval.ms:
This configuration defines the maximum time (in milliseconds) that a poll operation can block before the consumer is considered inactive and is kicked out of the consumer group. It helps prevent long-running processing tasks from blocking the consumer.

#### fetch.max.bytes:
This setting specifies the maximum amount of data (in bytes) that the consumer can fetch in a single request. It affects the size of batches fetched from Kafka brokers.

#### fetch.min.bytes and fetch.max.wait.ms:
These configurations work together to control the behavior of the consumer's fetch requests. 

- **fetch.min.bytes** sets the minimum amount of data that must be available before a fetch is returned to the client, and **fetch.max.wait.ms** specifies the maximum time to wait for that data.

#### heartbeat.interval.ms and session.timeout.ms:
These configurations control the frequency of heartbeats sent by the consumer to the group coordinator and the maximum time a consumer can be out of contact with the group before it is considered dead and rebalancing occurs.

#### max.partition.fetch.bytes:
This setting determines the maximum amount of data fetched per partition per request. It should be set to a value that can handle the largest message size in your topics.

#### key.deserializer and value.deserializer:
These configurations specify the deserializer classes for decoding the keys and values of messages retrieved from Kafka topics.

#### security.protocol, sasl.mechanism, sasl.jaas.config, ssl.truststore.location, ssl.keystore.location
If you're using Kafka with security features like SSL or SASL authentication, you'll need to configure these settings to establish secure connections with Kafka brokers.

#### enable.auto.offset.store:
If set to false, this configuration prevents the consumer from automatically storing its offset upon consuming a message. Useful for custom offset management scenarios.
