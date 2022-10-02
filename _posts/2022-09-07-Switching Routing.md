---
title: (K8S) 네트워크 기초 정리 - 스위치, 라우터, 게이트웨이
author: simon sanghyeon
date: 2022-09-07
categories: [Kubernetes]
tags: [Kubernetes, K8S, Network, Switch, Router, Gateway]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
이번에는 CKA 과정에서 다루는 네트워크 내용을 이해할 수 있도록 네트워크의 기본적인 개념들을 정리한다.

# 스위치(Switch)의 기본 개념

`스위치(switch)`는 `허브(Hub)`와 동일하게 여러 장비를 연결하고 통신을 중재하는 장비다. 허브와 스위치는 내부 동작 방식은 다르지만, 여러 장비를 연결하고 케이블을 한곳으로 모아주는 역할은 같다. 허브는 단순히 전기 신호를 재생성해 출발지를 제외한 모든 포트에 전기 신호를 내보내지만, 스위치는 허브와 달리 MAC주소를 이해할 수 있어 목적지 MAC 주소의 위치를 파악하고 목적지가 연결된 포트로만 전기 신호를 보낸다.

![fig01](/assets/img/2022-09-07-Switching Routing/fig01.png)

아래와 같이 `컴퓨터A`와 `컴퓨터B`가 있다고 할 때, A와 B가 통신하기 위해서는 A와 B 모두를 스위치에 연결해야 한다.
즉, 스위치는 두 컴퓨터를 연결해주는 네트워크를 만들어준다.

![fig02](/assets/img/2022-09-07-Switching Routing/fig02.png)

`ip link` 명령을 통해서 각 호스트의 인터페이스를 확인할 수 있으며, 위 예의 경우 `eth0`이 인터페이스가 된다. eth0 인터페이스를 통해 스위치에 연결된 각 컴퓨터는 `192.168.1.X`의 IP 주소를 할당하여 사용함으로써 하나의 네트워크를 이루게 된다. 각 컴퓨터의 IP 주소는 위 그림과 같이 `ip addr add {IP 주소}/24 dev eth0` 명령으로 등록할 수 있다. 이후 `ping`을 통해 서로 연결된 것 또한 확인할 수 있다.

**아무튼, 핵심은 스위치를 통해 컴퓨터가 연결되어 통신 될 수 있다는 점이다. 스위치를 통한 통신은 동일한 네트워크 상에서만 가능하다. 즉, 다른 네트워크로 패킷을 주고 받을 수 없다.**

---
# 라우터(Router)의 기본 개념
네트워크의 크기가 점점 커지고, 먼 지역에 위치한 네트워크와 통신해야 하는 필요성이 늘어나면서 `라우터(router)`가 필요해졌다. 라우터는 아래 그림처럼 네트워크 주소가 서로 다른 장비들을 연결할 때 사용한다. 최근 일반 사용자가 라우터 장비를 접하기는 어렵다.

![fig03](/assets/img/2022-09-07-Switching Routing/fig03.png)

---
# 게이트웨이(Gateway)의 기본 개념
아래와 같이 `192.168.1.X`를 사용하는 네트워크와, `192.168.2.X`를 사용하는 네트워크가 있다고 해보자.

![fig04](/assets/img/2022-09-07-Switching Routing/fig04.png)

만약, 192.168.1.11 `컴퓨터B`가 다른 네트워크에 있는 192.168.2.10 `컴퓨터C`에 통신하려면 어떻게 해야 할까? 서로 다른 네트워크에 접근해야 하기 때문에 라우터를 사용해야 한다.
위 예에서 라우터는 두 개의 분리된 네트워크를 연결해주므로, 각 네트워크로부터 각 IP 주소(192.168.1.1 및 192.168.2.1)를 할당 받는다.

`route` 명령어를 입력하면, routing 설정들이 있는 kernel의 routing table을 확인할 수 있다. 아무런 설정도 하지 않은 채, route 명령어를 입력하면, 아무런 정보가 나타나지 않는다. 즉, `컴퓨터B`는 `컴퓨터C`에 연결될 수 없다. `ip route add 192.168.2.0/24 via 192.168.1.1` 명령을 통해서 192.168.1.1 `gateway`를 거치면 192.168.2.0의 네트워크에 접근할 수 있다는 설정을 등록할 수 있다. 이러한 설정을 마치고 나면, `컴퓨터B`가 라우터를 통해 상대 네트워크에 있는 `컴퓨터C`에 접근할 수 있다.

한편, 이러한 설정은 일일이 수행해야 하는데, 예를 들어 `컴퓨터C`에서 `컴퓨터B`에 접근하기 위해서도 위와 같은 방식으로 `ip route add 192.168.1.0/24 via 192.168.2.1` 명령을 통해서 192.168.2.1 gateway를 거쳐 192.168.1.0 네트워크에 접근할 수 있다는 설정을 등록해야 한다. 수많은 네트워크마다 모두 이러한 방식으로 등록하는 것은 불가능하다. 따라서 `ip route add default via 192.168.2.1` 명령과 같이 `default gateway`를 설정해 줄 수도 있다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `토리대디 님의 블로그` : [네트워크 장비, 허브(Hub), 스위치(Switch), 라우터(Router) 개념 및 정리](https://yys630.tistory.com/27)
