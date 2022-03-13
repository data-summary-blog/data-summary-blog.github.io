---
layout: post
title: "좌충우돌 Spark 배우기 - Spark DataFrame & Spark SQL"
categories: spark
author: "Yongchan Hong"
---

# 좌충우돌 Spark 배우기 - Spark DataFrame & Spark SQL

## Spark SQL이란?  
`Spark SQL`은 정형 데이터 처리를 위한 Spark 모듈이다. Spark SQL을 통해 Spark 내부에서 관계형 처리를 하고, 스키마의 정보를 이용해 자동으로 최적화를 하며, 외부 데이터셋을 사용하기 쉽게 하는 것이 구체적인 목적이다. Spark SQL은 Spark Core에 기반해 Library형태로 지원이 되는데, `SQL`, `DataFrame`, `Dataset`의 3가지 주요 API가 있고, 2개의 백엔드 컴포넌트인 `Catalyst`(쿼리 최적화 엔진)과 `Tungsten`(시리얼라이저 - 용량 최적화)가 있다. 그렇다면 Spark Core와 Spark SQL에는 어떻게 다르고, 어떻게 비슷할까? 먼저, Spark Core에 RDD가 있듯 Spark SQL에는 DataFrame이라는 테이블 데이터 셋이 있다. 쉽게 설명하면 RDD에 스키마가 적용된 것이므로, RDD의 여러 개념들이 계승된다. Spark Core의 Spark Context와 비슷하게 Spark SQL에는 Spark Session이 있다. 다음과 같이 Spark Session을 만들 수 있다.  
```
spark = SparkSession.builder.appName('chan-test').getOrCreate()
``` 
이렇게 SparkSession을 통해 불러오는 데이터는 DataFrame이다. 그렇다면 이제 DataFrame에 대해서 조금 더 알아보자.

## DataFrame Deep Dive  
이전에 설명하였듯, DataFrame은 관계형 데이터셋이다. RDD가 Imperative API를 가졌다면, DataFrame은 Declartive API를 가지고 있다. 또한, DataFrame은 자동으로 최적화가 가능하고 타입이 없다. DataFrame은 RDD를 계승하였기에 다음과 같은 특징들을 가지고 있다.  
- Lazy Evaluation  
- 분산 저장  
- Immutable  

거기에 추가로 다음과 같은 특징들을 가지고 있다.
- Row 객체
- SQL 쿼리 실행 가능
- 스키마 존재, 최적화 가능
- CSV, JSON, Hive를 읽기/변환

### Dataset
분명 RDD, DataFrame, Dataset 크게 세가지 종류가 있다는 것을 기억한적 있을 것이다. 그렇다면 Dataset은 무엇일까? Dataset은 Type이 있는 DataFrame이다. 그러나 Python과 R은 Compile-Type type-safety를 가지고 있기에 필요하지않고, Java/Scala에서 쓰인다.

### RDD vs DataFrame vs Dataset
이 셋을 비교한다면 어떨까? 대부분의 상황에선 DataFrame 및 Dataset이 채택이 되고 있다. MLLib, Spark Streaming 등 다른 Spark Module과도 사용하기 편하고, 개발하기 편하며 무엇보다 자동으로 최적화가 되기 때문이다. RDD의 경우에는 low-level transaction/action을 사용하고 싶다면 사용한다. 
> DataFrame에서 RDD로 바꿀 수는 있다!  
> ```rdd = df.rdd.map(tuple)```을 통해서 바꿀 수는 있지만 자주 쓰이지는 않는다.  

