---
title: [고가용인프라]
date: 2020-09-24 14:30:19
tags: [infra]
categories: 
    - 개발
thumbnail: ""
permalink: ""
---

## 서버 환경
1.Web(2EA), DB(4EA), HAProxy(1EA), Backup(1EA)
2.CentOS 6.10, Apache : 2.4, PHP : 7.3 MariaDB : 10.3

## 서버 구성도
![](/images/infra.png)
<!-- more --> 
1. zenius 모니터링.
2. HAProxy(디비 로드밸런싱)
    - https://findstar.pe.kr/2018/07/27/install-haproxy/
    - https://en.wikipedia.org/wiki/HAProxy
3. 갈렐라 클러스터
    - http://rastalion.me/archives/760
    - https://bcho.tistory.com/1062

4.  RoseHA
    - http://www.ibinfo.co.kr/roseha
    - https://m.blog.naver.com/PostView.nhn?blogId=interikorea1999&logNo=220964505509&proxyReferer=https%3A%2F%2Fwww.google.com%2F

4. 기존에는 위에 그림처럼 HAProxy 위에  Galera로 운영을 하였다. 하지만 대용량 데이터를 인써트 업데이트 하기 때문에 Galera Replication 는 데이터 정합성을 중시하므로 시스템에 lock이 빈번히 걸려 시스템을 Galera를 제거, RoseHA 로 바꾸었다. 현재 별 문제 없이 잘 돌아감.
5. HAProxy, Galera -> RoseHA 로 변경
## 백업 및 복구
xtraback