---
title: 스파크는 어떻게 동작하는가? (하이 퍼포먼스 스파크 chap2)
date: 2018-10-17 13:00:00 +0900
categories: [dev, spark]
tags: [spark]
---

## RDD
- In Memory
- Immutable
  - Dependency
    - Narrow
    - Wide
  - Properties
    - Partitions
      - blocks, splits
    - Parent dependencies
    - Function (File, Pair, Shuffled, ...)
    - Partitioner (Opt.)
    - PreferredLocation (Opt.)
- Lazy Evaluation (Execution Plan)
  - Transformation
  - Action
- Memory Management
  - LRU Cache - per partition

## Spark Application
- Jobs (per Action)
  - Stages
    - Set of tasks
    - Stage boundary (Input, Shuffle)
      - Wide dependency
    - Sequencial
  - Tasks
    - Parallel
    - 1 Task / Partition
    - N Tasks / Executor (Slots)
- DAG Scheduler
  - Stage, Task 분할
- Task Scheduler
  - Cluster Manager를 통해 Task를 알맞은 Worker에 있는 Executor에 할당

## Driver and Executors
- Driver: Spark Application의 main
- Executor: 실제 데이터 처리 작업을 수행
  - 1 JVM
  - 1 core
  - Workers (Nodes)
    - 1대의 Worker에서 N개의 executor 실행 (Spark Cluster에서 동시에 여러 Application 실행)

