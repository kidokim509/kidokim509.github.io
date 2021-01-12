---
title: FLUME PERFORMANCE TUNING
date: 2015-05-22 23:00:00 +0900
categories: [dev, flume]
tags: [hadoop, flume]
---
## Batch Size (Source/Sink)
  * [How-to: Do Apache Flume Performance Tuning (Part 1)](http://blog.cloudera.com/blog/2013/01/how-to-do-apache-flume-performance-tuning-part-1/)
  * 하나의 트랜잭션에  Source가 Channel에 Put하거나 Sink가 Take하는 Event 개수
    * 만일 batch size가 적은 개수만큼 이벤트가 발생하면? 
      * batch timeout안에 batch size가 다 차지 않으면 Put/Take 수행함. (무한정 기다리지 않음)  
        * batch timeout은 Source에 따라 다름. 
        * ExecSource: batchTimeout = 기본 3000ms 
        * KafkaSource: batchDurationMillis = 기본 1000ms 
      * 초당 메시지 유입 건수를 고려하여 batch size와 batch timeout 설정을 결정 
  * Tradeoff
    * throughput / latency
    * performance / duplication
  * Source, Sink의 Property로 설정
  * batchSize of sources and sinks <= transactionCapacity of channels
    * transactionCapaciry: Channel의 max batch size. (Memory Channel의 경우, Put/Take용 queue size임)
  * To squeeze all the performance possible out of a Flume system, batch sizes should be tuned with care through experimentation.

## File Channel
* dataDirs
  * Using multiple directories on separate disks can improve file channel performance.
* [About Apache Flume FileChannel](http://blog.cloudera.com/blog/2012/09/about-apache-flume-filechannel/)
  * WAL and Queue
    * WAL: transaction log (transaction id, seq #, event data)
      * the WAL is named [Log](https://github.com/apache/flume/blob/trunk/flume-ng-channels/flume-file-channel/src/main/java/org/apache/flume/channel/file/Log.java).
      * the WAL is a set of files written and read from using the [LogFile](https://github.com/apache/flume/blob/trunk/flume-ng-channels/flume-file-channel/src/main/java/org/apache/flume/channel/file/LogFile.java) class and its subclasses.
    * Queue: event pointer queue (in-memory)
      * The queue described above is named [FlumeEventQueue](https://github.com/apache/flume/blob/trunk/flume-ng-channels/flume-file-channel/src/main/java/org/apache/flume/channel/file/FlumeEventQueue.java).
      * The queue itself is a circular array and is backed by a [Memory Mapped File](https://github.com/apache/flume/blob/trunk/flume-ng-channels/flume-file-channel/src/main/java/org/apache/flume/channel/file/EventQueueBackingStoreFile.java).
  * Transaction
    * Each transaction is written to the WAL based on the transaction type (Take or Put) and the queue is modified accordingly.
    * Each time a transaction is committed, fsync is called on the appropriate file to ensure the data is actually on disk and a pointer to that event is placed on a queue.
    * During a take, a pointer is removed from the queue. The event is then read directly from the WAL. (common for that read to occur from the os file cache)
  * Checkpoint
    * 장애 후 재기동 시, Flume은 WAL을 replay하면서 Queue를 복원
    * Checkpoint : Replaying WALs can be time consuming, so the queue itself is written to disk periodically. (disk에는 queue와 queue 저장 시 최종 seq #를 저장)
    * After a crash, the queue is loaded from disk and logs after last seq # will be replayed. (Replay 시간 절약)
    * During the checkpoint operation the channel is locked so that no Put or Take operations can alter it’s state.
    * Issue: Takes and Puts which are in progress at the time the checkpoint occurs are lost.