## DataFrame 만들기 & 조작하기
DataFrame은 어떻게 만들고 조작할 수 있을까? DataFrame은 RDD에서 스키마를 정의한 후 변형을 하거나, CSV, JSON 등 Structured 된 Data에서 받아올 수 있다.  
1. RDD로부터 DataFrame만들기  
다음과 같이 Schema를 유추하게 하거나, 직접적으로 정의하거나 두가지 방법이 가능하다.  
```
# 유추하기
df = spark.createDataFrame(rdd)
# 직접 정의하기 
schema = StructType(
    StructField("name", StringType(), True),
    StructField("price", IntegerType(), True)
)
df = spark.createDataFrame(rdd, schema)
```
2. 파일 읽어오기  
CSV, JSON, txt, Parquet 등 다양한 형식의 파일로부터 읽어올 수 있다.
```
# JSON
df = spark.read.json('test.json')
# TXT
df = spark.read.text('test.txt')
# CSV
df = spark.read.csv('test.csv')
# PARQUET
df = spark.read.load('test.parquet)
```
이렇게 받아온 DataFrame을 하나의 데이터베이스 테이블처럼 사용하려면 `createOrReplaceTemporaryView()`로 temporary view를 제작후 사용해야한다.
```
data.createOrReplaceTemporaryView('test')
spark.sql("SELECT * FROM test LIMIT 5)
```
이러한 DataFrame에서는 Hive Query Language와 비슷한 SQL문을 통해 질의를 하거나, 함수를 사용하여 질의를 할 수 있다. 
> DataFrame의 Schema는 어떻게 확인할 수 있을까?  
> 다음과 같은 방법을 통해 확인이 가능하다.  
> 1. dtypes  
> ```df.dtypes```   
> Column 명과 data type 을 list형태로 받을 수 있다.  
> 2. show()  
> ```df.show()```  
> table형태로 데이터를 출력하고, Default로는 첫 20개열만 보여준다. show내부에 첫번째 인자로는 숫자를 받아 보여줄 Row수를 정할 수 있고, 두번째 인자로는 True/False를 넣어 Result가 Truncate되어서 짤려서 보여줄지 정할 수 있다 (Default는 True다).
> 3. printSchema()  
> ```df.printSchema()```
> schema를 tree형태로 볼수 있어, nested된 것을 확인하기 편하다.

### Using Spark SQL
`spark.sql(SQL문)`을 통해서 SQL 쿼리를 질의할 수 있다. 위에서 설명하듯 Hive Query Language와 비슷하게 Select, From, Where, Count, Having, Group By, Order By, Sort By, Distinct, Join등이 모두 사용가능하며, 이에 대해서는 깊게 다루지 않겠다. (일반 SQL과 매우 유사하다)

### Using DataFrame Function
함수일 뿐이지 사실 SQL과 거의 유사하다. Select, Where, Limit, OrderBy, GroupBy, Join이 가능한데, 이해도를 높이기 위해 코드와 함께 첨부하겠다.
```
df.select('name','age').collect() # RDD와 비슷하게 collect같은 action이 필요하다.
df.select(df.name, (df.age + 10).alias('age')).collect() #alias는 컬럼 이름을 변경해준다.

from pyspark.sql import functions as F
df.agg(F.min(df.age)).collect() # Agg는 Aggregate의 약자로 그룹핑 후 데이터를 하나로 합치는 작업이다.
# [Row(min(age)=25)]

df.groupBy(df.name).avg().collect() # groupBy는 지정한 Column을 기준으로 데이터를 그룹핑한다.
# [Row(name='Chan', avg(age) = 25, count = 1), Row(name='Viotolo', avg(age) = 26, count = 1)]

df.join(df2, 'name').select(df.name, df2.height).collect() # name을 기준으로 join을 해준다.

df.select('name').where("country == 'Korea'").orderBy('age') # Korea인 경우에서의 name을 age순으로 order by 해준다.
```
필자는 보통 SQL을 사용하는게 가독성이 좋아 SQL을 자주 사용하기는 하나, 편리함이 우선시 되는 경우에는 DataFrame Function을 쓰기도 한다.

### User Defined Function (UDF)
UDF란 Spark SQL에서 제공하는 함수외에 사용자가 직접 함수를 제작하여 사용하는 것이다. 다음과 같은 과정을 거쳐서 UDF를 사용할 수 있다.
```
from pyspark.sql.functions import udf

# Decorator를 사용하여 Type지정가능
@udf("long")
def squared(n):
    return n*n

spark.udf.register("squared", squared)
spark.sql("SELECT price, squared(price) FROM transactions")
``` 

