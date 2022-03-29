---
layout: post
title: "네트워크를 공부하자! (5) 구글에 접속하면 생기는 일들"
categories: network
author: "Yongchan Hong"
---
# 네트워크를 공부하자! (5) 구글에 접속하면 생기는 일들
필자는 네트워크에 대해 지식이 더 있으면 좋겠다는 생각을 하였고, 이에 지금까지 알던 것 + 모르던 것을 모두 정리하고 다시 공부할겸 네트워크 쪽에 필수적인 지식들을 리뷰하는 세션을 가져보기로 했다. 그 최종장은 구글에 접속하면 생기는 일들이다.

## 구글에 접속한다면?
www.google.com을 접속하면.. 사실 앞에 http(https)와 뒤에 포트 (http시 80, https시 443)를 포함하여 https://www.google.com:443으로 HTTP Request를 보내는 것이다.  
도메인은 google.com이지만, 우린 IP를 알아야 함으로 DNS 서버를 통해 요청을 보내면 (google.com의 IP를 알려주세요~~) IP를 받을 수 있다. DNS 서버 주소는 컴퓨터에 보관되어 있다. (DNS서버는 UDP를 사용한다!)  
MAC주소 또한 필요한데, 우린 ARP Protocol을 통해 IP주소로부터 MAC주소를 알 수 있다. (우리의 MAC 주소는 이미 있다! 우리집 공유기 aka 게이트웨이에 대한 정보는 이미 있다).  

> 우리의 Private IP는 공유기를 통해 Public IP가 되고 (Network Address Translation), 라우팅을 거쳐 구글 라우터를 발견한다. 이후 ARP Protocol을 통해 요청을 받으면 MAC주소를 전달해준다. 

이후 HTTPS라면 TLS/SSL Handshake가 일어나고, 이후 TCP 3-way Handshake가 벌어져 연결을 만든다. (이후 TCP 4-way Handshake를 통해 연결을 끊어준다.)  
이후 패킷을 전달한다. 구체적으로, Application Layer에서 HTTP Request를 한다.
구글 서버는 이를 받고 응답을 돌려 보낸다. HTML 문서, image 등을 돌려보내는데, 브라우저의 렌더링 엔진 (크롬은 webkit, 파이어폭스는 Gecko)은 HTML을 파싱하고 DOM 트리를 구축한다. 또한 css를 파싱하여 스타일 구조체로 만들어준다. 

> DOM은 Document Object Model로 웹브라우저가 html을 인식하는 방식이다.

이 두가지를 합쳐서 랜더 트리로 만들게 된다. 이후 화면에 배치를 시작하고 UI 백엔드가 형상을 그린다. 


## Reference
https://www.youtube.com/watch?v=dh406O2v_1c  
https://www.youtube.com/watch?v=BEK354TRgZ8