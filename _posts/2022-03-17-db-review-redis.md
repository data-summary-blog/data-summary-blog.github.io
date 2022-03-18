---
layout: post
title: "오늘의 DB review - Redis"
categories: database
author: "Yongchan Hong"
---

# 오늘의 DB Review - Redis

## Redis 소개
Redis (Remote Dictionary Server)는 key-value 기반의 NoSQL로 일반 database, message broker, cache등에 쓰일 수 있다. 그렇다면 Redis는 왜 이리 각광받을까? 다음과 같은 이유들이 있다.  
- In-memory 데이터 베이스에 의한 빠른 성능  
Redis의 경우에는 디스크에서 데이터를 불러올 필요 없이 메모리에서 데이터를 엑세스할 수 있기 때문에 빠른 성능을 자랑한다.
- 다양한 데이터 구조 (collection) 제공    
Default인 String부터 bitmap, hash, list, sorted set, list 등 여러가지 데이터 구조를 사용할 수 있어 편리하다. 또한 자료 구조가 Atomic하여 Race Condition을 피할 수 있다 (제대로만 짰으면)
- 복제 지원  
Redis Cluster는 Master-Slave 아키텍처를 이용하여 Replication을 지원한다. Master는 읽기/쓰기가 가능하고 Slave는 읽기만 가능한데, Slave에 비동기적으로 복제를 한다. B에 장애가 일어나면 Slave중 하나가 승격이 되고 복구를 할 수 있다.
- 샤딩 지원  
Redis Cluster의 노드에 hash slot을 부여하여 데이터를 저장할때 주어진 hash를 바탕으로 나눈다.
- 영속성  
Redis는 놀랍게도 지속성을 보장하기 위해 데이터를 디스크에 저장할 수 있다! RDB (Snapshotting) 방식을 통해 메모리에 있는 전체 내용을 디스크에 옮겨 담거나, AOF (Append On File) 방식을 통해 데이터가 변경 될때마다 log 파일에 기록되는 형태를 사용할 수 있다. 
- 여러 언어로 사용 가능    
Java, Python, Go, C++ 등 다양한 언어가 지원된다.

## Redis 사용법
```
# 설치
brew install redis

brew services start redis (redis 서비스 시작)
brew services stop redis (redis 서비스 중지)

# redis-server (server 실행, brew services start redis 대신 사용 가능)

redis-cli (client 실행)

# String - Single Key
set <key> <value>
get <key>

# String - Multi Key
mset <key1> <value1> <key2> <value2> ... <keyN> <valueN>
mget <key1> <key2> ... <keyN>

# List
Lpush <key> A # key: (Z) -> (A, Z)
Rpush <key> A # key: (Z) -> (Z, A)

Lpop <key> # key: (A,B,C), pop A
Rpop <key> # key: (A,B,C), pop C

#set
SADD <key> <value> #이미 있으면 추가 안됨
SMEMBERS <key> #모든 Value 돌려줌
SISMEMBER <key> <value> #value 존재하면 1, 아니면 0

#sorted set - 랭킹에 따라 순서가 바뀜
ZADD <key> <score> <value> #value가 이미 있으면 score를 변경
ZRANGE <key> <startindex> <endindex>

#hash
Hmset <key> <subkey1> <value1> <subkey2> <value2>
Hgetall <key> #해당 키의 모든 subkey와 value가져옴
Hget <key> <subkey>
Hmget <key> <subkey1> <subkey2> ... <subkeyN>

keys * #현재 키들 확인 가능, 많은 경우 사용 주의

keys *search* #search를 포함한 key검색

#keys의 경우 Redis가 One Thread이기에 서버를 멈춰버린다. 따라서 scan이라는 기능을 추천한다.

scan 0 # keys *와 같음
scan 0 match *keyword*
scan 0 count number # number 만큼 읽어옴 


ttl key #남은 시간을 초로 환산해서 보여줌. cache나 이런곳에서 ttl설정함.

DEL key # 키 삭제
flushall #레디스 서버의 모든 데이터 삭제

```

이렇게 레디스와 그 사용법에 대해 알아보겠다. 기회가 된다면 추후에 고려해야할 요소도 다뤄보도록 하겠다.

### Reference 
https://aws.amazon.com/ko/elasticache/what-is-redis/
https://www.youtube.com/watch?v=mPB2CZiAkKM
https://brunch.co.kr/@jehovah/20