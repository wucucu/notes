Kafka
=======

Introduction to Apache Kafka

[Video][ref1] & [Slides][ref2] by [James Ward][ref3]

[Github codes][ref4]

## Records

- Key, Value, Timestamp
- Immutable
- Append Only
- Persisted

## Producers & Consumers

- Broker = Node in the cluster
- Producer writes records to a broker
- Consumer reads records from a broker
- Leader/Follower for cluster distribution

## Topics & Partitions

- Topic = Logical name with 1 or more partitions
- Partitions are replicated
- Ordering is guaranteed for a partition

## Offsets

- Unique sequential ID (per partition)
- Consumers track offsets
- Benefits: Replay, Different Speed Consumers, etc

## Producer Partitioning

- Writes to the leader of a partition
- Partitioning can be done manually or based on a key or auto by kafke
- Replication Factor is tobic-based
- Auto-Rebalancing

## Consumer Groups

- Logical name for 1 or more consumers
- Message consumption is load balanced across all consumers in a group

## Delivery Gurantees

  Producer Options
- Async (No Guarantee)
- Committed to Leader
- Committed to Leader & Quorum
  Consumer Options
- At-least-once (Default)
- At-most-once
- Effectively-once
- Exactly Once (Maybe)

## Cool Features

- Log compaction
- Disk no heap
- Pagecache to socket
- Balanced Partitions & Leaders
- Producer & Consumer & Quotas
- Heroku Kafka

[ref1]: https://www.youtube.com/watch?v=UEg40Te8pnE
[ref2]: http://presos.jamesward.com/introduction_to_apache_kafka/#/
[ref3]: https://www.jamesward.com/presos
[ref4]: https://github.com/jamesward/koober