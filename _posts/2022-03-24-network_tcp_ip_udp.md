---
layout: post
title: "네트워크를 공부하자! (4) TCP/IP"
categories: network
author: "Yongchan Hong"
---
# 네트워크를 공부하자! (4) TCP/UDP
필자는 네트워크에 대해 지식이 더 있으면 좋겠다는 생각을 하였고, 이에 지금까지 알던 것 + 모르던 것을 모두 정리하고 다시 공부할겸 네트워크 쪽에 필수적인 지식들을 리뷰하는 세션을 가져보기로 했다. 그 네번째는 바로 TCP/UDP이다.

## TCP
TCP(Transport Layer Protocol)이란 가장 대중적인 Transport Protocol로 양방향 통신이 가능하다. TCP는 `신뢰성`이 기반이 되어 데이터를 `순차적`으로, `안정적`으로 전달하고, `port` 번호에 해당하는 프로세스에 데이터를 전달한다. Transport Layer가 없다면 데이터를 순차적으로 전달할 수 없고, 송수신자 간의 데이터 처리 속도 문제로 `flow` 문제가 있을 수 있고, 또한 네트워크 문제로 `congestion` 문제가 있을 수 있다. 따라서  

- Flow Control (흐름제어)  
속도를 조절하기 위해 `Stop and Wait`, `Sliding Window` (수신측에서 설정한 윈도우 aka 버퍼 크기만큼 송신측에서 확인 응답 없이 세그먼트를 전송할 수 있게 하여 데이터 흐름을 동적으로 조절하는 기법) 등을 쓴다.
- Congestion Control (혼잡제어)  
네트워크 혼잡을 피하기 위해 송신 쪽의 속도를 조절함. `AIMD`(Additive Increase Mulitcative Decrease)로 패킷 성공/실패에 따라 window size를 1 늘리거나 절반으로 줄임. `Slow Start`는 비슷하지만 늘어나는게 지수함수 꼴을 가져서 속도 늘리는걸 빠르게 함. `Fast Retransmit` 와 `Fast Recovery`도 있다.
- Error Detection (오류제어)  
오류가 있을시 재전송을 통해 오류를 복구한다.

이 가능하다. TCP는 세그먼트를 전달하는데 TCP Header와 Data (Application Layer단의 것들)를 segment라 한다. TCP Header는 source port, destination port, flag (ack,syn, fin)등이 있다. 이 밑에 Packet (IP) 등이 있지만 이건 나중에 다루도록 하겠다. 그렇다면 TCP는 어떻게 연결을 이룰까?

## TCP 3-way, 4-way Handshake
TCP는 정확한 전송을 위해 3-way handshake를 통해 connection을 보장한다.  

1. 클라이언트가 서버에게 SYN 플래그를 보냄
2. 서버가 클라이언트에게 받았다는 신호인 ACK와 SYN을 보냄
3. 클라이언트가 서버에게 ACK를 보냄.  

이렇게 되면 연결이 이루어진다. 연결을 끊기 위에선 4-way handshake를 한다.

1. 클라이언트가 서버에게 종료한다는 FIN 플래그를 보냄.
2. 서버는 FIN을 받고 ACK을 보냄.
3. 서버가 데이터를 모두 보낸뒤 FIN 플래그를 보냄.
4. 클라이언트는 서버에게 ACK을 보냄.

이후 클라이언트는 소켓을 종료하고, time wait 후 client도 종료한다.

## UDP
UDP (User Datagram Protocol)은 TCP와는 사뭇 다르다. Segment가 아닌 Datagram단위로 나누며, 비연결형, 신뢰성 없는 전송 프로토콜이다. UDP는 데이터의 처리가 매우 빠르기 때문에 데이터가 덜 중요한 온라인 게임이나 실시간 방송에서 자주 쓰인다. 3-way 4-way handshake와 다르게 그냥 패킷을 보낸다. UDP는 오류검출은 UDP header중 `checksum`을 통해 할 수는 있다. 그러나 오류복구나 여타 flow control, congestion control, error detection은 불가하다.

### Reference
https://www.youtube.com/watch?v=ikDVGYp5dhg  
https://www.youtube.com/watch?v=BEK354TRgZ8  