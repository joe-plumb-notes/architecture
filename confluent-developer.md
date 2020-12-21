# CONFLUENT DEVELOPER SKILLS FOR BUILDING APACHE KAFKA® V5.3

## Fundamentals of Kafka Recap
### Distributed streaming platform
- Often referrd to as Event Streaming Platform
- Not just transporting events, but storing as well. More than just messaging, storage, processing.. Putting these together makes the event streaming platform.
- Backbone of an enterprise data architecture.
### Commit log
- Persistent representation of a sequence of events.
- They have an inherent order (that they arrive, and are stored). Each event is appended. Once written, cannot be updated (so, immutable) and unbounded (can keep coming). 
- Always appending to the end. Write once, read many.
- Many applications can read from the log simultaneously. Reading in Left to right, starting at oldest / earliest.
- Each application rading can maitnian their own reference to where they are. (Consumer offset)
- Very well suited to having multiple readers. 
- Being a distributed system, meaks it can provide high throughput, support massive scalability, and massive data volumes. 2 important principles are:
    - HA - ensure the data is always available and that failures will not cause you to lose access. HA provided through replication of your commit logs - Kafka takes single leader approach, where all reads and writes happen, and additional follower logs happen, acting as another form of consumer. There will be latency between followers and lead. You can have in sync replicas, a flag of how up to date the follower log is - more on this later.
    - Scalability - provided by partitioning of data, enabled by topics. A topic will usually have a type of data, with a particular structure (doesnt have to be!). Topic is distributed across the cluster. A topic has many partitions, each represents a subset of the data, spread out evenly across the nodes in the cluster. It's like sharding data. Partitions can have one or more segments - which is a single log file/commit log, which represents a subset of data in that partition. Being able to remove old segments as they reach a certain age enables log retention without affecting the data we are still actively writing to the active segment in the partition.
- Brokers also. This is a jvm process, typically run on different hosts (one on each). Replicas stored on these brokers. 
- 2 main types of retention:
    - Time based (removing old logs), retention period can be defined on a topic by topic basis. 
    - Compact retention policy. Works based on keys you define in your events. Multiple records with the same key, will keep the most up to date one and remove older versions. Useful when you want to manage some kind of mutable state - data constantly updating over time. e.g. inserts updates and deletes on a table. When a row is updated, publish an event, and keep the current state. This emulates mutable state, and relies on log compaction.
- Producers and consumers are your apps where your logic lives. Uses client libraries. 

### Kafka producer and consumer groups
- Way to scale out consumption. Could just have a single consumer .. but higher throughput, more parallelism, and leveraging scalability, done by consumer group. Think of each as a separate process or jvm. Each can consume a different subset from the topic(s). 
- Parallelize consumption by assigning different partitions to different consumers in the group.

### Event structure
- Body 
    - Key and value, key is optional unless compacted topic, and value i.e. your record. Key is typically used to identify some kind of _relationship between events_ that will be useful when consuming downstream), and 
- Metadata
    - Headers (originating IP address, tracking info, encryption key..) something that is important to associate, but keep logically distinct from the payload itself. 
    - Timestamp for event time. Can set in the producer. Kafka will assign these if you dont.

### Producer architecture
- Producer record encapsulates a single message. Topic and Value required, can also have key etc.
- `Send()` method will serialize the message, convert to a binary byte array, partition it to decide where it needs to be sent, then buffer, package with other messages, and then send to the broker. Broker with acknowledge reciept, and success or failure. Can auto-retry (configurable), and if keeps failing, can trigger exception.
- Serializer takes high level types and converts them to bytes. This is how Kafka stores data too. Structure/schema is not enforced - just stores byte arrays. Have to serialize before you send. Consumer will deserialize to convert back to higher level object. You can store whatever you want! It's up to your app to enforce the schema.
- Partitioning also happens in the producer - the partitioning algorithms (2 built in), hashing based on the key to determine which partition to send to. Default will send all messages with the same key to the same partition - obviously helpful for guaranteeing order or processing across the log. _Order is only guaranteed at the partition level_.
- If you dont need semantic partitioning, don't set a key, and then partitioning will be done round robin.
- Key based partitioning, be aware of your key cardinality and distributions. 
- Kafka doesnt handle unbalanced data across partitions for you, this is the developer responsibility. So may require you to write a custom partitioner.

