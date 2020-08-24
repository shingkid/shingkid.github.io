---
layout: post
title:  "Apache Kafka 102"
date:   2020-09-06 11:47:22 +0800
categories: Kafka Java
---
This is a continuation of [Apache Kafka 101](https://shingkid.github.io/kafka/cli/2020/08/30/apache-kafka-101.html). If you're completely new to Kafka, you should start there. This article introduces Java programming for Kafka.

## Creating a Kafka Project


## Advanced Configurations
### acks & min.insync.replicas
1. `acks=0` (no acks)
   * No response is requested
   * If the broker goes offline or an exception happens, we won't know and will lose data
2. `acks=1` (leader acks)
   * Leader response is requested, but replication is not a gurantee (happens in the background)
   * If an ack is not received, the producer may retry
   * If the leader broker goes offline but replicas haven't replicated the data yet, we have a data loss.
3. `acks=all` (replicas acks)
   * Leader + Replicas acks requested
   * `acks=all` must be used in conjunction with `min.insync.replicas`.
   * `min.insync.replicas` can be set at the broker or topic level (override).
   * `min.insync.replicas=2` implies that at least 2 brokers that are ISR (including leader) must respond that they have the data.
     * If you use `replication.factor=3`, `min.insync=2`, `acks=all`, you can only tolerate 1 broker going down, otherwise the producer will receive an exception on send.

### retries & max.in.flight.requests.per.connection
* In case of transient failures, developers are expected to handle exceptions, otherwise the data will be lost.
* E.g. NotEnoughReplicasException
* There is a `retries` setting:
  * defaults to 0
  * can be increased to a high number, e.g. `Integer.MAX_VALUE`
* In case of *retries*, by default **there is a chance that messages will be sent out of order** (if a batch has failed to be sent)
* If you rely on key-based ordering, that can be an issue.
* For this, you can control how many produce request can be made in parallel: `max.in.flight.requests.per.connection`
  * Default: 5
  * Can be set to 1 if you need to ensure ordering (may impact throughput)
* A better solution in Kafka>=1.0.0 is Idempotent Producer.

#### Idempotent Producer
* Problem: When a producer sends a message to Kafka, Kafka might commit the message and attempt to send an ack back. However, due to a network error, the producer does not receive Kafka's ack. So the producer sends the message again, and Kafka commits the same message a second time.
* An idempotent producer can detect duplicates so the same message is not committed twice.
* Idempotent producers are great to guarantee a stable and safe pipeline
* Parameters to set
  * `retires=Integer.MAX_VALUE` (2^31-1 = 2147483647)
  * `max.in.flight.requests=1` (Kafka >= 0.11 & <1.1) or `max.in.flight.requests=5` (Kafka >= 1.1, higher performance)
  * `acks=all`
* Just set `producerProps.put("enable.idempotence", true);`

```Java
public KafkaProducer<String, String> createKafkaProducer() {
  ...

  // Create safe producer
  properties.setProperty(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, "true");
  properties.setProperty(ProducerConfig.ACKS_CONFIG, "all");
  properties.setProperty(ProducerConfig.RETRIES_CONFIG, Integer.toString(Integer.MAX_VALUE));
  properties.setProperty(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, "5");

  ...
}
```

### Safe Producer
* Kafka < 0.11
  * `acks=all` (producer level)
    * Ensures data is properly replicated before an ack is received
  * `min.insync.replicas=2` (broker/topic level)
    * Ensures two brokers in ISR at least have the data after an ack
  * `retries=MAX_INT` (producer level)
    * Ensures transient errors are retried indefinitely
  * `max.in.flight.requests.per.connection=1` (producer level)
    * Ensures only one request is tried at any time, preventing message re-ordering in case of retries
* Kafka >= 0.11
  * `enable.idempotence=true` (producer level) + `min.insync.replicas=2` (broker/topic level)
    * Implies `acks=all`, `retries=MAX_INT`, `max.in.flight.requests.per.connection=5` (default)
    * While guranteeing order and improving performance
* Running a **safe producer** might impact throughput and latency, so always consider your use case.

### Producer/Message Compression
* Important to apply compression to the producer because it usually sends data that is text-based, e.g. JSON data
* Compression is enabled at the Producer level and doesn't require any configuration change in the Brokers or in the Consumers.
* `compression.type` can be `none` (default), `gzip`, `lz4`, `snappy`
* Compression is more effective the bigger the batch of messages being sent to Kafka!
* Benchmarks: https://blog.cloudflare.com/squeezing-the-firehose/
* The compressed batch has the following advantages:
  * Much smaller producer request size (compression ratio up to 4x!)
  * Faster to transfer data over the network => less latency
  * Better throughput
  * Better disk utilisation in Kafka (stored messages on disk are smaller)
* Disadvantages (very minor):
  * Producers must commit some CPU cycles to compression
  * Consumers must commit some CPU cylces to decompression
* Overall:
  * Consider testing snappy or lz4 for optimal speed / compression ratio
* Recommendations
  * Find a compression algorithm that gives you the best performance for your specific data. Test all of them!
  * Always usecompression in production and especially if you have high throughput
  * Consider tweaking `linger.ms` and `batch.size` to have bigger batches, and therefore more compression and higher throughput

### Producer Batching
By default, Kafka tries to send records as soon as possible. It will have up to 5 requests in flight, meaning up to 5 messages are sent together at a time. Afterwards, if more messages have to be sent while others are in flight, Kafka is smart and will start batching them while they wait to send them all at once.

This smart batching allows Kafka to increase throughput while maintaining very low latency. Batches have higher compression ratio and so better efficiency

`linger.ms`: Number of milliseconds a producer is willing to wait before sending a batch out. (default 0)
* By introducing some lag (e.g. `linger.ms=5`), we increase the chances of messages being sent together in a batch
* So at the expense of introducing a small delay, we can increase throughput, compression and efficiency of our producer.
* If a batch is full (see `batch.size`) before the ned of the `linger.ms` period, it will be sent to Kafka right away.

`batch.size`: Maximum number of bytes that will be included in a batch. (default 16KB)
* Increasing a batch size to 32KB or 64KB can help increas compression, throughput, and efficiency of requests.
* Any message that is bigger than the batch size will not be batched
* A batch is allocated per partition, so make sure that you don't set it to a number that's too high, otherwise you'll exceed or waste memory.

**NOTE:** You can monitor the average batch size metric using Kafka Producer Metrics.

### High Throughput Producer
Let's add *snappy* message compression in our producer. *snappy* is very helpful if your messages are text based (e.g. log lines or JSON documents). *snappy* has a good balance of CPU / compression ratio. We'll also increase the `batch.size` to 32KB and introduce a small delay through `linger.ms` (20ms).

```Java
public KafkaProducer<String, String> createKafkaProducer() {
  ...

  // High throughput producer at the expense of a bit of latency and CPU usage
  properties.setProperty(ProducerConfig.COMPRESSION_TYPE_CONFIG, "snappy");
  properties.setProperty(ProducerConfig.LINGER_MS_CONFIG, "20");
  properties.setProperty(ProducerConfig.BATCH_SIZE_CONFIG, Integer.toString(32*1024));

  ...
}
```

#### Producer Default Partitioner
By default, keys are hashed using the "murmur1" algorithm. It is best not to override the behaviour of the partitioner, but it is possible to do so (`paritioner.class`).

```Java
// Formula
targetPartition = Utils.abs(Utils.murmur2(record.key())) % numPartitions;
```

This means that the same key will go to the same partition, and adding partitions to a topic will completely alter the formula.

### max.block.ms & buffer memory
If the buffer is full (all 32MB), then the `.send()` method will start to block (won't return right away).

`max.block.ms=60000` is the time the `.send()` will block until throwing an exception. Exceptions are basically thrown when:
1. The producer has filled up its buffer
2. The broker is not accepting any new data
3. 60 seconds has elapsed

If you hit an exception, that usually means your brokers are down or overloaded as they can't respnd to requests.

### Delivery Semantics for Consumers
* At most once: offsets are committed as soon as the message batch is received. If the processing goes wrong, the message will be lost (it won't be read again).
* At least once: offsets are committed after the message is processed. If the processing goes wrong, the message will be read again. This can result in duplicate processing of messages. Make sure your processing is **idempotent** (i.e. processing again the messages won't impact your systems)
* Exactly once: Can be achieved for Kafka-to-Kafka workflows using Kafka Streams API. For Kafka-to-Sink workflows, use an idempotent consumer.

**NOTE:** For most applications you should use **at least once processing**.