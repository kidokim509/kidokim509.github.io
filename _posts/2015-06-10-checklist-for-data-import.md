---
title: 데이터 적재 체크 리스트
date: 2015-06-10 23:30:00 +0900
categories: [dev, data engineering]
tags: [hadoop, etl, hbase, nosql]
---
데이터 엔지니어링의 시작은 데이터를 저장소에 저장하는 것으로 부터 출발한다.
일회성으로 데이터를 적재하는 것이 아니라 지속적으로 생성되는 데이터를 저장 관리해야 한다면 어떤 것들을 먼저 고려해야 할까?

## 데이터 활용 목적
* 무엇을 분석하기 위한 데이터인가에 따라 데이터 저장 형태(스키마), 저장 시스템, 관리 체계 등이 달라질 수 있다.
* 가장 먼저 해야하는 것은 무엇을 위한 데이터인지 파악하는 것이다.

## 데이터 속성 파악
* 데이터 스키마
* 데이터 사이즈
  * 예) 최초 적재 데이터량, 일일 증가 예측량
* 원천 데이터 위치
  * 데이터 전송, 수집 방식에 영향
* 적재 주기
  * 예) 1일 1회, 매 시간 등
* 보관 주기 (retention) / 백업 여부
  * 예) 1년이 지난 데이터는 사용하지 않음
* 데이터 갱신(update) 여부, 쿼리 방식 / 량
  * Hive, HBase, NoSQL등 저장 시스템 선정 기준의 하나

## 적재 방식 설계
* 소스 / 타깃 시스템 정의 : 데이터 속성에 따라 결정됨. 예) API, FTP, 서버 Local, DBMS, 하둡
  * 소스가 운영 RDBMS일 경우, 데이터 량에 따라 대량 Traffic 발생 고려
    * Sqoop: 기본 Mapper 4개 (= DB Connection 4)
      * direct mode 
        * 예) MySQL, mysqldump 사용
  * 타깃이 운영 RDBMS일 경우, 데이터 량에 따라 대량 Traffic 발생 고려
    * Sqoop
      * non-direct mode
        * 예) MySQL
        * 1 Transaction = 10,000 Row insert & commit
      * direct mode
        * 예) MySQL, mysqlimport 사용 (LOAD INPATH FILE)
* ETL 설계
  * 소스 ~ 타깃 사이의 흐름

## 파일 생성 규칙
* 적재 폴더/파일명 규칙
  * 데이터의 기준 일자를 폴더, 파일명에 반영
  * 하나의 파일 = 하나의 테이블
    하나의 파일 = 당일 적재 분량
    파일 용량이 너무 큰 경우에는 파일 분리, 같은 폴더에 저장
  * 나머지는 비즈니스에 맞게 적용
* 파일 인코딩 (한글 사용 유의) :  UTF-8
* 스키마 모델링 반영

## 스키마 모델링
* 데이터 타입 결정
  * string, int, double, big decimal, timestamp …
  * 우리 플랫폼이 수용할 수 있는 데이터인가?
* 데이터 포맷 결정
  * 우리 플랫폼이 활용할 수 있는 데이터 형태인가?
  * 일자(2015-01-15) –> Timezone 유의!
  * 시간(14:21:30)
  * 금액 표기법 (콤마 사용 여부)
* Hive
  * managed / external
  * partition column
    * 데이터의 기준 일자 혹은 데이터의 분산 기준
  * 데이터 재적재 주기/주체에 따라 partition column을 만들어야 함.
    partition이 재적재 시 Overwrite의 범위를 결정 지음.

