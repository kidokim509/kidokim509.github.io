---
title:   조인(SQL과 코어 스파크) (하이 퍼포먼스 스파크 chap4)
date:    2018-11-07 23:59:00 +0900
categories: [dev, spark]
tags: [spark]
permalink: /spark/highperformancespark-chap4/
---

## 코어 스파크 조인
- 일반적인 조인은 로컬에서 작업을 수행할 수 있도록 각 RDD에 연관되는 키가 같은 파티션에 있기를 요구하므로 비용이 비싸다.
- 조인의 비용은 키의 개수와 레코드가 올바른 파티션에 위치하기 위해 움직여야 하는 규모에 비례해서 커진다.

### 조인 형태 선택하기
- 키 공간(개수)를 줄이기 위해 distinct나 combineByKey 연산을 수행하거나 중복키를 관리할 수 있는 cogroup을 사용하는 것이 나을 수 있다.
  - 조인하기 전에 조인할 키의 개수를 줄여놓는 것이 (중복없이) 낫다는 이야기
- 키가 양쪽 RDD에 모두 존재하는 것이 아니라면 예상치 못하게 데이터를 유실할 위험이 있다.
  - INNER냐 OUTER냐에 대한 이야기
- 규모가 큰 셔플을 피하기 위해 조인 전에 필터링 같은 방법으로 크기를 미리 감소시키는 것이 더 좋을 수 있다.
  - 수동으로 Predicate Pushdown을 하라는 이야기

### 실행 계획 선택하기
- 스파크의 기본 조인 구현: Shuffled Hash Join
  - RDD 키의 hash값을 가지고 파티션하여 각 파티션에 같은 키가 모이도록 함
  - 항상 작동하는 방식이긴한데, shuffle이 발생하므로 성능 cost 발생
- 셔플을 피하는 방법
  - 양쪽 RDD가 명시적인 파티셔너를 가진다. (이렇게 하면 RDD가 만들어질 때부터 키별로 파티션이 되어있기 때문에 조인에 의한 추가 재파티션이 없을 거라는 의미인 듯)
  - 한쪽의 데이터 사이즈가 메모리에 로드할 만큼 작다면, 브로드캐스트 해시 조인으로 해결
    - 스파크 SQL    
      - 알아서 브로드캐스트 조인 수행
      - 설정: spark.sql.autoBrodcastJoinThreshold, spark.sql.broadcastTimeout
    - 스파크 코어
      - 브로드캐스트 조인이 구현되어있지 않음 
      - 작은 쪽 RDD를 드라이버로 내린 뒤에, bigRDD.sparkContext.broadcast(smallRDDLocal) 방식으로 브로드캐스팅 시킨다.

## 스파크 SQL 조인
스파크 SQL은 조인의 효과를 높이기 위해 푸시다운을 수행하거나 연산 순서를 재정렬할 수 있다. 반면 사용자가 제어할 수는 없으므로 직접 셔플을 회피할 순 없다.
- DataFrame / DataSet의 Query Plan 보기
  - df.queryExecution.executedPlan
