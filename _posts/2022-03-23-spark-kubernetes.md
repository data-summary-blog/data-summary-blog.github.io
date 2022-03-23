---
layout: post
title: "좌충우돌 Spark 배우기 - Spark on Kubernetes"
categories: spark
author: "Yongchan Hong"
---

# 좌충우돌 Spark 배우기 - Spark on Kubernetes

## Spark-on-k8s-operator
![](https://raw.githubusercontent.com/GoogleCloudPlatform/spark-on-k8s-operator/master/docs/architecture-diagram.png)
다음 이미지는 GCP의 spark-on-k8s-operator에서 가져온 이미지다. 해당 Helm Chart는 [여기서](https://github.com/GoogleCloudPlatform/spark-on-k8s-operator/tree/master/charts/spark-operator-chart) 확인할 수 있다.여기서 알아야하는 컴포넌트는 다음과 같다.
- Spark Application Controller  
이 component는 Spark Application Object의 create, update, delete를 확인하고 이를 바탕으로 행동한다.
- Submission Runner  
Controller에서 받은 submission들을 spark-submit하는 역할을 한다.
- Spark Pod Monitor  
spark pod를 확인하고 controller에 상태를 업데이트한다.
- Mutating Admission Webhook  
Spark driver와 executor pod를 customize하는 역할을 한다.
- sparkctl  
커맨드라인 툴이다.

이에 대해 전체 프로세스를 설명하자면, 유저가 sparkctl이나 kubectl을 이용하여 SparkApplcation Object를 생성하게되면, Spark Application Controller는 API server를 보고 있다가 해당 object를 받게 된다. Spark Application Controller는 그 후 spark-submit argument를 가진 submission을 Submission Runner에 주면 Submission Runner는 실질적으로 이 application을 구동하게 된다. 구동하게 되면서 driver pod가 만들어지고, executor pod를 만들어주게 된다. 이 과정에서 Mutating Admission Webhook이 있다면 customize를 해준다. Spark Pod Monitor는 이러한 pod들의 상태를 확인하고 controller에게 업데이트시켜준다.  

## Scheudler?
스케줄러의 경우, Kubernetes에서 driver와 executor가 실행되게 schedule해주는 애이다. 만약 기존 Kubernetes Workload와 다르게 schedule하고 싶다면 driver와 executor가 동시에 실행되게 [Volcano](https://github.com/volcano-sh/volcano)를 사용할 수 있다.

## Spark Application
Application은 다음과 같은 형태를 띄울수 있다.
```
apiVersion: sparkoperator.k8s.io/v1beta2
kind: SparkApplication
metadata:
  name: spark-pi
  namespace: default
spec:
  type: Scala
  mode: cluster
  image: gcr.io/spark/spark:v3.1.1
  mainClass: org.apache.spark.examples.SparkPi
  mainApplicationFile: local:///opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar
```


### Reference
https://spark.apache.org/docs/latest/running-on-kubernetes.html  
https://github.com/GoogleCloudPlatform/spark-on-k8s-operator/blob/master/docs/user-guide.md#configuring-automatic-application-restart-and-failure-handling  