---
title:   jupyter notebook tips
date:    2018-10-18 15:18:00
categories: [dev, jupyter]
tags: [jupyter, notebook]
---

## SparkMagic
### spark add jars
```shell
%%configure -f
{"jars": ["/user/olaf.kido/spark-csv_2.10-1.5.0.jar", "/user/olaf.kido/commons-csv-1.6.jar"]}
```
첫번째 cell에서 %%configure 명령을 보내면 새로운 Spark Session이 시작됨.
jar 파일은 hdfs의 path이다.
자주쓰는 jar들은 livy server의 $LIVY_HOME/rsc_jars 폴더에 복사해두어도 됨. (확인 필요)
