---
layout: post
title: "DBT Review"
categories: dbt
author: "Yongchan Hong"
---
# DBT Review
## What is DBT?
DBT (Data Build Tool)는 ELT 구조에서 Trnasform을 담당하고 있는 tool이다. 
![](https://blog.getdbt.com/content/images/downloaded_images/What--exactly--is-dbt-/1-BogoeTTK1OXFU1hPfUyCFw.png)  
다음과 같이 DBT는 Data Loaders로 부터 Raw Data를 받고, 이를 Transform 시켜준뒤 다른 데이터 컨슈머나 BI Tool로 연결해주는 역할을 한다. 사실 DBT는 그렇게 복잡하지않다. 코드를 받고, SQL로 컴파일 시켜준뒤, Database에 구동을 시켜준다. DBT는 먼저 compiler와 runner로 이루어져있는데, compiler는 dbt 코드 (SQL, YAML 파일)을 컴파일하여 raw SQL로 만들어주고, runner는 이를 Database에 구동하고 테이블을 생성하는 등 한다. 참고로 Jinja Templating이 가능하며, 의존성을 가진 모델을 파악하여 DAG를 작성한다. DBT는 pip install로 쉽게 설치하여 사용이 가능하다.

DBT에 대해 이해하기 위해선 `Project`, `Source`, `Materialization`, `Model`을 이해해야 한다.

- Project  
하나의 DBT Project로서, SQL파일과 YAML 파일로 이루어진다. `dbt init [project name]`으로 생성 가능하며, dbt_project.yml 밑에서 configuration을 설정할 수 있다.

- Model  
Model은 Select Statement이며, Jinja Template을 사용할 수 있다. Model은 Models폴더 밑에 위치하는 것이 가능하며, `dbt run`을 하게 되면 해당 테이블을 Database에 생성하게 된다.

> Small Tip: dbt run시 --full-refresh를 주면 incremental model인 경우 drop하고 rebuild한다.

- Materialzation  
Materialization은 해당 Query를 바탕으로 어떻게 빌드 할지 정해주는 것이다. 크게 4종류가 있다.
    - view: Database에 View로 만들어놓는다. (Default)
    - table: Database에 Table로 만들어놓는다.
    - incremental: Table을 만들되 신규 데이터를 insert/update. 진짜 유용하다.
    - ephemeral: Database에 만들지 않고 다른 모델에서 dependency로 사용하도록 한다. 이것도 자주 쓴다.

- Source  
Source를 이용해 source table을 가져올 수 있고, 이를 통해 lineage를 관리할 수 있다.

## Why DBT?
그래서 왜 DBT를 이용해야할까? 다음과 같은 이유들이 있다.
1. Data를 하나의 Select Statement로 변환할 수 있다.
2. ref와 source를 통해 lineage를 관리하기가 매우 쉽다.
3. Staging Model을 쉽게 여러 곳에서 사용 가능하다.
4. 긴 쿼리를 여러개로 나눠서 관리할 수 있다.
5. DBT Test를 통해 Data Quality Check가 가능하다.
6. 오픈소스이고, 커뮤니티가 크다.

## How to use DBT
yaml파일에 관한 정보는 제외하고, 모델을 생성하는 sql파일은 대략 이렇게 제작할 수 있다.
```
{{ config(
    materialized='incremental',
    cluster_by=['datetime'],
) }}

WITH SAMPLE AS (
	SELECT
        datetime,
        value,
        example
    FROM
        {{ ref('example_model') }}
)

SELECT
    *
FROM
    SAMPLE
```

### Reference
https://blog.getdbt.com/what-exactly-is-dbt/  
https://docs.getdbt.com/tutorial/setting-up  
https://tommybebe.github.io/2021/07/09/dbt-101/