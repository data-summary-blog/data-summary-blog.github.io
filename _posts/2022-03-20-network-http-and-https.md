---
layout: post
title: "네트워크를 공부하자! (2) HTTP and HTTPS"
categories: network
author: "Yongchan Hong"
---
# 네트워크를 공부하자! (2) HTTP and HTTPS
필자는 네트워크에 대해 지식이 더 있으면 좋겠다는 생각을 하였고, 이에 지금까지 알던 것 + 모르던 것을 모두 정리하고 다시 공부할겸 네트워크 쪽에 필수적인 지식들을 리뷰하는 세션을 가져보기로 했다. 그 두번째는 바로 HTTP and HTTPS다.

## HTTP
HTTP (HyperText Transfer Protocol)은 텍스트 기반 통신 규약으로 인터넷에서 데이터를 주고 받을 수 있는 프로토콜 (규칙)이다.  
사용자 (Client)가 Server에 `request`를 보내면, Server가 사용자에 `response` 하는 형태로 구성이 되어 있다. request 메세지는 Method(Get등), Path, HTTP Protocol Version, Header로 이루어져 있고, response는 HTTP Protocol Version, Status Code, Status Message, Header로 이루어져 있다.   
HTTP의 특징은 다음과 같다.  
- TCP/IP를 이용하는 프로토콜이다 (추후에 다룰 예정). TCP 연결을 열고 HTTP 메세지를 전송한다.
- 연결상태를 유지하지 않는 `Connection-less` protocol이다
- 서버가 클라이언트 상태를 보존하지 않는 `Stateless` protocol이다.  
- HTTP 서버와 클라이언트에 의해 HTTP 메세지는 해석이 된다. 

그러나 HTTP는 텍스트 교환이라 신호를 가로채면 내용이 노출 될 수 있으므로 보안 이슈가 있다. 이를 해결하기 위해 HTTPS가 등장했다.

## HTTPS  
![](https://1.bp.blogspot.com/-alfxKGRbNUg/WsmLtikh1vI/AAAAAAAATSw/zNoMl22FD94ZZlyrggBKR6L3UuZAwzraACLcBGAs/s1600/12.png)  
HTTPS (HyperText Transfer Protocol Secure)은 HTTP에서 보안이 추가되어 서버와 클라이언트 사이의 통신 내용이 암호화 된다. 이러한 암호화는 SSL(Secure Sockets Layer)/TLS(Transport Layer Security)로 가능한데, 이 두가지는 추후에 설명하도록 하겠다. HTTPS의 통신 흐름은 다음과 같다.  
1. 서버를 가지고 있는 기업은 HTTPS를 적용하기 위해 `public key`와 `private key`를 만든다.  
> public key는 본인이 가진 private key로만 복호화 할 수 있다.  
2. CA 기업을 선택한후 CA 기업은 해당 기업의 이름, 서버 공개키, 서버 암호화 방법을 담은 Server Certificate을 만든다. 이러한 Certificate은 CA 기업의 개인 키로 암호화해서 전달된다.
> CA란? Certificate Authority로 인증기관이다. Let's Encrypt같은 곳이 있다.
3. 이 이후, 클라이언트가 서버에 접속하게 되면 서버는 이 인증서를 클라이언트에 전달한다.
4. 클라이언트는 CA를 통해 인증서를 대조하고, CA의 공개키 (브라우저에 신뢰받는 CA의 공개키들은 등록되어 있다)를 통해 인증서를 해독한다. 해독 이후 서버의 `public key`를 얻게되고, 클라이언트는 premaster secret key를 만든후 서버의 `public key`로 암호화하여 서버로 전송한다.
5. 서버는 이를 받아서 클라이언트의 premaster secret-key를 서버의 `private key`를 이용해 decrypt한다. 
6. 이제는 이 premaster secret-key를 이용하여 서버와 클라이언트가 동일하게 데이터를 암호화거나 복호화 할 수 있다.
> premaster secret-key는 `Symmetric key` 즉 대칭 키로서 암호화와 복호화에 같은 키가 사용이 된다.   

이러한 HTTPS는 신뢰받은 CA기업이 아닌 자체 인증서를 발급하는 경우 안전하지 않을 수 있으므로 주의하여야 한다. 

## 이모저모
HTTP는 80번 포트, HTTPS는 443번 포트로 지정되어 있다.


### Reference
https://velog.io/@surim014/HTTP%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80  
https://gyoogle.dev/blog/computer-science/network/HTTP%20&%20HTTPS.html  
http://i5on9i.blogspot.com/2015/07/https-ssl.html  