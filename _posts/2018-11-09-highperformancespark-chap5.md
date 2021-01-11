---
title:   효율적인 트랜스포메이션 (하이 퍼포먼스 스파크 chap5)
date:    2018-11-09 23:59:00 +0900
categories: [dev, spark]
tags: [spark]
---
## 좁은 트랜스포메이션 vs. 넓은 트랜스포메이션
### 좁은 종속성의 트랜스포메이션
- 부모 RDD의 각 파티션이 자식 RDD의 최대 하나의 파티션에 의해 사용되는 것
- 즉, 부모 파티션의 자식은 오직 하나
- 값을 모르고도 할 수 있는 것들
  - 변환하거나: map, filter, mapPartition, ...
  - 합하거나: coalesce (셔플 없이)

### 넓은 종속성의 트랜스포메이션
- 부모의 각 파티션마다 여러 자식 파티션에 종속되어 있는 것
- 즉, 부모 파티션의 자식이 여럿.
  - 셔플을 수반한: groupByKey, reduceByKey, sort, join, repartition

### coalesce의 특별한 경우
- 파티션의 개수를 줄이는 경우
  - 좁은 종속성의 트랜스포메이션
  - 태스크들은 자식 파티션에서 실행되므로 coalesce 연산을 포함한 스테이지에서 실행되는 태스크의 개수는 coalesce 트랜스포메이션의 결과 RDD의 파티션 개수와 동일하다.
    - 그래서 마지막에 coalesce(1)을 해버리면 앞서 수행하는 트랜스포메이션도 다 1개의 태스크에서 실행되는 불상사가...
    - 그런데, 입력 데이터의 block이 여러 노드에 분산되어있는 경우는 어떻게 되는걸까? (실험주제)
    - 해결 방법: shuffle 값을 true로 주고 coalesce를 호출하거나 repartition을 호출
- 파티션의 개수를 늘리는 경우
  - 넓은 종속성의 트랜스포메이션
  - 자식 파티션들에게 데이터를 공평하게 분산하는 것을 우선순위로 둠. 따라서 각 입력 파티션에 데이터가 얼마나 많이 들어 있느냐에 따라 데이터의 최종 위치가 달라지므로 초반에는 결과 레코드의 위치를 알 수 없다. shuffle이 필요하다는 이야기.

## 객체 생성 최소화하기
- GC 부담을 줄이자
  - 기존 객체 재활용하기
    - 새로운 객체 생성을 최소화.
    - 예를들어 aggregateByKey의 sequenceOp, compOp 같은 것을 구현할 때, value를 누적한 새 객체를 return 하는 것이 아니라, 기존 객체의 value를 업데이트해서 return 한다.
  - 더 작은 자료 구조 사용하기
    - 사용자 정의 클래스 보단 primitive type
    - case class나 tuple 보단 Array
  - 함수 내에서 중간 객체 생성을 피하기
    - 타입 간의 변환(예. 스칼라 컬렉션 형태 변환)은 중간 객체를 만든다.
    - 암묵적 변환이 운 나쁘게 성능에 영향을 끼치는 부분

## mapPartitions로 수행하는 반복자-반복자 트랜스포메이션
- mapPartitions 트랜스포메이션
  - 사용자가 한 파티션의 데이터를 대상으로 임의의 코드를 정의할 수 있게 해준다.
- 반복자-반복자 트랜스포메이션
  - RDD 파티션의 반복자는 단순 순회뿐만 아니라
    - 매핑(map, flatMap), 추가(++), 합치기(foldLeft, reduceRight, reduce), 원소 상태(forall, exists), 순회(next, foreach) 메서드를 갖는다.
    - RDD 트랜스포메이션과 차이는 반복자 트랜스포메이션은 반복자를 반환한다.
    - 반복자 트랜스포메이션은 순차적으로 실행된다.
  - 시간/공간적 이득
    - 파티션 전체를 메모리에 로드하지 않은 상태에서 트랜스포메이션이 가능하다.
    - 파티션을 모두 디스크에 쓰지 않고 필요한 레코드만큼만 사용하여 디스크 IO나 재연산 비용을 아낀다.
    - 중간 데이터용 자료 구조를 정의하지 않아도 된다.