## Catalyst Optimizer & Tungsten Project
![](https://user-images.githubusercontent.com/12586821/54877777-a72ebe80-4e66-11e9-993c-de5cd4d707d0.jpg)
Catalyst Optimzer와 Tungsten Project는 Spark가 쿼리를 돌리기 위해 사용하는 두가지 엔진이다. 그렇다면 이들은 뭐하는 애들일까?  
Catalyst는 Spark Core 위에 존재하는데, SQL과 DataFrame이 구조가 있는 데이터를 다룰 수 있게 해주는 모듈이다. Catalyst는 `Logical Plan`을 `Physcial Plan`으로 바꿔준다. 그렇다면 Logical Plan이란 무엇일까? Logical Plan이란 수행해야할 모든 Transformation단계를 추상화한 것으로 어떻게 데이터가 변할지 정의하지만 어디서 어떻게 변할지 정의하진 않는다. Physical Plan은 어떤 Cluster에서 실행될지 정의한다. 다음과 같은 과정을 거치게 된다.  
1. Analysis  
DataFrame의 Relation을 계산하고, 컬럼 타입과 이름을 확인한다.  
이를 통해 `Unresolved Logical Plan`을 `Logical Plan`으로 바꾼다.
2. Logical Plan Optimizations  
다음과 같은 Rule Based Optimization을 적용한다.
- Constant Folding  
상수로 표현된 표현식을 Compile Time에 계산한다 (X runtime)
- Predicate Pushdown  
Join & Filter -> Filter & Join으로 변경한다.  
Subquery 밖에 있는 where절을 안으로 밀어넣는다.  
- Projection Pruning   
연산에 필요한 칼럼만 가져온다.  
이 과정을 통해 `Logical Plan`에서 `Optimized Logical Plan`으로 변경이 된다.  
3. Physical Planning  
Optimized Logical Plan으로부터 Spark Exeuction Engine에서 실행될 수 있는 한 개 이상의 `Physical Plan`을 생성하게 된다. 여러개의 Physical Plan으로부터 Cost Model을 바탕으로 하나의 Physical Plan을 선정하게 된다.
4. Code Generation - Tungsten  
최적화된 Physical Plan을 Java ByteCode로 변환해준다. 

> **Plan 확인하기**  
> `spark.sql(query).explain(True)`를 넣어주면, Parsed Logical Plan, Analyzed Logical Plan, Optimized Logical Plan, Physical Plan을 확인할 수 있다. Parsed Logical Plan은 코드 그대로를 바꾼것으로 Unresolved Logical Plan이다. Analyzed Logical Plan은 analysis 과정을 거친 Logical Plan을 볼 수 있다. Optimzed Logical Plan에서는 제목 그대로 Optimized Logical Plan을 확인할 수 있다. Physical Plan에서는 최종 Physical Plan을 볼 수 있다. 

아까 Code Generation 옆에 Tungsten이라 붙은 것을 확인할 수 있다. Tungsten Project는 Catalyst Optimizer가 최적화된 Physical Planning까지 마치고 나면 이를 받아서 ByteCode로 변환해주는데, 스파크 엔진의 성능 향상이 목적이기에 메모리 관리 최적화와 캐시 활용 연산을 하여 코드 생성을 한다. 

여기까지 Spark SQL & DataFrame에 대해서 알아보았다. 추후에 MLLib이나 Spark Streaming에 대해서도 포스팅 해보겠다. 

### Reference
[실시간 빅데이터 처리를 위한 Spark & Flink Online](https://storage.googleapis.com/static.fastcampus.co.kr/prod/uploads/202112/185209-24/[%ED%8C%A8%EC%8A%A4%ED%8A%B8%EC%BA%A0%ED%8D%BC%EC%8A%A4]-%EA%B5%90%EC%9C%A1%EA%B3%BC%EC%A0%95%EC%86%8C%EA%B0%9C%EC%84%9C-%EC%98%AC%EC%9D%B8%EC%9B%90-%ED%8C%A8%ED%82%A4%EC%A7%80---%EC%8B%A4%EC%8B%9C%EA%B0%84-%EB%B9%85%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%B2%98%EB%A6%AC%EB%A5%BC-%EC%9C%84%ED%95%9C-spark---flink.pdf)  
[빅데이터 - 스칼라(scala), 스파크(spark)로 시작하기](https://wikidocs.net/book/2350)  
https://jjaesang.github.io/high-performance-spark/2019/03/22/dataframe_dataset.html