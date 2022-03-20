---
layout: post
title: "네트워크를 공부하자! (1) OSI 7 Layers"
categories: network
author: "Yongchan Hong"
---
# 네트워크를 공부하자! (1) OSI 7 Layers
필자는 네트워크에 대해 지식이 더 있으면 좋겠다는 생각을 하였고, 이에 지금까지 알던 것 + 모르던 것을 모두 정리하고 다시 공부할겸 네트워크 쪽에 필수적인 지식들을 리뷰하는 세션을 가져보기로 했다. 그 첫번째는 바로 OSI 7 Layers다.

## OSI 7 Layers
OSI (Open System Interconnection) 7 Layers Model는 네트워크 표준 모델로서, 네트워크를 단계별로 알 수 있고, 문제가 발생하면 어디를 봐야할지 추적할 수 있다.

![](https://s7280.pcdn.co/wp-content/uploads/2018/06/osi-model-7-layers-1.png)

1) Physical  
전기적 신호로 이루어진 데이터를 전달함  

2) Data-link  
Physical 계층에서 온 정보를 안전하게 전달되게 해주는 계층으로 MAC주소를 통해 통신한다. (스위치)
> Mac 주소란?  
> 통신을 위해 Data-link 계층에서 할당한 고유 식별자/물리 주소

3) Network  
데이터를 목적지까지 가장 빠르고 안전하게 운반하는 역할을 한다.  
따라서 라우터를 통해 최적의 경로를 찾아 논리주소인 IP주소를 지정하고, 데이터 단위인 패킷을 경로에 따라 전달해준다.  

4) Transport  
TCP/UDP를 통해 통신을 활성화 한다. 송/수신자간 데이터 전송을 할 수 있게 하며, 데이터 단위인 Segment를 보낸다. (port번호)

5) Session  
데이터의 통신을 위한 논리적 연결을 담당하고 있다. TCP/IP 세션을 없애거나 만드는 역할을 한다. 이 계층에선 동기화가 있는데, 송수신 중 오류가 발생하면 통신 양단에서 서로 동의하는 논리적인 지점인 동기점으로 돌아가 다시 재전송한다. 

6) Presentation  
데이터를 어떻게 표현할지 정하는 곳이다. 데이터 부호화, 데이터 압축, 데이터 복호화 등을 사용하며 JPEG, ASCII 등이 있다.

7) Application
최종 목적지로, 사용자들이 접하는 곳이다. HTTP, FTP, DNS등이 있다. 

## TCP/IP Model?
Session, Presentation, Application을 합쳐서 전부 Application Layer라 한다. 실제로 현재 인터넷은 이 구조를 따르고 있다. Application Layer에서 데이터를 요청할 때는 GET, POST, PATCH, DELETE 등의 HTTP 메서드, 자원 위치를 나타내는 URL, 그리고 HTTP 버젼을 물어본다. HTTP가 아닌 경우에는 SMTP, FTP등을 따른다.

### Reference
https://gyoogle.dev/blog/computer-science/network/OSI%207%EA%B3%84%EC%B8%B5.html  
https://www.youtube.com/watch?v=1pfTxp25MA8  