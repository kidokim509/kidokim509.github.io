---
title: FLUME KAFKASINK에서 KAFKA LOG의 PARTITIONING KEY 설정
date: 2015-05-28 23:00:00 +0900
categories: [dev, flume]
tags: [hadoop, flume]
---
Flume KafkaSink로 메시지를 Kafka에 쌓아 보았더니, 하나의 Partition에만 로그를 쌓다가 10분 간격으로 Partition이 변경되었다.

이렇게 된 이유는 Kafka에 로그를 쌓을 때 (KeyedMessage) Key를 명시하지 않으면 Kafka가 아래와 같이 Partitioning을 수행하기 때문이다.
>when the partitioning key is not specified or null, a producer will pick a random partition and stick to it for some time (default is 10 mins) before switching to another one.
><https://cwiki.apache.org/confluence/display/KAFKA/FAQ>

그런데 만일 Kafka에 쌓인 메시지를 가지고 실시간 분산 처리를 하려고 하면 동시에 여러 Partition에 로그가 분산되지 않아, Kafka Consumer 또한 10 주기로 하나씩만 일하게 되는 문제가 발생한다.

이를 개선하기 위해서는 Flume Event의 Header에 Partition Key를 추가해주면 된다. (Key 이름: “key”)

그러나 flume properties 파일에는 동적으로 Partition Key를 넘길 수 없기 때문에, Flume Interceptor를 하나 생성하여, Interceptor 처리를 하면서 Event 하나 혹은 Event Batch 마다 동일한 Partition Key를 Event Header 객체에 반영 시켜야 한다.
