---
layout: post
title: "Hadoop 깊게 알아보기"
categories: hadoop
author: "Yongchan Hong"
---
# Hadoop 깊게 알아보기 
Hadoop에 대해서 "조금" 아는것만으로 충분하지 않다. Hadoop Ecosystem을 정리했던 저번보다 더 알아보기 위하여 [빅데이터-하둡, 하이브로 시작하기](https://wikidocs.net/book/2203)를 참고하여 Hadoop 개념에 대해 조금이라도 더 깊게 알아보기로 했다.   
Hadoop은 Scale Up을 하는 방식 대신 Scale Out을 하여 컴퓨터 여러 대를 클러스터화하고 큰 크기의 데이터를 병렬로 처리해 속도를 높이는 분산 처리를 위한 프레임워크다.  
Hadoop V1은 HDFS와 MapReduce를 처음 정의 하였으며, Job Tracker와 Task Tracker로 MapReduce를 추적하였다. Hadoop V2는 잡트래커의 병목현상을 막기 위해 Yarn이 도입되게 되었다. Yarn 아키텍쳐를 도입하여 컨테이너 단위로 작업을 나누어 실행하게 되었다. 또한 Yarn 아키텍쳐 위에서 돌게 되어 MR로 구현되지않아도 Spark, Hbase, Storm등의 컴포넌트들이 돌 수 있다. Spark V3는 Erasure Coding (EC)가 도입되고, 고가용성을 위해 네임노드가 2개 이상 지원되었으며 YARN 타임리스 서비스 v2가 도입되었다. 이제 본격적으로 좀 깊이 알아보도록 하겠다.

## HDFS
HDFS (Hadoop Distributed File System)은 이름 그대로 File System으로서 분산처리를 목적으로 하고 장애 복구성을 지니고 있다. HDFS는 `블록`단위로 저장이 되며 해당 블록은 한번에 읽고 쓸 수 있는 데이터의 최대량이다. 따라서 블록 단위보다 큰 파일이 오면 블록으로 나누어 저장한다. 또한 블록 복제를 이용한 장애 복구가 가능하다. 블록의 기본 복제 단위인 3만큼 블록이 복제 되어 다른 Rack에 저장된다. 또한 HDFS는 한번 올리면 파일을 수정할 수 없기에 읽기 중심이다. 

### HDFS 구조
HDFS는 이전 에코시스템 리뷰에서도 말했지만 하나의 `네임 노드 (Name Node)`와 여러개의 `데이터 노드 (Data Node)`로 구성이 된다. 

네임 노드는 Namespace를 관리하는데, 이는 메타데이터 관리와 데이터 노드의 관리를 한다. 메타데이터는 전체적인 블록의 정보, 파일 이름 등으로 이루어지는데, `Fsimage`와 `Edits`가 있다. Fsimage는 현재 Namespace와 블록 정보를 가지고 있다면, Edits는 파일 생성 삭제 등 트랜잭션 로그를 들고 있다. 따라서 네임 노드는 Fsimage를 읽어 메모리를 적재하고 Edits를 읽어 변경내역을 반영한다. 그후 메모리 상태를 스냅샷으로 생성해 Fsimage파일을 다시 생성하며 데이터 노드로부터 블록 리포트를 수신해 매핑한다. 그리고 서비스를 시작한다. 

데이터 노드는 블록이 저장되는 노드로서, 블록 스캐너를 통해 오류를 확인하고 발생하면 수정하는 역할을 한다. 이러한 데이터 노드는 30-40개가 모여 랙(Rack)을 이루게 되고, 각 블록의 복제는 다른 랙에 들어가야한다. 데이터 노드는 주기적으로 하트비트를 데이터 노드에 전달해 동작 상태를 알려주고, 블록리포트를 통해 블록의 상태를 알려준다.

파일을 읽어야 하는 경우, Client는 요청하게 되면 Name Node를 통해 블록 위치를 받아오고, FSDataInputStream을 통해 데이터 노드에 파일블록을 읽을 수 있다. 파일을 써야 하는 경우, Name Node에 파일을 전송하고 블록을 써야할 노드 목록을 받게 되면 FSDataOutputStream을 통해 Packet으로 전달 하고 파일을 쓰게 된다. 서버와 클라이언트 통신시에는 RPC를 사용한다. 

![](https://wikidocs.net/images/page/23582/hdfsarchitecture.png)

### Secondary Name Node
Secondary Name Node에선 Fsimage와 Edits파일을 주기적으로 머지하여 최신 블록의 상태로 파일을 생성한다. 따라서 Edits 파일이 너무 길어지는 현상을 막을 수 있고, 네임노드가 빠르게 재구동 될 수 있게 해준다. 

![](https://charsyam.files.wordpress.com/2011/04/fsimage.png)

HDFS는 고가용성을 위해 standby 네임노드가 지원이 되어 active 네임노드가 죽게 된다면 변경되게 한다. 또한 standby 네임노드는 Secondary Namenode의 역할도 한다

### HDFS 이모저모
- HDFS Federation은 네임스페이스 단위로 네임 노드를 등록하여 사용하는 것으로 각 네임 노드 마다 독립적이다.
- HDFS는 세이프모드가 있어 데이터 노드를 읽기 전용 상태로 바꿀 수 있다. 서버 정비 할때 설정할 수 있다. 
- HDFS는 휴지통이 있어 파일 삭제 이후 휴지통으로 가고 복구할 수 있다. Trash Interval이 있어서 일정시간 후 Checkpoint가 날아간다.
- 기본적인 커맨드는 다음과 같다.  
```
hdfs dfs # 파일 시스템 쉘 명령어
hdfs dfs -ls #디렉토리 내부 파일 보여줌
hdfs dfs -get {src} {localdst} #로컬디렉토리로 다운로드
hdfs dfs -count #디렉토리 개수, 파일 개수 카운트

hdfs fsck #파일 시스템 상태 체크
```
- HDFS는 Hadoop V3에 Erasure Coding이란 것을 지원한다. EC는 데이터 저장 공간의 효율성을 높이려고 설계 된것으로, 이레이저 코드를 이용해 데이터를 인코딩하고, 손실시 디코딩을 하여 원본을 복구한다. 

## MapReduce
![](https://t1.daumcdn.net/cfile/tistory/2136A84B59381A8428)

MapReduce는 분산처리를 담당하는 프레임워크로, 맵과 리듀스로 나누어져있다. 맵은 데이터를 종류별로 모으고, 리듀스는 Filtering과 Sorting을 거쳐 데이터를 뽑아내는 역할을 한다. 맵리듀스 작업 단위는 잡이고, 맵 테스크와 리듀스 테스크르로 나누어져 있다. 맵의 경우 데이터 지역성으로 HDFS에서 일반 S3 같은 리모트 스토리보다 더 빠르게 구동이 된다 (네트워크 대역을 사용하지않고 데이터가 있는 노드에서 실행함. 그렇지 못하다면 같은 랙, 또 그렇지 못하다면 다른 랙에서 가져오게 됨). 맵리듀스의 처리 단계는 다음과 같다. 
1. Input  
InputFormat클래스를 사용해 데이터를 입력한다. Text, csv, gzip등의 형태 가능하다.
2. Map  
사용자가 Mapper Class를 상속해 만든 map 매소드를 이용해 입력을 분할하고 키별로 데이터를 정리한다.
3. Combiner  
네트워크를 통해 타고 넘어가는 데이터를 줄이기 위해 맵의 결과를 정리하는 것으로, 로컬 리듀서라고도 한다. 존재하지 않을 수도 있다.
4. Partitioner & Shuffle  
맵작업이 끝난 노드에서 리듀서로 데이터를 전달한다. 리듀서의 개수만큼 파티션을 생성한다. 
6. Sort  
리듀스 작업 전 키를 이용해 정렬을 수행한다.
7. Reduce
Reduce 클래스를 상속하여 reduce() 메소드를 구현할 수 있다. 데이터를 처리하고 저장한다.
8. Output  
리듀서의 결과를 정의된 형태 (csv, text 등)으로 저장한다. OutputForamt으로 형태를 정의하고 RecordWriter로 실제 파일을 기록한다. 

MapReduce는 보조 도구로 잡의 진행상황을 볼 수 있는 Counter와 공유되는 데이터를 이용해야할 때 쓰는 Distributed Cache를 사용할 수 있다. 

## YARN
YARN은 클러스터 리소스 관리 및 애플리케이션 라이프 사이클을 관리하는 아키텍쳐이다. YARN의 자원관리는 `Resource Manager`와 `Node Manager`, 애플리케이션 라이프 사이클 관리는 `Application Master`와 `Container`가 관리하고 있다. 

자원관리의 경우 노드 매니저가 각 노드마다 존재하는데, 이는 노드 내 자원 상태를 관리하고 리소스 매니저에 보고한다. 리소스 맨지너는 전체 클러스터의 장원을 관리한다. 스케줄러에 설정된 규칙에 따라서 자원이 분배가 된다. 

라이프 사이클의 경우 애플리케이션 마스터와 클라이언트가 사용이 된다. 클라이언트가 리소스매니저에 애플리케이션을 제출하게 되면, 리소스 매니저는 비어있는 노드에 애플리케이션 마스터를 실행하게 되고, 자원은 Node Manager를 통해 받아온다. 그 이후 자원을 할당 받으면 컨테이너를 각 노드에 실행하게 되고 작업이 종료되면 애플리케이션 마스터가 이를 취합해 리소스 매니저에 알리고 자원을 해제하게 된다.  
![](https://www.oreilly.com/library/view/hadoop-the-definitive/9781491901687/images/hddg_0402.png)

YARN의 스케줄러에는 크게 `FIFO Scheduler`, `Fair Scheduler`, `Capacity Scheduler`가 있다. FIFO는 먼저 들어온 작업을 처리하고, Fair는 제출된 작업이 동등하게 리소스를 점유하게 하고 (균등하게 자원 할당), Capacity는 큐를 선언해 각 큐별로 자원의 용량을 정하고 그 용량에 맞게 할당한다. YARN은 Node Label이 있어서 서버를 특성에 맞게 구분하여 작업을 처리할 수 있다.

## Hadoop Common
Hadoop Common은 작업 지원 도구이다. `DistCp`를 통해 클러스터 내의 대규모 데이터 이동을 맵리듀스로, 병렬로 복사할 수 있다. 또한 `Haddop Archive`로 작은 사이즈의 파일들을 묶어서 관리하게 해준다. 

### Reference
https://wikidocs.net/book/2203  
https://blog.naver.com/PostView.nhn?blogId=redhattt&logNo=221386458958&parentCategoryNo=28&categoryNo=22&viewDate=&isShowPopularPosts=false&from=postView  
https://12bme.tistory.com/154  