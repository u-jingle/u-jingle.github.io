---
title: "Spark Cluster 기본 개념"
layout: post
author: "JinggleJinggle"
header-style: text
tags:
  - spark
---

대량의 데이터를 다른 저장소로 옮길거나 새로운 포맷으로 변환이 필요한 경우 Apache Spark를 이용하고 있다.

스파크 사용자라면 누구나 한번쯤 읽는다는 '스파크 완벽 가이드' 책도 보고 직접 구현해 돌려도 봤지만,, 조금만 사용안하면 다시 개념을 잊어 버리고 말았다...

Spark Cluster 사용시 내가 자주 헷갈리는 기본 개념을 정리 해 보려고 한다.

# 동작방식

Spark Cluster는 크게 아래와 같이 동작한다.

![/img/in-post/2021-03-07/Spark-4.jpg](/img/in-post/2021-03-07/Spark-4.jpg){: width="95%"} 

1. Action을 호출하면 Job이 실행된다. (Job이 실행되면 host:4040 에서 Spark UI 확인이 가능하다)
하나의 Action은 하나의 Job. 
2.  Shuffle 작업 수에 따라 Stage가 발생된다.
3. 각 Stage 작업의 Partition수에 따라 Task가 발생된다.
Shuffle Partition은 기본 200개 이기 때문에 200개의 Task가 발생됨(spark.sql.shuffle.partitions 속성으로 변경 가능)
4. Executor는 한번에 하나의 Partition을 처리하며 Executor는 다수의 core로 구성된다.

# Partition

> 모든 Executor가 병렬로 작업을 수행할 수 있도록 데이터를 분할하는 단위이다.  
클러스터의 물리적 머신에 존재하는 로우의 집합이다.

### Executor와 Partition의 관계

하나의 Executor는 한번에 하나의 Partition을 처리한다.

Executor가 n개이고 Partitiondl 1개인 경우 병렬성은 1이되고, Executor가 1개이고 Partitiondl n개인 경우 또한 병렬성은 1이다.

![/img/in-post/2021-03-07/Spark-5.jpg](/img/in-post/2021-03-07/Spark-5.jpg){: width="90%"} 

### 병렬로 데이터 읽기

Spark에서 다수의 파일이 존재하는 폴더를 읽을때 폴더의 개별파일은 Dataframe의 Partition이 된다.

따라서 사용가능한 Executor를 이용해 병렬로 파일을 읽을 수 있다.(Executor를 넘어가는 파일은 대기)

아래와 같이 하나의 파일(Partition)을 여러 Executor가 분할해 읽을 수 없다.

![/img/in-post/2021-03-07/Spark-6.jpg](/img/in-post/2021-03-07/Spark-6.jpg){: width="90%"} 

### 병렬로 데이터 쓰기

파일이나 데이터의 수는 데이터를 쓰는 시점에 Dataframe이 가진 Partition 수에 따라 달라진다.

기본적으로 데이터 Partition당 하나의 파일이 만들어 진다.

- 파일당 레코드 수를 지정해 Partitioning하는 방법 : df.write.option("maxRecordsPerFile", 5000)

# Partition을 통한 성능개선 방법

### 파일포맷

분할가능한 파일포맷을 사용하면 여러 Task가 파일의 서로 다른 부분을 동시에 읽을 수 있다.

Json과 같은 분할 불가능한 포맷을 읽을 경우 하나의 코어에서 전체 파일을 읽어야 하므로 병렬성이 없다.

### 압축포맷

zip이나 tar 압축은 분할 불가능하므로 압축내부에 몇개의 파일이 있어도 하나의 코어를 사용한다.

그렇기 때문에 gzip, bzip2, lz4와 같은 분할 가능 압축 포맷을 사용할 것을 권장한다.

### Partition 개수

파티션 수가 너무 적으면 소수의 노드만 작업을 수행하기 때문에 데이터 치우침 현상이 발생한다.

또한 파티션 개수가 너무 많으면 많은 Task를 실행시켜야 하므로 부하가 발생한다.

- 처리해야할 데이터 양이 많은 경우, Shuffle Partition 개수 설정시 클러스터의 cpu 코어당 최소 2~3개의 Task를 할당하는 것이 좋다. (spark.sql.shuffle.partitions)