### Consumer group architecture
- Multiple parallel consumers assigned to different paritions in the topic. Automatic load balancing and fault tolerance.
- Same partition will not be assigned to more than 1 consumer. So limited to same number of partitions in the consumer group.

### Other topics to cover
- Kafka connect - from external system to Kafka, or from Kafka to an external system. Source and sink connectors. Saves you building a custom producer/consumer application to integrate. _Data transfer into and out of_ Kafka. Connectors are streaming applications. 

## Kafka Architecture
### Log file storage
- Commit log, adapted for event streams by Kafka, but nothing new.
- No need to deal with locking or record contention as append only. 
- Partitions stored as separate logs. Can think of a partition as a directory, whose name consists of topic name and partition number, and the files in the directory being the segments.

### Replication for HA and durability
- Replication factor manages number of replicas of the partitions across brokers. 3 is a good number for availability and cost. 
- Producers and consumers only interact with the leader replica. If leader fails, a follower can be elected. Kafka manages this process. Information about this is also propogated to the clients (producers, consumers, KSQL, Kafka Connect etc).

#### Detecting failure
- Controller is responsible. It's part of one of your brokers, and a function of the broker. Only one will be this. It's the first one to start by default.
- Controller monitors via zookeeper. Each broker is responsible for heartbeating into zookeeper. If not, controller assumes failure, triggers a leader election process, updates the ISR list, persists the new leader to ZooKeeper, and sends leader and ISR list changes to all brokers.
- Restarted broker will be sent leader and ISR list info.
- If current controller fails, another broker is assigned controller role.

### In-sync replicas
- Followers try to keep up with the data being produced to the leader by replicating with as low latency as possible. A follower that is keeping up, is called an in sync replica (ISR). 
- Distinction between ISR and out of sync replica is fairly significant. ISR is guaranteed to have all committed messages - i.e. all messages that have been produced to the cluster, that are visibile to the consumer. Followers may lag, but will be strongly consistent. Message is not committed until it has been replicated to all insync replicas.
- An insync replica will be selected to be the new leader if the leader fails, if possible. Otherwise you introduce inconsistency and data loss. You can enable this if needed. 
- High watermark point is the latest committed message. Can be more recent messages, but would not be visible to consumers until committed.

### Scaling using Partitions
- If a topic has only one partition, that means all clients will be communicating with the broker (even if the other brokers exist), because the leader exists only on one broker. By partitioning, we can scale by distributing the leadership of partitions across different brokers.
- _Ordering_ data is important. With the default hash partitioner, messages with the same key, from the same producer are delivered to the consumer in order. This is the only guarantee of order.
- Consumers will read messages across partitions in a loop, so that we don't starve a partition. You'll get little groups of messages from each partition, and won't have global ordering. You could have a topic with a single partition, but you'd limit scalability here. You could re-order/re-sequence at your application layer, but complex and resource intensive. 

### Consumer groups for scalability
- Extends the scalability of kafka to your consumers. Typically always run consumers in a consumer group, as it gives more flexibility and easier to do. 
- On startup, provide `group.id`, and those consumers that have an identical id will be co-ordinated by Kafka's Group Co-ordinator.  
- Think of consumer group as logical applications, doing different units of work.

