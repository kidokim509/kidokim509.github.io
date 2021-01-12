---
title: APACHE FLUME VS. LOGSTASH
date: 2015-06-20 23:00:00 +0900
categories: [dev, logstash]
tags: [hadoop, flume, logstash]
---
로그를 Tailing해서 Kafka로 전송하기 위해서 Apache Flume과 Logstash를 검토하다가 Logstash로 결정.

일단 요구사항은 다음과 같았다.
* Source
  * 로그 파일을 Tailing 해야 함. (즉, 계속 Write하고 있는 파일을 읽어들여야 함)
  * 특정 디렉토리 안에 있는 복수 개의 로그 파일을 읽어야 함.
  * 수집기가 의도적 혹은 장애로 중지되어서 재기동 하는 경우에 파일을 마지막에 읽었던 지점부터 읽기 시작해야 함.
* Target
  * 로그를 Kafka로 전송해야 함.
  * Kafka Partition에 로그를 Round Robin으로 배분할 수 있어야 함.

과정은 생략하고, 결론만 정리해보면 Flume은 파일 Tailing하는 목적으로는 부족한 면이 많다. Flume-NG에서는 TailSource를 지원하지 않기 때문에 Exec Source를 사용하여 Tail 명령이나 since 명령을 실행하는 식으로 사용을 해야 하는데, 이런 경우 파일 읽기를 담당하는 프로세스와 Flume 프로세스가 따로이기 때문에 파일을 읽어 전송하기까지의 과정을 Transactional하게 처리할 수 없다. Flume의 User Guide의 Exec Source 설명부에도 아래와 같은 내용이 주의사항으로 나와있다.

> Warning<br>
> The problem with ExecSource and other asynchronous sources is that the source can not guarantee that if there is a failure to put the event into the Channel the client knows about it. In such cases, the data will be lost. As a for instance, one of the most commonly requested features is the tail -F [file]-like use case where an application writes to a log file on disk and Flume tails the file, sending each line as an event. While this is possible, there’s an obvious problem; what happens if the channel fills up and Flume can’t send an event? Flume has no way of indicating to the application writing the log file that it needs to retain the log or that the event hasn’t been sent, for some reason. If this doesn’t make sense, you need only know this: Your application can never guarantee data has been received when using a unidirectional asynchronous interface such as ExecSource! As an extension of this warning – and to be completely clear – there is absolutely zero guarantee of event delivery when using this source. For stronger reliability guarantees, consider the Spooling Directory Source or direct integration with Flume via the SDK.

만일 장애시 데이터의 유실을 허용해도 되는 상황인데 Flume을 사용하고 싶다면 상관 없겠으나, 그렇지 않다고 하면 일단 파일을 Tailing하여 수집하는 경우에는 Flume은 불합격이다. (장애가 난 시점부터 다시 Tail을 재개하는 시점까지의 파일 내용은 수집을 하지 못함. 또는 해당 파일 전체를 다시 수집해야 함.)

Logstash는 복수개의 파일을 Tailing하여 수집하는 것이 가능하다. (File Input) 게다가 파일을 읽으면서 마지막 읽은 지점을 sincedb라 부르는 파일에 저장하기 때문에 Logstash를 중단하였다가 재개하는 경우, 최종으로 읽었던 지점 이후로부터 수집이 가능하다. 최근에 릴리즈된 1.5.0 GA부터는 Kafka Output을 지원하기 때문에 요구사항에도 적합했다.

Logstash 프로세스 다운의 경우를 테스트 해본 결과 다음과 같이 작동하였다.
* Graceful shutdown and start<br>
  shutdown 당시 읽었던 이후부터 파일을 읽기 시작함. 파일 내용 유실 없음.
* Force shutdown and start<br>
  장애에 의한 혹은 강제 종료 (eg. kill -9)되는 경우 현재 파일 offset을 미처 sincedb에 기록하지 못하고 프로세스가 죽는 경우가 발생한다. 이때엔 수집을 재개한 경우 기전송한 부분을 다시 읽게되어 전송 메시지에 중복이 발생하게 된다.<br>
  이에 대한 대비로, 수집한 데이터에 대한 중복 제거 배치 작업을 돌리거나, 데이터 활용 시 중복에 대한 예외 처리가 필요하다.

이 외에 Kafka로 메시지를 전송할 때에 Partitioning Key 지정을 잘 해줘야 하는데 앞 선 포스트에서 이야기 한 것과 같이 Flume은 별도의 Interceptor를 하나 개발하여 Partitioning Key를 지정해주어야 한다. (아마 누군가 contribute를 해서 앞으로는 개선될 것 같기는 함.) 그러나 Logstash에서는 Kafka Output 설정 시 partitioning_key_format (키로 사용할 값 – 파일 내용에서 파싱한 값이나 timestamp 값으로 사용가능), partitioner_class 등을 설정할 수 있다. 테스트에서는 각 메시지의 @timestamp 값을 Partition Key로 사용하였는데 고르게 메시지 분배가 일어났다.

* 참고: [Logstash Processing Pipeline](https://www.elastic.co/guide/en/logstash/current/pipeline.html)
