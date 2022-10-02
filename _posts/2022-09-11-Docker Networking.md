---
title: (K8S) 네트워크 기초 정리 - 도커의 네트워크
author: simon sanghyeon
date: 2022-09-11
categories: [Kubernetes]
tags: [Kubernetes, K8S, Network, Docker]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
도커의 네트워크 옵션 및 네트워크 네임스페이스와 관련한 내용들을 정리해보고자 한다.

# bridge(docker0)
![fig01](/assets/img/2022-09-11-Docker Networking/fig01.png){: width="500" height="500"}
*출처 : https://kangwoo.kr/2020/08/17/도커-네트워크/*

컨테이너의 네트워크 설정 옵션은 여러가지가 있지만, 기본적으로는 `bridge 네트워크(docker0)`가 있다. 이 네트워크 설정 옵션은 `docker run` 명령을 실행할 때 사용되는 기본값이므로, 별다른 설정을 추가할 필요가 없다.

bridge 네트워크는 `내부 사설 네트워크(internal private network)`로, 도커가 호스트에 설치될 때, default로 생성되며, 컨테이너와 도커 호스트를 연결해주는 역할을 한다. 이 네트워크는 default로 `172.17.0.x` 대역의 주소를 가진다. 기본적으로 bridge 모드를 사용할 경우, 컨테이너들은 분리된 네트워크 네임스페이스에 배치되며, 자동으로 172.17.0.x의 IP가 순차적으로 할당된다.

![fig02](/assets/img/2022-09-11-Docker Networking/fig02.png){: width="500" height="500"}

`docker network ls` 명령을 입력해보면 bridge 이름의 네트워크가 있는 것을 확인해 볼 수 있다. 도커가 설치된 호스트에서는 이러한 bridge 네트워크를 `docker0`라는 이름으로 부른다. `ip link` 명령으로 `docker0` bridge가 존재하는 것을 확인해볼 수 있다.

![fig03](/assets/img/2022-09-11-Docker Networking/fig03.png){: width="500" height="500"}

컨테이너가 생성될 때마다, 우선적으로 도커는 해당 컨테이너를 위한 네트워크 네임스페이스를 만든다. 그 후, 컨테이너 네트워크 네임스페이스 측에 연결될 `eth0@XXXX`이름의 인터페이스와 docker0 bridge 네트워크 측에 연결된 `vethXXXX` 이름의 인터페이스를 짝으로 만들며, 각각을 컨테이너와 docker0 bridge 네트워크에 연결한다. 내부적으로 `ip link add docker0 type bridge`와 같이 네트워크 네임스페이스에서 별도의 ip를 할당하고 연결하는 기술을 구현하는 것이다.

---
# --network none 옵션
![fig04](/assets/img/2022-09-11-Docker Networking/fig04.png){: width="200" height="200"}

이외에도, 사용자의 선택에 따라서 다른 네트워크 드라이버를 쓸 수도 있다. 예를 들어, `docker run --network none nginx`와 같이 `--network none` 옵션을 줄 수 있다. `--network none` 옵션을 통해 도커를 실행하면, 컨테이너는 어떠한 네트워크에도 연결되지 않게 된다. 즉, 컨테이너 내부에서 외부로 접근할 수도 없고, 외부로부터 컨테이너 내부로 접근할 수도 없다. 이 명령으로는 여러 컨테이너를 생성하여 실행한다고 하더라도 각각이 네트워크에 연결되어 있지 않기 때문에 컨테이너 간 통신이 될 수도 없고, 마찬가지로 각 컨테이너 내부에서 외부로 접근할 수도 없다.

---
# --network host 옵션
![fig05](/assets/img/2022-09-11-Docker Networking/fig05.png){: width="200" height="200"}

다음으로는, `docker run --network host nginx`와 같이 `--network host` 옵션을 줄 수도 있다. `--network host` 옵션은 컨테이너가 도커 호스트의 네트워크에 연결되도록 하는 옵션으로, 호스트와 컨테이너 간에 네트워크가 분리되지 않도록(no network isolation) 한다. `--network host` 옵션으로 도커를 실행한 경우, 만약 컨테이너의 80번 포트를 리스닝하는 웹 애플리케이션을 배포했다고 하면, 컨테이너와 호스트 간 별도의 포트 매핑 없이 호스트의 80번 포트에서도 웹 애플리케이션에 접근할 수 있게 된다. 한편, 동일한 포트를 리스닝하는 컨테이너의 인스턴스를 추가적으로 실행하면, 두 컨테이너의 프로세스는 동일한 호스트의 네트워크를 공유하는 꼴이기 때문에 제대로 동작하지 않게 된다.

# 기타

이외에도 도커가 자체적으로 제공하는 드라이버로는 컨테이너(container), 오버레이(overlay)가 추가적으로 있으며, 서드파티 플러그인으로는 weave, flannel, openvswitch 등이 있다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `도커 공식 documentation` : [Use bridge networks](https://docs.docker.com/network/bridge/)<br>
[4] `지구별 여행자 님의 블로그` : [도커 네트워크](https://kangwoo.kr/2020/08/17/%EB%8F%84%EC%BB%A4-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/)
