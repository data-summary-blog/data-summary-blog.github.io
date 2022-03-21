---
layout: post
title: "네트워크를 공부하자! (3) Load Balancing"
categories: network
author: "Yongchan Hong"
---

# 네트워크를 공부하자! (3) Load Balancing 
필자는 네트워크에 대해 지식이 더 있으면 좋겠다는 생각을 하였고, 이에 지금까지 알던 것 + 모르던 것을 모두 정리하고 다시 공부할겸 네트워크 쪽에 필수적인 지식들을 리뷰하는 세션을 가져보기로 했다. 그 세번째는 바로 Load Balancing이다.  

## Load Balancing

Load Balancing이란 네트워크 기술 중 하나로 둘 이상의 저장 장치나 컴퓨터 자원들에게 작업을 나누는 것이다. 쉽게 생각해보자. 웹사이트로 사람이 몰리면 어떨까? 하나의 서버로는 모든 트래픽을 감당하기 힘들 수 있다. 그럴때 필요한것이 `Scale-out`이고, 이에 여러 서버에 분산시켜주는 Load Balancing을 채택할 수 있다. `Load Balancer`는 이를 제공하는 서비스/장치로 클라이언트와 서버 사이에 있어서 특정 방식에 따라 부하를 분산 시켜주는 역할을 한다. 

## Load Balancing의 종류  
OSI 7계층에 따라 Load Balancing의 종류는 나뉠 수 있다.  
L2, L3, L4, L7이 있지만 대표적인 L4와 L7만 보도록 한다.  

### NLB (Network Load Balancer) - L4

L4의 Load Balancer로 IP와 포트 기반으로 서버 부하 분산을 한다. Client에서 Server로 연결할때 역할을 하며 Server에서 나가는 트래픽은 Client IP와 직접 통신한다. NLB는 고정 IP를 가지고 있다. 당연하지만 TCP/UDP 트래픽을 컨트롤한다. 트래픽이 극도로 많고 분산만 해야할 경우에는 NLB가 더 빠름으로 선호된다.

### ALB (Application Load Balancer) - L7

L7의 Load Balancer로 HTTP/HTTPS처리에 특화되어있다. Client와 Server 양방향 통신이 ALB를 통해 지나가야 한다. path-based routing이 지원되어 path별로 특정 서버로 연결해줄 수 있고, 기능도 NLB보다 많다. 유동적인 IP를 가지고 있다.

## Load Balancer가 서버를 선택하는 방식
다음과 같은 방식들이 대표적으로 있다. 
- Round Robin: 요청들을 순서대로 서버에 배정하는 방식
- Least Connections: 가장 연결 개수가 적은 서버를 선택하는 방식
- Source: Client IP를 해싱하여 분배하는 방식

### Reference
https://gyoogle.dev/blog/computer-science/network/Load%20Balancing.html  
https://dev.classmethod.jp/articles/load-balancing-types-and-algorithm/  