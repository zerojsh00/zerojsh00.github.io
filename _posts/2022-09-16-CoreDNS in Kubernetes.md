---
title: (K8S) 쿠버네티스에서의 CoreDNS
author: simon sanghyeon
date: 2022-09-16
categories: [Kubernetes]
tags: [Kubernetes, K8S, Network, CoreDNS]
render_with_liquid: true
# use_math: true
---
이 글은 Mumshad Mannambeth가 강의한 Udemy의 Certified Kubernetes Administrator (CKA) with Practice Tests 강의 커리큘럼을 토대로 공부한 내용을 정리하였음을 밝힙니다.

---
# 리눅스에서의 DNS

## /etc/hosts

리눅스 환경에서 DNS를 사용할 수 있는 방법은 `/etc/hosts` 파일을 이용하는 것이다. 이는 IP와 도메인 이름을 매핑하는 역할을 한다. 예를 들어, 아래와 같이 /etc/hosts 파일을 열어보면, `127.0.0.1`이 `localhost`로 등록되어 있는 것을 확인할 수 있다.

```bash
$ cat /etc/hosts

# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1	localhost
... 생략 ...
```

## /etc/resolv.conf

`/etc/hosts` 파일에 도메인이 등록되어 있지 않는 경우, `DNS resolver`, 즉, 사용하고자 하는 DNS 서버(네임 서버) 목록을 기록한 파일인 `/etc/resolv.conf`를 이용한다. resolv.conf의 `nameserver` 란에 해당 서버에서 사용할 DNS 서버를 지정할 수 있다.

---
# Core DNS의 필요성
![fig01](/assets/img/2022-09-16-CoreDNS in Kubernetes/fig01.png){: width="500" height="500"}

위와 같이 각각 IP 주소가 할당되어 있는 파란색 파드와 보라색 파드가 있다고 하자. 각 파드에서 위와 같이 /etc/hosts에 상대 파드의 IP 주소와 도메인 이름을 기입해두면, 각자 상대방의 도메인 이름을 이용할 수 있겠다. 문제는 클러스터 내에 파드가 매우 많다는 점이다. 즉, 이러한 방법은 적합하지 않다.

![fig02](/assets/img/2022-09-16-CoreDNS in Kubernetes/fig02.png){: width="500" height="500"}

반면, 10.96.0.10을 사용하는 DNS 서버가 있다고 하자. 이 경우, DNS 서버에 각 파드의 이름(실제로는 ‘.’ 대신 ‘-’를 사용한 파드의 IP 주소)을 저장해두고, 각 파드에서는 `/etc/resolv.conf`에 nameserver와  DNS 서버의 IP 주소를 매핑함으로써 DNS 서버에서 한꺼번에 도메인 이름을 관리할 수 있다. 이러한 방식이 쿠버네티스의 `CoreDNS`가 도메인 이름을 관리하는 방식이다.

---
# CoreDNS

클러스터 내에 배포되어 있는 `CoreDNS`는 CNCF 재단에서 관리하는 프로젝트로, 1.12 버전 이후 DNS 서버를 배포하는 역할을 한다. (1.12 버전 이전에는 `kube-dns`가 사용되었다.)

CoreDNS는 쿠버네티스 클러스터를 지켜보고 있다가 새로운 파드나 서비스가 생성되면 CoreDNS의 데이터베이스에 DNS record를 생성한다.

CoreDNS 서버는 kube-system 네임스페이스에서 레플리카셋을 통해 두 개의 파드로 배포되어 있다. 따라서 다른 컴포넌트가 CoreDNS 파드에 접근할 수 있도록 CoreDNS의 서비스를 이용한다.

![fig03](/assets/img/2022-09-16-CoreDNS in Kubernetes/fig03.png){: width="500" height="500"}

CoreDNS의 서비스는 kube-system 네임스페이스에서 `kube-dns`라고 부른다. 이러한 CoreDNS의 서비스, 즉, kube-dns의 IP 주소가 모든 파드의 `/etc/resolv.conf` 파일의 nameserver 란에 기입되어야 하는데, 이마저도 파드가 생성될 때 쿠버네티스가 자동으로 처리해준다.

---
# 참고 문헌

[1] `Mumshad Mannambeth의 강의` : [Certified Kubernetes Administrator (CKA) with Practice Tests](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/)<br>
[2] `시작하세요! 도커/쿠버네티스 (용찬호 지음)` : [시작하세요! 도커/쿠버네티스](http://www.yes24.com/Product/Goods/84927385)<br>
[3] `산티아고의 기술 블로그` : [Kubernetes CoreDNS](https://velog.io/@koo8624/Kubernetes-CoreDNS)
