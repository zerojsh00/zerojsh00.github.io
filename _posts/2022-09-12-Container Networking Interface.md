---
title: (K8S) 네트워크 기초 정리 - CNI
author: simon sanghyeon
date: 2022-09-12
categories: [Kubernetes]
tags: [Kubernetes, K8S, Network, CNI]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# CNI의 필요성
이전 ['도커의 네트워크 글'](https://zerojsh00.github.io/posts/Docker-Networking/)에서 bridge network를 통해서 여러 네트워크 네임스페이스에 배치된 컨테이너들을 연결하는 방법을 살펴보았다.
정리하자면, 그 과정은 다음과 같다.

- Create Network Namespace
- Create Bridge Network/Interface
- Create vEth Pairs (Pipe, Virtual Cable)
- Attach vEth to Namespace
- Attach Other vEth to Bridge
- Assign IP Addresses
- Bring the interfaces up
- Enable NAT - IP Masquerade

**위와 같은 절차로 컨테이너들을 연결하는 방식은 도커 뿐만 아니라 rkt, Mesos, 쿠버네티스 등 컨테이너의 네트워킹을 다루는 솔루션이라면 모두 사용하는 방식이다.
모두가 사용하는 방식인 만큼, 일종의 표준을 만들어 사용하는 것이 편리하므로, 이 과정을 하나의 프로그램으로 만들 필요가 생겼다.**

---
# CNI 개념
이러한 필요에 따라 `Container Networking Interface(CNI)`가 탄생하게 되었다.
CNI는 `Cloud Native Computing Foundation(CNCF)`의 프로젝트로, 네트워크 인터페이스를 구성하기 위해서 어떻게 플러그인이 개발되어야 하는지와 같은 상세한 내용과 라이브러리들로 구성되어 있다.
예를 들면 다음과 같은 내용들이 정의되어 있는 것이다.

- Container Runtime must create network namespace.
- Identify network the container must attach to.
- Container Runtime to invoke Network Plugin (bridge) when container is ADDed.
- Container Runtime to invoke Network Plugin (bridge) when container is DELeted.
- JSON format of the Network Configuration
- …
- Must manage IP Address assignment to PODs
- Must Return results in a specific format

CNI는 `BRIDGE`, `VLAN`, `IPVLAN`, `MACVLAN`, `WINDOWS`, `DHCP`, `host-local` 등의 플러그인들이 준비되어 있다.
이외에도 서드파티의 플러그인인 `Weavenet`, `Flannel`, `Cilium`, `vmware NSX`, `Calico` 등도 이용할 수 있으며, 이러한 모든 서드파티 플러그인들도 CNI의 표준을 구현한 것이다.

---
# CNI를 구현하지 않은 도커
한편, 참고로 도커는 CNI를 구현하지 않았다. 도커는 도커 자체의 표준인 `Container Network Model(CNM)`을 구현하였다.
따라서 도커에서 CNI의 플러그인들을 사용하기 위해서는 특별한 방법을 써야 한다.
`docker run --network=none nginx`처럼 컨테이너를 실행할 때 none 옵션을 주고 나서, `bridge add 2e34dcf34 /var/run/netns/2e34dcf34` 명령 처럼 CNI 플러그인을 수동으로 직접 세팅하여 활용할 수 있다.
사실 이러한 방법이 쿠버네티스가 활용하는 방법이며, none network로 도커 컨테이너를 생성한 후 CNI 플러그인을 활용한다.

---
# 쿠버네티스와 CNI
앞서 CNI는 네트워크 인터페이스를 구성하기 위해서 어떻게 플러그인이 개발되어야 하는지와 같은 상세한 내용과 라이브러리들로 구성되어 있다고 했다. 그렇다면, 쿠버네티스에서 이러한 CNI의 플러그인들을 사용하기 위해서는 어떻게 해야 할까?

![fig01](/assets/img/2022-09-12-Container Networking interface/fig01.png){: width="500" height="500"}

CNI 플러그인에 대한 설정은 클러스터 내 각 노드의 `kubelet.service`에서 정의된다. 이를 살펴보면,  `--network-plugin=cni`라고 명시되어 있는 것을 알 수 있다. `ps -aux | grep kubelet` 명령을 통해 실행 중인 kubelet service를 살펴보아도 동일한 설정을 확인할 수 있다.

## cni-bin-dir : CNI 플러그인
![fig02](/assets/img/2022-09-12-Container Networking interface/fig02.png)

이때, `--cni-bin-dir=/opt/cni/bin`로 설정되어 있는 것을 볼 수 있다.  `cni-bin-dir`에는 지원되는 모든 CNI 플러그인들이 실행 될 수 있는 상태(executables)로 구성되어 있다.

## cni-conf-dir : CNI 설정 파일
![fig03](/assets/img/2022-09-12-Container Networking interface/fig03.png)

또한, `--cni-conf-dir=/etc/cni/net.d`로 설정되어 있다. `cni-conf-dir`에는 CNI와 관련한 설정 파일(configuration files)들이 있다. 이 예에서는 bridge의 설정 파일이 있다. 이를 열어보면, 다음과 같이 정의되어 있는데, 이 포맷이 바로 플러그인 설정과 관련한 CNI 표준으로 정의되어 있는 포맷이다.

![fig04](/assets/img/2022-09-12-Container Networking interface/fig04.png){: width="400" height="400"}

위 bridge 설정 파일을 간단하게 살펴보자. 우선, mynet이라는 이름이며, `bridge` type이다. `isGateway` 설정은 bridge 네트워크 인터페이스가 gateway로써 사용되기 위해서 IP 주소를 필요로 한다는 뜻이다. `ipMasq` 설정은 IP Masquerade 기능에 대한 설정으로, 해당 네트워크로부터 외부로 나갈 때 IP Masquerade 기능을 사용할지 여부를 명시한다. `ipam` 설정은 파드에 할당될 서브넷, IP 주소 등 `IPAM(IP Address Management)`에 관한 설정을 한다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
