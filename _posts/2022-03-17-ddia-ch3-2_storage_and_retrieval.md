---
layout: post
title: "DDIA Study -  Chapter 3 : Storage and Retrieval (KOR) (2)"
categories: DDIA
author: "Yongchan Hong"
---


# Chapter 3 : Storage and Retrieval (2)

> In this DDIA category, I will post a summary of Designing Data Intensive Applications (DDIA) from chapter 1 to 12.

In this chapter, we learn about storage and retrieval.

## OLAP AND OLTP
- `Transcation`: 데이터베이스 상태를 변화시키기 위해 수행하는 작업이자 읽기/그룹의 논리 단위 형태
    - Transaction 처리는 ACID속성을 가질 필요 없다! 트랜잭션 처리는 지연시간이 낮은 읽기와 쓰기를 가능하게 한다는 뜻이다.
- `OLTP` = Online Transaction Processing
    - 사용자 입력을 바탕으로 삽입/갱신
    - 전체 칼럼 접근 필요
    - 빠르게 처리해야함
- `OLAP` = Online Analytic Processing
    - 분석을 위해 사용함
    - 많은 수의 레코드를 읽고 일부 칼럼을 읽어 집계함
- `OLTP` vs `OLAP`  
    |   특성  |   OLTP   |    OLAP    |
    |---|---|---|
    |읽기|적은 수의 레코드, 키 기준으로 전체 칼럼을 많이 가져옴|많은 레코드에 대한 aggregation, 일부 칼럼 접근|
    |쓰기|사용자가 입력, 낮은 지연시간 필요|ETL 또는 스트리밍|
    |사용자|많은 일반 유저/소비자|내부 분석가|
    |데이터 표현|데이터의 최신상태|이벤트의 이력|
    |데이터양|적음|많음|
    |병목|디스크탐색 (키의 데이터를 찾아야함) |디스크 대역폭 (짧으시간에 여러 레코드 스캔해야함)|
- Database, Data Warehouse, Data Mart, Data Lake


## Data Warehouse
- 분석가들의 OLTP 작업에 영향을 주지않고 마음껏 질의 할 수 있는 데이터베이스
- OLTP Database에서 ETL을 통해 적재함
- 분석 접근에 맞춰서 최적화 시킬수 있음 (색인이 그다지 유용하지않음)
- 데이터모델로는 관계형 모델 사용
- Redshift, Hive, Spark SQL, Impala, Presto, Drill 등
- 분석용 스키마로는 `Star Schema`나 `Snowflake Schema` 사용
    - Star Schema
        - 스키마 중심에 Fact Table을 기준으로 여러 Dimension Table들이 퍼져나감
        - Dimension Table들은 외래키를 참조함
        - Fact Table의 각 로우는 이벤트를 나타냄.
        - Dimension Table의 Dimension은 육하원칙을 나타냄 (누가, 언제, 어디서, 무엇을, 어떻게, 왜)
    - Snowflake Schema
        - Star Schema에서 중복을 줄이기 위함
        - Dimension Table에서 더 하위차원으로 세부화됨
        - Join이 매우 많고, 작업이 복잡할 수 있어 덜 선호됨

## Column-Oriented Database
- OLAP의 사실 테이블에 접근하는 경우 특정 칼럼만 접근하는 경우가 많음.
- 일반 로우-지향 저장소를 사용하게 되면 모든 로우를 메모리로 적재한 다음 필터링을 해야한다.
- `칼럼 지향 저장소`: 각 칼럼별로 모든 값을 함께 저장함.
    - Parquet: 문서 데이터 모델을 지원하는 칼럼 저장소 형식
- `칼럼 지향 저장소의 각 로우는 모두 같은 순서이다`
- 칼럼 압축
    - 각 칼럼 값별로 비트맵 부호화
        - product_id 10에 대해서 로우별로 일치/불일치에 대해 1, 0 으로 나타냄
        - 0이 너무 많으면 run-length 부호화로 더 줄임: 0이 1개, 1이 3개, 나머지는 0
    - 비트맵 OR/AND로 Where문 연산 가능
- 칼럼패밀리와의 차이: 칼럼 패밀리는 메타데이터만 기입해놓은걸로 각 ROW마다 다른 칼럼이 사용가능하다. (HBase, Cassandra). 고로 칼럼 패밀리는 굳이 칼럼 압축을 사용하지않고 대부분 로우 지향이다.
- 칼럼 지향 저장소/칼럼 압축을 이용하여 L1캐시에 칼럼의 더 많은 로우를 저장하고 비트 AND/OR연산자로 바로 연산할 수 있게 설계 가능하다 (vectorized processing)
- 정렬해야하는 칼럼을 선택하면 각 로우가 모두 같은 순서라는 걸 이용해 재구성할 수 있다. 정렬을 하면 칼럼 압축에 도움이 된다 (같은 값이 연속해서 길게 반복될 수 있음으로)
- C-Store는 복제 데이터를 다양한 방식으로 정렬해서 저장하여 질의할때 적절한 버전을 찾아 질의하게 된다.
    - 로우의 2차 색인과 비슷하나 로우의 2차 색인은 포인터만 포함하는 반면 여기선 값을 포함하는 칼럼만 있다.
- 칼럼 지향 저장소에 쓰기를 할때는 B 트리 접근방식은 압축된 칼럼에서 불가하고 (로우를 삽입하기 위해 모든 칼럼을 갱신해야함 으로) LSM트리 방식으로 가능하다.
- 칼럼 지향 저장소 예시: Redshift, MariaDB

## 집계: 데이터큐브와 구체화 뷰
- Count, Sum, AVG 등 집계를 위해 질의에 자주 쓰이는 합이나 카운트를 캐시해두자!
- `Materialized View`
    - 가상 뷰와는 그저 질의를 작성하는 단축본이지만 구체화 뷰는 실제 복사본
    - 원본 데이터 변경시 갱신해야함
- `Data Cube`
    - Materialized View의 사례
    - n차원으로 되어있는 외래키를 1차원으로 축소함
        - Example: date, product로 되어있는 2차원을 축소하여 price의 집계값 등도 얻을 수 있음
    - 특정 질의를 계산하여 매우 빠름
    - 유연성은 없음
