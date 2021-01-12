---
title: HIVE QUERY의 OUTPUT이 작은 파일로 쪼개지는 경우 
date: 2015-06-10 23:00:00 +0900
categories: [dev, hive]
tags: [hadoop, hive]
---
MapReduce 처리에 있어서 Input 파일들이 Block 크기 이하로 잘게 쪼개져 있는 경우는 좋지 않다.

MapReduce Job에서 Map Task의 개수는 Input의 Block 개수에 의해 결정이 되는데, Block 크기 이하의 파일들이 많으면 그만큼 많은 Mapper가 생성되고, 각 Mapper는 작은량의 데이터만 처리하게 되기 때문이다.

Hive에서 Query를 수행하는 경우 Output 파일의 개수가 많고 각 파일의 크기가 Block 크기 이하로 작게 생성되는 경우가 있다. 이는 Hive가 Reduce Task의 개수를 결정하는 방식과 관련이 있다. Hive는 전체 Input 파일 용량을 기준으로 1GB (0.14 버전 이후 부터는 256MB)당 1개의 Reducer를 생성한다. (아래 관련 설정 참고)

mapred.reduce.tasks
> * Default Value: -1
> * Added In: Hive 0.1.0
> The default number of reduce tasks per job. Typically set to a prime close to the number of available hosts. Ignored when mapred.job.tracker is “local”. Hadoop set this to 1 by default, whereas Hive uses -1 as its default value. By setting this property to -1, Hive will automatically figure out what should be the number of reducers.

hive.exec.reducers.bytes.per.reducer
> * Default Value: 1,000,000,000 prior to Hive 0.14.0; 256 MB (256,000,000) in Hive 0.14.0 and later 
> * Added In: Hive 0.2.0; default changed in 0.14.0 with [HIVE-7158](https://issues.apache.org/jira/browse/HIVE-7158) (and [HIVE-7917](https://issues.apache.org/jira/browse/HIVE-7917))
> Size per reducer. The default in Hive 0.14.0 and earlier is 1 GB, that is, if the input size is 10 GB then 10 reducers will be used. In Hive 0.14.0 and later the default is 256 MB, that is, if the input size is 1 GB then 4 reducers will be used.

그러므로 30GB를 처리하는 Query를 실행하면 특별히 Reducer의 개수를 지정하지 않는 한 30개의 Reducer가 돌게 된다. MapReduce에서 Output 파일의 개수는 Reducer의 개수에 의해 결정 되므로, 이 Query의 Output 파일 개수는 30개가 된다. 이 때, 각 Reducer에서 처리한 최종 결과물이 작다면 Block 용량보다 작은 크기의 파일들이 30개가 생길 수도 있다.

이런 경우, 최종 Output 파일이 잘게 쪼개지는 것을 막기 위해 다음의 2가지 방법을 사용해볼 수 있다.

* Reduce Task 개수(mapred.reduce.tasks)를 명시하여 Query를 실행
* hive.merge.mapredfiles 옵션을 활성화하여 Query 수행 후, 잘게 나뉜 파일들을 다시 Merge 함
  * Output 파일들의 평균 용량이 hive.merge.smallfiles.avgsize 이하면 Merge 수행
  * Merge를 위한 Map Task가 Stage 마지막에 추가됨
  * Merge한 파일의 Max 크기는 hive.merge.size.per.task 설정을 따름

hive.merge.mapredfiles
> Default Value: false
> Added In: Hive 0.4.0
> Merge small files at the end of a map-reduce job.

hive.mergejob.maponly
> Default Value: true
> Added In: Hive 0.6.0
> Removed In: Hive 0.11.0
> Try to generate a map-only job for merging files if CombineHiveInputFormat is supported. (This configuration property was removed in release 0.11.0.)

hive.merge.size.per.task
> Default Value: 256000000
> Added In: Hive 0.4.0
> Size of merged files at the end of the job.

hive.merge.smallfiles.avgsize
> Default Value: 16000000
> Added In: Hive 0.5.0
> When the average output file size of a job is less than this number, Hive will start an additional map-reduce job to merge the output files into bigger files. This is only done for map-only jobs if hive.merge.mapfiles is true, and for map-reduce jobs if hive.merge.mapredfiles is true.

* 참고
  * <http://blog.cloudera.com/blog/2009/02/the-small-files-problem>
  * <https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties>
