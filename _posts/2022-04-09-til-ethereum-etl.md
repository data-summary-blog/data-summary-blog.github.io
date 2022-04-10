---
layout: post
title: "TIL: Ethereum ETL"
categories: til
author: "Yongchan Hong"
---
# TIL: Ethereum ETL

블록체인에 대해 찾아보다보니 Ethereum ETL에 대해서 알게 되었다. 필자는 해당 [BigQuery 글](https://cloud.google.com/blog/products/data-analytics/ethereum-bigquery-how-we-built-dataset)과 [Medium 글](https://evgemedvedev.medium.com/exporting-and-analyzing-ethereum-blockchain-f5353414a94e)을 참고하여 작성하였다.

[Ethereum ETL](https://github.com/blockchain-etl/ethereum-etl)은 말 그대로 블록체인 데이터를 CSV나 RDBMS에 적재 가능한 형태로 바꾸어주는 역할을 한다. 

![](https://storage.googleapis.com/gweb-cloudblog-publish/images/image2_PTI1kuT.max-1400x1400.png)

BigQuery에서 제시한 전체적인 ETL과정은 다음과 같다. 먼저 전체적인 과정은 Airflow를 사용하여 주기적으로 Batch가 이루어진다. 처음 Ethereum에서 Export하는 과정은 위에 제시됐던 Ethereum ETL Github을 사용이 되었으며, Ethereum node를 JSON RPC를 이용하여 연결이 되어 정보를 가져온다. 다음과 같이 날짜별로 block의 range를 알 수 있으며,  
```
python get_block_range_for_date.py --date 2018-01-01
```
이 결과에서 나온 Block의 결과를 바탕으로, Block과 Transaction의 내용을 다음과 같이 얻어 낼 수 있다.   
```
ethereumetl export_blocks_and_transactions --start-block 0 --end-block 500000 \
--blocks-output blocks.csv --transactions-output transactions.csv \
--provider-uri https://mainnet.infura.io/v3/7aef3f0cd1f64408b163814b22cc643c
```
이렇게 해서 얻어낸 csv는 storage에 적재가 된다. 개인적으로 csv는 성능이 조금 더딜수 있음으로 Medium글에서 추천하듯 CSV file들을 Columnar Dataset인 Parquet로 변환한뒤 적재한다면 OLAP Database에서 쿼리하기 좋을 것이다. 이렇게 완료가 되었다면 이제부터 원하는대로 쿼리하여 가장 인기가 좋은 토큰, 가스가 사용된 트랜잭션 수 등을 알 수 있다.

Ethereum ETL 깃헙 도큐멘테이션을 보아하니 Streaming또한 있던데, 이거도 나중에 찾아 봐야겠다. 