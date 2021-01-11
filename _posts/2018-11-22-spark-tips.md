---
title:   spark tips
date:    2018-11-22 01:34:00 +0900
categories: [dev, spark]
tags: [spark]
---

## Spark UnitTest 공통 모듈 만들기
- 만들려는 것
  - 로컬에서 하둡 클러스터 연결 없이 Spark core, Spark sql 모두 테스트 하기
  - File은 물론, Hive Table로 부터의 DataFrame 로드도 모두 로컬에서 가능하게 하기
    - Hive Table을 Sampling해서 파일로 Export해서 Project의 리소스로 추가
  - RDD나 DataFrame, DataSet에 대한 비교로 Assert 체크 하기
    - Holden Karau의 https://github.com/holdenk/spark-testing-base를 활용하자

1. Spark Context, Session 구성
- master: local
  - 이 상태에서는 로컬에 하둡 Client가 구성되어있어도 HDFS Cluster를 바라보지 않는다.
- SparkSession 만들 때 enableHiveSupport는 하지 않는다.
  - 로컬이든 원격이든 어딘가에 있을 Hive Metastore와 연결하지 않고 프로젝트 코드 안에서 작업을 마치기 위함
  - 프로젝트 코드 안에 Hive용 mini cluster를 갖추거나, 로컬에 아예 개발용으로 하둡, Hive 클러스터를 구성하면 enable도 가능
- SparkSession의 config로 "spark.sql.warehouse.dir"을 프로젝트 안의 resource 폴더 중 하나로 지정

2. Hive에서 파일 추출하기
- 처음에는 orc 포맷으로된 테이블에서 파일을 꺼내어 사용
  - spark.sql.warehouse.dir 아래의 aaa.db/my_table 과 같은 hive path 구조에 맞춰 넣음
  - 코드에서 Spark SQL의 CREATE TABLE my_table (...) USING ORC; 쿼리로 테이블 생성하고
  - spark.sql("select * from my_table")로 쿼리 실행하자 스키마 정보가 틀리다는, xxx 컬럼을 찾을 수 없다는 에러 뜸
    - spark.read.format("orc").load(path)와 같은 식으로 파일을 DataFrame으로는 잘 읽었으나 
  - orc dump로 hive에서 추출한 파일을 열어보니 컬럼이 모두 _col0, _col1, _col2... 과 같은 형식으로 들어있음
    - [HIVE-4243](https://issues.apache.org/jira/browse/HIVE-4243)
- 여러차례 다른 시도 끝에 text 파일 포맷 사용하기로
  - hive -e "INSERT OVERWRITE LOCAL DIRECTORY 'path' STORED AS TEXTFILE SELECT * FROM my_table"로 파일 추출

3. 추출한 파일을 Spark SQL로 쿼리하기
- Spark SQL 테이블 생성
  - CREATE TABLE my_talbe (...) USING OPTIONS (header='false', delimiter='\001', nullValue='\\\\N')
- Spark SQL로 SELECT 및 UDF 쿼리 실행
  - spark.sql("SELECT * FROM my_table LIMIT 10").show()
  - 정상 실행 확인!
  
