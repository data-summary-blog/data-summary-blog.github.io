---
layout: post
title: "Hadoop 깊게 알아보기 - Hive편"
categories: hadoop
author: "Yongchan Hong"
---
# Hadoop 깊게 알아보기 - Hive편 (1)
Hadoop에 대해서 "조금" 아는것만으로 충분하지 않다. Hadoop Ecosystem을 정리했던 저번보다 더 알아보기 위하여 [빅데이터-하둡, 하이브로 시작하기](https://wikidocs.net/book/2203)를 참고하여 Hadoop 개념에 대해 조금이라도 더 깊게 알아보기로 했다. 이번 시간엔 Hive에 대해 리뷰해보기로 했다.  
Hive란 데이터 웨어하우징용 솔루션으로 테이블과 같은 형태로 HDFS에 저장된 데이터를 정의하고 데이터를 대상으로 쿼리할 수 있는 HiveQL를 제공한다. Hive 1.0은 SQL을 이용한 맵리듀스 처리, 파일 데이터의 논리적 표현을 지원하였다. Hive 2.0은 Spark 지원을 강화하였고, LLAP(Live Long and Process)를 추가하여 핫데이터를 캐쉬하여 빠른속도로 데이터를 처리할 수 있게 되었으며 기본엔진을 Tez로 바꿨다. Hive 3.0에서는 Materialized View를 추가하였고, Kafka-Hive 커넥터가 나왔다. 

## Hive Architecture
![](https://wikidocs.net/images/page/23282/hive-hadoop-architecture.png)  
Hive의 전체적인 구조는 다음과 같이 생겼다.  
- UI  
사용자가 쿼리나 기타 작업을 제출하는 사용자 인터페이스
- Driver  
쿼리를 입력받고 작업을 처리하는 곳. JDBC 인터페이스 API를 제공한다.
- Compiler  
메타 스토어를 참고하여 쿼리 구문을 분석하고 실행계획을 생성하는 곳
- Metastore  
DB, Table, Partition등의 내용을 저장하는 곳  
- Execution Engine 
Compiler에 의해 생성된 실행 계획을 실행하는 곳  

전체적인 흐름을 설명하면, 사용자가 SQL 문을 제출한다면, 드라이버가 컴파일러에 요청하여 Metastore정보를 바탕으로 쿼리문을 적절하게 컴파일한다. 컴파일된 SQL을 Execution Engine으로 실행하면 리소스 매니저에서 자원을 적절히 활용하여 쿼리를 실행하고 결과를 사용자에게 반환한다. 

> beeline이란? HiveServer2에 접속하여 쿼리를 실행하기 위한 도구이다. ./beeline으로 실행한뒤 `!connect jdbc:hive2://{hive ip & port} {user} {password} org.apache.hive.jdbc.HiveDriver` 형식으로 HiveServer2에 접속하고 쿼리할 수 있다.

## Hive Metastore
Hive Metastore는 파일의 물리적인 위치와 데이터에 대한 정보를 보관하고 사용자 요청에 따라 제공하는 곳이다. 메타스토어는 크게 `Embedded`, `Local`, `Remote`의 세가지가 있는데, Embedded는 Derby DB를 사용해 한번에 한명만 접근가능해 테스트 목적으로 쓰인다. Local는 메타 데이터를 외부 RDBMS에 저장하는데 보통 MySQL이 많이 쓰인다. Remote는 메타스토어 자체를 별도의 JVM위에서 구동하게 한다. 우리는 Local Metastore를 사용해 AWS RDS에 저장을 하여 사용하였다. Schema Manager로 스키마 변경시 Metastore내에서 업데이트하도록 하였고, Json을 Parquet로 변경하는 프로세스도 거쳤다. 우리는 별도의 Hive Query Engine 없이 Spark를 필요시 띄워 사용하도록 하였다. 따라서 Querybook을 사용할때는 Spark Thrift Server를 띄워서 사용해야됐다.

> Spark Thrift Server란?  
> Spark Thrift Server는 JDBC/ODBC로 Spark SQL을 날릴 수 있게 해주는 서버이다. k8s에 띄우는건 이걸 참조.. [띄우는 법](https://mightytedkim.tistory.com/m/46)


Table과 Database에 대해서는 2탄에 알아보겠다. 근데 일단 우리는 Metastore를 사용했으므로.. 조금 나중에... 

### Reference
https://wikidocs.net/book/2203  
https://mightytedkim.tistory.com/m/46  