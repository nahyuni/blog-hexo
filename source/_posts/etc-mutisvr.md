---
title: [멀티 플레이어 서버 고찰]
date: 2020-02-06 14:30:19
tags: [server]
categories: 
    - 그 외
thumbnail: ""
permalink: ""
---
<!-- toc -->
## 1. 생각해야 될점
 1. 반응속도 - 우리게임은 초당 몇번의 패킷을 보내야 하는가.(ex 초당 10번을 목표로 동기화(최대RTT - 100ms)) - 모바일 환경에서의 RTT(불규칙) 
   https://aws.amazon.com/ko/blogs/korea/route53-latency-based-routing/
   
<!-- more -->
 2. 동기화  
   2.1 P2P/CS 사용.
     모바일 환경에서 유용한가? 

 3. 보안(실시간이 아닌 검증으로 되는가?) - 유저간의 레이싱 시작할때 맵정보를 받고 레이싱 끝날때 검증을 한다.(게임서버) 
    맵정보에서 없는 좌표가 들어왔을시 또는 1초 동안의 최대 나올수 있는 좌표 범위를 서버에서 계산해서 문제가 있을 경우 블랙유저로 처리한다.

## 2. 망 구조 정리(장단점)
* P2P(보안 약함, 홀펀칭 등 무선 네트워크에서는 변수가 있음.)

1. 대칭피어방식으로 패킷을 보낸다.
2. P2P Lockstep(RTS) 동기화 방식.
 2.1 명령어를 동시에 내리면 정확히 같은 시점에 모든 피어가 이 명령
 어를 처리해야 한다.(랜덤 요소가 없어야 한다.) 최대 지연시간 200M 초로 디파인한다.
 2.2 각 피어(클라이언트)는 저마다 다른 플레임 레이트로 구동되고 접속환경도 품질이 서로 차이가 난다. - 이를 해결하기 위해 턴타이머를 구현해야 한다.
 2.3 턴 타이머는 명령대기열을 받는 즉시 처리하지 않고 예를들면 한턴(200M)일 경우 두번의 턴이 지난 다음에 처리하므로써 화면에 반영하기까지는 지연시간 까지 합쳐 600M초가 된다.
     지연시간에도 불구하고 딱 두턴만 기다려주면 모든 플레이어가 명령을 동시에 처리할수 있다.
 2.4 최대 지연시간 200M 초를 넘기면 플레이어를 내보내도록 처리한다.
 2.5 명령어를 메모리에 담기 때문에 리플레이시에 메모리 용량이나 처리부담이 적다.
 2.6 게임 요소에 난수로 검사할 경우(예를 들면 A인스턴스는 궁수가 보병을 명중시키고, B는 명중을 못시킬 경우) 같은 시드로 같은 횟수로 해야 한다.
3. 홀펀칭이 안되는 경우 릴레이 서버를 이용한다.

* CS 구조(보안이 필요할 경우, 서버,클라이언트 공수가 큼.)
1. Pure Client-Server - (LOL)
서버와 클라이언트가 동기화를 맞춰준다. (Ex: LOL)
클라이언트에서 Input을 주면, 서버에서 게임 로직을 처리한다. 서버가 게임 오브젝트들의 상태를 동기화한다.
상태정보가 틀어지기 때문에 클라이언트는 서버에서 보내주는 값들을 자연스럽게 보간 및 예측만 해주면 된다.
Peer-to-Peer LockStep처럼 가장 느린 클라이언트에 동기화를 맞춰주는게 아니라 서버에 동기화를 맞춰주기 때문에 다른 클라이언트에 영향을 받지 않는다. 
당연하지만 서버가 느려질 경우에는 모두가 느려진다.
서버의 허락을 받고 명령이 수락되는 방식이기 때문에 해킹에 강한 편이지만, 서버에 허락을 받아야 하기 때문에 반응 속도가 떨어지는 편이다.
그리고 Latency가 조금이라도 높아지는 날에는 게임이 지옥으로 변하게 된다. - 온라인에서나 가능할거 같음.

2. Client-Side Prediction (Reconciliation) 
Client-Server 방식과 비슷하지만, 클라이언트에서 먼저 계산하고, 서버와 오차를 보정해 나가면서 동기화를 맞추는 것이 차이점이다.
주로 빠른 반응성이 필요한 FPS(Ex: 콜오브듀티) 게임 등에서 많이 사용되고 있다.
서버와 클라이언트 간의 오차를 줄여주는 것이 매우 힘든 작업이다.

3. CS with Lockstep 방식(starcraft)
 3.1 서버에서 각 피어의 역활을 한다. 그외에 방식은 다 똑같음.
 3.2 서버에서 게임로직 처리.


* 혼합 구조.(RTS 방식이 아님)
1. 클라이언트는 홀펀칭을 시도후에 안되면 게임서버(Relay)에 접속한다. 
2. 홀펀칭이 성공하면 P2P로 1초에 4번 정도 이동패킷을 보내며, 이중에 5~6번마다 한번씩 서버에게 보낸다.(실시간 검증 - 이동 패킷이 허용범위를 넘었을 경우 해킹 유저로 보고 끊는다.)
3. 서버(게임서버)는 이것을 다른 유저에게 relay한다. 이동패킷을 제외한 패킷의 경우는 P2P로 보낸후 서버에게도 패킷을 보낸다.(평균 2~3초에 한번 보냄)
4. 패킷을 받는 클라이언트는 P2P든, 서버든 가장 빨리 "보내는 쪽"을 먼저 처리하고 뒤이어 온 패킷은 "무시" 한다.
5. 서버는 TCP/IP를 사용한다.


* 단순 릴레이 구조.(CS 구조)
1. 클라이언트에서 보내는것을 게임서버는 그대로 Relay를 한다.
2. 게임 세션 및 검증을 게임서버에서 한다.(실시간이 아닌 검증)

* 자료
모바일 게임에서의 실시간 멀티플레이에서 P2P와 CS 네트워킹의 비교 
 1. http://www.codeinfo.net/bbs/board.php?bo_table=pra_view&wr_id=6
CS 동기화 관련 자료 
 1. http://yakolla.tistory.com/62
P2P RTS 와 CS 관련 자료
 2. https://pegasuskim.wordpress.com/2015/12/23/%EC%84%9C%EB%B2%84-%EB%8F%99%EA%B8%B0%ED%99%94-%EB%AA%A8%EB%8D%B8/ 


## 3. 그 밖에 참고
1. EA코리아에서 만든 realracing3은 Time Shifted Multiplayer (비실시간 멀티플레이어) 로 만듦. 뜻은 시간을 옮겨놓은(즉, 서로간의 플레이 시간이 일치하지 않는) 멀티플레이어를 뜻한다
TSM 은 사용자들의 주행 데이터를 기반으로 한 인공지능 봇이다. 인터넷이 켜져 있으면 다른 사용자들의 아이디와 프로필 사진, 랩타임을 이용하는 것이고, 꺼져있으면 기본 탑재된 TSM 이 나온다.
가끔 TSM 끼리 서로 경쟁하는 모습도 보이곤 하지만 그렇게 적극적인 것은 아니며, 모든 TSM 이 레이싱라인을 따라서 너무나도 질서정연하게 달리기 때문에, TSM 에 실시간 멀티플레이와 같은 변칙적이며 전투적인 플레이를 기대해서는 안 된다



