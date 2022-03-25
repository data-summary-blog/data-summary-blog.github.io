---
layout: post
title: "Kafka 훑고 지나가기 (2) - MSK, strimzi"
categories: kafka
author: "Yongchan Hong"
---

# Kafka Advanced - MSK and Strimzi
Kafka를 클라우드 환경에 쓰기 위해 적합한! MSK와 Strimzi를 소개하고자 한다.

## Amazon MSK
Amazon MSK(Managed Streaming for Kafka)는 Amazon에서 제공하는 완전 관리형 Kafka이다. 
![](https://docs.aws.amazon.com/ko_kr/msk/latest/developerguide/images/msk-architecture.png)  
다이어그램과 같이 Amazon MSK클러스터를 위해 각 az당 생성할 브로커 노드 수를 지정하면, 다음과 같이 az별로 브로커를 생성해 준다. 또한 MSK는 자동으로 주키퍼도 같이 생성해준다. 또한 쉽게 Producer, Consumer와 Topic을 만들 수 있다. MSK의 장점으론 뭐가 있을까?
- Autoscaling이 가능하다. 원래 scale-in이 불가했었는데 업데이트로 인해 자동으로 Broker 수와 Volume이 scale-in과 scale-out가 가능하다.
- Broker log는 S3, Cloudwatch, Firehose등에서 받을 수 있다.
- User Authentication을 TLS 등으로 가능하다.
- Rolling Update가 가능해서 Cluster를 멈추지않고 가능하다.

## Strimzi 
![](https://strimzi.io/docs/operators/latest/images/operators.png)  
Kafka를 Kubernetes 환경에서 사용할 수 있게 해준다.  
이 부분은 좀 더 공부해보겠다.



### Reference
https://docs.aws.amazon.com/ko_kr/msk/latest/developerguide/what-is-msk.html  
https://strimzi.io/