## RDD 재사용
### 반복적인 연산
동일한 부모 RDD를 여러 번 사용하는 트랜스포메이션이라면 부모 RDD(반복적으로 사용하는)를 영속화하자.
```scala
val keyValuePairs = Array(1.0, 2.0, 3.0, 4.0, 1.0, 2.0, 3.0, 4.0).zipWithIndex
test("Itereative Computations "){
    def rmse(rdd : RDD[(Int, Int )]) = {
      val n = rdd.count()
      math.sqrt(rdd.map(x => (x._1 - x._2) * (x._1 - x._2)).reduce(_ + _) / n)
    }

    val validationSet = sc.parallelize(keyValuePairs)

    // tag::iterativeComp[]
    val testSet: Array[RDD[(Double, Int)]] =
      Array(
        validationSet.mapValues(_ + 1),
        validationSet.mapValues(_ + 2),
        validationSet)
    validationSet.persist() //persist since we are using this RDD several times
    val errors = testSet.map( rdd => {
        rmse(rdd.join(validationSet).values)
    })
    // end::iterativeComp[]

    // the one where we didn't change anything should have the
    // lowest root mean squared error
    assert(errors.min == errors(2))

  }
```
위 코드의 DAG를 확인해보면 validationSet을 parallelize를 3번 수행 (count action 3번 호출에 의한 validationSet RDD 로딩)
DAG Plan 상에는 parallelize가 훨씬 더 많이 등장하는데 skipped task로 빠짐

### 동일 RDD에 대해 여러 번의 액션 호출
- RDD를 재사용하지 않는다면 RDD에서 호출되는 액션들은 매번 RDD 트랜스포메이션들로 이루어진 전체 계보를 바탕으로 자체적인 잡을 따로 호출
- 영속화와 체크포인팅은 RDD의 계보를 끊게 되므로 persist나 checkpoint 호출 이전의 트랜스포메이션들은 한 번만 실행
```scala
val sorted = rddA.sortByKey()
val count = sorted.count()
val sample: Long = count / 10
val sampled = sorted.take(sample.toInt)
```
sorted RDD의 sortByKey는 count, take에 의해서 총 2번 수행됨. 
아래와 같이 persist나 checkpoint를 호출한다면 계보 그래프를 RDD의 생성부터 혹은 영속화나 체크포인트된 RDD 부터 만들게 되므로 sortByKey는 한 번만 실행됨
```scala
val sorted = rddA.sortByKey()
sorted.persist()
val count = sorted.count()
val sample: Long = count / 10
val sampled = sorted.take(sample.toInt)
```

### 각 파티션의 연산 비용이 너무 큰 경우
- 모든 좁은 트랜스포메이션이 클러스터의 executor의 처리량보다 더 큰 GC 오버헤드나 메모리 부담을 만들어 낸다면 체크포인팅이나 off_heap 영속화가 도움
- 반대로 때로는 체크포인팅이나 persist를 저장하고 다시 읽는 것보다 재연산이 더 빠른 경우도 있음
  - IO vs. Computing
  - 특히 체크포인팅은 무조건 disk에 중간 결과를 저장하므로 IO 비용이 높기 때문에 재연산을 피하는 경우 조차도 전체적인 성능 향상은 크지 않다. 말 그대로 체크포인팅이 중요할 때 사용.

### 재사용의 형태
- RDD StorageLevel은 아래 5가지의 조합으로 이루어진다.
  - useDisk: 메모리에 들어가지 않는 파티션은 디스크에 기록
  - useMemory: 메모리에 저장
  - useOffHeap: 타키온(Tachyon)등 executor 바깥 메모리 저장
  - deserialized: RDD를 직렬화하지 않은 자바 객체로 저장 (용량 부담 큼. serialization을 쓰면 메모리 사용량은 줄어드나 성능 부담은 deserialized 보다는 커짐)
  - replication: 영속화 데이터의 복사본 개수를 제어. 기본은 1.

### 체크포인팅
- RDD를 HDFS나 S3 같은 외부 저장소에 저장
- RDD의 계보를 잃어버림
- 스파크 어플리케이션 종료 후에도 유지. (persist는 유지되지 않음)
- RDD의 평가를 강제
- 외부 저장 시스템의 공간 문제보다 실패나 재연산의 비용에 대한 우려가 클 경우에 쓰는 것이 가장 좋음
  - 중간 실패가 우려되는 잡
  - 잡이 메모리 부족 에러로 실패 (체크포인팅을 쓰면 executor의 메모리를 사용하지 않고 비용과 오류 가능성을 낮출 수 있음 - 체크포인팅 하면 부모 RDD들 다 GC 시키고, 중간 결과만 로드해서 쓰게 되므로)

### 셔플 파일
- 스파크는 셔플하는 동안 디스크(worker node의 로컬 디렉토리)에 데이터를 쓴다. 이 파일들이 '셔플 파일'
- 대개 mapper에 의해 정렬된 각 입력 파티션의 모든 레코드를 포함
- 만일 드라이버 프로그램이 이미 셔플된 RDD를 재사용한다면 셔플 파일들을 써서 셔플 지점까지의 재연산을 하지 않을 수 있다.
  - spark GUI에서 Skipped Stages가 바로 이런 경우

### 어큐뮬레이터 주의 사항
- RDD의 일부가 재연산되어야 하는 상황이라면 스파크는 재연산되는 만큼 어큐뮬레이터에 값을 계속 더할 것이고 재연산된 부분은 값이 곱절로 커질 수 있다.

