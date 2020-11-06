---
title: [aws gamelift ]
date: 2020-01-11 14:30:19
tags: [aws, gamelift]
categories:
    - 개발
thumbnail: ""
permalink: ""
---
# install gamelift SDK for linux
0. 
0.1.
0.2.
0.3.
0.4

1. SDK 다운(https://ap-northeast-2.console.aws.amazon.com/gamelift/home?region=ap-northeast-2#/)
2. cmake 설치.(yum install cmake)
3. 해당 폴더(/home/gamelift/gamelift_sdk)에서 cmake CMakeLists.txt(makefile 생성)
4. make
<!-- more -->
5. cd thirdparty
6. cmake CMakeLists.txt
7. make(makefile 생성)
8. cd /home/gamelift/gamelift_sdk/prefix/lib
 8.1 libprotobuf.so, libboost_date_time.so, libboost_system.so, libboost_random.so, libprotoc.so (해당 동적 라이브러리가 생성되었는지 확인)
9. library PATH 넣어주기.
 9.1 vi /etc/profile
 9.2 export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/gamelift/gamelift_sdk/prefix/lib 넣어준다.
 9.3 source /etc/profile(바로 적용)
 9.4 echo $LD_LIBRARY_PATH(위에 라이브러리가 추가 되었는지 확인)

# gamelift 사용하는 방법(예제)

0. 문서(GameLift_dist.pdf) 

https://github.com/padhansell/GameLift/blob/master/GameLift_dist.pdf


1. 다운로드(https://github.com/zeliard/GameLift)
2. GameLiftLinuxServer 폴더에서 cmake CMakeLists.txt(makefile 생성)
3. make

# 자신의 서버 소스에 gamelift를 추가 하는 방법

1. 소스 컴파일(make, make install)
 1.1 vi makefile
 1.2 -L/home/gamelift/gamelift_sdk/prefix/lib  -lsioclient -lboost_date_time -lboost_random -lboost_system -lprotobuf -lpthread (lib 추가)
      -I/home/gamelift/gamelift_sdk/prefix/include(include 추가)
      GameLiftManager.cpp Log.cpp 추가(gamelift 예제 소스에서 가지고 옴)
 1.3 make

# 아마존 게임리프트 콘솔에 올리는 방법

1. 아마존 게임리프트 콘솔 도움말
http://docs.aws.amazon.com/ko_kr/gamelift/latest/developerguide/fleets-creating.html

2. gamelift console 설정(https://ap-northeast-2.console.aws.amazon.com/gamelift/home?region=ap-northeast-2#/r/fleets/create) - 아마존 계정을 만들어야 한다.
 2.1 aws 액세스 키 생성
  https://docs.aws.amazon.com/ko_kr/general/latest/gr/managing-aws-access-keys.html
 2.2 게임서버에 session key, secret key 추가.
  2.2.1 해당 게임서버에서 AWS cli 설치 및 키 추가(http://pyrasis.com/book/TheArtOfAmazonWebServices/Chapter30/02)
  2.2.2 해당 서버에 실행 폴더를 AWS cli 통해서 gamelift에 올림.
   aws gamelift upload-build --name GameLiftLinuxServer5 --build-version 0.0.1 --build-root /home/gamelift/GameLift-master/GameLiftLinuxServer/build --operating-system AMAZON_LINUX --region ap-northeast-2
  2.2.3 aws gamelift describe-build --build-id (올라 갔는지 확인)
 2.3 gamelift cosole 접속(직접 들어가서 해야 한다.)
