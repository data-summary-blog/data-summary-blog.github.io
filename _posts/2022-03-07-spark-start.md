---
layout: post
title: "좌충우돌 Spark 배우기 - Spark 설치하기 & 시작하기"
categories: spark
author: "Yongchan Hong"
---

# 좌충우돌 Spark 배우기 - Spark 설치하기 & 시작하기

## Spark를 배우자
최근에 Spark에 대해서 강의와 자료들을 찾아가며 제대로 공부하기로 결심하였다!
아무래도 Data Engineering에 핵이 되는 스택이자 빅데이터를 위해선 필요하기 때문임이 그 이유이다.  
이전에는 Spark SQL 및 RDD에서 관하여 가장 기본적인 사용법과 개념만 알았다면, 이 기회에 조금 더 깊게 파보고 Spark의 원리 및 다양한 사용법에 대해 공부해 보려한다.  
가장 기초적인 자료는 [실시간 빅데이터 처리를 위한 Spark & Flink Online](https://storage.googleapis.com/static.fastcampus.co.kr/prod/uploads/202112/185209-24/[%ED%8C%A8%EC%8A%A4%ED%8A%B8%EC%BA%A0%ED%8D%BC%EC%8A%A4]-%EA%B5%90%EC%9C%A1%EA%B3%BC%EC%A0%95%EC%86%8C%EA%B0%9C%EC%84%9C-%EC%98%AC%EC%9D%B8%EC%9B%90-%ED%8C%A8%ED%82%A4%EC%A7%80---%EC%8B%A4%EC%8B%9C%EA%B0%84-%EB%B9%85%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%B2%98%EB%A6%AC%EB%A5%BC-%EC%9C%84%ED%95%9C-spark---flink.pdf) 강의를 수강하며 배웠고, 또한 Spark 공식 문서 및 다른 자료들을 참조하였다. 

## Spark란?
Spark란 아파치 재단에서 오픈소스로 관리되고 있는 In-memory 기반의 대용량 데이터 고속 처리 엔진이다. 또한, Spark는 Hadoop의 연산엔진인 MapReduce를 대체하는 프로젝트이다. Spark는 MapReduce와 달리 디스크 대신 메모리에 중간 결과를 저장하고, 데이터를 쪼개서 여러 노드의 메모리에서 처리하기에 MapReduce보다 메모리 상에선 100배, 디스크상에선 10배 더 빠르게 처리한다. 또한 Spark의 특징으로는 다양한 언어 지원이 되고 (Python, Java, Scala, R, SQL), Yarn 등 다양한 클러스터에 동작가능하고 다양한 파일 포맷을 지원한다. 또한, Lazy Evaluation을 통해 테스크를 정의할때는 연산을 하지 않다가 결과 필요할때 최적화한 것을 연산한다 (DAG의 형식으로 최적화한다). Spark는 다음과 같은 컴포넌트들로 구성되어 있다.
- Spark Core  
메인 컴포넌트로, 작업 스케줄링, 메모리 관리 등의 기능을 제공하고, RDD, Dataset, Dataframe을 이용한 스파크 연산을 처리한다.
- Spark SQL  
SQL을 이용하여 RDD, Dataset, Dataframe 작업을 처리한다.
- Spark Streaming  
실시간 데이터 스트림을 처리한다.
- MLlib  
머신러닝 기능을 제공한다.
- GraphX  
그래프 프로세싱을 가능하게 해준다.

## Spark Cluster 구조
![](https://wikidocs.net/images/page/26630/cluster-overview.png)  
Spark Application은 다음과 같이 **Driver Program**, **Worker Node**, **Cluster Manager**로 이루어져 있다.  
- **Driver Program**  
Driver Program는 Python, Java, Scala 등의 Script로 Task를 만들고 정의한다. 이러한 Task는 Cluster Manager로 전달이 된다.이에 대해서 깊게 설명을 하자면, Driver Program는 main 함수를 실행하고 Spark Context 객체를 생성한다 (또한 이것을 통해 RDD를 생성한다). 이러한 SparkContext는 Cluster Manager로부터 Executor실행을 위한 리소스를 요청한다. Transformation, Action등을 저장하고, Worker Node에게 전송하는 역할도 하는데, Application을 Task단위로 나누어 Executor에게 전송한다.
- **Cluster Manager**  
Cluster Manager는 Spark Context로부터 요청이 오면 리소스를 할당하고 Worker Node (Executor)를 실행한다. 유명한 Cluster Manager로는 YARN이나 Mesos가 있다.  
- **Worker Node**  
Worker Node는 Executor를 포함하며 실제 작업이 이뤄지는 곳이다. Executor는 실제로 연산(Task)을 실행하고, 연산 결과를 Driver Program에 전송한다. Task는 Executor가 하는 실제 작업으로, Executor의 Cache를 공유하여 작업 속도를 높일 수 있다.  


## Spark 설치하기  
Spark를 Python 환경에서 사용하기 위해선 다음과 같은 것들이 필요하다.  
- Python
- Spark
- Java/Scala
- PySpark  

맥의 경우에는 Java/Scala를 Brew로 설치할 수 있으며, Spark는 `brew install apache-spark`로 설치 가능하다. Python에서 Spark를 사용하기 위하여 쓰는 PySpark는 `pip install pyspark`를 통해 설치 가능하다. Terminal에 pyspark를 치면 spark를 실행시킬수 있다.

> 맥의 경우에는, Local PySpark에서 `Service 'sparkDriver' could not bind on a random free port`라고 뜨며 잘 안되는 경우가 있다. 그런 경우 SPARK_LOCAL_IP가 localhost로 잘 세팅이 되어있지 않은 경우로, pyspark를 실행할 때는 다음과 같이 해결하거나, `pyspark -c spark.driver.bindAddress=127.0.0.1`, sparkConf에서 다음 문을 추가하면 된다. `.set('spark.driver.bindAddress', '127.0.0.1')`.

spark를 시행하면 다음과 같은 곳 (`http://127.0.0.1:4040`) 에서 모니터링 결과를 확인할 수 있다 (로컬 기준)    
![](https://i.stack.imgur.com/xjQwI.png)

## Spark Context 시작하기
SparkContext는 Spark Cluster와 연결시켜주는 객체로, Driver Program에서 job을 전달 시키는 등의 action을 위해 필요하다. 프로그램당 하나씩 만들 수 있다.  
다음과 같은 코드 형태로 만들 수 있다. 
```
from pyspark import SparkConf, SparkContext
conf = SparkConf().setMaster("local").setAppName("Chan_test")
sc = SparkContext.getOrCreate(conf=conf)
```
설명을 하면, setMaster는 개발용 로컬환경을 쓰겠다고 정의하고, setAppName은 Spark UI에서 확인할 수 있는 Application의 이름을 정의한다. SparkConf()는 전체적인 Context의 Configuration을 정의할 수 있는 곳이다. SparkContext는 3번째 줄과 같이 정의를 하고, getOrCreate을 통해서 Driver Program이 해당 Context가 존재하면 get을 하고 존재하지 않으면 새로 생성해주는 역할을 한다.


다음 챕터부터는 RDD에 대해 깊게 알아보기로 하며 사용 예시 또한 알아보기로 한다.
### Reference
https://wikidocs.net/26630  
[실시간 빅데이터 처리를 위한 Spark & Flink Online](https://storage.googleapis.com/static.fastcampus.co.kr/prod/uploads/202112/185209-24/[%ED%8C%A8%EC%8A%A4%ED%8A%B8%EC%BA%A0%ED%8D%BC%EC%8A%A4]-%EA%B5%90%EC%9C%A1%EA%B3%BC%EC%A0%95%EC%86%8C%EA%B0%9C%EC%84%9C-%EC%98%AC%EC%9D%B8%EC%9B%90-%ED%8C%A8%ED%82%A4%EC%A7%80---%EC%8B%A4%EC%8B%9C%EA%B0%84-%EB%B9%85%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%B2%98%EB%A6%AC%EB%A5%BC-%EC%9C%84%ED%95%9C-spark---flink.pdf)