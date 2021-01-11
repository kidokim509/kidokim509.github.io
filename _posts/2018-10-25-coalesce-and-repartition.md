---
title:   spark rdd의 coalesce와 partition의 차이
date:    2018-10-25 23:55:00 +0900
categories: [dev, spark]
tags: [spark]
---

## coalesce
- full shuffle을 하지 않고 파티션의 개수를 조절한다.
- transform 연산 후에 결과를 1개의 파일로 저장하기 위해 coalesce(1)를 하면, transform을 1개의 executor로 실행한다.
  - 왜 그런지 추측해보면...
  - coalesce 작업은 full shuffle을 하지 않으므로 DAG Schedule상 stage 분할이 되지 않는다.
  - stage 분할 없이 partition을 1개로 합하려면 애초에 executor 하나로 partition을 몰아 넣고 시작하게 되는 것 같음

## repartition
- full shuffle을 해서 파티션의 개수를 조절한다.
- transform 연산 후에 마찬가지로 repartition(1)을 하면, transform은 N개의 executor에서 실행됨
  - 왜 그런지 추측해보면...
  - full shuffle 하므로 repartition 전후로 stage가 나뉨
  - repartion (full shuffle) 후에 partition 1개짜리 ShuffledRDD가 생성됨

## 참고
[https://hackernoon.com/managing-spark-partitions-with-coalesce-and-repartition-4050c57ad5c4](https://hackernoon.com/managing-spark-partitions-with-coalesce-and-repartition-4050c57ad5c4)
