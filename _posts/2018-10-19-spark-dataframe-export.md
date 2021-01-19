---
title:   spark dataframe을 csv로 export하기
date:    2018-10-19 16:50:00 +0900
categories: [dev, spark]
tags: [spark]
permalink: /spark/spark-dataframe-export/
---

## spark dataframe을 csv로 export하기
### jupyter, sparkmagic kernel 기준

```shell
%%configure -f
{"jars": ["/user/olaf.kido/spark-csv_2.10-1.5.0.jar", "/user/olaf.kido/commons-csv-1.6.jar"]}

```

```scala
# Spark 1.6이라면
sqlContext = HiveContext(sc)
```

```scala
df = sqlContext \
.sql("select * from source_table") \
.coalesce(1)

# HDFS에 저장
df\
.write\
.mode("overwrite")\
.format("com.databricks.spark.csv")\
.save("/user/kidokim509/data.csv")

# pySpark이고 DN에 pandas가 설치 되어있다면
df.\
toPandas().\
to_csv("data.csv")
```