#### Partition assignment
- Exactly one consumer in a consumer group per partition. All messages with the same key, to the same consumer in the group. `partition.assignment.strategy` in the consumer config. There are three:
    - Range (default) - goes one topic at a time, and assigns as evenly as possible. Rage assigner assigns matching partitions from different topics to the same consumer - good for joins. If you're doing hash partitioning on different topics on the same key, then the events for that key will be on the same partitions. This is known as co-partitioning. Common strategy and sometimes a requirement for functionality (e.g. KSQL)
    - Round Robin - maybe more suitable if you're not joining across topics. No attempt to match consumers and partitions across topics.
    - Sticky - similar to round robin in initial assignment, however when something changes in the consumer group and a re-balancing occurs, it will try to minimize changes and therefore minimize the impact on your application. 
- Doesn't matter if you're only consuming from one topic, as effect is pretty much the same.

- Limited at the top end of scalability by the number of partitions that you have to consumer from.
- You can change the number of partitions in the topic, but be careful - it's a simple administrative process to do this, but it does not move existing data. So if you're using the hash partitioner, you're changing the formula. So your data may end up going to a different partition, giving you messages with the same key in different partitions. Work around is to create a new topic with new partitions, then replicate data across to the new topic, so you can guarantee data with the same key is on the same partition.

### Foundational security for the cluster
- Data encrypted in transit by TLS/SSL, between HTTP clients and REST APIs of other services (Mutual TLS or Basic auth).
- ZooKeeper may now support TLS encryption for comms between Kafka cluster and that. 
- No ootb at rest encryption - but can use volume level encryption (file system encryption), or, encrypt in your client application (do your own encryption). Taking advantage of kafka's "just moving binary data"-ness.
- Between Native clients and Kafka cluster, can use Mutual TLS and SASL (OAuth2, Kerberos, ScRAM, and more)
- Authorization is done in Kafka using ACLs. Who can do what. ACLs defined at the broker level, stored in ZooKeeper.
- RBAC supported in Confluent 5.3. Extends ACLs to the other components and beyond. 

## Developing with Kafka
- Can access Kafka with a wide array of native clients. Communicate via HTTP if the language you're working in doesn't have a good native client, or if you want a thin client. 
- Confluent Enterprise also has MQTT proxy. Directly interfaces with Kafka broker. 

### Writing a producer
- In Java, use `KafkaProducer` class. Thread safe, so you can share it across threads in your instance (Typically faster).
- Create a `Properties` object and pass it to the producer. Clients must specify `bootstrap.servers`, with a seed list of 2 or 3 brokers so that you can connect and make a metadata request to the cluster, to get the topics leaders, etc. Normally at least 2. 
- `key.serializer` and `value.serializer` also required. Must implement the `Serializer` interface. 
- `acks` determines at which point the producer will recieve acknowledgement of success or failure. `0` = fire and forget, `1` = producer will wait for leader to acknowledge successful reciept of message. Default value. Balance between thrpoughput and durability. When `acks=all`, producer will wait until all in-sync replicas have acknowledged reciept. You can get unlucky and experience data loss in any of these configs. There is additional config that can be used to improve durability further, later.
- Helper classes `ProducerConfig`, `ConsumerConfig` and `StreamsConfig` provides a more typesafe way of passing details to your client. Less error prone than passing literal strings.
- Wrap your key and value in `ProducerRecord`, along with other details, and `producer.send(record);`. Not a syncronous call - partially async, thus non-blocking. Waits for the message to be buffered locally. Returns `Future` with `RecordMetadata`, which will be populated on the send (future will throw exception when failure comes back).
    - Force blocking with `producer.send(...).get()`
- Message is sent onto the cluster when? It depends. You can batch events to improve producer performance. Manually flush with `flush()`, but uncommon. Few ways to config this .. :
    - Low latency scenario:
        - `batch.size` defaults to 16KB
        - `linger.ms` defaults to 0
        - Lots of smaller network requests which reduces throughput
    - `buffer.memory` default is 32MB, defines total memory allocated on heap in your producer jvm for all partition level buffers. There is a buffer for each topic partition that the producer is sending the data to. Modern java apps run with 1GB or more of heap. *Dont want to run out of buffer space* - if you do, your send call will block, which will have a propogating negative impact on your producer. 
