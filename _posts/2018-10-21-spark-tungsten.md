---
title:   spark tungsten
date:    2018-10-21 23:59:00 +0900
categories: [dev, spark]
tags: [spark, tungsten]
---

## spark tungsten
Tungsten is the codename for the umbrella project to make changes to Apache Spark’s execution engine that focuses on substantially improving the efficiency of memory and CPU for Spark applications, to push performance closer to the limits of modern hardware. This effort includes the following initiatives:

- Memory Management and Binary Processing: leveraging application semantics to manage memory explicitly and eliminate the overhead of JVM object model and garbage collection
- Cache-aware computation: algorithms and data structures to exploit memory hierarchy
- Code generation: using code generation to exploit modern compilers and CPUs
- No virtual function dispatches: this reduces multiple CPU calls which can have a profound impact on performance when dispatching billions of times.
- Intermediate data in memory vs CPU registers: Tungsten Phase 2 places intermediate data into CPU registers. This is an order of magnitudes reduction in the number of cycles to obtain data from the CPU registers instead of from memory
- Loop unrolling and SIMD: Optimize Apache Spark’s execution engine to take advantage of modern compilers and CPUs’ ability to efficiently compile and execute simple for loops (as opposed to complex function call graphs).

## spark's memory model
- Spark Executor Memory (spark.executor.memory)
  - Reserved Memory
    - Reserved for system, hard-coded 300MB
  - Spark + User Memory (spark.executor.memory - Reserved Memory)
    - Spark Memory
      - spark.memory.fraction * (Spark + User Memory)
      - Storage Memory: RDD Cache, Broadcast variables 등의 저장영역
        - spark.memory.storageFraction * Spark Memory
      - Execution Memory: Shuffle, Join, Sort, Aggregation 계산과정의 임시 데이터 저장영역
    - User Memory
      - (1 - spark.memory.fraction) * (Spark + User Memory)
      - 사용자가 생성한 object들 저장 공간
- Memory Overhead
  - spark.executor.memoryOverhead (== spark.yarn.executor.memoryOverhead. v2.3 부터 depreated)
  - default: spark.executor.memory * 0.10, with minimum of 384MB
  - The amount of off-heap memory to be allocated per executor.

### 참고
[https://www.tutorialdocs.com/article/spark-memory-management.html](https://www.tutorialdocs.com/article/spark-memory-management.html)
[https://0x0fff.com/spark-memory-management/](https://0x0fff.com/spark-memory-management/)
[http://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-2/](http://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-2/)
[https://www.slideshare.net/laclefyoshi/apache-spark-70339069](https://www.slideshare.net/laclefyoshi/apache-spark-70339069)