#### Retries
- `retries` are to manage failures or timeout, in cases where `acks` is 1 or all. If you're using Kafka to integrate, you need to use this. Do you want to handle this all at the app level? Or would you like the client to handle this for you?
    - Does not apply to all failure modes, but failures that are considered retryable and transient failures e.g. request timeout, invalid leader, etc. 
- 1MB is default message size limit.
    - `retries` is number of times to retry. Default `MAX_INT`
    - `retry.backoff.ms` will add a pause between retries and not putting excessive load on brokers. Default is `100` (ms). Linear backoff.
- broker failure and leader election typically takes in the region of 6-10 seconds, so need to allow for this. 
- `max.inflight.requests.per.connection` enables you to issue parallel requests (request pipelining) to a broker. Can break ordering guarantees. So, if sensitive, set this to 1, or use an idempotent producer (more later). 
- Batch of messages succeeds or fails atomically (all succeed or all fail). Don't have a massive batch size because if (when) it fails, creates lot of overhead.
#### `send()` and Callbacks
- `send(record, null)` is preferred. The second parameter is an implementation of a `Callback` interface with an `onCompletion` method, that will be called by the sender thread when the ack comes back from the broker. 

```
onCompletion(RecordMetadata metadata, java.lang.Exception exception)
```

- Won't have to block or poll from the main sender thread, so this is prefereable for recieving ack. Ack returns `metadata` and `exception`, one will be `null` - `metadata` if there is an exception, or `exception` if send was successful. e.g.:

```
producer.send(record, (recordMetadata, e) -> {
    if (e != null) {
        e.printStackTrace();
    } else {
        System.out.println("Message String = " + record.value() + 
                            ", Offset = " + recordMetadata.offset());
    }
});
```

- But how to you associate it with the message? The message is not references in `onCompletion`... _a lambda function defines a closure_ which captures surrounding context, so even though the producer record is not part of the param list, you can bind it through the closure in the lambda function. Or if implementing your own class that implements that interface, you could define a constructor argument that passes in the producer record to bind it to the callback.
- Just make sure you're doing something with it - the reason for doing this in the first place is to better handle retries and issues to prevent data loss!
- `producer.close();` blocks until all previosuly sent requests complete - close gracefully if at all possible, to prevent duplicate messages when you startup again. `finally` block or `try with resources`. jvm shutdown hook also. Will wait indefinitely, or, `producer.close(timeout, timeUnit);` if you need to time box it.

#### REST proxy
- Confluent REST API to integrate with Kafka. REST call translated into java calls into the cluster, response comes back too. 
- POST JSON, base64-encoded json or Avro-encoded json.
- Use GET to retrieve data from Kafka.
- Supports ~66% of throughput of native producer, ~20% on consumer. So not optimal for performance, but is very flexible.

### Consumers and Offsets
- Consumers track their progress using the `__consumer_offsets` - a pointer to a message offset, which represents the _next_ message that needs to be read. 
- Stored in an internal topic in Kafka. 
- Consumer offsets are auto-committed by default. It's a compacted topic. 
    - You can turn this off, and manage it yourself if you like. `enable.auto.commit`. 
- Can subscribe to 1 or more topics, and can use regex for subscription to subscribe to new topics on the fly. e.g. Topics on month
- Syncronous, blocking api that consumes the messages. Loop forever (unless wait time set)
- Limit number of messages returned with 
    - `poll(Duration)`
    - `max.partition.fetch.bytes` - partition per topic, so can get more back depending on number of topics subscribed to (1MB default)
    - `max.poll.records` to manage total size per batch
- Performance tune with:
    - `fetch.max.wait.ms` will intoduce some delay, but can reduce requests on broker on quieter topics. Default, 500ms
    - `fetch.min.bytes` default of 1, good for low latency.
- `poll()` and `fetch()` are different and it's not always 1 for 1. Internal fetching process may collect next events before you call `poll()` so it's already there for the application rather than waiting to go to the broker again.
- Prevent resource leaks by closing the consumer in a `finally` block.

